# 16. 自定义 Provider -- 接入第三方模型提供商

## 概述

**什么时候需要自定义 provider？** 当 pi 内置的 provider（anthropic / openai / google / deepseek 等）满足不了你的接入场景时，就需要自定义。典型两类：

- **走代理 / 网关**：公司内部有 AI 网关（统一鉴权、审计、路由），请求要先打网关再转发到真实模型。内置 provider 的端点改不掉，得自定义 → 用**模式 1（标准 API 格式）**或**模式 4（覆盖已有 provider 的 baseUrl）**。
- **非标准 API**：目标服务不是标准 Anthropic / OpenAI 格式（私有协议、特殊鉴权、需要预处理请求/响应），内置 stream 处理器跑不通 → 用**模式 2（自定义流式 streamSimple）**。
- **需要 OAuth/SSO**：企业 provider 要走 SSO 登录拿 token → 用**模式 3（OAuth）**。
- **需要动态模型列表 / 复杂过滤**：模型目录要运行时从远端拉、或按凭证过滤暴露的模型 → 用**模式 5（完整 Provider 对象）**。

不自定义会怎样？pi 只认内置 provider，你的网关/私有模型在 `/model` 选择器里根本看不到，调用也就无从发起。

---

`pi.registerProvider(name, config)` 允许扩展注册自定义的 AI 模型提供商。通过此 API，你可以接入任何兼容 Anthropic Messages API、OpenAI Responses API、OpenAI Chat Completions API 或自定义流式协议的模型服务。

每个 provider 可包含多个模型定义，支持 API Key（含环境变量插值）和 OAuth 两种认证方式。注册后的 provider 会出现在 `/model` 选择器中和 `/login` 支持的 provider 列表中。

## API 签名

```ts
// 在 ExtensionAPI 上
registerProvider(provider: Provider): void                      // 重载 1：注册完整 pi-ai Provider 对象（高级用法，见模式 5）
registerProvider(name: string, config: ProviderConfig): void    // 重载 2：注册静态配置（常用，见模式 1-4）

// 注销已注册的 provider
unregisterProvider(name: string): void
```

`registerProvider` 在扩展加载阶段和运行时均可调用：
- 加载阶段：注册请求排队，等核心就绪后批量处理
- 运行时：立即生效，直接更新 ModelRuntime

**两种重载的差异**：

| 重载 | 输入 | 能力 | 适用场景 |
|------|------|------|---------|
| `registerProvider(provider: Provider)` | 完整 pi-ai Provider 对象 | 含 `getModels()` / `refreshModels()` / `filterModels()` / `stream()` / `streamSimple()` / `auth` 运行时能力 | 动态模型刷新、复杂认证、模型过滤、自定义流 |
| `registerProvider(name, config: ProviderConfig)` | 静态配置对象 | 描述性配置（baseUrl + apiKey + models 列表） | 简单 OpenAI 兼容 API、固定模型列表 |

证据：`registerProvider` 双重载见 `core/extensions/types.ts`，分发逻辑见 `model-registry.ts`，Provider 接口见 `packages/ai/src/models.ts`。

## ProviderConfig 参数

```ts
interface ProviderConfig {
  /** UI 中显示的 provider 名称 */
  name?: string;

  /** API 端点地址（定义模型时必填） */
  baseUrl?: string;

  /** API Key。支持字面值、$ENV_VAR / ${ENV_VAR} 插值、!command 前置命令 */
  apiKey?: string;

  /** API 协议类型 */
  api?: Api;

  /** 自定义流式处理器（用于非标准 API 格式） */
  streamSimple?: (model: Model<Api>, context: Context, options?: SimpleStreamOptions) => AssistantMessageEventStream;

  /** 刷新此 provider 的模型列表。返回的模型列表由框架自动通过 `context.publish({ update })` 发布，
   *  无需手动 publish。Full provider extensions 能力。 */
  refreshModels?(context: RefreshModelsContext): Promise<ProviderModelConfig[]>;

  /** 自定义请求头 */
  headers?: Record<string, string>;

  /** ⚠️ **默认 `false`**。设为 `true` 时，使用 API Key 自动添加 `Authorization: Bearer` 头（协议无关，源码 `provider-composer.ts` 的 `withConfiguredAuth`）。
   * 注意：openai-completions 下 OpenAI SDK 已无条件带 Bearer（apiKey 配好即有），**无需此字段**；
   * 主要用于「要求 Bearer 的非 openai 协议代理」（如 Anthropic 代理只认 Bearer）。自定义鉴权用 `headers`。
   * 优先级：若同时设了 `headers: { Authorization: ... }`，**显式 header 优先**，`authHeader` 生成的值被覆盖。*/
  authHeader?: boolean;

  /** 模型列表。如果提供，替换该 provider 的所有已有模型 */
  models?: ProviderModelConfig[];

  /** OAuth 认证配置（支持 /login 命令） */
  oauth?: OAuthConfig;
}
```

