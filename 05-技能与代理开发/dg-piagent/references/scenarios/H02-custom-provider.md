# 场景：接入自定义模型 Provider (H02)

## 这是什么 / 不是什么

**是**：通过 `pi.registerProvider()` 在扩展中注册第三方 Provider，把兼容标准 API 格式（Anthropic Messages / OpenAI Responses / OpenAI Completions / Azure / Bedrock / Codex / Mistral / Google / Vertex 等 10 种）或自定义协议的模型服务接入 pi-agent。注册后模型出现在 `/model` 选择器、provider 出现在 `/login` 列表。

**不是**：
- **不是改 Agent loop**：注册 provider 只换"调用哪个 LLM + 怎么发请求"，不改 agent 的循环、工具调用、上下文管理逻辑。要换 loop 行为去看 [H01](H01-full-control.md) 或 agent-core。
- **不是改默认 model**：注册成功后模型只是"可选"，用户还要 `/model <provider>/<id>` 切换；或通过 `defaultModelPerProvider` 兜底（见 sdk_doc/05）。
- **不是 auth 管理**：provider 的 OAuth/auth status 查询走 `authStorage`，不在本场景。

## 什么时候用 / 不用会怎样

| 触发场景 | 用 H02 | 不用会怎样 |
|---------|--------|-----------|
| 公司搭了 Anthropic/OpenAI 兼容代理网关，要接入 | 模式 1 标准 API | 每次切换模型都要手动改配置文件 |
| 已有 provider 要换 API 端点（如 anthropic 走 corp gateway） | 模式 4 覆盖 baseUrl | 改不了，只能在请求层手改 |
| 服务用非标准 API（自研协议） | 模式 2 streamSimple | 接不进来 |
| 服务要 OAuth 登录（如 GitHub Copilot、corp SSO） | 模式 3 OAuth | 没法用 `/login` 命令 |
| 仅测试 / 不发真请求 | 用 [H03 Faux Provider](H03-faux-provider.md)，**不要**用本场景 | 测试代码会真的发 HTTP |
| 想完全控制 ModelRegistry（如从数据库读模型） | 去 [H01](H01-full-control.md) 自定义 ResourceLoader | 本场景只控 provider 配置 |

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `pi.registerProvider(name, config)` | 注册自定义 Provider；扩展加载阶段排队、运行时立即生效 | [sdk_doc/16](../sdk_doc/16-custom-provider.md) |
| `pi.unregisterProvider(name)` | 移除 provider 并 reload 内置模型（restore overrides） | [sdk_doc/16](../sdk_doc/16-custom-provider.md) |
| `ProviderConfig` | Provider 配置类型（含 baseUrl/apiKey/api/authHeader/headers/models/oauth/streamSimple） | [sdk_doc/16](../sdk_doc/16-custom-provider.md) |
| `ProviderModelConfig` | 模型定义（id/name/api/baseUrl/reasoning/input/cost/contextWindow/maxTokens/compat 等） | [sdk_doc/16](../sdk_doc/16-custom-provider.md) |

> ⚠️ **不要混入 `createAgentSessionServices`**：该工厂函数（agent-session-services）**接受** `modelRuntime` 参数但**不接受**自定义 ResourceLoader——内部强制 `new DefaultResourceLoader(...)`。如果你要自定义 ResourceLoader（如修改内置模型加载），用 [H01](H01-full-control.md) 的 `createAgentSession({ resourceLoader })`。

## ⚠️ `authHeader` 的真相（容易想反）

**字段行为**：`ProviderConfig.authHeader` **默认 `false`**；显式传 `true` 时，provider-composer 会把 `Authorization: Bearer <apiKey>` 注入请求头（源码 `provider-composer.ts` 的 `withConfiguredAuth`，`if (authHeader)` 严格真值检查；`core/extensions/types.ts` 的 `authHeader` 字段注释「If true, adds Authorization: Bearer header with the resolved API key.」）。传 `true` 但 apiKey 解析不到会直接 throw `"authHeader requires a resolved API key"`。

**但鉴权头不是你想象的只有 authHeader 一条路**——各协议有自己的默认行为：

