# 场景：认证配置 (B01)

## 目标

为 Agent 配置 LLM Provider 的认证凭据——API Key、OAuth Token 或自定义 Provider 密钥，并控制密钥的来源、持久化方式与运行时覆盖。

## 什么时候用 / 不用会怎样

**适合用本场景**：

- 想从**自定义路径**加载 `auth.json`（多租户 / 沙盒 / CI 临时密钥）
- 想**运行时注入密钥**而不写磁盘（CI/CD、 Secrets Manager 取密钥）
- 想**自定义存储后端**（如 KMS / Vault / 数据库，不写本地文件）
- 想为**自定义 Provider**（通过 `registerProvider` 注册的）配置认证

**不适合用本场景**：

- 只想跑一次、用环境变量——什么都不传，SDK 会自动从 `process.env` 读 `ANTHROPIC_API_KEY` 等环境变量（详见下方「方式一」）
- 想做 OAuth 登录流程——本场景只覆盖「已有凭据如何传给 SDK」，OAuth 触发流程见 [sdk_doc/05 §login/logout](../sdk_doc/05-auth-model-registry.md)

**不用会怎样**：不配置认证时 `createAgentSession` 不立即报错（`??` 短路兜底），报错延迟到首次 `session.prompt()`。常见坑：创建 session 成功，prompt 却报 No API key——这是认证延迟校验设计（见下方陷阱 1）。

## 前置条件

- 已安装 `@earendil-works/pi-coding-agent`
- 已知目标 Provider 的 ID（如 `anthropic` / `openai` / `google` 等内置 Provider，或自定义 Provider 名）
- 若用文件存储：**目录可写**——`FileAuthStorageBackend` 会自动 `mkdirSync(dir, { recursive: true, mode: 0o700 })` 创建父目录（`ensureParentDir()`），并 `chmodSync(path, 0o600)` 设置文件权限（`ensureFileExists()`，写文件时也用 `AUTH_FILE_WRITE_OPTIONS.mode = 0o600`）

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `ModelRuntime` | 推荐入口：统一管理模型列表、凭据存储与请求认证解析 | [sdk_doc/05-auth-model-registry.md](../sdk_doc/05-auth-model-registry.md) |
| `ModelRuntime.create({ credentials, authPath, modelsPath })` | 创建 ModelRuntime 实例，可注入自定义 CredentialStore | [sdk_doc/05-auth-model-registry.md](../sdk_doc/05-auth-model-registry.md) |
| `readStoredCredential(provider)` | 一次性读取 auth.json 中某 provider 的凭据（从包根导出） | [sdk_doc/05-auth-model-registry.md](../sdk_doc/05-auth-model-registry.md) |
| `createAgentSession({ modelRuntime, agentDir })` | 把自定义 ModelRuntime 挂到 session 上 | [02-agent-session.md](../sdk_doc/02-agent-session.md) |

> ⚠️ **旧 API 已废弃**：`AuthStorage` 类不再从包根导出，`ModelRegistry.create(authStorage)` 静态工厂已删除（`ModelRegistry` 类降级为扩展层兼容包装器，构造签名改为 `new ModelRegistry(runtime)`）。`createAgentSession` 的 options 不再接 `authStorage` / `modelRegistry`，只接 `modelRuntime`。

## 默认行为：什么都不传会怎样

**`createAgentSession()` 不传任何认证参数时，SDK 内部会自动创建默认 `ModelRuntime`**：

```ts
// SDK 内部等价于：
const modelRuntime = await ModelRuntime.create();   // ~/.pi/agent/auth.json + ~/.pi/agent/models.json
```

默认路径是 `~/.pi/agent/auth.json`（**不是** `.pi/auth.json`），由 `getAgentDir()` 决定。`getAgentDir()` 受 **`PI_CODING_AGENT_DIR`** 环境变量覆盖（由 `APP_NAME.toUpperCase() + "_CODING_AGENT_DIR"` 拼成，APP_NAME 默认 `"pi"`）——设了就走自定义目录，否则 `~/.pi/agent/`。

