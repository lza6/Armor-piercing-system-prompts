# 场景：用 Faux Provider 做测试 (H03)

## 这是什么 / 不是什么

**是**：通过 `registerFauxProvider()` 注册一个**完全本地、无网络**的模拟 AI Provider，按预设响应序列回答请求。用于在不调真实 LLM 的前提下，测试扩展逻辑、工具调用流程、Agent Loop 行为、错误恢复、流式处理、token 估算等。

**不是**：
- **不是真发请求**：所有响应来自本地队列，不会创建 TCP 连接，`baseUrl` 是字面 `http://localhost:0`（faux）。要测真实 HTTP 行为请用 [H02](H02-custom-provider.md) 接代理网关。
- **不是 Mock 整个 AgentSession**：Faux 只替换"模型调用"那一层（`stream` / `streamSimple`），不改 Agent loop、不改工具执行、不改 session 状态机。要完全替换用 [H01](H01-full-control.md)。
- **不是改默认 model**：`faux.getModel()` 拿到的是 Faux Model 实例，要把它传给 `createAgentSession({ model })` 才生效（见下方"最大陷阱"——光传 model 还不够）。
- **不是 auth 管理**：`fauxProvider()`（非 compat 路径，faux）的 auth resolve 永远返回 `{ auth: {} }`；但本文档通篇用的 compat `registerFauxProvider`（compat）**不走** auth resolve——它直接 `registerApiProvider`。不管哪条注册路径，Agent 的 streamFn 仍然会查 ModelRuntime，**必须**给 ModelRuntime 也配上（见最大陷阱节）。

## 什么时候用 / 不用会怎样

| 触发场景 | 用 H03 | 不用会怎样 |
|---------|--------|-----------|
| 写扩展拦截 `before_provider_request` / `after_provider_response` | 必用 Faux | 真发请求烧钱、响应不稳定 |
| 测试工具调用流程（多轮 toolCall → toolResult） | Faux + `fauxToolCall` | 难复现特定 LLM 响应序列 |
| 测试错误恢复（断网、429、500、abort） | Faux + `stopReason: "error"` 或 factory throw | 错误时机不可控 |
| 测试流式 UI（thinking、text、toolcall delta） | Faux 配 `tokensPerSecond` | 真 LLM 流式时序不稳定 |
| 测试 prompt cache 行为 | Faux 内置 promptCache per sessionId（faux） | 真 LLM cache 行为昂贵且难复现 |
| 测试多模型分支（reasoning on/off、不同 contextWindow） | Faux + `models: [...]` | 要给每个模型分别配真 API key |
| 想测真实 SDK 错误处理（重试、超时） | **不要用 Faux**，去 [H02](H02-custom-provider.md) 配代理网关 | Faux 走的是本地 stream，不会触发 HTTP 错误 |

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `registerFauxProvider(options?)` | 注册 Faux Provider，返回带 `state`/`setResponses`/`getModel`/`unregister` 的 handle | 本场景 + 源码 `packages/ai/src/compat` |
| `fauxAssistantMessage(content, options?)` | 构造一个完整的 `AssistantMessage`（含 usage / stopReason / errorMessage / responseId / timestamp） | 本场景 + 源码 `packages/ai/src/providers/faux` |
| `fauxToolCall(name, args, options?)` | 构造一个 `ToolCall` content block，`id` 默认 `tool:<ts>:<rand>` | 源码 `packages/ai/src/providers/faux` |
| `fauxText(text)` / `fauxThinking(text)` | 构造 `TextContent` / `ThinkingContent` block | 源码 `packages/ai/src/providers/faux` |
| `faux.setResponses([...])` | **替换**待消费响应队列 | 本场景 |
| `faux.appendResponses([...])` | **追加**到现有队列末尾 | 本场景 |
| `faux.getPendingResponseCount()` | 查队列剩余数量（断言"已消费完"用） | 源码 `packages/ai/src/providers/faux` |
| `faux.state.callCount` | 读已调用次数（累加，不会因 setResponses 重置） | 源码 `packages/ai/src/providers/faux` |
| `faux.unregister()` | 从 api-registry 删除 Faux（**不**清理 ModelRuntime，见陷阱 #2） | 源码 `packages/ai/src/compat`（`unregisterApiProviders`）/ `172-174`（`unregister` 方法） |

