# 场景：获取可用模型列表 (B03)

## 目标

列出当前 SDK 能识别的所有模型，**按是否已配置 API Key 过滤**，用于启动前检查、UI 模型选择器、CI 可用性验证等。区别于 [场景 A02「精确指定一个模型」](A02-model-selection.md)——A02 是「我知道要用哪个」，B03 是「我先看看有哪些能用」。

## 什么时候用 / 不用会怎样

**适合用本场景**：

- 启动前检查「这台机器到底配了哪些 key」——给用户一个友好提示而不是延迟到 `prompt()` 才 throw
- 写**模型选择器 UI**——`getAvailableSnapshot()` 的同步 + 不刷 OAuth 特性正是为 UI 列表场景设计的（`modelRuntime.getAvailableSnapshot()`，旧 `modelRegistry.getAvailable()` 是同语义兼容包装器）
- CI 脚本里**验证环境**——部署前确认目标模型确实有 key
- 列出 `models.json` 自定义模型，确认配置生效

**不适合用本场景**：

- 代码里写死要用某个模型 → 用 [A02「精确指定模型」](A02-model-selection.md)（推荐 `getBuiltinModel`）
- 配置 API Key → 用 [B01「认证配置」](B01-auth-config.md)
- 想做 OAuth 登录流程 → 用 [sdk_doc/05 §login/logout](../sdk_doc/05-auth-model-registry.md)

**不用会怎样**：不主动列模型，你就不知道当前环境能用哪些——SDK 会走 `findInitialModel()` 兜底，挑一个默认模型；挑不到时 session 仍正常返回，延迟到 `session.prompt()` 时才抛 `No model selected.` / `No API key found for "..."`（[A01「延迟报错机制」](A01-minimal-startup.md)）。这是 SDK 的「延迟 throw」设计——B03 的价值就是让你**在 prompt 之前就知道列表是空的**。

## 前置条件

- 已安装 `@earendil-works/pi-coding-agent`
- 若要查到「有 key 的」模型，需要先配过认证（环境变量 / auth.json / models.json 三选一，详见 [B01](B01-auth-config.md)）
- 若要列出自定义模型，需要 `models.json` 存在（默认路径 `~/.pi/agent/models.json`）

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `ModelRuntime` | 推荐入口：列出 / 查找 / 过滤模型 + 绑定认证 | [sdk_doc/05-auth-model-registry.md](../sdk_doc/05-auth-model-registry.md) |
| `modelRuntime.getAvailableSnapshot()` / `getModels()` | 新方法名（同步、不刷 OAuth） | [sdk_doc/05-auth-model-registry.md](../sdk_doc/05-auth-model-registry.md) |
| `getBuiltinModel` | **推荐**：按 provider + modelId 从内置目录精确取一个 | [sdk_doc/05-auth-model-registry.md](../sdk_doc/05-auth-model-registry.md) |
| `getModel` | ⚠️ **@deprecated**，等价于 `getBuiltinModel`，仅为兼容旧代码保留 | 同上 |

> ⚠️ **旧 API 已废弃**：`AuthStorage` 类不再从包根导出，`ModelRegistry.create()` 静态工厂已删除。新代码请用 `ModelRuntime.create()` + `modelRuntime.getAvailableSnapshot()` / `modelRuntime.getModels()`。`ModelRegistry` 类降级为扩展层兼容包装器（`new ModelRegistry(runtime)`），其 `getAll()` / `getAvailable()` 仍可调用但内部委托 ModelRuntime。

## 默认行为：什么都不传会怎样

**`createAgentSession()` 不传 `modelRuntime` 时，SDK 内部自动创建默认实例**：

```ts
// SDK 内部等价于：
const modelRuntime = await ModelRuntime.create();   // ~/.pi/agent/auth.json + ~/.pi/agent/models.json
```

