# ExtensionAPI (pi 对象)

## 这是什么

`ExtensionAPI` 是扩展的**核心操作接口**。每个扩展函数接收一个 `pi` 参数，通过它订阅事件、注册工具、发送消息、控制 UI 等。

**什么时候写扩展**：当你需要拦截/修改工具调用、注入系统提示词、接自定义 Provider、变换用户输入、或订阅生命周期做埋点/后处理时，写一个扩展文件并 `export default (pi) => {...}`。普通"响应事件做记录"的需求也能用外部 `session.subscribe()`（见 04-events.md），但只有扩展能拦截/修改事件流、注册工具与命令。

```ts
// 扩展文件 (extensions/my-ext.ts) 或 extensionFactory
export default (pi: ExtensionAPI) => {
  pi.on("agent_start", (event, ctx) => { /* ... */ });
  pi.registerTool({ name: "my_tool", /* ... */ });
};
```

## 完整接口

### 事件订阅

```ts
pi.on(eventName, handler): void
```

支持的所有事件名（详见 [04-events.md](04-events.md)）：

> **派发层提示**：标 ⭐ 的事件是**扩展独有**，只经 `pi.on`（扩展层 `_extensionRunner.emit`）派发，**外部 `session.subscribe()` 收不到**（不在 `AgentSessionEvent` union 中）。下表所有 `session_*` 事件（`session_start` / `session_before_*` / `session_compact` / `session_shutdown` / `session_tree`）也都是扩展独有，**除 `session_info_changed` 外** subscribe 层均收不到。`agent_settled` 的"两层"指扩展层 `pi.on` + session subscribe 层（详见 04-events.md）。

| 事件名 | Handler 签名 | 典型用途 |
|--------|-------------|---------|
| `project_trust` | `(event: ProjectTrustEvent, ctx: ProjectTrustContext) => ProjectTrustEventResult` — **特殊签名**（不是通用 ExtensionHandler） | 决定是否信任项目（安全确认） |
| `resources_discover` | `(event, ctx) => ResourcesDiscoverResult` | 动态提供资源路径 |
| `session_start` | `(event: SessionStartEvent, ctx) => void` | 会话初始化（startup/reload/new/resume/fork，payload reason 字段标明触发原因） |
| `session_before_switch` | `(event, ctx) => SessionBeforeSwitchResult` | 切换会话前确认/保存状态 |
| `session_before_fork` | `(event, ctx) => SessionBeforeForkResult` | 分叉前定制行为 |
| `session_before_compact` | `(event, ctx) => SessionBeforeCompactResult` | 自定义压缩策略 |
| `session_compact` | `(event, ctx) => void` | 压缩完成后的后续处理 |
| `session_shutdown` | `(event, ctx) => void` | 清理资源、保存状态 |
| `session_before_tree` | `(event, ctx) => SessionBeforeTreeResult` | 定制树导航行为 |
| `session_tree` | `(event, ctx) => void` | 追踪导航操作 |
| `context` | `(event, ctx) => { messages? }` — 修改发给 LLM 的消息 | ⭐ 修改消息列表（扩展独有，subscribe 收不到） |
| `before_provider_request` | `(event, ctx) => unknown` — 替换请求体 | 修改/记录发往 Provider 的原始请求（⭐ 扩展独有，subscribe 收不到） |
| `before_provider_headers` | `(event, ctx) => void` — **in-place mutate** `event.headers`（返回值被忽略；`ProviderHeaders` 类型为 `Record<string, string \| null>`，设为 `null` 即删除该 header） | 注入鉴权 / tracing / session header（与 `before_provider_request` 语义不同）（⭐ 扩展独有，subscribe 收不到） |
| `after_provider_response` | `(event, ctx) => void` | 记录 Provider 响应状态（⭐ 扩展独有，subscribe 收不到） |
| `before_agent_start` | `(event, ctx) => BeforeAgentStartEventResult`（即 `{ systemPrompt?, message? }`） | ⭐ 修改系统提示词（扩展独有，subscribe 收不到） |
| `agent_start` | `(event, ctx) => void` | 记录 Agent 开始时间、显示状态 |
| `agent_end` | `(event, ctx) => void` | 记录结果、触发后处理（不可作唯一结束信号，见 04-events.md 坑 1） |
| `agent_settled` | `(event, ctx) => void` — 无 payload | 所有 retry/compaction/queue 处理完才触发，两层都派发（可靠结束信号，见 04-events.md） |
| `session_info_changed` | `(event, ctx) => void` | 会话名变更通知（pi.on 与 subscribe 都派发） |
| `turn_start` | `(event, ctx) => void` | 注入上下文、预加载数据 |
| `turn_end` | `(event, ctx) => void` | 记录/分析本轮、自动总结 |
| `message_start` | `(event, ctx) => void` | 消息到来通知 |
| `message_update` | `(event, ctx) => void` | 流式输出、实时渲染（逐 token） |
| `message_end` | `(event, ctx) => { message? }` — 替换消息 | 修改最终消息、持久化落库 |
| `tool_execution_start` | `(event, ctx) => void` | 显示工具执行状态 |
| `tool_execution_update` | `(event, ctx) => void` | 流式渲染工具部分输出 |
| `tool_execution_end` | `(event, ctx) => void` | 记录工具执行结果 |
| `model_select` | `(event, ctx) => void` | 记录/通知模型切换（⭐ 扩展独有，subscribe 收不到） |
| `thinking_level_select` | `(event, ctx) => void` | 记录思考等级变化（⭐ **扩展独有，subscribe 收不到**；subscribe 层对应事件是 `thinking_level_changed`，payload 仅 `level`，无 `previousLevel`） |
| `tool_call` | `(event, ctx) => { block?, reason? }` — 可拦截 | ⭐ 拦截/阻止/修改工具调用（扩展独有，subscribe 收不到） |
| `tool_result` | `(event, ctx) => { content?, details?, isError?, usage? }` — 可修改 | ⭐ 修改工具输出（扩展独有，subscribe 收不到） |
| `user_bash` | `(event, ctx) => { operations?, result? }` | 自定义 bash 执行（⭐ 扩展独有，subscribe 收不到） |
| `input` | `(event, ctx) => { action: "continue" } \| { action: "transform"; text: string; images?: ImageContent[] } \| { action: "handled" }` | 变换/拦截用户输入（⭐ 扩展独有，subscribe 收不到） |