> ⚠️ **不要用 `createAgentSessionServices`**：该工厂（`agent-session-services`）强制内部 `DefaultResourceLoader`，且其内部 ModelRuntime 没配 faux provider，会让 faux 调用失败。本场景必须用 `createAgentSession` 并显式传 `modelRuntime`（见最大陷阱节）。

## ⚠️ 最大陷阱：光传 `model: faux.getModel()` 不够，ModelRuntime 也要配

**事实**：`createAgentSession` 创建的 Agent 在调 LLM 时走 `streamFn`，而 streamFn 第一步就是向 `modelRuntime` 请求认证：

```ts
// model-runtime.ts: prepareRequest() 内部（streamFn → modelRuntime.streamSimple → prepareRequest）
const resolution = await this.getAuth(model, { apiKey, env, signal });
if (!resolution) {
  throw new ModelsError("auth", `Provider is not configured: ${model.provider}`);
}
```

如果你只写 `createAgentSession({ model: faux.getModel() })`——默认 `ModelRuntime.create()` 里**没有** faux provider 的配置，`getAuth` 返回 `undefined`，agent 第一次 prompt 就抛 `Provider is not configured: faux`（`ModelsError`，auth 类，model-runtime）错误。

**证据**：`packages/coding-agent/src/core/sdk.ts`（streamFn 内部委托 modelRuntime）、`packages/coding-agent/src/core/model-registry`（兼容层 `getApiKeyAndHeaders` 委托 `runtime.getAuth`）。对照 `packages/coding-agent/test/suite/harness` 的真实用法——它显式做了 4 步配置才让 faux 跑起来。

> **注意 harness 与 createAgentSession 的 streamFn 路径差异**：harness（`test/suite/harness.ts`）的 `streamFn` 直接用 compat 的 `streamSimple`，走 api-registry，**绕过** ModelRuntime 的 `prepareRequest` / `getAuth`——所以 harness 不需要配 ModelRuntime 的 auth。而 `createAgentSession` 的 streamFn 走 `modelRuntime.streamSimple` → `prepareRequest` → `getAuth`，有 auth 校验。harness 虽然也传了 `modelRuntime: getModelRuntime(modelRegistry)`，但那只是给 AgentSession 做 model 解析用，streamFn 路径完全不同。

**正确做法**（最小可用模板，用 `ModelRuntime.create`）：

```ts
import { registerFauxProvider } from "@earendil-works/pi-ai/compat";
import { fauxAssistantMessage, fauxToolCall, fauxText } from "@earendil-works/pi-ai/providers/faux";
import {
  ModelRuntime,
  SessionManager,
  createAgentSession,
} from "@earendil-works/pi-coding-agent";

// 1. 注册 Faux Provider
const faux = registerFauxProvider({ tokensPerSecond: 100 });

// 2. ★ 关键：给 ModelRuntime 配上 faux provider 的 auth + 模型列表
//    不再有 AuthStorage.create() / ModelRegistry.create() 静态工厂
//    方案 A：直接用 ModelRuntime.create() 然后调 registerProvider
const modelRuntime = await ModelRuntime.create({ modelsPath: null, allowModelNetwork: false });

//    给 credentials 注入 faux 的 runtime key（通过 session 暴露的兼容 modelRegistry）
//    或在创建 ModelRuntime 时传 credentials 自定义 CredentialStore
modelRuntime.registerProvider(faux.models[0].provider, {
  baseUrl: faux.models[0].baseUrl,      // http://localhost:0
  apiKey: "faux-key",
  api: faux.api,                         // "faux" 或随机 id
  models: faux.models.map((m) => ({
    id: m.id,
    name: m.name,
    api: m.api,
    reasoning: m.reasoning,
    input: m.input,
    cost: m.cost,
    contextWindow: m.contextWindow,
    maxTokens: m.maxTokens,
    baseUrl: m.baseUrl,
  })),
});

// 3. 预设响应序列
faux.setResponses([
  fauxAssistantMessage("Hello! I'm a mock assistant."),
  fauxAssistantMessage([
    fauxText("Let me check that for you."),
    fauxToolCall("read", { file_path: "/src/config.ts" }),
  ], { stopReason: "toolUse" }),
]);

// 4. 创建 session 时同时传 model + modelRuntime
const { session } = await createAgentSession({
  model: faux.getModel(),
  sessionManager: SessionManager.inMemory(),
  modelRuntime,                          // ★ 不能漏（取代 modelRegistry）
  tools: ["read"],
});

try {
  await session.prompt("hi");
  await session.prompt("check config");
} finally {
  session.dispose();
  faux.unregister();
}
```