### Api 类型说明

| Api 值 | 说明 |
|--------|------|
| `"anthropic-messages"` | Anthropic Messages API |
| `"openai-responses"` | OpenAI Responses API |
| `"openai-completions"` | OpenAI Chat Completions API（含智谱、DeepSeek、Ollama 等兼容 API） |
| `"azure-openai-responses"` | Azure OpenAI Responses API |
| `"openai-codex-responses"` | OpenAI Codex Responses API |
| `"mistral-conversations"` | Mistral Conversations API |
| `"bedrock-converse-stream"` | AWS Bedrock Converse Stream |
| `"google-generative-ai"` | Google Gemini API |
| `"google-vertex"` | Google Vertex AI |
| `"pi-messages"` | pi 内部协议（高级用法，通常不需要手动指定） |
| `(string & {})` | 自定义 API 标识符（需配合 `streamSimple`） |

### OAuthConfig 结构

```ts
oauth?: {
  /** 登录 UI 中显示的 provider 名称 */
  name: string;
  /** @deprecated 仅为源码兼容保留，规范化的鉴权流程会忽略此字段 */
  usesCallbackServer?: boolean;
  /** 执行登录流程，返回凭证 */
  login(callbacks: OAuthLoginCallbacks): Promise<OAuthCredentials>;
  /** 刷新过期凭证。⚠️ 必须接收 signal 参数， aborted 时应抛出 AbortError */
  refreshToken(credentials: OAuthCredentials, signal: AbortSignal): Promise<OAuthCredentials>;
  /** 将凭证转换为 API Key 字符串 */
  getApiKey(credentials: OAuthCredentials): string;
  /** 可选：根据凭证修改模型配置 */
  modifyModels?(models: Model<Api>[], credentials: OAuthCredentials): Model<Api>[];
}
```

> 💡 **callbacks 完整方法**：上面示例只展示了 `onAuth` + `onPrompt`。`OAuthLoginCallbacks` 还包含 `onDeviceCode`（设备码流程）、`onSelect`（多选项选择）、`onManualCodeInput`、`onProgress`、`signal` 等方法。完整定义见 `OAuthLoginCallbacks` 接口（`packages/ai/src/compat/extension-oauth-types.ts`）。

## ProviderModelConfig 参数

```ts
interface ProviderModelConfig {
  /** 模型 ID（如 "claude-sonnet-4-20250514"） */
  id: string;

  /** 显示名称（如 "Claude 4 Sonnet"） */
  name: string;

  /** API 类型覆盖（可覆盖 provider 级别的 api） */
  api?: Api;

  /** API 端点覆盖（可覆盖 provider 级别的 baseUrl） */
  baseUrl?: string;

  /** 是否支持扩展思考（extended thinking） */
  reasoning: boolean;

  /** 思考级别映射（将 pi 的 thinking level 映射为 provider 专用值） */
  thinkingLevelMap?: { [level: string]: string | null };

  /** 支持的输入类型 */
  input: ("text" | "image")[];

  /** Token 价格（用于成本追踪，可为 0） */
  cost: { input: number; output: number; cacheRead: number; cacheWrite: number; tiers?: { inputTokensAbove: number; input: number; output: number; cacheRead: number; cacheWrite: number }[] };

  /** 最大上下文窗口（tokens） */
  contextWindow: number;

  /** 最大输出 tokens */
  maxTokens: number;

  /** 自定义请求头（模型级别覆盖） */
  headers?: Record<string, string>;

  /** OpenAI 兼容性设置。对于非 OpenAI 原生的兼容 API（智谱、DeepSeek、Ollama 等），
      通常需要覆盖默认值以避免发送不支持的参数。见下方"compat 常见覆盖"章节。 */
  compat?: {
    supportsDeveloperRole?: boolean;
    supportsStore?: boolean;
    maxTokensField?: "max_tokens" | "max_completion_tokens";
    supportsReasoningEffort?: boolean;
    // ... 更多字段见 pi-ai 源码 detectCompat 函数
  };
}
```