> **重要**：B 系列 SDK 集成场景**绝大多数不需要手动 `ModelRuntime.create()`**——只要走环境变量（`ANTHROPIC_API_KEY` 等）或预置好 `~/.pi/agent/auth.json`，SDK 默认行为就够了。下面三种方式是「需要自定义」时才用。

## 密钥优先级（核心机制）

`ModelRuntime.getAuth(model)` 实际解析密钥时按以下顺序回退：

| 级别 | 来源 | 持久化 | 适用场景 |
|------|------|--------|---------|
| 1 | **请求级 override**（`streamSimple` / `prompt` 透传的 `options.apiKey`，resolve.ts:73） | ❌ 单次请求生效 | 同一进程内每次请求换不同 key（CI 多租户） |
| 2 | **stored credential**——auth.json 中**每个 provider 唯一**的一条凭据（`api_key` 或 `oauth`；OAuth 类型自动刷新） | ✅ 写文件 | 持久化主场景、OAuth 登录后持久化 |
| 3 | **`models.json` 中的 `providerConfig.apiKey`**（或 `registerProvider({ apiKey })`），即 composeApiKeyAuth 的"无 credential 时回退到 rawKey"路径 | ✅ 写 models.json | 自定义 Provider / 代理网关 |
| 4 | **环境变量**（`ANTHROPIC_API_KEY`、`OPENAI_API_KEY` 等，含 ADC / AWS profile 等 ambient 来源） | ❌ | 开发期最常用，CI 也常用 |

> **关于第 2 级的三个要点**：
>
> 1. **一个 provider 只存一条凭据**——`CredentialStore` 契约明确「keyed by `Provider.id`, one credential per provider」（types.ts:50-54）。`modify` 是覆盖式写入，后写的覆盖先写的，**不存在「同时存 api_key 和 oauth 然后按优先级选」**。`resolveProviderAuthWithSignal`（resolve.ts:87-104）只做一次 `credentials.read`，拿到的是 `api_key` 就走 `resolveApiKey`，是 `oauth` 就走 `resolveStoredOAuth`（自动刷新）。
> 2. **`setRuntimeApiKey` 也落在这一层**——`RuntimeCredentials`（runtime-credentials.ts:4）实现 `CredentialStore`，`setRuntimeApiKey` 往 `overrides` Map 里塞一个 `{ type: "api_key", key }`，在 `read()`（runtime-credentials.ts:24-28）里优先返回。因此进程级 overlay 实际表现为「第 2 级的 stored credential」，**不是**独立的更高级别。CI/CD 注入临时密钥（不想写盘、进程退出即失效）用的就是这个。
> 3. **自定义 `CredentialStore` 注入**完全取决于实现——若 `read` 返回固定 key 则同属第 2 级；若 `modify` 内部对接外部 KMS 则是另一回事。

> **注意**：旧 `ModelRegistry.getApiKeyAndHeaders` 仍可作兼容包装器调用，内部委托 `modelRuntime.getAuth`。新代码直接用 `modelRuntime.getAuth(model)`。环境变量（第 4 级）在「是否已配置」检查时使用，实际请求路径上要等 providerConfig 也没配才走到 env。

**环境变量特殊行为**：

- **Anthropic 有三个 env 变量**：`ANTHROPIC_AUTH_TOKEN` / `ANTHROPIC_OAUTH_TOKEN` / `ANTHROPIC_API_KEY`。在「是否已配置认证」检查时三者都参与（优先级 `ANTHROPIC_AUTH_TOKEN` 最高）；但实际取 key 时 `getEnvApiKey()` **跳过 `ANTHROPIC_AUTH_TOKEN`**（因为它须作 `Authorization: Bearer` 传递，不能当普通 API key），取 `ANTHROPIC_OAUTH_TOKEN` 或 `ANTHROPIC_API_KEY`
- **Google Vertex 走 ADC**：不读 `GOOGLE_CLOUD_API_KEY` 时 fallback 到 `gcloud auth application-default login` 生成的 ADC 凭据