| 协议 | apiKey 有值时的鉴权头 | authHeader 的作用 |
|------|---------------------|------------------|
| `openai-completions` | **`Authorization: Bearer <apiKey>` 必带**——pi-ai 在 `createClient` 里 `new OpenAI({ apiKey, baseURL, defaultHeaders })`（`openai-completions.ts`），OpenAI Node SDK 的 client 构造函数接收 `apiKey` 后请求时无条件以 `Authorization: Bearer <apiKey>` 发送（SDK 标准行为，与 authHeader 无关） | **冗余**。写了也还是同一个头，不写也带 |
| `anthropic-messages` | `x-api-key: <apiKey>`——Anthropic SDK 的 apiKey 参数默认走 x-api-key | 额外加 Bearer 头；**代理网关只认 Bearer（不认 x-api-key）时才有用** |

**结论**：

1. 接 **OpenAI 兼容代理**（`api: "openai-completions"`）：**只要 apiKey 配好，Bearer 头一定在，不需要 authHeader**。「不写 authHeader 网关就 401」是常见误解——401 通常是 apiKey 没解析到（Key 没配 / 配错来源），不是缺 authHeader。
2. 接 **Anthropic 兼容代理**（`api: "anthropic-messages"`）：默认走 x-api-key；只有代理明确要求 Bearer 头时才加 `authHeader: true`。
3. **自定义鉴权**（网关要求非标准格式，如 `Authorization: Token xxx` / 自定义签名头）：用 `headers` 字段直接写，与 authHeader 无关。

## 核心代码

### 模式 1：标准 API 格式的 Provider（最常用）

目标服务实现了标准 Anthropic Messages / OpenAI Responses / OpenAI Completions 等格式时，只需指定 `api`，框架会用内置流式处理器：

```ts
// 模式 1a：OpenAI 兼容代理（智谱 / DeepSeek / Ollama / LiteLLM / vLLM 等都属此类）
// 注意：openai-completions 下 Bearer 由 SDK 自动带（apiKey 配好即有），不需要 authHeader（见上方「authHeader 的真相」）
export default (pi: ExtensionAPI) => {
  pi.registerProvider("my-proxy", {
    baseUrl: "https://api.mycompany.com/v1",
    apiKey: "$MY_COMPANY_API_KEY",        // 支持 $ENV / ${ENV} / !command / 字面值 4 种格式
    api: "openai-completions",
    name: "My Company AI",                // UI 显示名（可选）
    models: [{
      id: "corp-model-v1",
      name: "Corp Model v1",
      reasoning: false,
      input: ["text", "image"],
      cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
      contextWindow: 128000,
      maxTokens: 16384,
      // OpenAI 兼容性覆盖（非 OpenAI 原生的兼容 API 通常需要）
      compat: {
        supportsDeveloperRole: false,     // 不发 developer 角色
        supportsStore: false,             // 不发 store 参数
        maxTokensField: "max_tokens",     // 用 max_tokens 而非 max_completion_tokens
        supportsReasoningEffort: false,   // 不发 reasoning_effort
      },
    }],
  });
};

// 模式 1b：Anthropic 兼容代理（apiKey 注入 x-api-key，无需 authHeader；仅代理要求 Bearer 时才加 authHeader: true）
export default (pi: ExtensionAPI) => {
  pi.registerProvider("my-anthropic-proxy", {
    baseUrl: "https://anthropic-proxy.mycompany.com",
    apiKey: "$MY_ANTHROPIC_KEY",
    api: "anthropic-messages",
    // 不需要 authHeader —— anthropic-messages API 内部用 x-api-key 头
    models: [{
      id: "claude-sonnet-4-20250514",
      name: "Claude 4 Sonnet (Proxy)",
      reasoning: true,
      input: ["text", "image"],
      cost: { input: 3, output: 15, cacheRead: 0.3, cacheWrite: 3.75 },
      contextWindow: 200000,
      maxTokens: 64000,
    }],
  });
};
```

**支持的 `api` 值**（10 种已知 + 自定义字符串）：

| Api 值 | 适用场景 |
|--------|---------|
| `"anthropic-messages"` | Anthropic 原生或兼容代理 |
| `"openai-responses"` | OpenAI Responses API |
| `"openai-completions"` | OpenAI Chat Completions（含智谱、DeepSeek、Ollama、vLLM、LiteLLM 等兼容） |
| `"azure-openai-responses"` | Azure OpenAI Responses |
| `"openai-codex-responses"` | OpenAI Codex |
| `"mistral-conversations"` | Mistral |
| `"bedrock-converse-stream"` | AWS Bedrock |
| `"google-generative-ai"` | Google Gemini |
| `"google-vertex"` | Google Vertex AI |
| `"pi-messages"` | pi 原生消息格式 |
| `(string & {})` 自定义字符串 | 需配合 `streamSimple`（模式 2） |