## 关键模式

### 模式 1：标准 API 格式的 Provider

当目标服务实现了标准 Anthropic Messages API 或 OpenAI 兼容格式时，只需设置 `api` 即可，无需自定义 `streamSimple`：

```ts
pi.registerProvider("my-proxy", {
  baseUrl: "https://proxy.example.com",
  apiKey: "$PROXY_API_KEY",
  api: "anthropic-messages",   // 使用内置的 Anthropic 流式处理
  authHeader: true,            // 仅当代理要求 Bearer（而非 x-api-key）时；Anthropic 协议使用 x-api-key 头（anthropic-messages.ts:287），默认不需要 authHeader
  name: "My Proxy",
  models: [
    {
      id: "claude-sonnet-4-20250514",
      name: "Claude 4 Sonnet (Proxy)",
      reasoning: true,
      input: ["text", "image"],
      cost: { input: 3, output: 15, cacheRead: 0.3, cacheWrite: 3.75 },
      contextWindow: 200000,
      maxTokens: 64000,
    },
  ],
});
```

### 模式 2：自定义流式 API

当目标服务使用非标准 API 格式时，需要实现 `streamSimple` 处理器。该函数接收 `model`、`context`、`options`，返回 `AssistantMessageEventStream`：

```ts
import {
  type AssistantMessage,
  createAssistantMessageEventStream,
} from "@earendil-works/pi-ai";

pi.registerProvider("custom-api", {
  baseUrl: "https://api.custom.com",
  apiKey: "$CUSTOM_API_KEY",
  api: "custom-api-format",   // 自定义标识符
  streamSimple(model, context, options) {
    const stream = createAssistantMessageEventStream();

    (async () => {
      // 1. 初始化 output AssistantMessage（所有事件都引用同一个 output 对象）
      const output: AssistantMessage = {
        role: "assistant",
        content: [],
        api: model.api,
        provider: model.provider,
        model: model.id,
        usage: {
          input: 0,
          output: 0,
          cacheRead: 0,
          cacheWrite: 0,
          totalTokens: 0,
          cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0, total: 0 },
        },
        stopReason: "pending",
        timestamp: Date.now(),
      };

      try {
        // 2. push start 事件（partial 字段始终是当前 output 的引用）
        stream.push({ type: "start", partial: output });

        // 3. 构建并发送请求
        const response = await fetch(model.baseUrl, {
          method: "POST",
          headers: {
            "Authorization": `Bearer ${options?.apiKey}`,
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            messages: context.messages,
            system: context.systemPrompt,
            tools: context.tools,
            max_tokens: options?.maxTokens,
          }),
          signal: options?.signal,
        });

        // 4. 读取 SSE 流或 JSON 响应，边读边 push 内容事件
        //    （以 text 块为例；thinking/toolcall 块的事件序列同理，用 contentIndex 区分）
        //    output.content.push({ type: "text", text: "" });
        //    stream.push({ type: "text_start", contentIndex: 0, partial: output });
        //    while (...) { block.text += delta; stream.push({ type: "text_delta", contentIndex, delta, partial: output }); }
        //    stream.push({ type: "text_end", contentIndex: 0, content: block.text, partial: output });
        //
        //    流结束时把 output.stopReason 设为 "stop" / "length" / "toolUse"。

        // 5. 收尾：检查 stopReason，push done 或抛错进入 catch
        if (output.stopReason === "pending") {
          throw new Error("Provider stream ended without a stop reason");
        }
        if (output.stopReason === "error" || output.stopReason === "aborted") {
          throw new Error(output.errorMessage || "An unknown error occurred");
        }

        stream.push({ type: "done", reason: output.stopReason, message: output });
        stream.end();
      } catch (error) {
        // ⚠️ error 字段必须是一个 AssistantMessage（即 output），不能是 raw Error 对象。
        //    先把 stopReason / errorMessage 写到 output 上，再把 output 作为 error 字段 push 出去。
        output.stopReason = options?.signal?.aborted ? "aborted" : "error";
        output.errorMessage = error instanceof Error ? error.message : String(error);
        stream.push({ type: "error", reason: output.stopReason, error: output });
        stream.end();
      }
    })();

    return stream;
  },
  models: [ /* ... */ ],
});
```