默认加载路径是 `~/.pi/agent/models.json`（**不是** `.pi/models.json`），由 `getAgentDir()` 决定；`getAgentDir()` 受 `PI_CODING_AGENT_DIR` 环境变量覆盖（`ENV_AGENT_DIR = \`${APP_NAME.toUpperCase()}_CODING_AGENT_DIR\``，`APP_NAME` 默认 `"pi"`）。

> **重要**：B 系列 SDK 集成场景**绝大多数不需要手动 `ModelRuntime.create()`**——默认行为就够。只有在「自定义 models.json 路径」「内存模式」「需要 registerProvider 运行时注入」等场景下才需要手动构造。

## 核心方法对比：`getModels()` vs `getAvailableSnapshot()`

这两个方法在 `ModelRuntime` 上并列存在，语义完全不同——选错会得到错误的结果。旧的 `ModelRegistry.getAll()` / `getAvailable()` 是同语义的兼容包装器（内部委托 ModelRuntime）：

| 方法 | 返回 | 语义 | 同步？ | 刷 OAuth？ |
|------|------|------|--------|-----------|
| `modelRuntime.getModels(providerId?)` | `readonly Model<Api>[]` | **所有**已注册模型（内置 + models.json + registerProvider）；传 `providerId` 只列该 provider 的模型 | ✅ 同步 | ❌ 不刷 |
| `modelRuntime.getAvailableSnapshot()` | `readonly Model<Api>[]` | 上者过滤后**仅保留已配 key 的**（基于最近一次 availability 刷新的快照） | ✅ 同步 | ❌ 不刷 |

**何时用哪个**：

- 想「**这台机器能跑什么**」→ `getAvailableSnapshot()`
- 想「**SDK 识别哪些模型**（包括没配 key 的）」→ `getModels()`——例如做模型商店 UI、展示全部可选项
- 两者都是**同步方法**，返回快照而不是 Promise——因为内部只读缓存的存在性检查，不触发 OAuth 刷新。这是设计特性：UI 列表场景不能阻塞在 token 刷新上
- `modelRuntime.getAvailable()` 是 **async** 方法（返回 Promise），会触发实时 availability refresh——UI 列表场景不建议用，适合启动时需要确保数据最新的场景

## 核心代码

### 方式一：列出已配 key 的模型（最常用）

```ts
import { ModelRuntime } from "@earendil-works/pi-coding-agent";

const modelRuntime = await ModelRuntime.create();

// 同步方法，不刷 OAuth token——适合 UI 列表
const available = modelRuntime.getAvailableSnapshot();

// 顺序由内部快照决定（内置 + 自定义注册顺序），不保证按字母或按 provider 分组
// 不要依赖顺序做「取第一个 = 最优选」——要精确指定请用 find() 或 getBuiltinModel()
console.log(available.map((m) => `${m.provider}/${m.id}`));
// 可能输出（仅示例，实际 ID 随 SDK 版本变化）：
// ["anthropic/claude-opus-4-8", "anthropic/claude-sonnet-4-5", "openai/gpt-5.5", ...]
```

### 方式二：列出所有模型（含未配 key 的）

```ts
const all = modelRuntime.getModels();
console.log(`SDK 识别 ${all.length} 个模型，其中 ${modelRuntime.getAvailableSnapshot().length} 个已配 key`);
```

### 方式三：精确查找一个模型

拿到 Model 引用后传给 `createAgentSession({ model })`，有三种路径——**推荐 `getBuiltinModel`**（有类型推导）：

```ts
// 内置模型（推荐）：编译期校验 provider + modelId
import { getBuiltinModel } from "@earendil-works/pi-ai/providers/all";

const opus = getBuiltinModel("anthropic", "claude-opus-4-5");
//  ↓ 对内置模型等价于 modelRuntime.getModel("anthropic", "claude-opus-4-5")
//    区别：getBuiltinModel 只查静态目录，modelRuntime.getModel() 查运行时全量（含自定义注册）

// 自定义模型（从 models.json 加载的）：必须走 modelRuntime.getModel()
const myModel = modelRuntime.getModel("my-proxy", "my-custom-model");
if (!myModel) throw new Error("没找到，请检查 models.json");
```