> **测试场景的简化方案**：coding-agent 内部测试 harness 用 `createInMemoryModelRegistry(authStorage)` + `getModelRuntime(registry)` 组合构造。第三方测试代码可以直接 `ModelRuntime.create({ modelsPath: null, allowModelNetwork: false })` 然后用 `registerProvider` 注入 faux 配置。

**简化的替代方案**：如果你只关心 stream 层（不走 AgentSession），可以直接用 compat 的 `complete()` / `stream()`：

```ts
import { complete, registerFauxProvider, fauxAssistantMessage } from "@earendil-works/pi-ai/compat";

const faux = registerFauxProvider();
faux.setResponses([fauxAssistantMessage("hi")]);

const response = await complete(faux.getModel(), {
  messages: [{ role: "user", content: "hello", timestamp: Date.now() }],
});
// response.content === [{ type: "text", text: "hi" }]
faux.unregister();
```

这种方式不需要配 ModelRegistry——`complete` 直接走 api-registry，不经过 AgentSession 的 auth 校验。

## 核心代码：多轮 AgentSession 测试

```ts
import { registerFauxProvider } from "@earendil-works/pi-ai/compat";
import {
  fauxAssistantMessage,
  fauxToolCall,
  fauxText,
  fauxThinking,
} from "@earendil-works/pi-ai/providers/faux";
import {
  ModelRuntime,
  SessionManager,
  createAgentSession,
} from "@earendil-works/pi-coding-agent";

const faux = registerFauxProvider({ tokensPerSecond: 100 });

// 配 ModelRuntime（取代旧的 AuthStorage.inMemory + ModelRegistry.inMemory 组合）
const modelRuntime = await ModelRuntime.create({ modelsPath: null, allowModelNetwork: false });
modelRuntime.registerProvider(faux.models[0].provider, {
  baseUrl: faux.models[0].baseUrl,
  apiKey: "faux-key",
  api: faux.api,
  models: faux.models.map((m) => ({
    id: m.id, name: m.name, api: m.api,
    reasoning: m.reasoning, input: m.input, cost: m.cost,
    contextWindow: m.contextWindow, maxTokens: m.maxTokens, baseUrl: m.baseUrl,
  })),
});

// 预设完整的多轮交互序列
faux.setResponses([
  // 第 1 轮：纯文本回复
  fauxAssistantMessage("Hello! I'm a mock assistant."),

  // 第 2 轮：thinking + text + tool call
  fauxAssistantMessage(
    [fauxThinking("user wants config..."), fauxText("Let me check."),
     fauxToolCall("read", { file_path: "/src/config.ts" })],
    { stopReason: "toolUse" },
  ),

  // 第 3 轮：动态响应（工厂函数，4 个参数）
  (context, _options, state, model) => {
    const userMsg = context.messages.at(-1);
    return fauxAssistantMessage(
      `[${model.id}] Call #${state.callCount}. Tools: ${context.tools?.length ?? 0}`,
    );
  },

  // 第 4 轮：模拟错误（stopReason="error" → stream 发 error 事件）
  fauxAssistantMessage("Simulated failure", {
    stopReason: "error",
    errorMessage: "Simulated failure",
  }),
]);

const { session } = await createAgentSession({
  model: faux.getModel(),
  sessionManager: SessionManager.inMemory(),
  modelRuntime,                  // 接 modelRuntime（不接 modelRegistry）
  tools: ["read"],
});