> ⚠️ **事件序列与 contentIndex**（完整顺序，见 `AssistantMessageEvent` 类型，`packages/ai/src/types.ts`）：
> 1. `start`（携带 `partial: AssistantMessage`）——必须先发
> 2. 内容事件（可重复，每个内容块用同一个 `contentIndex` 串起来）：
>    - 文本：`text_start` → `text_delta`(多次) → `text_end`
>    - 思考：`thinking_start` → `thinking_delta`(多次) → `thinking_end`
>    - 工具调用：`toolcall_start` → `toolcall_delta`(多次) → `toolcall_end`
> 3. 终止事件（二选一，发完立即 `stream.end()`）：
>    - `done` —— `reason: "stop" | "length" | "toolUse"`，`message: AssistantMessage`
>    - `error` —— `reason: "aborted" | "error"`，**`error: AssistantMessage`**（不是 Error 对象）
>
> 每个 `partial` 字段都是**当前 output 状态的快照引用**——边收数据边更新 `output.content` / `output.usage` / `output.stopReason`，然后把 `output` 放进 `partial` 推出去。

#### 模式 2b：复用内置 stream 实现（推荐写法）

如果目标 API 是标准 Anthropic / OpenAI 格式但需要预处理请求/响应（如注入网关签名、改 header），可复用 `pi-ai` 内置的 stream 函数，避免从零实现 SSE 解析：

```ts
// 新入口：每个 API 模块同时导出 stream 和 streamSimple，槽位里必须用 streamSimple
import { streamSimple as streamSimpleAnthropic } from "@earendil-works/pi-ai/api/anthropic-messages";
import { streamSimple as streamSimpleOpenAIResponses } from "@earendil-works/pi-ai/api/openai-responses";

pi.registerProvider("corp-gateway", {
  baseUrl: "https://ai.corp.com",
  apiKey: "$CORP_API_KEY",
  api: "anthropic-messages",
  headers: { "X-Corp-Signature": "$CORP_SIG" },   // 网关签名
  streamSimple(model, context, options) {
    // 直接转发给内置 streamSimple，无需自己解析 SSE
    return streamSimpleAnthropic(model, context, options);
  },
  models: [ /* ... */ ],
});
```

> ⚠️ **`stream` 和 `streamSimple` 不能互换**：每个 `api/*.ts` 模块同时导出两个函数，签名不同——`stream` 接收 `AnthropicOptions`/`OpenAIResponsesOptions` 等**协议专属选项**，`streamSimple` 接收统一的 `SimpleStreamOptions`（由 pi-ai 框架填充 `apiKey`/`signal`/`reasoning` 等通用字段）。`ProviderConfig.streamSimple` 槽位的类型签名是 `(model, context, options?: SimpleStreamOptions)`，所以槽位里**必须**调用 `streamSimple`。两者返回类型相同（都是 `AssistantMessageEventStream`），传 `stream` 也能跑，但 options 形状不匹配，属于命名误导。
>
> 💡 **import 路径选择**：
> - **新代码**（推荐）：`from "@earendil-works/pi-ai/api/{anthropic-messages|openai-responses|openai-completions}"`（按需导入 `stream` 或 `streamSimple`）
> - **旧代码**：`streamSimpleAnthropic` / `streamSimpleOpenAIResponses` 现从 `@earendil-works/pi-ai/compat` 导入（deprecated alias）
> - **根入口**（`@earendil-works/pi-ai`）：不导出这些 stream 函数