### 模式 2：自定义流式 API（非标准协议）

服务用自研协议时，必须实现 `streamSimple`，并指定自定义 `api` 字符串：

```ts
import { createAssistantMessageEventStream } from "@earendil-works/pi-ai";

export default (pi: ExtensionAPI) => {
  pi.registerProvider("custom-api", {
    baseUrl: "https://api.custom.com",
    apiKey: "$CUSTOM_API_KEY",
    api: "custom-api-format",          // 任意字符串，框架不会按内置格式处理
    streamSimple(model, context, options) {
      const stream = createAssistantMessageEventStream();
      (async () => {
        try {
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
          // 推送事件：start / text_delta / done / error 等（详见 pi-ai 的 AssistantMessageEvent 类型）
          // error 事件 error 字段必须是 AssistantMessage 类型，不能塞任意 Error 对象
          stream.push({ type: "start", partial: /* ... */ });
          stream.push({ type: "done", reason: "stop", message: /* ... */ });
        } catch (err) {
          stream.push({ type: "error", reason: "error", error: /* AssistantMessage */ });
        }
        stream.end();   // done / error 后必须调
      })();
      return stream;
    },
    models: [/* ... */],
  });
};
```

> 💡 **复用内置 stream**：如果目标 API 是标准 Anthropic / OpenAI 格式但需要预处理请求/响应（如注入网关签名），可复用 `pi-ai` 内置 stream 函数避免从零解析 SSE。新代码从 `@earendil-works/pi-ai/api/{anthropic-messages|openai-responses|openai-completions}` 导入 `stream`。详见 [sdk_doc/16 §模式 2b](../sdk_doc/16-custom-provider.md)。

### 模式 3：OAuth 登录

需要 `/login <provider-name>` 流程时（如 GitHub Copilot、corp SSO），实现 `oauth` 配置块：

```ts
export default (pi: ExtensionAPI) => {
  pi.registerProvider("corp-ai", {
    baseUrl: "https://ai.corp.com",
    api: "openai-responses",
    models: [/* ... */],
    oauth: {
      name: "Corporate AI",               // 必填，/login UI 显示名
      async login(callbacks) {
        // callbacks 完整方法见下方表格
        callbacks.onAuth({ url: "https://corp.com/oauth/authorize" });
        const code = await callbacks.onPrompt({ message: "Paste the authorization code:" });
        const { access_token, refresh_token, expires_in } = await exchangeToken(code);
        return {
          refresh: refresh_token,
          access: access_token,
          expires: Date.now() + expires_in * 1000 - 5 * 60 * 1000,   // 提前 5 分钟过期
        };
      },
      async refreshToken(creds, signal) {
        // 用 creds.refresh 换新 token，返回新 OAuthCredentials
        return newCreds;
      },
      getApiKey(creds) {
        return creds.access;              // 把 credentials 转成 API Key 字符串
      },
      // 可选：根据凭证修改模型配置（如把 baseUrl 改成凭证里的 endpoint）
      // modifyModels(models, creds) { return models; }
      // 注意：oauth 块里有一个 `usesCallbackServer?: boolean` 字段，但已被 @deprecated
      // 标注「Retained for source compatibility; ignored by canonical auth flows.」——勿用
    },
  });
};
```

**`OAuthLoginCallbacks` 完整方法**（源码：pi-ai `compat/extension-oauth-types.ts` 的 `OAuthLoginCallbacks`）：

| 方法 | 用途 |
|------|------|
| `onAuth(info)` | 通知前端打开 `info.url` 让用户登录（可带 `instructions`） |
| `onDeviceCode(info)` | 设备码流程：显示 `userCode` + `verificationUri` |
| `onPrompt(prompt)` | 弹输入框让用户贴 code；返回 `Promise<string>` |
| `onSelect(prompt)` | 多选项选择（如选账号）；返回 `Promise<string \| undefined>` |
| `onProgress?(message)` | 可选：显示进度文本 |
| `onManualCodeInput?()` | 可选：用户手动贴 code（用于 callback server 场景） |
| `signal?` | 可选：`AbortSignal`，用户取消登录时触发 |

