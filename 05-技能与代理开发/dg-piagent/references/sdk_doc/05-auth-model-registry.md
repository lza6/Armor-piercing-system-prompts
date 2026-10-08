# 05 - 模型/认证运行时 (ModelRuntime) + ModelRegistry 兼容包装器

> `CreateAgentSessionOptions.authStorage` 和 `modelRegistry` 参数已被 `modelRuntime` 整体替换。`ModelRuntime` 类统一管理认证凭据 + 模型注册表；`AuthStorage` 类不再从包根导出；`ModelRegistry` 仍导出但降级为「扩展层兼容包装器」，内部委托 `ModelRuntime`。末尾给出旧 API 迁移对照表。

## 这是什么

`ModelRuntime` 是 pi-agent 的**认证 + 模型一体化运行时**（`packages/coding-agent/src/core/model-runtime.ts`）。它把凭据存储（`auth.json`）、模型目录（`models.json`）、Provider 注册表、内置模型目录四件事打包成一个对象，供 `createAgentSession` 和扩展上下文使用。

扩展内访问点：`ctx.modelRegistry` —— 一个**同步兼容包装器**（`ModelRegistry` 类，`packages/coding-agent/src/core/model-registry.ts`），内部全部委托 `ModelRuntime`。

## 第一部分：ModelRuntime（主 API）

### 静态工厂

```ts
import { ModelRuntime } from "@earendil-works/pi-coding-agent";

// 标准创建（默认从 <agentDir>/auth.json + <agentDir>/models.json 加载）
const runtime = await ModelRuntime.create({ authPath, modelsPath });

// 自定义凭据后端（如内存）
const runtime = await ModelRuntime.create({
  credentials: myCredentialStore,
  modelsPath: null,            // null = 不加载 models.json
  allowModelNetwork: false,    // 默认 false，禁止 create() 阶段联网刷新模型目录
});
```

**`CreateModelRuntimeOptions`**（`model-runtime.ts`）：

| 字段 | 类型 | 说明 |
|------|------|------|
| `credentials?` | `CredentialStore` | 凭据存储后端，默认从 `authPath` 读 |
| `authPath?` | `string` | auth.json 路径 |
| `modelsPath?` | `string \| null` | models.json 路径，`null` 表示不加载 |
| `modelsStore?` | `ModelsStore` | 自定义模型存储（高级用法） |
| `modelsStorePath?` | `string` | 自定义 models-store.json 路径（只改路径、不改实现时用，与 `modelsStore` 互斥） |
| `allowModelNetwork?` | `boolean` | `create()` 阶段是否允许联网刷新模型目录（默认 false） |
| `modelRefreshTimeoutMs?` | `number` | 联网刷新超时 |
| `catalogBaseUrl?` | `string` | 内置模型 catalog 基础 URL |
| `signal?` | `AbortSignal` | create 阶段的取消信号 |

### 关键实例方法

#### 模型查询

```ts
getModel(providerId: string, modelId: string): Model<Api> | undefined  // 精确查找
getModels(): readonly Model<Api>[]                                       // 全部模型（含未配置认证）
getAvailableSnapshot(): readonly Model<Api>[]                           // 已配置认证的可用模型
hasConfiguredAuth(providerId: string): boolean                          // 快速检查，不刷 OAuth
isUsingOAuth(providerId: string): boolean                               // 是否走 OAuth 模式
getProvider(providerId: string): Provider | undefined                   // 原生 Provider 对象
getProviderAuthStatus(providerId: string): AuthStatus                   // 脱敏状态（不执行命令）
```

#### 认证解析

```ts
getAuth(model: Model<Api>, overrides?: ModelRuntimeAuthOverrides): Promise<AuthResult | undefined>
getAuth(providerId: string, overrides?: ModelRuntimeAuthOverrides): Promise<AuthResult | undefined>
```

两个重载都有可选第三参 `overrides?: ModelRuntimeAuthOverrides`（`model-runtime.ts:76-81`），可传 `apiKey`/`env`/`minOAuthValidityMs` 做**请求级**覆盖（如临时换 key、要求 OAuth token 至少还剩 N 毫秒有效期），不写入 auth.json。