### 模式 3：OAuth 登录

支持通过 `/login <provider-name>` 命令进行 OAuth 认证。实现 `oauth` 配置块即可：

```ts
pi.registerProvider("corporate-ai", {
  baseUrl: "https://ai.corp.com",
  api: "openai-responses",
  models: [ /* ... */ ],
  oauth: {
    name: "Corporate AI",
    login: async (callbacks) => {
      callbacks.onAuth({ url: "https://corp.com/oauth/authorize?..." });
      const code = await callbacks.onPrompt({
        message: "Paste the authorization code:"
      });
      // 用 code 换取 token
      const { access_token, refresh_token, expires_in } = await exchangeToken(code);
      return {
        refresh: refresh_token,
        access: access_token,
        expires: Date.now() + expires_in * 1000 - 5 * 60 * 1000,
      };
    },
    refreshToken: async (credentials) => {
      // 用 credentials.refresh 换取新 token
      return newCredentials;
    },
    getApiKey: (credentials) => credentials.access,
  },
});
```

### 模式 4：覆盖已有 Provider 的 API 地址

不提供 `models`，仅提供 `baseUrl`，可覆盖已有 provider 的 API 端点而不替换模型列表：

```ts
// 将 anthropic provider 的请求代理到自定义网关
pi.registerProvider("anthropic", {
  baseUrl: "https://my-gateway.example.com",
});
```

### 模式 5：注册完整 pi-ai Provider 对象

当 ProviderConfig 静态配置不够用时（需要动态模型刷新、复杂认证、模型过滤、自定义流式协议），可以构造完整的 pi-ai `Provider` 对象并注册。这是 "Full provider extensions" 的核心 API：

`CreateProviderOptions` 接口字段（`ai/src/models.ts`）：

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | `string` | Provider ID |
| `name?` | `string` | 显示名称，默认取 `id` |
| `baseUrl?` | `string` | API 端点 |
| `headers?` | `ProviderHeaders` | 自定义请求头 |
| `auth` | `ProviderAuth` | 认证配置（`apiKey` 或 `oauth`） |
| `models` | `readonly Model<TApi>[]` | 静态基线模型列表（空数组则纯动态） |
| `fetchModels?` | `(context) => Promise<readonly Model<TApi>[]>` | 动态拉取模型覆盖层，createProvider 自动事务性恢复并发布 |
| `filterModels?` | `(models, credential) => readonly Model<TApi>[]` | 按凭证过滤暴露的模型 |
| `api` | `ProviderStreams \| Partial<Record<TApi, ProviderStreams>>` | 流式实现（单个 API 或按 model.api 分派） |

```ts
import { createProvider } from "@earendil-works/pi-ai";
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";

const myProvider = createProvider({
  id: "my-dynamic-provider",
  name: "My Dynamic Provider",
  auth: {
    apiKey: {
      name: "API key",
      login: async (interaction) => ({ /* ... */ }),
      check: async (input) => { /* ... */ },
      resolve: async (input) => ({ auth: { apiKey: "..." }, source: "..." }),
    },
    // 或 oauth: { ... }
  },
  // 静态基线模型列表（必填，空数组则纯动态）
  models: [ /* Model<Api> 对象 */ ],
  // 动态拉取模型（可选，createProvider 自动事务性恢复并发布）
  async fetchModels(context) {
    const resp = await fetch("https://api.mycompany.com/v1/models");
    const data = await resp.json();
    return data.models.map((m) => ({ /* Model<Api> */ }));
  },
  // 按凭证过滤暴露的模型（可选）
  filterModels(models, credential) {
    return models.filter((m) => !m.id.includes("legacy"));
  },
  // 流式实现：单个 ProviderStreams 对象，或按 model.api 分派的 map
  api: openAICompletionsApi(),  // 参考 deepseek.ts / anthropic.ts 的真实用法
});

export default function (pi: ExtensionAPI) {
  // 重载 1：直接注册 Provider 对象
  pi.registerProvider(myProvider);
}
```

**ProviderConfig vs Provider 选择决策**：