> **兼容旧代码**：`@earendil-works/pi-ai/compat` 子路径的 `getModel` 行为与 `getBuiltinModel` 完全一致（`export const getModel = getBuiltinModel`），但标记为 `@deprecated`。新代码请用 `getBuiltinModel`。同理，旧 `modelRegistry.find()` 改为 `modelRuntime.getModel()`。

### 完整示例：列模型 → 筛选 → 启动 session

```ts
import { getBuiltinModel } from "@earendil-works/pi-ai/providers/all";
import { ModelRuntime, createAgentSession } from "@earendil-works/pi-coding-agent";

const modelRuntime = await ModelRuntime.create();

// 1. 列出可用模型
const available = modelRuntime.getAvailableSnapshot();

if (available.length === 0) {
  // SDK 此处不会 throw——主动检查给用户友好提示
  // 不检查的话，createAgentSession 会返回带 modelFallbackMessage 的 session
  // 真正的 throw 延迟到 session.prompt() 时（A01 横向）
  throw new Error(
    "没有可用的模型，请先配置 API Key（见 B01-auth-config.md）"
  );
}

// 2. 按优先级挑选：优先 opus，否则取第一个可用
const preferred =
  available.find((m) => m.provider === "anthropic" && m.id === "claude-opus-4-5") ??
  available[0];

// 3. 启动 session
const { session, modelFallbackMessage } = await createAgentSession({
  model: preferred,
  modelRuntime,
});

// 4. 生产建议：检查 fallback 警告
if (modelFallbackMessage) {
  console.error("⚠️ SDK model fallback:", modelFallbackMessage);
}

try {
  await session.prompt("Hello");
} finally {
  session.dispose();
}
```

## Model 对象关键字段

`getAvailable()` / `getAll()` / `find()` 返回的 `Model<Api>` 对象：

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | `string` | 模型 ID，如 `"claude-opus-4-5"` |
| `name` | `string` | 显示名，如 `"Claude Opus 4.5"` |
| `provider` | `ProviderId` | Provider ID，如 `"anthropic"` / `"openai"` |
| `api` | `Api` | API 格式，如 `"anthropic-messages"` / `"openai-responses"` |
| `baseUrl` | `string` | 请求 base URL |
| `reasoning` | `boolean` | 是否支持 thinkingLevel（`false` 时只支持 `"off"`） |
| `input` | `("text" \| "image")[]` | 支持的输入类型 |
| `cost` | `ModelCost`（`{ input, output, cacheRead, cacheWrite }` + 可选 `tiers?: ModelCostTier[]`） | 单价（$/百万 token） |
| `contextWindow` | `number` | 上下文窗口大小 |
| `maxTokens` | `number` | 单次最大输出 token |
| `samplingParams?` | `Record<string, unknown>` | 默认采样参数，per-request 可覆盖 |
| `thinkingLevelMap?` | `ThinkingLevelMap` | thinkingLevel 映射（覆盖 provider 默认） |
| `headers?` | `Record<string, string>` | 模型级请求头 |
| `compat?` | 视 `api` 而定 | 兼容性覆盖（OpenAI/Anthropic 特有） |

## 自定义 models.json（B 档）

默认路径 `~/.pi/agent/models.json`，可放用户自定义的 provider + 模型。SDK 行为：

- **路径不存在不报错**——`ModelRuntime.create({ modelsPath: "/custom/path.json" })` 即使文件不存在也正常构造，自定义模型列表为空
- **解析错误不崩溃**——错误存到 `getError()` 返回值，内置模型仍可用
- **热重载**：`modelRuntime.refresh()` 从磁盘重新读取（重新 compose 所有 provider，registerProvider 注册的数据保留在 extensionProviders 中，不清空注册）