`AuthResult` 含 `auth: { apiKey?, headers?, baseUrl? }` + `env?: ProviderEnv` + `source?: string`（认证来源的可读标签，如 `"ANTHROPIC_API_KEY"` / `"OAuth"`），是请求 provider 时最终注入的认证信息。

`getApiKeyAndHeaders()`（`ModelRegistry` 包装）是基于此的高层封装，返回 `ResolvedRequestAuth = { ok: true; apiKey?; headers?; env? } | { ok: false; error }`。

#### Provider 动态注册

ModelRuntime 上是两个**独立**方法（不是重载）：

```ts
// 注册静态 ProviderConfig（简单场景）
registerProvider(providerId: string, config: ProviderConfigInput): void

// 注册完整 pi-ai Provider 对象（复杂场景）
registerNativeProvider(provider: Provider): void

unregisterProvider(providerId: string): void
```

> ⚠️ **`registerProvider` 的「双重载」在 `ModelRegistry` 兼容包装器和扩展层 `ExtensionAPI` 上，不在 `ModelRuntime` 上**。包装器/扩展层的 `registerProvider(provider: Provider)` 重载内部委托 `runtime.registerNativeProvider()`。直接对 `ModelRuntime` 实例调 `runtime.registerProvider(provider)` 会编译失败——请用 `runtime.registerNativeProvider(provider)`。

详见 [16-custom-provider.md](16-custom-provider.md)。

#### 凭据写入（运行时登录/登出/设 API key）

ModelRuntime 自身就是运行时凭据写入主入口，无需再绕 `CredentialStore` 接口：

```ts
setRuntimeApiKey(providerId, apiKey, options?): Promise<void>      // 写入 API key
removeRuntimeApiKey(providerId, options?): Promise<void>          // 删除 API key
login(providerId, type, interaction): Promise<Credential>         // OAuth 登录
logout(providerId, options?): Promise<void>                       // 登出
listCredentials(options?): Promise<readonly CredentialInfo[]>     // 列出已存凭据（脱敏元信息）
```

> 写入后会同步刷新本地模型/认证快照；若快照同步失败会抛 `CredentialSynchronizationError`（凭据本身已提交）。详细签名见 `packages/coding-agent/src/core/model-runtime.ts`。

#### 其他

```ts
refresh(options?: ModelsRefreshOptions): Promise<ModelsRefreshResult>  // 重新加载 models.json + 联网刷新
getError(): string | undefined                                         // models.json 解析错误
complete(model, context, options?): Promise<AssistantMessage>          // 一次性补全调用（不走流）
getCompatibilityRequestConfig(model): { authHeader, headers }          // 请求级兼容配置
registerNativeProvider(provider: Provider): void                       // 注册原生 Provider（被 registerProvider 重载 2 委托）
getRegisteredProviderConfig(name): ProviderConfigInput | undefined
getRegisteredNativeProvider(name): Provider | undefined
getRegisteredProviderIds(): readonly string[]
```

> ModelRuntime 还完整实现了 pi-ai 的 `Models` 接口（`packages/ai/src/models.ts`），包括 `stream()`/`streamSimple()`/`completeSimple()`/`checkAuth(providerId)`/`getAvailable(providerId?)`（**异步**，可能触发联网刷新）/`getAvailableSnapshot()`（**同步**只读快照）/`getProviders()`。SDK 集成通常不直接调这些（`createAgentSession` 内部会消费），但调用 `getAvailable()` 与 `getAvailableSnapshot()` 的语义差异在并发场景要注意：前者会 await 刷新、后者立即返回快照。完整签名见源码。

## 第二部分：ModelRegistry 兼容包装器

`ModelRegistry` 仍是公开导出，但**仅在扩展上下文使用**——`ctx.modelRegistry` 类型是 `ModelRegistry`。它是 155 行的同步包装器（`model-registry.ts`），全部方法委托给内部 `runtime: ModelRuntime`。

### 构造（一般用户不直接调）

```ts
import { ModelRegistry, ModelRuntime } from "@earendil-works/pi-coding-agent";

const registry = new ModelRegistry(myModelRuntime);
```

**不推荐**直接 `new ModelRegistry(...)`——通常由 `createAgentSession` 内部从 `modelRuntime` 自动构造，扩展通过 `ctx.modelRegistry` 拿到。