| 场景 | 选择 |
|------|------|
| 标准 OpenAI 兼容 API + 固定模型列表 | ProviderConfig（模式 1-4） |
| Anthropic 原生 API + 网关签名 | ProviderConfig + streamSimple（模式 2b） |
| 动态模型目录（运行时从远端拉） | Provider（模式 5） |
| 复杂 OAuth 流程 + 模型过滤 | Provider（模式 5） |
| 完全自定义流式协议 | Provider（模式 5）或 ProviderConfig + streamSimple（模式 2） |

## 使用示例（完整扩展）

参考官方示例：

- **自定义 Anthropic Provider**：`packages/coding-agent/examples/extensions/custom-provider-anthropic/` -- 完整的 OAuth 登录 + 自定义流式处理的 Anthropic provider 实现（含 Claude Code stealth mode）。
- **GitLab Duo Provider**：`packages/coding-agent/examples/extensions/custom-provider-gitlab-duo/` -- 通过 GitLab AI Gateway 代理 Anthropic 和 OpenAI 模型，复用 `pi-ai` 内置的 `streamSimpleAnthropic` 和 `streamSimpleOpenAIResponses`（旧导出名，现从 `@earendil-works/pi-ai/compat` 导入；新代码建议从 `@earendil-works/pi-ai/api/*` 导入）。

基本扩展结构：

**async 工厂动态拉取模型列表**：扩展工厂函数可以是 `async`，在工厂里 `fetch` 远端模型列表再 `registerProvider`。这比 `session_start` 事件更好——pi 启动时会等工厂跑完才继续，所以模型在 interactive 启动和 `pi --list-models` 时都已可用。

```ts
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";

export default async function (pi: ExtensionAPI) {
  // 工厂里拉远端模型列表
  const response = await fetch("http://localhost:1234/v1/models");
  const payload = (await response.json()) as {
    data: Array<{ id: string; name?: string; context_window?: number; max_tokens?: number }>;
  };

  pi.registerProvider("local-openai", {
    baseUrl: "http://localhost:1234/v1",
    apiKey: "$LOCAL_OPENAI_API_KEY",
    api: "openai-completions",
    models: payload.data.map((m) => ({
      id: m.id,
      name: m.name ?? m.id,
      reasoning: false,
      input: ["text"],
      cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
      contextWindow: m.context_window ?? 128000,
      maxTokens: m.max_tokens ?? 4096,
    })),
  });
}
```

```ts
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";

export default function (pi: ExtensionAPI) {
  pi.registerProvider("my-provider", {
    baseUrl: "https://api.mycompany.com/v1",
    apiKey: "$MY_API_KEY",
    api: "openai-completions",
    // openai-completions 下 Bearer 由 SDK 自动带（apiKey 配好即有），不需要 authHeader；自定义鉴权用 headers 字段
    name: "My Company AI",
    models: [
      {
        id: "my-model-v2",
        name: "MyModel v2",
        reasoning: true,
        input: ["text"],
        cost: { input: 1, output: 5, cacheRead: 0.1, cacheWrite: 0 },
        contextWindow: 128000,
        maxTokens: 16384,
      },
    ],
  });
}
```

## 关键细节与注意事项

### 1. API Key 插值语法

`apiKey`（以及自定义 `headers` 的值）使用与 `models.json` 相同的配置值语法：
- `"$ENV_VAR"` -- 从环境变量读取（如 `"$OPENAI_API_KEY"`）
- `"${ENV_VAR}"` -- 花括号形式（如 `"${MY_SECRET}"`）
- `"!command"` -- 执行命令获取整个值（如 `"!pass show api/token"`）
- `$$` -- 字面 `$`（当 apiKey 里真的需要 `$` 字符时用，否则会被当成插值前缀吞掉）
- `$!` -- 字面 `!`（当值以 `!` 开头但你不想触发命令执行时用）
- 纯字符串 -- 直接作为 API Key 使用

### 2. 模型替换 vs 追加

提供 `models` 字段会**替换**该 provider 下所有已有模型（包括内置模型）。如果只想追加模型而不影响已有模型，需要手动合并（但通常不建议：应使用不同的 provider 名称）。

### 3. 自定义 API 标识符