> `OAuthCredentials` 形状：`{ refresh: string; access: string; expires: number; [key: string]: unknown }`。`expires` 是绝对时间戳（毫秒）。

### 模式 4：覆盖已有 Provider 的 API 端点

不传 `models`、仅传 `baseUrl`（和可选 `headers`），会**保留**该 provider 已有的所有模型（内置或之前注册的），只改它们的 `baseUrl`：

```ts
export default (pi: ExtensionAPI) => {
  // 把 anthropic 的所有请求代理到自定义网关，模型列表不变
  pi.registerProvider("anthropic", {
    baseUrl: "https://my-gateway.example.com",
    // 不传 models —— 触发 provider-composer 的 "override only" 分支
  });
};
```

**前提**：provider 名（如 `"anthropic"`）必须在 ModelRegistry 中已有模型，覆盖才有效。如果该 name 是首次注册且无 `models`、仅传 `baseUrl`，**不会报错，但会注册一个 0 模型的空 provider**（`/login` 可能看到 provider 名但无模型可选）——因为 `composeModelProvider` 在无 `apiKey`/`oauth` 时仍会生成默认的 API-key 登录方式，不抛 `no authentication method configured`。

## ProviderConfig 完整字段速查

| 字段 | 类型 | 何时必填 | 含义 |
|------|------|---------|------|
| `name` | `string` | 可选 | UI 中显示的 provider 名称 |
| `baseUrl` | `string` | 定义 models 时必填 | API 端点地址 |
| `apiKey` | `string` | 定义 models 时必填（除非有 oauth） | 支持 `$ENV` / `${ENV}` / `!command` / 字面值 |
| `api` | `Api` | provider 级或 model 级至少一处 | API 协议类型（见上方 api 值表） |
| `streamSimple` | `function` | 自定义 api 时必填 | 非标准 API 的流式处理器 |
| `headers` | `Record<string,string>` | 可选 | 自定义请求头（如网关签名 `X-Corp-Sig: $SIG`） |
| `authHeader` | `boolean` | 可选（默认 false） | 为 true 时往请求头加 `Authorization: Bearer <apiKey>`（协议无关）。openai-completions 下 SDK 已自动带 Bearer，**无需此字段**；主要用于「要求 Bearer 的非 openai 协议代理」（如 Anthropic 代理只认 Bearer） |
| `models` | `ProviderModelConfig[]` | 想注册新模型时 | 提供则**替换**该 provider 的所有模型 |
| `oauth` | `{ name; login; refreshToken; getApiKey; ... }`（`ProviderConfig` 内联属性块） | 需要 `/login` 流程时 | OAuth 登录配置 |
| `refreshModels` | `(context) => Promise<ProviderModelConfig[]>` | 可选 | 动态刷新模型列表（如从远程 API 拉取） |

**校验规则**（分散在 `provider-composer.ts` 的 3 个函数里，均在模型注册/重载时触发）：

| # | 规则 | 抛错信息 | 所在函数 |
|---|------|---------|---------|
| 1 | 有 `streamSimple` 时必须有 `api` | `Provider <id>: "api" is required when registering streamSimple.` | `validateExtensionProvider`（入口校验） |
| 2 | 有 `models`（且非空）时必须有 `baseUrl` | `Provider <id>: "baseUrl" is required when defining custom models.` | `applyExtension`（被 `validateExtensionProvider` 调用，扩展层路径）/ `modelFromJson`（models.json 路径） |
| 3 | 有 `models` 且无 `oauth` 时必须有 `apiKey` | `Provider <id>: no authentication method configured.` | `composeModelProvider`（组合 provider 时校验 auth） |
| 4 | 每个 model 必须能在 model 级或 provider 级解析到 `api` | `Provider <id>, model <id>: no "api" specified. Set at provider or model level.` | `applyExtension` / `modelFromJson` |

## `registerProvider` 签名

**扩展层** `pi.registerProvider`（`core/extensions/types.ts`）有两个重载：