## 核心方式

### 方式一：环境变量（推荐——开发期最常用）

**什么都不传**，SDK 自动从 `process.env` 读取：

```ts
import { getBuiltinModel } from "@earendil-works/pi-ai/providers/all";
import { createAgentSession } from "@earendil-works/pi-coding-agent";

// 设置环境变量（在 shell / .env / CI secrets 中）
// export ANTHROPIC_API_KEY=sk-ant-xxx

const { session } = await createAgentSession({
  model: getBuiltinModel("anthropic", "claude-sonnet-4-6"),
});
```

> **注**：旧示例用的 `getModel`（`@earendil-works/pi-ai/compat`）已 `@deprecated`（compat.ts:62），推荐改用 `getBuiltinModel`（`providers/all`）或 `Models.getModel()`。模型 id 随目录更新而变（当前为 `claude-sonnet-4-6` / `claude-opus-4-7` 系列），以实际内置目录为准。

完整 Provider → 环境变量映射见 SDK 源码 `env-api-keys.ts`。常见 Provider：

| Provider | 环境变量 |
|---------|---------|
| `anthropic` | `ANTHROPIC_AUTH_TOKEN`（discovery 用，getEnvApiKey 跳过）/ `ANTHROPIC_OAUTH_TOKEN`（优先）/ `ANTHROPIC_API_KEY` |
| `openai` | `OPENAI_API_KEY` |
| `google` | `GEMINI_API_KEY` |
| `google-vertex` | `GOOGLE_CLOUD_API_KEY` 或 ADC（`gcloud auth`） |
| `deepseek` | `DEEPSEEK_API_KEY` |
| `github-copilot` | `COPILOT_GITHUB_TOKEN` |

### 方式二：运行时注入密钥（CI/CD / Secrets Manager）

通过 `ModelRuntime.create({ credentials })` 注入自定义 `CredentialStore` 实现达到运行时注入密钥——适合从 AWS Secrets Manager / Vault / CI Environment 取密钥：

```ts
import { ModelRuntime, createAgentSession } from "@earendil-works/pi-coding-agent";

// 1. 自定义 CredentialStore 实现（或复用 AuthStorage 类，已不再从包根导出但可内部实现）
//    最简方案：复用默认 file-based store，通过 setRuntimeApiKey 注入
const modelRuntime = await ModelRuntime.create();   // ~/.pi/agent/auth.json + models.json

// 2. 通过 modelRuntime 注入 runtime key（不持久化，进程退出即失效）
await modelRuntime.setRuntimeApiKey("anthropic", process.env.ANTHROPIC_API_KEY!);

// 3. 传入 createAgentSession（只接 modelRuntime，不接 authStorage/modelRegistry）
const { session } = await createAgentSession({
  modelRuntime,
  // model 省略时走 findInitialModel 兜底，挑一个已配 key 的模型
});

try {
  await session.prompt("Hello!");
} finally {
  session.dispose();  // ⚠️ 必须释放
}
```

> **方式 2.5：请求级注入（真正的最高优先级）**：`streamSimple` / `prompt` 透传的 `options.apiKey` 是**单次请求**的 override（resolve.ts:73-85，`resolveProviderAuthWithSignal` 第一个分支），优先级高于所有 stored / runtime / env。CI 场景里「同一进程、每次请求换不同 key」用这个，而不是 `setRuntimeApiKey`（后者是**进程级** overlay，整个 session 期间固定一个 key）。
>
> ```ts
> // 每次请求传不同的 apiKey（请求级，不写盘、不污染 session 级状态）
> await modelRuntime.streamSimple(model, ctx, { apiKey: tenantKeyA });
> await modelRuntime.streamSimple(model, ctx, { apiKey: tenantKeyB });
> ```
>
> 区别：`setRuntimeApiKey` 是「给这个进程的 anthropic 绑定一个 key」（`RuntimeCredentials.read` overlay）；`options.apiKey` 是「这一次请求用这个 key」（`resolveProviderAuthWithSignal` 第一分支）。两者落在不同层，互不干扰。