> ⚠️ 旧代码中 `ModelRegistry.create(authStorage, modelsJsonPath)` / `ModelRegistry.inMemory(authStorage)` 已**不存在**（类没有静态工厂）。照抄会编译失败。

### 实例方法（全部委托 ModelRuntime）

```ts
find(provider, modelId)                  // 委托 runtime.getModel
getAll() / getAvailable()                // 委托 runtime.getModels / getAvailableSnapshot
hasConfiguredAuth(model)                 // 委托 runtime.hasConfiguredAuth(model.provider)
getApiKeyAndHeaders(model): Promise<ResolvedRequestAuth>  // 基于 runtime.getAuth 的高层封装
getApiKeyForProvider(provider)           // 基于 runtime.getAuth(provider)
getProvider(provider) / getProviderDisplayName(provider) / getProviderAuthStatus(provider) / getProviderAuth(provider)
isUsingOAuth(model)                      // 委托 runtime.isUsingOAuth(model.provider)
registerProvider(...) / unregisterProvider(name)  // 双重载，委托 runtime
getRegisteredProviderConfig(name) / getRegisteredNativeProvider(name) / getRegisteredProviderIds()
refresh(options?) / getError()
complete(model, context, options?)       // 一次性补全调用
```

详细签名见 `packages/coding-agent/src/core/model-registry.ts` 或 `node_modules/@earendil-works/pi-coding-agent/dist/**/*.d.ts`。

## 第三部分：readStoredCredential —— 一次性读取 auth.json

`AuthStorage` 类不再公开导出，但根入口导出了 `readStoredCredential()` 用于一次性读取凭据（`index.ts` re-export from `./core/auth-storage.ts`）：

```ts
import { readStoredCredential } from "@earendil-works/pi-coding-agent";

// 签名：readStoredCredential(providerId, authPath = <agentDir>/auth.json)，同步返回
const cred = readStoredCredential("anthropic", "/path/to/auth.json");
// → { type: "api_key", key: "sk-ant-..." } | undefined
```

适合写脚本、做迁移、诊断工具等"读一次就够"的场景。运行时长期持有/管理认证，请用 `ModelRuntime`。

## 第四部分：getModel() —— 从内置目录查找

> ⚠️ **已废弃**：`getModel` 来自 `@earendil-works/pi-ai/compat`，源码标 `@deprecated`（完整消息：`Static catalog read. Use getBuiltinModel from "@earendil-works/pi-ai/providers/all" or Models.getModel().`）。新代码请改用 `@earendil-works/pi-ai/providers/all` 的 `getBuiltinModel`。

```ts
import { getModel } from "@earendil-works/pi-ai/compat";        // 已废弃
// 推荐：import { getBuiltinModel } from "@earendil-works/pi-ai/providers/all";

getModel(provider, modelId)   // 即 typeof getBuiltinModel：强类型泛型签名
```

```ts
const model = getModel("anthropic", "claude-opus-4-5");
const model = getModel("openai", "gpt-5");
```

> `getModel()` 只查找内置目录。自定义模型（`models.json` 或 `registerProvider`）需要通过 `modelRuntime.getModel()` / `registry.find()` 查找。

## 组合使用示例

### ModelRuntime（推荐）

```ts
import { ModelRuntime, createAgentSession } from "@earendil-works/pi-coding-agent";
import { getModel } from "@earendil-works/pi-ai/compat";

// 1. 创建 ModelRuntime（一次创建认证 + 模型注册表）
const modelRuntime = await ModelRuntime.create({
  authPath: "/my-project/.pi/auth.json",
  modelsPath: "/my-project/models.json",
});

// 2. 选择模型（getModel 用内置 catalog 的真实 ID，如 claude-sonnet-4-5）
const model = getModel("anthropic", "claude-sonnet-4-5")
  ?? modelRuntime.getModel("my-provider", "my-model");

// 3. 传入 createAgentSession
const { session } = await createAgentSession({ model, modelRuntime });

try {
  await session.prompt("Hello!");
} finally {
  session.dispose();
}
```

### 全默认（最简）