```ts
const rt = await ModelRuntime.create();                              // 默认 ~/.pi/agent/models.json
const rt = await ModelRuntime.create({ modelsPath: "/custom/path.json" }); // 自定义路径
const rt = await ModelRuntime.create({ modelsPath: null });          // 不加载 models.json

// 检查加载错误
const err = rt.getError();
if (err) console.error("models.json 解析错误：", err);
```

完整 `models.json` schema 见 [sdk_doc/05 §registerProvider](../sdk_doc/05-auth-model-registry.md)。

## 运行时 registerProvider（B 档）

程序化注入自定义 provider + 模型，不依赖 `models.json` 文件：

```ts
modelRuntime.registerProvider("my-proxy", {
  baseUrl: "https://proxy.example.com",
  apiKey: "$PROXY_API_KEY",       // 支持 $ENV_VAR 语法
  api: "anthropic-messages",
  models: [{
    id: "claude-sonnet-4-5-via-proxy",
    name: "Claude Sonnet 4.5 (via proxy)",
    reasoning: true,
    input: ["text", "image"],
    cost: { input: 3, output: 15, cacheRead: 0.3, cacheWrite: 3.75 },
    contextWindow: 200000,
    maxTokens: 16384,
  }],
});

// 之后 getAvailableSnapshot() / getModel() 都能查到
modelRuntime.unregisterProvider("my-proxy");  // 移除
```

⚠️ **覆盖行为**（实现于 `provider-composer.ts`）：

- 传入 `models` → **替换**该 provider 下的**所有**已有模型
- 只传 `baseUrl` / `headers` → 覆盖已有模型的 URL（不替换模型本身）
- 传入 `oauth` → 注册 OAuth provider（用于 `modelRuntime.login()`）

完整字段表见 [sdk_doc/05 §ProviderConfigInput](../sdk_doc/05-auth-model-registry.md)。

## 其他查询方法（B 档）

写模型选择 UI / 认证状态面板时常用（完整签名见 [sdk_doc/05](../sdk_doc/05-auth-model-registry.md)）：

| 方法 | 返回 | 用途 |
|------|------|------|
| `modelRuntime.hasConfiguredAuth(providerId)` | `boolean` | 单 provider 认证检查（不刷 OAuth） |
| `modelRuntime.isUsingOAuth(providerId)` | `boolean` | 该 provider 是否走 OAuth 凭据（订阅模式） |
| `modelRuntime.getProvider(id)` | `Provider \| undefined` | 取 provider 实例（含 displayName） |
| `modelRuntime.getProviderAuthStatus(providerId)` | `AuthStatus` | Provider 维度状态（参数为 providerId 字符串，不暴露 key 值，不执行命令） |
| `modelRuntime.getAuth(providerId \| model)` | `Promise<AuthResult \| undefined>` | Provider(providerId 字符串) 或 Model 维度取认证（会刷 OAuth） |
| `modelRuntime.getError()` | `string \| undefined` | models.json 解析错误（`undefined` = 无错） |
| `modelRuntime.refresh(options?)` | `Promise<ModelsRefreshResult>` | 重新从磁盘加载 + 重建动态注册 |

## 陷阱与已知问题

### 陷阱 1：`getAvailable()` 返回空数组时 SDK 不 throw

`getAvailable()` 返回 `[]` 时，SDK 不会主动报错——`createAgentSession({ model: undefined })` 会走 `findInitialModel()` 兜底失败，返回带 `modelFallbackMessage` 的 session；真正的 throw 延迟到 `session.prompt()` 时（[A01 延迟 throw](A01-minimal-startup.md)）。

**正确做法**：列模型后主动检查长度，别等 SDK 帮你发现：