当使用自定义 `api` 值时（如 `"custom-api-format"`），`pi-ai` 不会尝试按内置格式处理请求。此时必须提供 `streamSimple` 实现。如果不提供 `streamSimple`，对该 provider 的模型调用将失败（发生在 stream 时：`getApiProvider(model.api)` 返回 undefined，抛 `No API provider registered for api: <custom>`）。

> ⚠️ **`streamSimple` 必须搭配 `api` 字段（硬约束）**：注册时 `validateExtensionProvider` 会检查——只要提供了 `streamSimple` 但没设 `api`，**注册期直接抛错** `Provider <name>: "api" is required when registering streamSimple.`（`provider-composer.ts:410-412`）。哪怕你的 `streamSimple` 完全自包含、不依赖任何内置协议，也必须给一个 `api` 标识符（哪怕是自定义字符串）。

### 4. streamSimple 的错误处理

`streamSimple` 中捕获的异常必须通过 `stream.push({ type: "error", ... })` 报告，不能直接抛出。`done` 和 `error` 事件发出后必须调用 `stream.end()`。

> ⚠️ **`error` 字段必须是 `AssistantMessage`，不是 raw Error 对象**。`AssistantMessageEvent` 的 error 分支定义为 `{ type: "error"; reason: "aborted" | "error"; error: AssistantMessage }`（`packages/ai/src/types.ts`）。正确做法是先把 `stopReason` / `errorMessage` 写到 output 上，再把 output 整个作为 `error` 字段 push 出去——和 `done` 分支的 `message: AssistantMessage` 对称。

```ts
// ❌ 错误：直接 throw —— 异常不会被 pi-agent 捕获，调用方收不到错误信息
streamSimple(model, context, options) {
  const stream = createAssistantMessageEventStream();
  fetch(model.baseUrl, { /* ... */ })
    .then(res => res.json())
    .then(data => { /* ... */ })
    .catch(err => { throw err; });  // ← 错误！
  return stream;
}

// ❌ 错误：把 raw Error 塞进 error 字段 —— 产出结构非法 event
//   stream.push({ type: "error", reason: "error", error });  // error 是 catch 到的 Error 对象

// ✅ 正确：catch 后先填 output.stopReason/errorMessage，再 push output 作为 error 字段
streamSimple(model, context, options) {
  const stream = createAssistantMessageEventStream();
  (async () => {
    const output: AssistantMessage = { /* ...见模式 2 的完整初始化... */ };
    try {
      stream.push({ type: "start", partial: output });
      const response = await fetch(model.baseUrl, { /* ... */ });
      // ... 处理响应，推送内容事件，设置 output.stopReason ...
      stream.push({ type: "done", reason: output.stopReason, message: output });
      stream.end();
    } catch (error) {
      output.stopReason = options?.signal?.aborted ? "aborted" : "error";
      output.errorMessage = error instanceof Error ? error.message : String(error);
      stream.push({ type: "error", reason: output.stopReason, error: output });
      stream.end();
    }
  })();
  return stream;
}
```

### 5. thinkingLevelMap

`thinkingLevelMap` 将 pi 的 thinking level（`"off"` | `"minimal"` | `"low"` | `"medium"` | `"high"` | `"xhigh"` | `"max"`）映射为 provider 特有的值。例如对于某些模型，`"xhigh"` 被映射为 provider 侧的最大值。将某个 level 设为 `null` 表示该 provider 不支持此 thinking 级别。

### 6. Token 成本字段

`cost` 字段中的 `cacheRead` 和 `cacheWrite` 用于 Anthropic 风格的 prompt caching。对于不支持缓存的价格模型的 provider，将这些值设为 `0`。

### 7. getModel 工具函数

`pi-ai` 提供 `getModel(provider, id)` 函数（**已废弃**），可以从内置模型注册表中查找模型定义。替代方案：
- `import { getBuiltinModel } from "@earendil-works/pi-ai/providers/all"` —— 静态目录读取
- 或运行时 API `Models.getModel()`（推荐，走当前会话的 model registry）

在构建自定义 provider 时，可用于复用已有模型配置作为默认值。

### 8. 扩展卸载时的清理

调用 `pi.unregisterProvider(name)` 可以移除已注册的 provider，恢复其覆盖的内置模型。这在扩展热重载时很重要：先移除旧注册，再添加新注册，避免残留。