try {
  await session.prompt("hi");
  await session.prompt("check config");
  await session.prompt("dynamic");
  await session.prompt("fail me");

  console.log("Total LLM calls:", faux.state.callCount);
  console.log("Pending:", faux.getPendingResponseCount());
} finally {
  session.dispose();
  faux.unregister();
}
```

## ⚠️ compat vs 非 compat 返回值不可混用

本文档通篇使用 compat 路径 `registerFauxProvider()`（返回 `FauxProviderRegistration`）。非 compat 路径 `fauxProvider()` 返回 `FauxProviderHandle`。两者字段不同，**不能混用**：

| 字段 | `FauxProviderRegistration`（compat） | `FauxProviderHandle`（非 compat） |
|------|--------------------------------------|-----------------------------------|
| `.provider` | ❌ 无 | ✅ `Provider` 对象 |
| `.api` | ✅ `string` | ✅ `string` |
| `.models` | ✅ `Model[]` | ✅ `Model[]` |
| `.getModel()` | ✅ | ✅ |
| `.state` | ✅ | ✅ |
| `.setResponses()` | ✅ | ✅ |
| `.appendResponses()` | ✅ | ✅ |
| `.getPendingResponseCount()` | ✅ | ✅ |
| `.unregister()` | ✅ | ❌ 无 |

**P0 教训**：如果你用 compat 路径（`registerFauxProvider`），`faux.provider` 是 `undefined`，传给 `modelRuntime.registerProvider()` 会报错。compat 路径应改用 `faux.models[0].provider`（string 类型）。非 compat 路径（`fauxProvider()`）才有 `.provider`（Provider 对象）。

## `RegisterFauxProviderOptions` 完整字段

源码：`packages/ai/src/providers/faux`

| 字段 | 类型 | 默认 | 含义 |
|------|------|------|------|
| `api` | `string` | `faux:<ts>:<rand>`（`randomId(DEFAULT_API)`，不传时；faux）。显式传 `api` 才固定为该值 | API 标识。**传自定义值时**：调 `registerFauxProvider({ api: "faux:test" })` 后，所有 message 的 `.api` 字段都会被改写成 `"faux:test"`（见 faux `cloneMessage`） |
| `provider` | `string` | `"faux"` | Provider 标识。和 `api` 一样会被改写到 message |
| `models` | `FauxModelDefinition[]` | 单模型 `faux-1` | 模型列表。第一个模型是 `getModel()` 的默认返回 |
| `tokensPerSecond` | `number` | `undefined`（即瞬时） | 流式速率。`<= 0` 或 `undefined` 都表示不延迟（faux）。**用于模拟真实流式时序** |
| `tokenSize` | `{ min?, max? }` | `{ min: 3, max: 5 }` | 每个 delta chunk 的 token 大小区间（字符数 = tokenSize × 4）。**只影响 delta 粒度，不影响总内容** |

### `FauxModelDefinition` 字段

源码：`packages/ai/src/providers/faux`

| 字段 | 类型 | 默认 | 含义 |
|------|------|------|------|
| `id` | `string` | **必填** | 模型 id（如 `"faux-fast"`） |
| `name` | `string?` | = `id` | 显示名 |
| `reasoning` | `boolean?` | `false` | 是否支持 thinking。**影响**：`streamWithDeltas` 不会因 reasoning false 拒发 thinking 块，但上层 Agent 会在 `thinkingLevel !== "off"` 时检查 model.reasoning 决定是否发 thinking 参数 |
| `input` | `("text"\|"image")[]?` | `["text", "image"]` | 支持的输入类型 |
| `cost` | `{ input, output, cacheRead, cacheWrite }?` | 全 0 | 单价（Faux 永远算 0 成本） |
| `contextWindow` | `number?` | `128000` | 上下文窗口 |
| `maxTokens` | `number?` | `16384` | 单次最大输出 |

## 工厂函数：动态生成响应

队列里除了 `AssistantMessage`，还可以放 **工厂函数**。源码：`packages/ai/src/providers/faux`

```ts
export type FauxResponseFactory = (
  context: Context,
  options: StreamOptions | undefined,
  state: { callCount: number },
  model: Model<string>,
) => AssistantMessage | Promise<AssistantMessage>;
```

| 参数 | 类型 | 含义 |
|------|------|------|
| `context` | `Context` | 完整上下文——`systemPrompt` / `messages`（含历轮 toolResult）/ `tools`（含 schema） |
| `options` | `StreamOptions \| undefined` | stream 调用选项，含 `sessionId`、`cacheRetention`、`signal`（abort）、`apiKey` 等 |
| `state` | `{ callCount: number }` | 调用计数器（**注意**：调用 factory 前 callCount 已经自增，所以首次 factory 收到的是 `callCount: 1`） |
| `model` | `Model<string>` | 当前请求的 model 实例（多模型时用于分支） |

**返回值**：`AssistantMessage` 或 `Promise<AssistantMessage>`（支持 async）。返回 Promise 时会 await，适合做异步上下文读取。

**常见用法**：

```ts
// 按 model.id 分支
faux.setResponses([
  (_ctx, _opt, _state, model) =>
    model.reasoning
      ? fauxAssistantMessage([fauxThinking("hmm..."), fauxText("ok")])
      : fauxAssistantMessage("ok"),
]);