```ts
const available = modelRuntime.getAvailableSnapshot();
if (available.length === 0) {
  throw new Error("没有可用模型，请配置 API Key（见 B01-auth-config.md）");
}
```

### 陷阱 2：顺序不保证

`getAvailable()` 的返回顺序由内部 `models` 数组决定——内置模型按 providers 字典序加载（`all.ts` 遍历顺序），自定义模型按 `models.json` / `registerProvider` 的调用顺序 push。**不保证**按字母、按 provider 分组、按优先级排序。

**错误做法**：

```ts
const model = modelRuntime.getAvailableSnapshot()[0];  // ❌ "取第一个"不等于"最优"
```

**正确做法**：用 `find()` 或 `getBuiltinModel()` 精确指定，或用 `Array.find()` 按条件筛选（见「完整示例」）。

### 陷阱 3：内置模型清单随版本变快

`anthropic.models.ts`、`openai.models.ts` 等是**自动生成**的（文件顶部有「This file is auto-generated by scripts/generate-models.ts」注释），随 SDK 版本发布更新。**不要在代码里写死假设「某个模型 ID 一定存在」**——例如 `claude-opus-4-5` 现在存在，但旧版本 SDK 可能没有。

**正确做法**：

```ts
import { getBuiltinModel } from "@earendil-works/pi-ai/providers/all";

const opus = getBuiltinModel("anthropic", "claude-opus-4-5");
if (!opus) {
  // 降级到 getAvailable() 取一个，或 throw 友好提示
  throw new Error("当前 SDK 版本不支持 claude-opus-4-5，请升级 @earendil-works/pi-ai");
}
```

### 陷阱 4：`registerProvider()` 会覆盖同 provider 的所有模型

如果你 `modelRuntime.registerProvider("anthropic", { models: [...] })`，**原有 anthropic 下所有内置模型会被清空**（覆盖逻辑在 `provider-composer.ts`）。这是「替换」不是「合并」。

**正确做法**：自定义 provider 用独立名字（如 `"my-anthropic-proxy"`），不要覆盖内置 provider 名。

## 关键细节

- 推荐入口是 `ModelRuntime.create()`，默认路径 `~/.pi/agent/models.json`（受 `PI_CODING_AGENT_DIR` 覆盖）
- `getModels()` / `getAvailableSnapshot()` / `getModel()` 都是**同步方法**——内部读缓存快照（不刷 OAuth、不读磁盘）
- `hasConfiguredAuth(providerId)` 也是同步的——只检查存在性，不触发 token 刷新
- `ModelRuntime.create({ modelsPath: null })` **不加载 `models.json`**——只能查到内置模型
- `ModelRuntime.create({ modelsPath })` 即使 path 不存在也不报错——自定义模型为空
- `getError()` 检查 models.json 解析错误——解析错误时内置模型仍可用
- `refresh()` 从磁盘重新加载 + 重新 compose 所有 provider（registerProvider 注册的数据保留在 extensionProviders 中，不清空）
- `registerProvider()` 会覆盖同 provider 下所有已有模型（**替换不是合并**）
- 认证系统**完全独立于 `cwd`**——ModelRuntime 不依赖工作目录

## 变体与延伸

- **精确指定模型**（`getBuiltinModel` + `createAgentSession({ model })`） → [场景 A02](A02-model-selection.md)
- **配置 API Key**（让 `getAvailableSnapshot()` 能查到更多模型） → [场景 B01](B01-auth-config.md)
- **自定义 models.json 路径 / `agentDir` 整目录切换** → [场景 B01](B01-auth-config.md)
- **内存模式 runtime**（不依赖 models.json） → 本文「自定义 models.json」节
- **动态注册 provider / OAuth 登录** → [sdk_doc/05-auth-model-registry.md](../sdk_doc/05-auth-model-registry.md)
- **未传 model 的兜底逻辑**（findInitialModel 算法） → [场景 A02 §未传 model 的兜底逻辑](A02-model-selection.md)