> **一次性读 auth.json 的便捷 API**：`readStoredCredential(provider)` 从包根导出，适合 CI 脚本只需读一次密钥而不创建完整 session 的场景。
>
> **自定义 CredentialStore 的完整方案**：`ModelRuntime.create({ credentials })` 接收任意实现 `CredentialStore` 接口的对象（含 `read` / `list` / `modify` / `delete` 四个方法）。内置实现是 `AuthStorage`（类仍在 core 内部，但不再从包根导出）。如需 KMS / Vault 后端，自行实现该接口后传入。
>
> **`agentDir` 替代方案**：只想换存储目录（而非完全自定义 credentials）时，传 `agentDir` 给 `createAgentSession` 更简洁——SDK 会自动构造指向 `<agentDir>/auth.json` 和 `<agentDir>/models.json` 的 `ModelRuntime`。见下方「方式四」。

### 方式三：完全自定义（自定义路径 / 内存后端 / 自定义存储）

这些场景统一通过 `ModelRuntime.create({ credentials, authPath, modelsPath })` 实现。

**3a. 自定义 auth.json 文件路径**——多租户、按用户隔离密钥文件：

```ts
const modelRuntime = await ModelRuntime.create({
  authPath: "/data/tenant-42/auth.json",
});
```

**3b. 纯内存后端**——测试场景、绝不能落盘的场景：

自行实现 `CredentialStore` 接口（或复用 core 内部的 `AuthStorage.inMemory()`，但它不从包根导出，SDK 集成场景无法直接调用），传入 `ModelRuntime.create({ credentials })`。如需内存后端，实现 `CredentialStore` 接口或用 `InMemoryAuthStorageBackend`（同样不在公开导出）。`InMemoryAuthStorageBackend` 后端**完全跳过文件 I/O**——`withLock` / `withLockAsync` 退化为纯内存操作（见 `auth-storage.ts` 的 `InMemoryAuthStorageBackend` 类）。

**3c. 自定义后端**（实现 `CredentialStore` 接口）——KMS / Vault / 数据库：

```ts
import { ModelRuntime, createAgentSession } from "@earendil-works/pi-coding-agent";
import type { CredentialStore } from "@earendil-works/pi-ai";

// 最小 CredentialStore 实现：读 / 列 / 改 / 删（写通过 modify 完成，无独立 write 方法）
const myCredentials: CredentialStore = {
  async read(providerId) { /* 从 KMS / Vault 读取 */ },
  async list() { /* 返回凭据元信息列表 */ },
  async modify(providerId, fn) { /* 序列化写：读当前 → fn → 写回 */ },
  async delete(providerId) { /* 删除凭据 */ },
};

const modelRuntime = await ModelRuntime.create({ credentials: myCredentials });
```

### 方式四：`agentDir` 整目录切换

**适合场景**：想把 `auth.json` + `models.json` + `settings.json` + `sessions/` 全部放到自定义目录（如便携版、容器化、多用户隔离）。

```ts
const { session } = await createAgentSession({
  agentDir: "/data/my-app/pi-agent",  // ← 整目录切换
  // SDK 自动构造：
  //   ModelRuntime.create({
  //     authPath:   "/data/my-app/pi-agent/auth.json",
  //     modelsPath: "/data/my-app/pi-agent/models.json",
  //   })
  //   SettingsManager.create(cwd, "/data/my-app/pi-agent")
});
```