// 异步读取数据库
faux.setResponses([
  async (ctx) => {
    const expected = await db.query("select ...");
    return fauxAssistantMessage(`expected: ${expected}`);
  },
]);

// 测试 abort
faux.setResponses([
  async (_ctx, opt) => {
    await new Promise((resolve) => setTimeout(resolve, 1000));
    if (opt?.signal?.aborted) {
      return fauxAssistantMessage("aborted", { stopReason: "aborted", errorMessage: "aborted" });
    }
    return fauxAssistantMessage("done");
  },
]);
```

## 错误注入：3 种方式

### 方式 1：`stopReason: "error"` + `errorMessage`（构造错误 AssistantMessage）

```ts
faux.setResponses([
  fauxAssistantMessage("Simulated 500", {
    stopReason: "error",
    errorMessage: "Internal Server Error",
  }),
]);
```

**行为**（faux）：stream 会发 `{ type: "error", reason: "error", error: message }` 事件，然后 `stream.end(message)`。**注意**：内容 `"Simulated 500"` 仍然会被流式输出（走 text_start / text_delta / text_end），只是在末尾发 error 事件而不是 done。

### 方式 2：Factory 抛异常

```ts
faux.setResponses([
  (_ctx, _opt, _state, _model) => {
    throw new Error("boom");
  },
]);
```

**行为**（faux）：被 try/catch 捕获，发出 `{ type: "error", reason: "error", error: errorMessage }`，其中 errorMessage 是 `error.message`（Error 实例）或 `String(error)`（其他值）。**和方式 1 区别**：不会流式输出任何 content（content 是空数组）。

### 方式 3：队列耗尽自动错误

不预设足够响应即可：

```ts
faux.setResponses([fauxAssistantMessage("only one")]);
await session.prompt("first");   // OK
await session.prompt("second");  // 抛 "No more faux responses queued"
```

**行为**（faux）：构造 `errorMessage: "No more faux responses queued"` 的 AssistantMessage，stopReason 为 "error"。

> **测试断言**：所有 3 种错误都让最终 `AssistantMessage.stopReason === "error"`、`errorMessage` 非空。差异在 stream 中途是否有 content delta。

## 流式模拟行为

Faux Provider 不创建真连接，但**模拟**流式时序，方便测试 UI delta 处理。

### `tokensPerSecond` 控制整体速率

```ts
const faux = registerFauxProvider({ tokensPerSecond: 10 });  // 慢速
```

- `undefined` 或 `<= 0`：每个 chunk 走 `queueMicrotask`，几乎瞬时
- 正数：每个 chunk 延迟 `(tokens / tokensPerSecond) * 1000` ms（faux）

### `tokenSize` 控制 delta 粒度

```ts
const faux = registerFauxProvider({ tokenSize: { min: 1, max: 1 } });  // 每次只 1 token
```

输入文本被切成 `tokenSize * 4` 个字符的 chunk（faux），每个 chunk 之间按 `tokensPerSecond` 延迟。

### Abort 行为

streamWithDeltas 在每个 chunk 边界检查 `signal?.aborted`（faux、341-346 等）：

- **stream 开始前 abort**：发 `{ type: "error", reason: "aborted", error: abortedMessage }`，`content: []`
- **stream 中途 abort**：已发出的 delta 保留，后续不发；末尾发 `error` 事件，`stopReason: "aborted"`、`errorMessage: "Request was aborted"`

## Usage 估算：自动 + 可选 prompt cache 模拟

**自动估算**（faux）：每次响应都会算 usage，无需手动指定。

| 字段 | 算法 |
|------|------|
| `input` | `Math.ceil(serializeContext(context).length / 4)` |
| `output` | `Math.ceil(assistantContentToText(content).length / 4)` |
| `cacheRead` | 见下方 cache 逻辑 |
| `cacheWrite` | 见下方 cache 逻辑 |
| `totalTokens` | `input + output + cacheRead + cacheWrite` |
| `cost` | 永远全 0（faux） |

**Context 序列化**（faux）：`system:<prompt> + 每条 message + tools JSON`。具体见 `serializeContext`。

### Prompt cache 模拟（per sessionId）

如果 `options.sessionId` 存在且 `cacheRetention !== "none"`（faux）：

- **首次**：`cacheWrite = input tokens`、`cacheRead = 0`
- **后续**（相同 sessionId）：算与上次 prompt 的**公共前缀**长度，前缀部分计入 `cacheRead`，差异部分计入 `cacheWrite`、`input = promptTokens - cacheRead`
- **跨 sessionId**：不共享 cache（faux 查 `promptCache.get(sessionId)`）

> **测试断言**：`first.usage.cacheWrite > 0`、`second.usage.cacheRead > 0`、`crossSession.usage.cacheRead === 0`。源码测试：`packages/ai/test/faux-provider.test`。

## `FauxContentBlock` 类型限制

源码：`packages/ai/src/providers/faux`

```ts
export type FauxContentBlock = TextContent | ThinkingContent | ToolCall;
```

**不能放 ImageContent**——faux 的 `fauxAssistantMessage` 只接受上述 3 种 block。如果想测试 image 输入，要在 **user message** 那侧塞 image（context.messages[].content 支持 ImageContent），faux 的 `serializeContext` 会把 image 算成 `[image:mime:length]` 估算 token（faux）。

## `fauxAssistantMessage` 完整签名

源码：`packages/ai/src/providers/faux`

```ts
function fauxAssistantMessage(
  content: string | FauxContentBlock | FauxContentBlock[],
  options?: {
    stopReason?: AssistantMessage["stopReason"];  // 默认 "stop"
    errorMessage?: string;                        // 默认 undefined
    responseId?: string;                          // 默认 undefined
    timestamp?: number;                           // 默认 Date.now()
  },
): AssistantMessage;
```

**重要**：返回的 message 的 `api` / `provider` / `model` 字段是**占位**（`"faux"` / `"faux"` / `"faux-1"`，见 faux）。在 stream 流式时会被 `cloneMessage`（faux）**改写**为 `registerFauxProvider` 时传入的 `api` / `provider` / 当前 model id。所以测试时如果断言 `message.provider === "my-custom"`，要记得 `registerFauxProvider({ provider: "my-custom" })`。

## `AssistantMessageEvent` 协议（stream 事件类型）

Faux 发出的事件符合 `packages/ai/src/types` 的协议。完整 12 种：

| 事件 type | 何时发 | 关键字段 |
|-----------|-------|---------|
| `start` | stream 开始 | `partial: AssistantMessage`（content 空数组） |
| `text_start` | 文本块开始 | `contentIndex` / `partial` |
| `text_delta` | 文本块增量 | `delta: string` / `contentIndex` / `partial` |
| `text_end` | 文本块结束 | `contentIndex` / `content: string`（完整文本）/ `partial` |
| `thinking_start` | thinking 块开始 | `contentIndex` / `partial` |
| `thinking_delta` | thinking 块增量 | `contentIndex` / `delta: string` / `partial` |
| `thinking_end` | thinking 块结束 | `contentIndex` / `content: string`（完整文本）/ `partial` |
| `toolcall_start` | 工具调用开始 | `contentIndex` / `partial`（arguments 空对象） |
| `toolcall_delta` | 工具参数 JSON 增量 | `contentIndex` / `delta: string`（JSON 片段）/ `partial` |
| `toolcall_end` | 工具调用结束 | `contentIndex` / `toolCall: ToolCall`（完整对象）/ `partial` |
| `done` | 正常结束 | `reason: "stop" \| "length" \| "toolUse"` / `message` |
| `error` | 异常结束 | `reason: "aborted" \| "error"` / `error: AssistantMessage` |

> **`done` 和 `error` 是互斥终止事件**：每个 stream 必发其一，且是最后一个事件。

## 常见误期待与陷阱

1. **❌ 只传 `model: faux.getModel()` 就期望能跑** → 默认 ModelRuntime 不认识 faux provider，streamFn 会因 `getAuth` 返回 undefined 抛错。**必做**：同时传 `modelRuntime`（已配 faux provider 的）。见最大陷阱节。
2. **❌ 期望 `faux.unregister()` 会清理 ModelRuntime** → 不会。`unregister` 只调 `unregisterApiProviders(sourceId)` 删 api-registry（compat 定义 / 172-174 方法体），ModelRuntime 中通过 `registerProvider` 注册的 provider 配置仍然在。如果测试间需要"干净状态"，要重建 `ModelRuntime.create({ modelsPath: null })` 或单独调 `modelRuntime.unregisterProvider("faux")`。
3. **❌ 期望 `state.callCount` 在 `setResponses` 后重置** → 不会。`callCount` 是 streamFn 调用次数累加（faux），只增不减。`setResponses` 只换队列。要重置 callCount 必须重新 `registerFauxProvider()`。
4. **❌ 期望 `tokensPerSecond: 0` 表示"尽可能快"** → 实际是走 `queueMicrotask`（faux），即下一个微任务就发 chunk，**仍然不是同步**。要完全同步只能不传（undefined）。
5. **❌ 期望 factory 抛错会终止 agent loop** → 不会。Faux 把异常包装成 `AssistantMessage(stopReason: "error")` 返回（faux），agent loop 看到的是"一次失败调用"，可能触发重试或传给上层错误处理。要彻底终止需要让 agent 进入 fatal 状态（如连续错误超过 retry 上限）。
6. **❌ 期望 factory 里的 `state.callCount` 是 0（首次调用）** → 实际是 1（faux 在 factory 调用前就自增）。第 N 次调用收到的是 N。
7. **❌ 期望 `fauxAssistantMessage` 接受 `ImageContent`** → 不接受（faux `FauxContentBlock` 只有 3 种）。要测 image 输入走 user message 侧。
8. **❌ 期望 `api: "anthropic-messages"` 能让 faux 伪装成 anthropic** → `api` 字段只是个标识，不会走真实 anthropic 协议。streamFn 是 faux 自己的本地实现（faux），和 anthropic SDK 完全无关。要测真实 anthropic 行为去用 Mock Service Worker 或 H02 代理。
9. **❌ 期望 abort 后 content 是完整的** → 不是。abort 时已经发出的 delta 保留在 partial.content 里，但**没发完的 block 会停在中间**（如 text 只有一半）。`stopReason: "aborted"`、`errorMessage: "Request was aborted"`。
10. **❌ 期望多模型共用一个响应队列** → 不是。`pendingResponses` 是 per-provider 的（faux），所有模型共享同一队列。如果要多模型分别响应，要在 factory 内根据 `model.id` 分支。

## 变体与延伸

- 接入真实自定义 Provider → [H02](H02-custom-provider.md)
- 完全手写 ResourceLoader / ModelRegistry → [H01](H01-full-control.md)
- 测试工具拦截扩展 → [场景 E01](E01-tool-intercept.md)
- Faux 完整源码 → `packages/ai/src/providers/faux.ts`
- Faux 官方测试用例 → `packages/ai/test/faux-provider.test.ts`（覆盖 cache / abort / 多模型 / 错误注入等）
- coding-agent 用 Faux 的真实测试 harness → `packages/coding-agent/test/suite/harness`