Handler 函数签名：`(event: EventType, ctx: ExtensionContext) => Promise<ResultType | void> | ResultType | void`（支持 async/await）

### 工具注册

```ts
pi.registerTool(tool: ToolDefinition): void
```

运行时动态注册一个 LLM 可调用的工具。参见 [06-tools.md](06-tools.md) 的 ToolDefinition 详解。

### 命令/快捷键/CLI Flag

```ts
pi.registerCommand(name: string, options: {
  description?: string;
  handler: (args: string, ctx: ExtensionCommandContext) => Promise<void>;
  getArgumentCompletions?: (argumentPrefix: string) => AutocompleteItem[] | null | Promise<AutocompleteItem[] | null>;
}): void

pi.registerShortcut(shortcut: KeyId, options: {
  description?: string;
  handler: (ctx: ExtensionContext) => Promise<void> | void;
}): void

pi.registerFlag(name: string, options: {
  description?: string;
  type: "boolean" | "string";
  default?: boolean | string;
}): void

pi.getFlag(name: string): boolean | string | undefined
```

### 消息渲染

```ts
pi.registerMessageRenderer<T>(customType: string, renderer: MessageRenderer<T>): void
pi.registerMarkdownTransformer(transformer: MarkdownTransformer): void  // 用户/助手 Markdown 渲染前变换
pi.registerEntryRenderer<T>(customType: string, renderer: EntryRenderer<T>): void  // 自定义 entry 渲染器（CustomEntry，不进 LLM 上下文）
```

- `registerMessageRenderer` 为 `CustomMessageEntry` 注册渲染器
- `registerMarkdownTransformer` 在用户/助手 Markdown 渲染到交互式 transcript 前做变换（如链接展开、代码块高亮预处理）
- `registerEntryRenderer` 为 `CustomEntry`（不参与 LLM 上下文的会话条目，如日志/状态行）注册渲染器

### 消息发送

```ts
pi.sendMessage<T>(message: {
  customType: string;
  content: T;
  display?: string;
  details?: unknown;
}, options?: {
  triggerTurn?: boolean;
  deliverAs?: "steer" | "followUp" | "nextTurn";
}): void

pi.sendUserMessage(
  content: string | (TextContent | ImageContent)[],
  options?: { deliverAs?: "steer" | "followUp" }
): void
```

