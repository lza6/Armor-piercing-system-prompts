# 扩展层事件系统（`pi.on`）

## 这是什么

pi-agent 事件系统分**两层**，本文聚焦**扩展层** `pi.on`：

- **扩展层 `pi.on`** ← **本文档主体**。写在 extension factory 内，通过 `extensionFactories` 注入，
  能收到全部事件（含 `context` / `tool_call` / `tool_result` / `before_agent_start` / `input` / `model_select` / `thinking_level_select` 等**扩展独有事件**），
  handler 可拦截/修改 Agent 行为（返回值能改消息/工具参数）
- **外部层 `session.subscribe`** ← 详见 [02-agent-session.md §subscribe](02-agent-session.md#subscribe)。
  写在 server / CLI 等外部宿主内，订阅式只读，收不到上述 7 个扩展独有事件，但额外能收
  `queue_update` / `compaction_*` 等 **session 状态事件**（扩展层也收不到；`session_info_changed` 是例外，两层都派发）

> 两层多数事件共有（`agent_start` / `turn_*` / `message_*` / `tool_execution_*`），具体派发差异
> 见本文末尾[坑 4](#坑-47-个扩展独有事件-subscribe-静默收不到--最大集成坑)。

---

## 事件分类（扩展层视角）

pi-agent 通过**事件驱动**架构运行。所有 Agent 生命周期、消息流、工具执行、会话管理都通过事件传递。扩展通过 `pi.on(eventName, handler)` 订阅事件来介入 Agent 行为。

## 事件分类

### 会话生命周期

| 事件名 | 触发时机 | 可修改 | 典型用途 |
|--------|---------|--------|---------|
| `session_start`* | 会话启动 | 否 | 初始化、打印欢迎信息 |
| `session_shutdown` | 关闭会话 | 否 | 清理资源、保存状态 |
| `session_before_switch` | 切换会话前 | 是 | 确认切换、保存当前状态 |
| `session_before_fork` | 分叉会话前 | 是 | 定制 fork 行为 |
| `session_before_compact` | 压缩前 | 是 | 自定义压缩策略 |
| `session_compact` | 压缩后 | 否 | 压缩完成的后续处理 |
| `session_before_tree` | 导航会话树前 | 是 | 定制树导航 |
| `session_tree` | 会话树操作后 | 否 | 追踪导航 |

### Agent 生命周期

| 事件名 | 触发时机 | 可修改 | 典型用途 |
|--------|---------|--------|---------|
| `before_agent_start` | 用户提交 prompt 后，agent loop 前 | 是 | 检查/修改系统提示词、拦截请求 |
| `agent_start` | agent loop 开始 | 否 | 记录开始时间、显示状态 |
| `agent_end` | agent loop 结束 | 否 | 记录结果、触发后处理 |
| `agent_settled` ⭐ v0.83.0 | agent run 完全稳定后（所有 retry/compaction/queue 处理完才触发） | 否 | **坑 1 官方解药**：可靠结束信号、写库收尾、SSE done |
| `turn_start` | 每个 turn 开始 | 否 | 注入上下文、预加载数据 |
| `turn_end` | 每个 turn 结束 | 否 | 记录、分析、自动总结 |

### 消息生命周期

| 事件名 | 触发时机 | 可修改 | 典型用途 |
|--------|---------|--------|---------|
| `message_start` | 消息开始（user/assistant/toolResult） | 否 | 消息到来通知 |
| `message_update` | assistant 消息流式更新（逐 token） | 否 | 流式输出、实时渲染 |
| `message_end` | 消息结束 | 是 | 修改最终消息、注入内容 |

### 工具生命周期

| 事件名 | 触发时机 | 可修改 | 典型用途 |
|--------|---------|--------|---------|
| `tool_call` | 工具执行前 | 是 | 拦截/阻止/修改参数 |
| `tool_execution_start` | 工具开始执行 | 否 | 显示执行状态 |
| `tool_execution_update` | 工具执行中的部分输出 | 否 | 流式渲染 |
| `tool_execution_end` | 工具执行结束 | 否 | 记录结果 |
| `tool_result` | 工具结果返回后 | 是 | 修改工具输出、自定义渲染 |

### 模型事件

| 事件名 | 触发时机 | 可修改 | 典型用途 |
|--------|---------|--------|---------|
| `model_select` | 模型切换时 | 否 | 更新 UI、记录日志 |
| `thinking_level_select` | 思考等级变化时 | 否 | 记录、通知 |

### Provider 事件

| 事件名 | 触发时机 | 可修改 | 典型用途 |
|--------|---------|--------|---------|
| `context` | LLM 调用前 | 是 | 修改发给 LLM 的消息列表 |
| `before_provider_request` | Provider 请求发送前 | 是 | 修改请求体 |
| `before_provider_headers` ⭐ v0.83.0 | headers 组装完成、HTTP 请求发出前 | 是（in-place mutate） | 注入鉴权 / tracing / session header，`null` 删除该 header |
| `after_provider_response` | Provider 响应返回后 | 否 | 记录响应状态 |

#### Provider 事件可获取数据范围

| 事件 | payload | 能拿到 | 拿不到 |
|------|---------|--------|--------|
| `context` | `{ messages: AgentMessage[] }` | 即将发给 LLM 的消息列表（含历史 + 当前 turn） | **system_prompt**（要去 `before_agent_start` 事件的 `event.systemPrompt`） |
| `before_provider_request` | `{ payload: 原始请求体 }` | model / messages / tools / temperature 等发往 Provider 的完整请求 | — |
| `before_provider_headers` | `{ headers: ProviderHeaders }` | `Record<string, string \| null>`（mutate；`null` 删除该 header，返回值被忽略） | — |
| `after_provider_response` | `{ status, headers }` | HTTP 状态码、响应头 | **响应体、token usage**（需从 `message_end` 的 AssistantMessage.usage 获取） |

> ⚠️ **`before_provider_headers` 与 `before_provider_request` 语义不同**：前者要求 in-place mutate `event.headers`（返回值被忽略），后者要求 return 新 payload。两者顺序：`before_provider_request` 改请求体 → headers 组装 → `before_provider_headers` 改 headers → HTTP 发出。证据：`ext-types.ts:681-689`、`sdk.ts:318-328` 的 `streamSimple` `transformHeaders` 回调。

#### 事件触发频次

| 频次 | 事件 |
|------|------|
| 每 prompt 一次 | `agent_settled`（v0.83.0，确保所有 retry/compaction/queue 处理完才触发）、`before_agent_start`（在 `prompt()` 内、retry 循环**之前** emit，整个 prompt 只触发一次，与 trace 数无关） |
| 每 trace 一次 | `agent_start`、`agent_end`（retry 场景下一次 prompt 可能产生多个 trace，每个 trace 各一对 start/end） |
| 每 turn 一次 | `turn_start`、`turn_end` |
| 每轮 LLM 调用一次 | `context`、`before_provider_request`、`after_provider_response`、`message_end` (assistant) |
| 每次工具调用一次 | `tool_call`、`tool_execution_*`、`tool_result`、`message_end` (toolResult) |

#### handler 执行顺序

同一事件多个 handler **按扩展加载顺序串行执行**（extensionFactories 数组顺序）。对顺序敏感的逻辑（如「先捕获原始数据 → 后做遮蔽」），必须显式控制 `extensionFactories` 顺序。

### 输入事件

| 事件名 | 触发时机 | 可修改 | 典型用途 |
|--------|---------|--------|---------|
| `input` | 收到用户输入 | 是 | 变换/拦截用户输入（扩展独有，subscribe 收不到） |
| `user_bash` | 用户执行 `!command` | 是 | 自定义 bash 执行 |

### 资源事件

| 事件名 | 触发时机 | 可修改 | 典型用途 |
|--------|---------|--------|---------|
| `resources_discover` | session_start 后 | 是 | 动态提供资源路径 |

> \* `session_start` 可通过 `pi.on("session_start", handler)` 订阅（`ext-types.ts:1200`），payload 含 `reason: "startup"|"reload"|"new"|"resume"|"fork"` + `previousSessionFile?` 字段（`ext-types.ts:562-568`）。它由 `createAgentSession` 的 `sessionStartEvent` 参数触发，扩展运行时通过通用 `emit()` 派发。

## 事件数据结构

> **渐进式披露**：本节按使用频率分级覆盖 payload 结构。
>
> - **本文详述**：消息生命周期（`message_*` + 流式 12 子事件）、工具调用（`tool_call` / `tool_result`）、Provider 事件、Agent 生命周期（5 种）、工具执行（3 种）—— 都是**一般 Web/Server 集成必备**
> - **按需查源码**（未列出 payload）：模型事件、输入事件（`input` / `user_bash`）、资源事件（`resources_discover`）、8 种 `session_*` 事件（多数 CLI 专属）—— 见 `packages/coding-agent/src/core/extensions/types.ts`
>
> **模型事件 payload 速查**（两者都是 [扩展独有](#坑-47-个扩展独有事件-subscribe-静默收不到--最大集成坑)、subscribe 收不到）：
> - `model_select`：`{ model: Model, previousModel: Model | undefined, source: "set" | "cycle" | "restore" }`（`ext-types.ts:794-799`；派发 `_emitModelSelect` `agent-session.ts:1558-1570`，模型未变时不触发）
> - `thinking_level_select`：`{ level: ThinkingLevel, previousLevel: ThinkingLevel }`（`ext-types.ts:802-806`；派发 `agent-session.ts:1691-1695`）。注意 session 层对应的是 `thinking_level_changed`（subscribe 能收），别与 select 混淆。
>
> 源码路径简写：`ext-types.ts` = `packages/coding-agent/src/core/extensions/types.ts`，`agent-types.ts` = `packages/agent/src/types.ts`。

### message_start / message_end 事件

```ts
MessageStartEvent  { type: "message_start", message: AgentMessage }
MessageUpdateEvent { type: "message_update", message: AssistantMessage, assistantMessageEvent: ... }
MessageEndEvent    { type: "message_end", message: AgentMessage }
```

`event.message` 是联合类型（UserMessage | AssistantMessage | ToolResultMessage）。**AssistantMessage 是获取 LLM 调用详情的关键载体**，关键字段：

| 字段 | 类型 | 用途 |
|------|------|------|
| `content` | `(TextContent \| ThinkingContent \| ToolCall)[]` | LLM 输出内容块（文本 / 思考 / 工具调用） |
| `usage` | `Usage` | **token 统计**（input / output / cacheRead / cacheWrite / totalTokens / cost）— 想拿 token 必读这里 |
| `stopReason` | `"pending" \| "stop" \| "length" \| "toolUse" \| "error" \| "aborted"` | finish_reason（`"pending"` 为流式中途态，终态为其余五值） |
| `errorMessage?` | `string` | 错误信息（stopReason=error 时） |
| `provider`, `model` | `string` | 实际响应的 provider / model |
| `responseId?` | `string` | Provider 返回的响应 ID |
| `timestamp` | `number` | Unix ms |

> **AssistantMessage 类型参考**：完整定义（含 usage / stopReason 等原生字段）见 [18-compaction.md](18-compaction.md) 的 usage 使用示例，或直接查 SDK 源码 `@earendil-works/pi-ai` 类型定义。

**获取 token usage 的正确做法**：从 `message_end` 事件的 `event.message.usage` 直接读。**不要**因为 `after_provider_response` 事件拿不到 usage 而去包装 Provider——`createAgentSession` 内部硬编码 streamFn，外部无法注入。

### assistantMessageEvent 子事件类型（流式增量协议）

`message_update` 事件的 `assistantMessageEvent` 字段是一个**流式增量协议**，共 12 个子事件类型（源码：`packages/ai/src/types.ts` 的 `AssistantMessageEvent`），按固定时序在单个 assistant 响应内触发。

> 注意：`message_update` **只对 assistant 消息触发**（源码注释明确：「Only emitted for assistant messages during streaming」）。user 消息和 toolResult 消息只有 `message_start` + `message_end`，没有中间增量。

**流控制**（每个响应必发 1 次 `start` + 1 次 `done`/`error`）：

| 子事件 | 携带 | 语义 |
|--------|------|------|
| `start` | `partial: AssistantMessage` | 流开始，携带初始快照 |
| `done` | `reason: "stop" \| "length" \| "toolUse"`, `message: AssistantMessage` | 流正常终止，携带最终 message |
| `error` | `reason: "aborted" \| "error"`, `error: AssistantMessage` | 流异常终止 |

**text 通道**（模型不输出文本时整组省略）：

| 子事件 | 携带 |
|--------|------|
| `text_start` | `contentIndex` |
| `text_delta` | `contentIndex`, `delta: string`（token 级，通常 1-3 字符/次） |
| `text_end` | `contentIndex`, `content: string`（组装好的完整字符串） |

**thinking 通道**（模型不思考时整组省略）：`thinking_start` / `thinking_delta` / `thinking_end`，字段结构与 text 通道相同。

**toolcall 通道**（每个工具调用触发一组）：

| 子事件 | 携带 |
|--------|------|
| `toolcall_start` | `contentIndex` |
| `toolcall_delta` | `contentIndex`, `delta: string` |
| `toolcall_end` | `contentIndex`, `toolCall: ToolCall`（组装好的完整工具调用） |

#### 关键性质

1. **三通道相互独立**：thinking、text、toolcall 可任意组合出现。**不要假设「模型在 thinking 里说了 = text 也会有」**——把 `thinking_delta` 直接转发为前端 status 是常见错误根源（thinking 内容用户不可见，且是 token 碎片）。

2. **`contentIndex` 区分同类型多段**：单次响应内可以有多段 text（不同 index）或多个 toolcall。按通道缓冲时需按 `contentIndex` 分桶，否则会跨段拼接出乱码。

3. **`*_end` 子事件已含完整内容**：`text_end.content` / `thinking_end.content` / `toolcall_end.toolCall` 是 SDK 已组装好的完整值，**不需要自己累积 delta**。消费侧优先用 `*_end.content`，`delta` 仅在需要"逐字符打字机效果"时使用。

4. **`done` / `error` 是流终止信号**：此后同响应不会再发任何 `message_update`。如需「响应彻底结束后做收尾」（如持久化、发 SSE done），用 `done`/`error` 或外层的 `message_end` 都可，但**不要**用某个 `text_end` 当作响应结束。

#### 推荐消费模式

```ts
// 按通道分桶，到 *_end 一次性取完整内容（无需自己累积 delta）
const textBuffers = new Map<number, string>()

pi.on("message_update", (e) => {
  const ae = e.assistantMessageEvent

  if (ae.type === "text_delta") {
    // 仅当需要打字机效果时累积；否则可忽略，等 text_end 拿完整 content
    textBuffers.set(ae.contentIndex, (textBuffers.get(ae.contentIndex) ?? "") + ae.delta)
  } else if (ae.type === "text_end") {
    const full = ae.content  // 已是完整字符串
    handleText(full)
    textBuffers.delete(ae.contentIndex)
  }
  // thinking_delta / toolcall_delta 同理按通道处理
})
```

### tool_call 事件（按工具名区分）

```ts
// 每个内置工具有自己的事件类型（v0.80.4+ 所有 tool_call / tool_result 事件都带 toolCallId）
BashToolCallEvent    { type: "tool_call", toolCallId: string, toolName: "bash", input: BashToolInput }
ReadToolCallEvent    { type: "tool_call", toolCallId: string, toolName: "read", input: ReadToolInput }
EditToolCallEvent    { type: "tool_call", toolCallId: string, toolName: "edit", input: EditToolInput }
WriteToolCallEvent   { type: "tool_call", toolCallId: string, toolName: "write", input: WriteToolInput }
GrepToolCallEvent    { type: "tool_call", toolCallId: string, toolName: "grep", input: GrepToolInput }
FindToolCallEvent    { type: "tool_call", toolCallId: string, toolName: "find", input: FindToolInput }
LsToolCallEvent      { type: "tool_call", toolCallId: string, toolName: "ls", input: LsToolInput }
CustomToolCallEvent  { type: "tool_call", toolCallId: string, toolName: string, input: Record<string, unknown> }
```

> **`toolCallId` 字段**（`ext-types.ts:853-856`）：v0.80.4+ 所有 tool_call / tool_result 事件都带这个字段，用于关联同一次工具调用的 call / result。多工具并发场景下务必用 `toolCallId` 关联，不要靠 toolName + 时序猜。

类型守卫辅助函数：`isBashToolResult(e)`, `isReadToolResult(e)`, `isEditToolResult(e)` 等。

### tool_result 事件（同样按工具名区分）

`BashToolResultEvent`, `ReadToolResultEvent`, `EditToolResultEvent`, `WriteToolResultEvent`, `GrepToolResultEvent`, `FindToolResultEvent`, `LsToolResultEvent`, `CustomToolResultEvent`

### Agent 生命周期事件

#### `before_agent_start` ⭐ 扩展独有（改系统提示词的权威入口）

- **源码**：ext-types.ts:699-709
- **payload**：

| 字段 | 类型 | 含义 |
|------|------|------|
| `prompt` | `string` | 用户原始 prompt（扩展后） |
| `images?` | `ImageContent[]` | 用户附加图片 |
| `systemPrompt` | `string` | **完整组装好的系统提示词字符串**（能直接读/改） |
| `systemPromptOptions` | `BuildSystemPromptOptions` | 结构化构建选项（供扩展理解 Pi 加载了哪些资源） |

- **handler 返回值**：`{ systemPrompt?: string, message?: CustomMessage }` — 返回 `systemPrompt` 替换本轮系统提示词，多个扩展链式覆盖（后覆盖前）

#### `agent_start`

- **源码**：ext-types.ts:712-714 + agent-types.ts:415
- **payload**：`{ type: "agent_start" }` 无额外字段
- **三层差异**：扩展层 / agent-core / session 层 **payload 完全一致**

#### `agent_end`

- **源码**：ext-types.ts:717-720 + agent-session.ts:141-147
- **扩展层 payload**：`{ type: "agent_end", messages: AgentMessage[] }`
- **session 层 payload**：`{ type, messages, willRetry: boolean }` — 多 `willRetry`（是否即将自动重试）
- **三层差异**：⚠️ session 层独享 `willRetry`，扩展层**看不到**该字段（扩展层 emit 先于 session 层 willRetry 注入）

#### `agent_settled` ⭐ v0.83.0 新增（坑 1 官方解药）

- **源码**：ext-types.ts:722-725（事件类型）+ agent-session.ts:148（union 成员）+ agent-session.ts:587-595（两层都派发的 `_emitAgentSettled`）+ agent-session.ts:1054-1066（在 `_runAgentPrompt` 的 finally 块触发）
- **payload**：`{ type: "agent_settled" }` 无字段
- **派发**：**两层都派发**（扩展层 `pi.on("agent_settled")` 和外部层 `session.subscribe` 都能收）
- **核心价值**：在所有 retry / compaction / queue 处理完才触发。与 `agent_end` 对比：

| 维度 | `agent_end` | `agent_settled`（v0.83.0+） |
|------|------------|------------------------------|
| 触发次数 / prompt | 1+ 次（retry 场景会多次） | **1 次** |
| 触发时机 | agent loop 退出 | _runAgentPrompt finally 块（所有 retry/compaction/queue 处理完） |
| 适合做"真正结束"信号 | ❌（提前触发） | ✅ |
| 派发层 | 两层（扩展层 willRetry 被丢弃） | 两层（一致） |

#### `turn_start`

- **源码**：ext-types.ts:728-732 + agent-types.ts:418
- **扩展层 payload**：`{ type: "turn_start", turnIndex: number, timestamp: number }`
- **session 层 payload**：`{ type: "turn_start" }` 无额外字段
- **三层差异**：⚠️ 扩展层比 session 层多 `turnIndex` / `timestamp`

#### `turn_end`

- **源码**：ext-types.ts:735-740 + agent-types.ts:419
- **扩展层 payload**：`{ type: "turn_end", turnIndex: number, message: AgentMessage, toolResults: ToolResultMessage[] }`
- **session 层 payload**：`{ type, message, toolResults }` 无 `turnIndex`
- **三层差异**：⚠️ 扩展层比 session 层多 `turnIndex`

> **三层 payload 差异速查**（核查 v0.80.2 源码发现的关键信息）：
> `agent_end` / `turn_start` / `turn_end` 在扩展层和 session 层字段不同。集成时如果跨层传递事件，**不能假设 payload 同构**——例如 server 层收到 `agent_end` 时读 `willRetry` 是有效的，但扩展层 handler 读不到。

### 工具执行事件

工具执行三事件两层 payload **完全一致**（不像 `turn_*` / `agent_end` 有字段差异），可放心跨层使用。

#### `tool_execution_start`

- **源码**：ext-types.ts:762-767 + agent-types.ts:426

| 字段 | 类型 | 含义 |
|------|------|------|
| `toolCallId` | `string` | 工具调用唯一 ID |
| `toolName` | `string` | 工具名 |
| `args` | `any` | 工具调用参数 |

#### `tool_execution_update`

- **源码**：ext-types.ts:770-776 + agent-types.ts:427

| 字段 | 类型 | 含义 |
|------|------|------|
| `toolCallId` / `toolName` / `args` | 同上 | — |
| `partialResult` | `any` | 部分/流式输出 |

#### `tool_execution_end`

- **源码**：ext-types.ts:779-785 + agent-types.ts:428

| 字段 | 类型 | 含义 |
|------|------|------|
| `toolCallId` / `toolName` | 同上 | — |
| `result` | `any` | 工具执行结果 |
| `isError` | `boolean` | 是否为错误结果 |

## 可修改的事件返回值

部分事件 handler 可以返回值来修改行为：

| 事件 | 返回值类型 | 作用 |
|------|-----------|------|
| `tool_call` | `{ block?: boolean, reason?: string }` | 阻止工具执行 |
| `tool_result` | `{ content?, details?, isError?, usage? }` | 修改工具结果（`usage?: Usage` v0.81.0 新增，可覆盖工具的 token 统计） |
| `context` | `{ messages?: AgentMessage[] }` | 替换发给 LLM 的消息（messages 被 `structuredClone` 深拷贝，不污染原始数据） |
| `input` | `{ action: "continue"/"transform"/"handled" }` | 变换或拦截输入 |
| `message_end` | `{ message?: AgentMessage }` | 替换最终消息 |
| `user_bash` | `{ operations?, result? }` | 自定义 bash 执行 |
| `before_agent_start` | `{ systemPrompt?, message? }` | 替换系统提示词（链式覆盖） |

## 关键细节

- `pi.on()` 是同步注册、异步回调。同一事件可有多个 handler。**但"异步回调"仍被派发方 `await`**（`runner.ts:805-811` 的扩展双层循环里 `const handlerResult = await handler(event, ctx)`）——handler 里 `await` 慢 I/O（DB 写、二次 LLM 调用、网络请求）会阻塞 agent loop、延长 `prompt()` 的 resolve。落库这类重活必须 **fire-and-forget**（handler 内 `queueMicrotask` / 推队列后立刻返回，重活交独立 worker）。对比之下 `session.subscribe` 的 listener 不被 await（`agent-session.ts:554-558` 的 `_emit` 同步 `for (const l of this._eventListeners) l(event)`，返回的 Promise 被丢弃），可直接 `await` 异步 I/O 而不阻塞 Agent——纯观测类落库优先走 subscribe。
- **错误处理两层不同，务必注意**：扩展层 handler 抛错 / reject 会被 `runner.ts:819-828` 的 `try/catch` 捕获并经 `emitError` 路由（不中断后续 handler），相对安全；但 `session.subscribe` 的 listener **没有任何 try/catch**——同步抛错会沿调用栈上冒、返回 rejected Promise 则变成 **unhandled rejection**（错误静默丢失，仅控制台一条未捕获告警）。所以 subscribe listener 内部需自行 `try/catch` 兜底，否则日志/落库失败会无声消失。
- `tool_call` 的 `event.input` 可以直接修改（mutate in place），后续 handler 看到的是修改后的值
- `session_before_compact` 可以返回自定义的 compact 指令来改变压缩行为
- 事件 handler 的 ctx 提供了 `abort()`, `shutdown()`, `compact()`, `cwd`, `model` 等上下文

## 集成踩坑

事件契约层的不直觉点，每条给陷阱 + 对策。

### 坑 1：`agent_end` 不可靠（不一定触发 / retry 时多次触发）

外层 `await session.prompt()` 可能因此不 resolve，SSE 连接挂住。

**对策**：优先用 `agent_settled`——每 prompt 只触发一次，所有 retry/compaction/queue 处理完才 emit，扩展层与 subscribe 层都派发。旧版本无此事件，用 try/finally 兜底：

```ts
let done = false
const unsub = session.subscribe(e => {
  if (e.type === "agent_end" && !done) { done = true; sendDone() }
})
try {
  await session.prompt(msg)
} finally {
  if (!done) { done = true; sendDone() }  // agent_end 没触发也要结束
  unsub()
}
```

### 坑 2：`message_end` 单轮多次触发（粒度是「每次 LLM 调用」非「每条消息」）

一次提问（调一次工具再回复）会产生多条 assistant `message_end`：预文本 + 空消息 + 最终回答。无脑落库会让消息表膨胀。

**对策**：跳过空 content；同 role 连续消息合并；恢复会话只读最后非空。

```ts
pi.on("message_end", e => {
  const text = extractText(e.message.content)
  if (!text.trim()) return  // 跳过空消息
  // persistMessage 内部必须 fire-and-forget（推队列后台写），
  // 不能是同步阻塞、也不能返回被派发方 await 的 Promise——
  // message_end 一轮多次触发，每次都卡 agent loop 会显著拖慢整轮。
  persistMessage(e.message.role, text)
})
```
> 💡 **更优解**：`message_end` 两层都派发，若只为存消息内容（不需要 `ctx`），改用 `session.subscribe("message_end", async e => { await db.insert(...) })` 更合适——subscribe 不 await listener，写库在后台跑，零阻塞。

### 坑 3：`context` 事件配对要容错

依赖 `context` 抓「发给 LLM 的原始 messages」做 trace 配对时，可能出现 `message_end(assistant)` 无对应 `context` 事件（历史版本首次 LLM call 可能漏触发；新版源码已修，遇异常先实测）。

**对策**：配对失败走容错路径（独立落库但关联字段留空，标 `pair_failed: true`）；或用 `before_provider_request`（payload 含完整请求体）兜底。

### 坑 4：7 个扩展独有事件 subscribe 静默收不到 ⭐ 最大集成坑

`context` / `tool_call` / `tool_result` / `before_agent_start` / `input` / `model_select` / `thinking_level_select` **只在扩展层 `pi.on` 派发**，`session.subscribe` 完全收不到，且**不报错**——handler 被调用但 `event.type` 分支永不命中，整轮对话日志可能全空而控制台无任何提示。

**派发分类**：

| 事件 | pi.on | subscribe |
|---|:---:|:---:|
| `context` / `tool_call` / `tool_result` / `before_agent_start` / `input` / `model_select` / `thinking_level_select` | ✓ | ✗ 扩展独有 |
| `agent_start` / `agent_end` / `agent_settled` / `message_*` / `turn_*` / `tool_execution_*` / `session_info_changed` | ✓ | ✓ 两层都有（payload 字段有差异，见下） |
| `queue_update` / `compaction_*` / `auto_retry_*` / `thinking_level_changed` / `summarization_retry_*` 等 | ✗ | ✓ subscribe 独有 |

**分层原则**：server 层（subscribe）做高层信号转发；需要这 7 个事件的逻辑（日志/trace/拦截/落库）必须写成扩展走 `pi.on`。

```ts
// ❌ subscribe 里写 tool_call 永不命中，且无报错
session.subscribe(e => { if (e.type === "tool_call") log(e.input) })

// ✓ 写成扩展，走 pi.on
export default (pi) => {
  pi.on("tool_call", e => log(e.input))
  pi.on("context", e => logMessages(e.messages))
}
```

> **两层 payload 差异**：`turn_*` 扩展层多 `turnIndex`/`timestamp`；`agent_end` subscribe 层多 `willRetry`（扩展层收不到）。`input` 事件每次循环重建对象，**mutation 无效**，必须 `return transform`。

### 坑 5：扩展层感知不到重试

`auto_retry_start` / `auto_retry_end` 是 subscribe 独有；`agent_end.willRetry` 转发给扩展层时被主动丢弃（扩展层只收到 `{ type, messages }`）。结果：扩展层完全看不到 SDK 正在重试，按 `agent_start`/`agent_end` 建账的逻辑会被一次提问的重试拆成 N 份。

**对策**：优先用 `agent_settled`（所有 retry 处理完才触发一次）替代手动判定；或在 `agent_end` 里自己跑重试正则——`isRetryableAssistantError` 可直接 `import` 自 `@earendil-works/pi-ai`，识别到可重试则不 close 上下文、等下一次 `agent_start` 复用。