```ts
const { session } = await createAgentSession();
// modelRuntime 由 createAgentSession 用 ~/.pi/agent/auth.json + models.json 自动构造
```

## v0.80.2 → v0.80.8 迁移对照表

| v0.80.2 旧 API | v0.80.8+ 新 API | 备注 |
|----------------|-----------------|------|
| `import { AuthStorage } from "@earendil-works/pi-coding-agent"` | 不再导出 | 一次性读取用 `readStoredCredential()`，长期持有用 `ModelRuntime` |
| `AuthStorage.create(path?)` | `ModelRuntime.create({ authPath })` | ModelRuntime 一次性吃掉 auth + models |
| `AuthStorage.inMemory(data?)` | `ModelRuntime.create({ credentials: inMemoryStore })` | 自定义 `CredentialStore` |
| `auth.getApiKey(provider)` | `modelRuntime.getAuth(provider).then(r => r?.auth.apiKey)` | 或 `ctx.modelRegistry.getApiKeyForProvider(provider)` |
| `auth.setRuntimeApiKey(p, k)` | `modelRuntime.setRuntimeApiKey(providerId, apiKey)` | 写凭据走 `ModelRuntime`，不绕 `CredentialStore`。通常 CLI 用 `--api-key`，SDK 用 `ModelRuntime.create({ credentials })` 注入 |
| `auth.get(provider)` / `auth.set(p, c)` | `modelRuntime.getAuth(provider)` 读、`modelRuntime.setRuntimeApiKey(...)` 写 | ⚠️ `CredentialStore` 接口只有 `read`/`list`/`modify`/`delete`（**无** `get`/`set`）。自定义 store 实现这几个方法即可 |
| `ModelRegistry.create(authStorage, modelsPath?)` | `ModelRuntime.create({ authPath, modelsPath })` | 旧静态工厂不存在 |
| `ModelRegistry.inMemory(authStorage)` | `ModelRuntime.create({ modelsPath: null })` | 同上 |
| `createAgentSession({ authStorage, modelRegistry })` | `createAgentSession({ modelRuntime })` | **最关键的破坏性变更** |
| `session.modelRegistry` getter | `session.modelRuntime` getter（`agent-session.ts`） | 旧 getter 已删除 |
| `registry.find(p, m)` / `getAll()` / `getAvailable()` 等方法 | 仍可用，但通过 `ctx.modelRegistry`（扩展内）或 `modelRuntime` 直接调 | ModelRegistry 作为兼容包装器保留 |

## 关键细节

- **凭据类型**仍是 `ApiKeyCredential` (`{ type: "api_key"; key?: string; env?: Record<string, string> }`) 或 `OAuthCredential`，定义来自 `@earendil-works/pi-ai`
- **`ApiKeyCredential.env`**：同一 key 走不同网关时可注入额外 env（如 `PROXY_BASE_URL`），写入 auth.json 的 env 字段，请求时自动合并到 process.env
- **OAuth Token 过期自动刷新**：链路是 `getAuth()` → `models.getAuth()` → 若 token 过期则走 `CredentialStore.modify` → `AuthStorage.modify` 内通过 `proper-lockfile` 文件锁串行化（`auth-storage.ts`），避免并发刷新冲突。锁串行化的是 `modify`（刷新发生在其内部），不是 `getAuth` 调用本身
- **`allowModelNetwork` 默认 false**：`create()` 默认不联网刷新模型目录，适合离线 / CI 场景
- **`ModelRuntime.create()` 是 async**：联网刷新模型目录、读 auth.json 都是异步操作
- **`models.json` 缺失不报错**：`ModelRuntime.create({ modelsPath })` 即使路径不存在也返回正常 runtime，自定义模型为空
- **`registerProvider()` 会覆盖同 Provider 下所有已有模型**
- **双重载**：`registerProvider(provider: Provider)` 可注册完整 pi-ai Provider（含动态模型刷新、过滤、自定义流），远比 `ProviderConfigInput` 静态配置强大——详见 [16-custom-provider.md](16-custom-provider.md) 模式 5
- **扩展上下文 `ctx.modelRegistry`** 是 ModelRegistry 包装器（同步兼容），**不是 ModelRuntime**；扩展要做一次性 `complete()` 调用、解析 API key 走包装器即可