`sendUserMessage` 在 Agent 空闲时立即触发 turn；流式中按 `deliverAs` 入队（steer/followUp）。`sendMessage` 默认不触发 turn（需显式 `triggerTurn: true`）。

### 会话状态

```ts
pi.appendEntry<T>(customType: string, data?: T): void
pi.setSessionName(name: string): void
pi.getSessionName(): string | undefined
pi.setLabel(entryId: string, label: string | undefined): void
```

### Shell 执行

```ts
pi.exec(command: string, args: string[], options?: ExecOptions): Promise<ExecResult>
```

### 工具/Tools 管理

```ts
pi.getActiveTools(): string[]
pi.getAllTools(): ToolInfo[]
pi.setActiveTools(toolNames: string[]): void
pi.getCommands(): SlashCommandInfo[]
```

`ToolInfo` = `Pick<ToolDefinition, "name" | "description" | "parameters" | "promptGuidelines"> & { sourceInfo: SourceInfo }`

### 模型控制

```ts
pi.setModel(model: Model<any>): Promise<boolean>
pi.getThinkingLevel(): ThinkingLevel
pi.setThinkingLevel(level: ThinkingLevel): void
```

### Provider 注册（双重载）

```ts
// 重载 1：注册完整 pi-ai Provider 对象
pi.registerProvider(provider: Provider): void

// 重载 2：注册静态配置
pi.registerProvider(name: string, config: ProviderConfig): void
```

这是接入自定义模型的关键 API。ProviderConfig 主要字段（详见 [16-custom-provider.md](16-custom-provider.md)）：
- `name?` — UI 显示名
- `baseUrl` — API 端点
- `apiKey` — 密钥（支持 `$ENV_VAR` 引用 / `!command` / 字面值）
- `api` — API 格式。已知值：`"anthropic-messages"` / `"openai-responses"` / `"openai-completions"` / `"azure-openai-responses"` / `"openai-codex-responses"` / `"mistral-conversations"` / `"bedrock-converse-stream"` / `"google-generative-ai"` / `"google-vertex"` / `"pi-messages"`，也支持自定义 string
- `streamSimple?` — 自定义流式处理器（用于非标准 API 格式）
- `refreshModels?` — 模型列表刷新回调（支持 `/refresh` 命令动态拉取模型目录）
- `models` — 模型列表（含 contextWindow, maxTokens, cost 等）
- `headers` — 自定义请求头（可选，`Record<string, string>`）
- `authHeader` — 默认 false。为 `true` 时用 API Key 自动添加 `Authorization: Bearer` 头（协议无关）。注意 openai-completions 下 SDK 已自动带 Bearer，此字段可省；主要给「要求 Bearer 的非 openai 协议代理」用
- `oauth` — OAuth 配置（可选，含 `refreshToken(credentials, signal: AbortSignal)` 必须接收 signal 参数）

```ts
pi.registerProvider("my-proxy", {
  baseUrl: "https://proxy.example.com",
  apiKey: "$PROXY_API_KEY",
  api: "anthropic-messages",
  authHeader: true,            // 仅当代理要求 Bearer（而非 x-api-key）时；anthropic-messages 默认走 x-api-key
  models: [{
    id: "claude-sonnet-4-20250514",
    name: "Claude 4 Sonnet (proxy)",
    reasoning: false,
    input: ["text", "image"],
    cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
    contextWindow: 200000,
    maxTokens: 16384
  }]
});
```