`agentDir` 优先级：`options.agentDir` > `PI_CODING_AGENT_DIR` 环境变量 > `~/.pi/agent/`（`sdk.ts` 中 `agentDir` 三元求值；`getAgentDir()` in `config.ts`）。

> **`agentDir` vs 自定义 `modelRuntime`**：如果同时传了 `options.modelRuntime` 和 `options.agentDir`，**`modelRuntime` 优先**（`??` 运算符先检查左侧）。`agentDir` 计算出的 `authPath` / `modelsPath` 仅在未传 `modelRuntime` 时用于构造默认 ModelRuntime（sdk.ts:176）。

## 变体与延伸

- **持久化 API Key 到磁盘**：用 `modelRuntime.login("anthropic", "api_key", interaction)`（公开 API，内部走 `Models.login` + `synchronizeCredentialState`）。注：`modelRuntime.credentials` 是 `private readonly`（model-runtime.ts:124），SDK 用户**无法直接调用** `.credentials.modify(...)`；底层写入靠 `login` 或自定义 `CredentialStore`（实现接口后传给 `ModelRuntime.create({ credentials })`）。`ModelRegistry` 仅在 ExtensionRunner 内部使用，不在 session 上暴露
- **OAuth 登录触发**：`modelRuntime.login(providerId, type, interaction)`——`type` 是认证类型（`"api_key"` / `"oauth"` 等），`interaction` 包含回调与 abort signal。OAuth provider 必须先注册（内置：`anthropic` / `github-copilot` / `openai-codex` / `openrouter` / `radius` / `xai` / `kimi-coding` 已注册；完整列表 `grep "oauth:" providers/*.ts`）
- **provider 级环境变量注入**：同一个 key 走不同 base URL / 网关时，可在 auth.json 中存 env：`{ type: "api_key", key: "sk-xxx", env: { PROXY_BASE_URL: "https://gw.corp.com" } }`。请求时作为 provider-level env 传给 API 调用（合并到 request options.env，不修改 `process.env`）。详见 [sdk_doc/05](../sdk_doc/05-auth-model-registry.md) 的「认证解析」与「凭据写入」节
- **`api_key.key` 支持命令模式**：`AuthStorage.read`（auth-storage.ts:333）对 `api_key` 类型会调 `resolveConfigValue(credential.key, credential.env)`——即 `key` 可以是命令（如 `"op://vault/anthropic/apiKey"` 走 1Password CLI）而非明文字符串。这是 `models.json` 中 `_command` source 的来源，可对接任意 vault CLI；详见 `resolve-config-value.ts`
- **获取可用模型列表** → 见 [场景 B03](B03-available-models.md)
- **指定模型运行** → 见 [场景 A02](A02-model-selection.md)

## 陷阱与已知问题

### 陷阱 1：认证缺失延迟报错

`createAgentSession` 用 `??` 短路兜底，**即使完全没配置任何认证也不会报错**。报错延迟到 `session.prompt()` 时通过 `modelRuntime.getAuth` 返回 `undefined`（或兼容层 `modelRegistry.getApiKeyAndHeaders` 返回 `{ ok: false }`）后抛出。

**正确做法**：在 `prompt` 前**显式检查**：

```ts
if (!modelRuntime.hasConfiguredAuth(model.provider)) {
  throw new Error(`No auth configured for ${model.provider}/${model.id}`);
}
// 注：session 上只有 modelRuntime，没有 modelRegistry
```