```ts
// 重载 1：传 Provider 实例（高级用法，注册原生 streamSimple 的 provider 对象）
registerProvider(provider: Provider): void

// 重载 2：传 provider 名 + 配置（最常用，本场景所有示例都用这种）
registerProvider(name: string, config: ProviderConfig): void
```

> ⚠️ **注意层级区别**：ModelRuntime 实现层（`model-runtime.ts`）不是重载，而是两个**独立方法**：`registerProvider(providerId, config)` 和 `registerNativeProvider(provider)`。扩展层的重载 1 最终调用的就是 `registerNativeProvider`，重载 2 调用的是 `registerProvider`。

本场景所有示例都用重载 2。重载 1 主要用于 coding-agent 内部注册和测试 harness。

## ProviderModelConfig 完整字段

```ts
interface ProviderModelConfig {
  id: string;                  // 模型 ID
  name: string;                // 显示名
  api?: Api;                   // 覆盖 provider 级 api
  baseUrl?: string;            // 覆盖 provider 级 baseUrl
  reasoning: boolean;          // 是否支持 extended thinking
  thinkingLevelMap?: Record<string, string | null>;
                               // 把 pi 的 thinking level 映射为 provider 专用值
  input: ("text" | "image")[]; // 输入类型
  cost: { input: number; output: number; cacheRead: number; cacheWrite: number };
                               // token 价格，不支持 cache 的设 0
  contextWindow: number;       // 最大上下文窗口（tokens）
  maxTokens: number;           // 最大输出 tokens
  headers?: Record<string, string>;       // 模型级请求头
  compat?: {                               // OpenAI 兼容性覆盖（OpenAI 兼容 API 常用）
    supportsDeveloperRole?: boolean;
    supportsStore?: boolean;
    maxTokensField?: "max_tokens" | "max_completion_tokens";
    supportsReasoningEffort?: boolean;
    thinkingFormat?:
      | "openai"            // reasoning_effort（默认）
      | "openrouter"        // reasoning: { effort }
      | "deepseek"          // thinking: { type } + reasoning_effort
      | "together"          // reasoning: { enabled } + reasoning_effort
      | "baseten"           // 可配置 chat_template_args + reasoning_effort
      | "zai"               // thinking: { type }
      | "qwen"              // 顶层 enable_thinking: boolean
      | "chat-template"     // 可配置 chat_template_kwargs
      | "qwen-chat-template"// chat_template_kwargs.enable_thinking + preserve_thinking
      | "string-thinking"   // 顶层 thinking: string
      | "ant-ling";         // reasoning: { effort }（仅映射值非 null 时）
    // 共 11 个值，完整定义见 pi-ai 源码 types.ts:550-561
  };
  // 上表只列了最常用的 5 个 compat 字段；OpenAICompletionsCompat 共 20+ 字段
  //（含 supportsUsageInStreaming / requiresToolResultName / cacheControlFormat /
  //  openRouterRouting / vercelGatewayRouting / sendSessionAffinityHeaders 等），
  // 完整定义见 pi-ai 源码 types.ts:528-586 的 OpenAICompletionsCompat 接口
}
```

## 变体与延伸

- **Faux Provider 测试**：不发真请求的测试用 → [H03](H03-faux-provider.md)
- **完全手动组装**：自定义 ModelRegistry 实例、改 authStorage 等 → [H01](H01-full-control.md)
- **streamSimple 深度模式（含复用内置 stream）** → [sdk_doc/16 §模式 2b](../sdk_doc/16-custom-provider.md)
- **OAuth 完整 callbacks 文档** → [sdk_doc/16 §OAuthConfig](../sdk_doc/16-custom-provider.md)，源码在 `pi-ai/src/compat/extension-oauth-types.ts` 的 `OAuthLoginCallbacks`
- **compat 完整字段表** → [sdk_doc/16 §compat 常见覆盖](../sdk_doc/16-custom-provider.md)，源码在 `pi-ai/src/api/openai-completions.ts` 的 `detectCompat`
- **API Key 插值（含 `!command`）** → [sdk_doc/16 §API Key 插值语法](../sdk_doc/16-custom-provider.md)
- **内置模型定义读取**：`getBuiltinModel(provider, id)` 从 `@earendil-works/pi-ai/providers/all` 导入（**注意**：`getModel` 是 `compat.ts` 的已废弃别名，请用 `getBuiltinModel`）
- **官方示例**：
  - `packages/coding-agent/examples/extensions/custom-provider-anthropic/`（OAuth + 自定义 streamSimple 的 Anthropic provider）
  - `packages/coding-agent/examples/extensions/custom-provider-gitlab-duo/`（GitLab Duo Gateway，复用内置 stream）