> 完整 Provider 对象注册详见 [16-custom-provider.md 模式 5](16-custom-provider.md#模式-5注册完整-pi-ai-provider-对象v0810)。

### 扩展间事件总线（EventBus）

```ts
pi.events: EventBus  // 共享事件总线，用于扩展间通信
```

跨扩展通信的轻量级事件总线。与 `pi.on`（pi-agent 生命周期事件）不同，`pi.events` 用于扩展自定义事件，多个扩展之间可借此协调状态。详见 SDK 源码 `extensions/types.ts`。

### Provider 反注册

```ts
pi.unregisterProvider(name: string): void
```

移除已注册的 provider 及其所有模型，恢复被覆盖的内置模型。若 provider 未注册则无操作（不抛错）。适用于 provider 热更新、测试清理等场景。

```ts
pi.unregisterProvider("my-proxy");
```

## ExtensionContext

每个 event handler 的第二个参数 `ctx` 提供了操作上下文：

| 属性/方法 | 类型 | 说明 |
|-----------|------|------|
| `ctx.ui` | `ExtensionUIContext` | 终端 UI 操作（CLI 专属；Web 二次开发通常用不到，本 skill 未收录详细文档，查 SDK 源码） |
| `ctx.hasUI` | `boolean` | 是否有可用 UI（TUI/RPC 模式为 true，print/json 模式为 false） |
| `ctx.mode` | `"tui" \| "rpc" \| "json" \| "print"` | **运行模式**。扩展可根据模式差异化行为：tui 有交互 UI；rpc/json 给 SDK 调用方；print 一次性输出 |
| `ctx.cwd` | `string` | 当前工作目录 |
| `ctx.isProjectTrusted()` | `() => boolean` | **项目是否被用户信任**。用于决定是否执行潜在危险操作（写文件、跑 bash 等） |
| `ctx.sessionManager` | `ReadonlySessionManager` | 只读会话管理器（14 个只读方法的 Pick 子集，**含 `buildContextEntries()` 不含 `buildSessionContext()`**） |
| `ctx.modelRegistry` | `ModelRegistry` | 模型注册表兼容包装器（内部委托 `ModelRuntime`） |
| `ctx.model` | `Model<any> \| undefined` | 当前模型 |
| `ctx.scopedModels` | `readonly ScopedModel[]` | 会话解析的模型作用域快照（来自 `--models` / `enabledModels` 设置，空 = 所有可用模型） |
| `ctx.thinkingLevel?` | `ThinkingLevel \| undefined` | 当前思考等级（会话运行时提供时） |
| `ctx.isIdle()` | `() => boolean` | Agent 是否空闲 |
| `ctx.signal` | `AbortSignal \| undefined` | 当前 AbortSignal |
| `ctx.abort()` | `() => void` | 中止当前操作 |
| `ctx.shutdown()` | `() => void` | 关闭 pi |
| `ctx.hasPendingMessages()` | `() => boolean` | 是否有待处理消息 |
| `ctx.getContextUsage()` | `() => ContextUsage \| undefined` | 上下文用量 |
| `ctx.compact()` | `(options?: CompactOptions) => void` | 触发压缩。`CompactOptions = { customInstructions?: string; onComplete?: (result: CompactionResult) => void; onError?: (error: Error) => void }` |
| `ctx.getSystemPrompt()` | `() => string` | 获取当前系统提示词 |

## ExtensionCommandContext（命令 handler 专用）

命令 handler 的 ctx 额外提供：

| 方法 | 类型 | 说明 |
|------|------|------|
| `waitForIdle()` | `() => Promise<void>` | 等待 Agent 空闲 |
| `getSystemPromptOptions()` | `() => BuildSystemPromptOptions` | 获取系统提示词构建选项，可用于分析当前 system prompt 配置 |
| `newSession(options?)` | `(options?: { parentSession?, setup?, withSession? }) => Promise<{ cancelled: boolean }>` | 创建新会话 |
| `fork(entryId, options?)` | `(entryId: string, options?: { position?, withSession? }) => Promise<{ cancelled: boolean }>` | 从指定条目分叉 |
| `navigateTree(targetId, options?)` | `(targetId: string, options?: { summarize?, customInstructions?, replaceInstructions?, label? }) => Promise<{ cancelled: boolean }>` | 导航会话树 |
| `switchSession(sessionPath, options?)` | `(sessionPath: string, options?: { withSession? }) => Promise<{ cancelled: boolean }>` | 切换到其他会话文件 |
| `reload()` | `() => Promise<void>` | 重载扩展/skills/prompts/themes |

## ReplacedSessionContext（withSession 回调参数）

`newSession()` / `fork()` / `switchSession()` 的 `withSession` 回调接收此上下文。它继承 `ExtensionCommandContext` 全部方法，并额外提供绑定到新会话的消息发送能力：

| 方法 | 类型 | 说明 |
|------|------|------|
| `sendMessage<T>()` | `(message, options?) => Promise<void>` | 向新会话发送自定义消息（支持 `triggerTurn` / `deliverAs`） |
| `sendUserMessage()` | `(content, options?) => Promise<void>` | 向新会话发送用户消息（支持 `deliverAs`） |