**若需知道 key 从哪来**（UI 场景）：用 `modelRuntime.getProviderAuthStatus(providerId)`，返回 `{ configured: boolean, source?: "stored" | "runtime" | "environment" | "fallback" | "models_json_key" | "models_json_command", label? }`（provider-composer.ts + model-runtime.ts），比 `hasConfiguredAuth` 的布尔值更细。
```

### 陷阱 2：Anthropic 三个环境变量的优先级与特殊行为

Anthropic 有三个环境变量：`ANTHROPIC_AUTH_TOKEN` / `ANTHROPIC_OAUTH_TOKEN` / `ANTHROPIC_API_KEY`。在认证发现时三者按此顺序检查；但实际取 API key 时 `getEnvApiKey()` 会**跳过 `ANTHROPIC_AUTH_TOKEN`**（因为它须作 `Authorization: Bearer` 头传递），取 `ANTHROPIC_OAUTH_TOKEN` 或 `ANTHROPIC_API_KEY`（`env-api-keys.ts` 中 `getEnvApiKey` 的 anthropic 分支）。如果你同时设了 `ANTHROPIC_OAUTH_TOKEN` 和 `ANTHROPIC_API_KEY`，SDK 会读 `ANTHROPIC_OAUTH_TOKEN`（当作 OAuth token）。如果你想做 API Key 认证却设了 OAuth token 变量，会走到 OAuth 分支失败。

### 陷阱 3：文件锁机制

`FileAuthStorageBackend` 用 `proper-lockfile` 加锁保护 auth.json 的并发写入（见 `auth-storage.ts` 的 `FileAuthStorageBackend` 类）。锁机制：

- 同步锁：最多重试 10 次，每次间隔 20ms（`acquireLockSyncWithRetry`）
- 异步锁：自定义重试循环（`maxRetries=10`），指数退避延迟 `min(round((rand+1)*100*2^attempt), 10000)` ms；proper-lockfile 自身选项 `retries: 0, stale: 30000`（30s 过期）

如果你的进程持有锁超过 30 秒，其他进程会判定锁 stale 并抢占——OAuth 刷新时要注意不要在锁内做长耗时操作。

### 陷阱 4：自定义存储后端必须实现锁语义

实现 `AuthStorageBackend` 时**必须正确处理并发**——SDK 的 OAuth 自动刷新依赖锁来防止多个进程同时刷新 token。如果自定义后端是「无锁的」（如直接读写 KMS），可能造成重复刷新或 token 覆盖。

## 关键细节

- 推荐入口是 `ModelRuntime.create()`，默认路径 `~/.pi/agent/auth.json` + `~/.pi/agent/models.json`（**不是** `.pi/auth.json`）
- `AuthStorage` 类**不再从包根导出**——如需自定义后端，实现 `CredentialStore` / `AuthStorageBackend` 接口后通过 `ModelRuntime.create({ credentials })` 注入
- `ModelRegistry.create()` 静态工厂已删除——`ModelRegistry` 降级为兼容包装器（`new ModelRegistry(runtime)`），仅用于扩展层；新代码直接用 `modelRuntime`
- `auth.json` 的文件权限自动设为 `0o600`，父目录 `0o700`（`FileAuthStorageBackend.ensureParentDir()` / `ensureFileExists()`）
- OAuth token 过期时 `getApiKey()` 自动通过文件锁刷新，避免并发刷新冲突
- `modelRuntime.getAvailableSnapshot()` 的快速检查**不刷新 OAuth token**——适合 UI 列表场景；实际请求时才走 `modelRuntime.getAuth()` 触发刷新
- `ModelRuntime.create({ modelsPath: null })` **不加载 `models.json`**，只能使用内置模型
- `ModelRuntime.create({ modelsPath })` 即使 `modelsPath` 不存在也不会报错——自定义模型为空
- `registerProvider()` 的 `config.models` 会替换该 Provider 的模型列表（provider config 本身是合并而非覆盖，保留未覆盖的字段）
- `login(providerId, type, interaction)` 中的 `providerId` 必须是已注册的 OAuth provider（内置：`anthropic` / `github-copilot` / `openai-codex` / `openrouter` / `radius` / `xai` / `kimi-coding` 等，自定义需通过 `pi.registerProvider({ oauth })` 注册）
- 认证系统**完全独立于 `cwd`**——ModelRuntime 不依赖工作目录，跨项目可复用