## 常见误期待与陷阱

1. **「openai-completions 不写 authHeader 网关就收不到 Bearer」** → **反了**。apiKey 有值时 OpenAI SDK 无条件加 `Authorization: Bearer <apiKey>`，与 authHeader 无关；authHeader 只对「要求 Bearer 的非 openai 协议代理」（如 Anthropic 代理）有意义。
2. **「我注册 provider 名 `anthropic` 不传 models 会清空内置模型」** → **错**。不传 models 时进 "override only" 分支，只改 baseUrl/headers，模型列表保留（`provider-composer.ts` 的 `applyExtension` override-only 分支）。
3. **「streamSimple 失败直接 throw 就行」** → **错**。throw 后异常不会被 agent-loop 捕获，调用方收不到错误。必须 `stream.push({ type: "error", reason: "error", error: AssistantMessage })` 后调 `stream.end()`。`error` 字段类型是 `AssistantMessage`（pi-ai `types.ts` 的 `AssistantMessage` 接口），不是任意 Error 对象。
4. **「扩展加载时 registerProvider 立即生效」** → **部分错**。加载阶段调用会排队进 `pendingProviderRegistrations`，等 `runner.bindCore` 完成后批量 flush（`runner.ts` 的 `bindCore` flush 循环）。运行时（命令处理 / 事件回调中）调用才立即生效。
5. **「OAuth 不需要 `name` 字段」** → **错**。`ProviderConfig.oauth.name` 必填（`core/extensions/types.ts`），是 `/login` UI 显示名。
6. **「自定义 `api` 值可以不传 streamSimple」** → **错**。框架对未知 api 值不内置处理，调用会失败。必须提供 streamSimple（`validateExtensionProvider` 会校验 streamSimple + api 组合）。
7. **「registerProvider 失败会被吞掉」** → **错**。`validateExtensionProvider` 会 throw，扩展加载时被 runner 捕获并报 diagnostics，运行时调用则直接抛给调用方。生产代码应 try/catch。
8. **「unregisterProvider 只是删模型」** → **错**。除了从内部 provider 注册表（`ModelRuntime` 的 `extensionProviders` + `nativeExtensionProviders` 两个 Map）删除，还会调 `recomposeProvider`（extension config 删除后内置 provider 恢复原始定义）+ `refresh()` 重新加载 models.json（`model-runtime.ts` 的 `unregisterProvider`）。
9. **「OAuth 的 `expires` 是相对秒数」** → **错**。是绝对时间戳（毫秒），通常写 `Date.now() + expires_in * 1000 - 提前量`。
10. **「OpenAI 兼容 API 不需要 compat」** → **看情况**。OpenAI 原生不需要；非 OpenAI 原生（智谱、DeepSeek、Ollama 等）几乎都需要至少覆盖 `supportsDeveloperRole` / `maxTokensField` 等字段，否则服务端会因不支持的字段而拒。

## 官方示例参考

- **自定义 Anthropic Provider**：`packages/coding-agent/examples/extensions/custom-provider-anthropic/` — OAuth 登录 + 自定义 streamSimple 的完整实现（含 Claude Code stealth mode）。
- **GitLab Duo Provider**：`packages/coding-agent/examples/extensions/custom-provider-gitlab-duo/` — 通过 GitLab AI Gateway 代理 Anthropic 和 OpenAI 模型，用 `anthropicMessagesApi()` / `openAIResponsesApi()` 工厂函数复用 `pi-ai` 内置 stream。

> 新代码请从 `@earendil-works/pi-ai/api/{anthropic-messages|openai-responses|openai-completions}` 导入 `stream`/`streamSimple`，或从 `@earendil-works/pi-ai/compat` 导入 `anthropicMessagesApi()` / `openAIResponsesApi()` 工厂函数（旧导出名 `streamSimpleAnthropic` / `streamSimpleOpenAIResponses` 等已弃用）。详见 [sdk_doc/16](../sdk_doc/16-custom-provider.md)。