### 9. compat 常见覆盖

pi-agent 对未知 provider 默认按 OpenAI 原生行为发送参数。非 OpenAI 原生的兼容 API（智谱、DeepSeek、Ollama、vLLM、LiteLLM 等）通常不支持部分参数，需要通过 `compat` 覆盖：

```ts
compat: {
  supportsDeveloperRole: false,   // 不发 developer 角色（改用 system）
  supportsStore: false,           // 不发 store 参数
  maxTokensField: "max_tokens",   // 用 max_tokens（不是 max_completion_tokens）
  supportsReasoningEffort: false, // 不发 reasoning_effort
  thinkingFormat: "deepseek",     // thinking 格式（11 个可选值："openai"|"openrouter"|"deepseek"|"together"|"baseten"|"zai"|"qwen"|"chat-template"|"qwen-chat-template"|"string-thinking"|"ant-ling"）
}
```

以上是最常见的覆盖项。完整字段列表见 pi-ai 源码 `OpenAICompletionsCompat` 接口（约 25 个字段，`types.ts`），`detectCompat` 函数（`api/openai-completions.ts`）负责根据 URL 自动推断。

> **compat 不限于 openai-completions**。各协议有专属 compat 类型：`OpenAICompletionsCompat`、`OpenAIResponsesCompat`、`AnthropicMessagesCompat`、`BedrockCompat` 等，条件类型映射见 `types.ts` 的 `TApi -> Compat`。

### 10. 上下文溢出错误归一化（自定义 provider 上线必读）

当请求超出模型的上下文窗口时，pi 能自动 compact 对话并重试一次。但这个自动恢复**只在 pi 认出这是溢出错误时才触发**——判断依据是最终 assistant message 的 `stopReason === "error"` 且 `errorMessage` 命中 pi 已知的 overflow pattern（见 `packages/ai/src/utils/overflow.ts`）。

如果你的自定义 provider 返回的溢出错误信息 pi 不认识，**自动 compaction 不会触发**，用户会直接看到报错。解决办法：在注册 provider 的同一个扩展里，用 `message_end` handler 把 `errorMessage` 改写成 pi 认识的短语（最稳妥的通用回退是 `context_length_exceeded`）。

```ts
const MY_PROVIDER_OVERFLOW_PATTERN = /your provider's overflow phrase/i;

export default function (pi: ExtensionAPI) {
  pi.registerProvider("my-provider", { /* ... */ });

  pi.on("message_end", (event, ctx) => {
    const message = event.message;
    if (message.role !== "assistant") return;
    if (message.stopReason !== "error") return;
    // 限定只改自己 provider 的错误
    if (
      message.provider !== "my-provider" &&
      ctx.model?.provider !== "my-provider"
    )
      return;

    const errorMessage = message.errorMessage ?? "";
    // 幂等：已经含 context_length_exceeded 就别再改
    if (errorMessage.includes("context_length_exceeded")) return;
    // 只匹配自己 provider 的专属溢出 pattern
    if (!MY_PROVIDER_OVERFLOW_PATTERN.test(errorMessage)) return;

    return {
      message: {
        ...message,
        errorMessage: `context_length_exceeded: ${errorMessage}`,
      },
    };
  });
}
```

`message_end` 在 pi 把 assistant message 记录进 live context（用于 auto-compaction 判断）**之前**运行，所以改写后的 `errorMessage` 就是 pi 检查的值。改写成功后 pi 会：丢掉这条失败的 assistant message → 运行 compaction → 重试请求一次。

**三条 guard 必须严格遵守**：
- **限定 provider**：用 `message.provider` 和 `ctx.model?.provider` 判断，别误改其他 provider 的错误。
- **匹配专属 pattern**：只匹配你自己 provider 的溢出文案，**不要**匹配 pi 的通用 overflow pattern。尤其别改写 rate-limit / throttling 类错误（`rate limit`、`too many requests`），否则会把限流误判成溢出、触发 compaction 而不是 pi 正常的退避重试。
- **幂等跳过**：`errorMessage` 已含 `context_length_exceeded` 就 return，避免重复改写。
