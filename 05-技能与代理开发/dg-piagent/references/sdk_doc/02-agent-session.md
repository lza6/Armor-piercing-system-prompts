# AgentSession

## 这是什么

`AgentSession` 是 `createAgentSession()` 返回的**会话实例**。它提供对话、中断、模型切换、事件订阅、状态读取等所有核心操作。一个 Session 代表一次完整的对话生命周期。

## 核心方法

### prompt()

```ts
session.prompt(text: string, options?: PromptOptions): Promise<void>
```

发送用户消息，触发 Agent 循环。这是最常用的方法。

```ts
await session.prompt("What is 2 + 2?");
```

`PromptOptions`:
- `images?: ImageContent[]` — 附加图片
- `expandPromptTemplates?: boolean` — 是否展开文件式 prompt 模板（默认 true）
- `streamingBehavior?: "steer" | "followUp"` — **流式时必填**。当 `isStreaming === true` 时调用 `prompt()` 必须指定此字段，决定新消息入队方式（`steer` 中断 / `followUp` 排队）；非流式时可省略
- `source?: InputSource` — 输入来源（默认 "interactive"）
- `preflightResult?: (success: boolean) => void` — RPC 模式内部 hook，一般用户用不到

### subscribe()

```ts
session.subscribe(listener: (event: AgentSessionEvent) => void): () => void
```

订阅会话事件流，用于获取 LLM 响应输出。**返回一个 unsubscribe 函数**，调用它可以取消订阅（长连接/Web 场景断开时务必调用，防止监听器泄漏）。

```ts
session.subscribe((event) => {
  if (event.type === "message_update" && event.assistantMessageEvent.type === "text_delta") {
    process.stdout.write(event.assistantMessageEvent.delta);
  }
  if (event.type === "agent_end") {
    console.log("Agent finished");
  }
});
```

#### subscribe 能收到的事件清单

`AgentSessionEvent` 共 **23 种**（注：源码 union 中 `auto_retry_end` 重复声明两次，实际 distinct 类型 = 13 session 独有 + 10 来自 AgentEvent）。事件分两类：

**A. 来自 AgentEvent（10 种，扩展层 `pi.on` 也派发）**

| 事件 | 触发时机 | 关键字段 |
|------|---------|---------|
| `agent_start` | Agent 主循环开始 | — |
| `agent_end` | Agent 主循环结束（含重试场景） | `messages` / `willRetry` |
| `turn_start` | turn 开始（一次 LLM 调用 + 工具链的边界） | **无字段** |
| `turn_end` | turn 结束 | `message` / `toolResults` |
| `message_start` / `message_update` / `message_end` | 任意角色消息生命周期 | `message`（`message_update` 多 `assistantMessageEvent`） |
| `tool_execution_start` | 工具执行开始 | `toolCallId` / `toolName` / `args` |
| `tool_execution_update` | 工具执行过程 | `toolCallId` / `toolName` / `args` / `partialResult` |
| `tool_execution_end` | 工具执行结束 | `toolCallId` / `toolName` / `result` / `isError` |

**B. session 独有（13 种，扩展层 `pi.on` 收不到）** ⭐

| 事件 | 触发时机 | 关键字段 |
|------|---------|---------|
| `agent_settled` | agent run 完全稳定后触发（所有 retry/compaction/queue 处理完） | — |
| `queue_update` | `steer()` / `followUp()` 入队消息变化 | `steering` / `followUp`（当前队列内容，`readonly string[]`） |
| `compaction_start` | 上下文压缩开始 | `reason`（manual / threshold / overflow） |
| `compaction_end` | 上下文压缩结束 | `reason` / `result?` / `aborted` / `willRetry` / `errorMessage?` |
| `entry_appended` | 会话文件追加条目 | `entry: SessionEntry` |
| `session_info_changed` | 会话名字变更 | `name: string \| undefined` |
| `thinking_level_changed` | 思考等级变化 | `level: ThinkingLevel` |
| `auto_retry_start` | 自动重试开始（错误恢复） | `attempt` / `maxAttempts` / `delayMs` / `errorMessage` |
| `auto_retry_end` | 自动重试结束 | `success` / `attempt` / `finalError?` |
| `summarization_retry_scheduled` | 摘要重试排定 | `attempt` / `maxAttempts` / `delayMs` / `errorMessage` |
| `summarization_retry_attempt_start` | 摘要重试开始（branchSummary / compaction 两变体） | `source` / 可选 `reason` |
| `summarization_retry_finished` | 摘要重试完成 | — |
| `bash_execution_update` | bash 工具执行过程增量输出 | `id?` / `delta` |

> 两层事件互补：A 类多数共有，但扩展层 payload **不一定更丰富**（因事件而异）——
> - `turn_start` 扩展层多 `turnIndex` + `timestamp`（session 层无字段）；`turn_end` 扩展层只多 `turnIndex`（无 `timestamp`）。
> - `agent_end` 扩展层反而**更简单**：只有 `messages`，**没有 session 层的 `willRetry`**。
>
> B 类（session 独有）只有 subscribe 能收；扩展独有事件（`context` / `tool_call` / `tool_result` / `before_agent_start` / `input` / `model_select` / `thinking_level_select`）只有 `pi.on` 能收。

> ⚠️ **subscribe 静默收不到的 7 个事件**：`context` / `tool_call` / `tool_result` / `before_agent_start` / `input` / `model_select` / `thinking_level_select` ——
> 这些事件只在扩展层（`_extensionRunner.emit`）派发、不在 `AgentSessionEvent` union 中，必须写成扩展走 `pi.on`，详见 [04-events.md 坑 4](04-events.md#坑-47-个扩展独有事件-subscribe-静默收不到--最大集成坑)。
>
> ⚠️⚠️ **`thinking_level` 系列与 `model` 系列易混**：两者在 `setThinkingLevel` / `setModel` 中是对称的双层派发模式，但有一个关键差异——
> - **model 系列**：`setModel()` 只派发扩展层 `model_select`，session 层**没有任何对应事件**（subscribe 完全收不到模型切换）。
> - **thinking_level 系列**：`setThinkingLevel()` 同时派发**两层**——session 层 `thinking_level_changed`（subscribe **能收**）+ 扩展层 `thinking_level_select`（subscribe **收不到**）。
>
> 所以监听思考等级变化用 `thinking_level_changed`（subscribe 可用），监听模型切换必须写成扩展走 `pi.on("model_select", ...)`。千万别把 `thinking_level_select` 写进 subscribe——它和 `thinking_level_changed` 只差一个词，却是静默失败陷阱。

> 需要"agent 真正结束"信号时，订阅 **`agent_settled`** 替代 `agent_end`——后者在 retry/compaction/queue 场景下会提前或多次触发，前者确保所有自动行为处理完才 emit，且两层都派发。

> 扩展层 `pi.on` 的完整事件分类与生命周期 → [04-events.md](04-events.md)

### steer()

```ts
session.steer(text: string, images?: ImageContent[]): Promise<void>
```

**入队**一条控制消息到会话队列。不会打断当前流程，在下个 `turn` 开始前被消费。

```ts
session.steer("记得先写单元测试再重构");
```

> 注意：steer() 只是入队，不打断当前流程。如果 agent loop 已结束，steer 的消息不会生效。

### followUp()

```ts
session.followUp(text: string, images?: ImageContent[]): Promise<void>
```

`steer()` 的姊妹方法。入队到 **follow-up 队列**（而非 steering 队列），只在 agent 无更多工具调用/steering 消息时消费。两者都会触发 `queue_update` 事件。

**steer vs followUp 的差异**：

| 维度 | `steer()` | `followUp()` |
|------|----------|-------------|
| 队列 | steering | followUp |
| 消费时机 | 下个 turn 开始前 | agent 无更多工具调用/steering 时 |
| 适用场景 | 修正当前对话方向（中断语义） | 排队跟进任务（非中断语义） |
| 排空模式控制 | `steeringMode` / `setSteeringMode` | `followUpMode` / `setFollowUpMode`（对称 API） |

> 排空模式 API：两个队列都支持自定义排空行为，签名对称。默认排空模式详见源码。

### abort()

```ts
session.abort(): Promise<void>
```

立即中止当前 Agent 操作（LLM 调用或工具执行）。**注意：`abort()` 会 `await waitForIdle()`——即 resolve 时 agent 已完全停止**（源码 JSDoc："Abort current operation and wait for agent to become idle"），调用后无需手动等待即可安全操作 state。

```ts
session.abort();
```

### setModel()

```ts
session.setModel(model: Model<any>): Promise<void>
```

运行时切换模型。

```ts
session.setModel(getModel("openai", "gpt-5"));
```

> ⚠️ **`model_select` 是扩展层独有事件，`session.subscribe` 收不到**。
> `setModel()` 会经扩展 runner 派发 `model_select`（含 `model / previousModel / source`），但该事件**不在 `AgentSessionEvent` 中**——subscribe 监听器里写 `event.type === "model_select"` 永远不会命中（静默失败）。需要监听模型切换的话，必须写成扩展走 `pi.on("model_select", ...)`。

### dispose()

```ts
session.dispose(): void
```

释放会话资源（关闭文件描述符、停止监听器等）。**必须调用。**

```ts
try {
  const { session } = await createAgentSession();
  await session.prompt("hello");
} finally {
  session.dispose();
}
```

### bindExtensions()

```ts
await session.bindExtensions(bindings: ExtensionBindings): Promise<void>
```

绑定扩展到 session（异步）。通常由 createAgentSession 内部调用，手动使用 `AgentSessionRuntime` 时需要显式调用。

## 属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `session.state` | `AgentState` | 当前会话状态（来自 `@earendil-works/pi-agent-core`） |
| `session.sessionId` | `string` | 会话唯一 ID |
| `session.sessionFile` | `string \| undefined` | 会话持久化文件路径 |

### AgentState 字段（pi-agent-core/dist/types）

```ts
interface AgentState {
  systemPrompt: string;                  // 每次请求发送的系统提示词
  model: Model<any>;                     // 当前模型（不可为 undefined）
  thinkingLevel: ThinkingLevel;          // 推理深度："off"|"minimal"|"low"|"medium"|"high"|"xhigh"|"max"
  tools: AgentTool<any>[];               // 可用工具数组（赋值会拷贝顶层）
  messages: AgentMessage[];              // 对话历史（赋值会拷贝顶层）
  readonly isStreaming: boolean;         // 是否正在处理 prompt/continuation（持续到 agent_end 监听器结束）。⚠️ 此为 agent-core 层信号，在 subscribe 监听器内读此值恒为 true；判断 agent 是否完全结束用 session.isStreaming（持续到 agent_settled）或订阅 agent_settled
  readonly streamingMessage?: AgentMessage;  // 当前流式响应的局部助手消息
  readonly pendingToolCalls: ReadonlySet<string>;  // 正在执行的工具调用 ID
  readonly errorMessage?: string;        // 最近一次失败/中止的错误信息
}
```

> 注意：`model` 在类型层面是 `Model<any>`（非 undefined），但 **运行时可能为空**——`session.model` getter 的实际签名是 `Model<any> | undefined`。`createAgentSession()` 不会校验模型是否存在；真正调用 `session.prompt()` 时，若 `!this.model` 会 **throw** `formatNoModelSelectedMessage()`。所以**第一次 prompt 前必须确保已设置模型**（通过配置 Provider 或 `session.setModel()`）。

## 生命周期

```
createAgentSession()
  ↓
session.subscribe()       ← 订阅事件流
  ↓
session.prompt("hi")      ← 第一次对话
  ↓  (Agent 循环中...)
  ↓  可能：session.steer()   ← 注入消息
  ↓  可能：session.abort()   ← 中断
  ↓  可能：session.setModel() ← 换模型
  ↓
session.prompt("more")    ← 继续对话
  ↓
session.dispose()         ← 释放（必须）
```

## 关键细节

- `subscribe()` 可以在 `prompt()` 前或后调用
- `steer()` 只是**入队**，效果在下个 turn 开始时才体现
- `state.messages` 在 `agent_end` 事件后才更新完全
- 如果 agent loop 已完成，`steer()` + `prompt()` 才会触发新 turn

## 其他公开方法（按需查源码）

上面只列了最常用的 8 个方法。`AgentSession` 还有以下公开成员，一般场景用不到，需要时直接查 `agent-session.ts`：

| 方法/getter | 一句话说明 |
|------------|------------|
| `cycleModel(direction?)` | 在 `--models` 列表或全部已配置模型中循环切换 |
| `setThinkingLevel(level)` | 设置推理深度（同步） |
| `compact(customInstructions?)` | 手动触发上下文压缩（⚠️ 内部第一行 `await this.abort()`，即调用时会先中止正在运行的 agent 操作） |
| `reload(options?)` | 重载会话（用于扩展 hot-reload、会话替换后刷新上下文） |
| `executeBash(command, ...)` | 在会话 cwd 下执行 bash 命令（默认入消息历史，`excludeFromContext: true` 时排除出 LLM 上下文） |
| `getSessionStats()` | 返回 `SessionStats`（消息/token/cost 统计） |
| `get modelRuntime()` | 访问 `ModelRuntime`（替代 `modelRegistry`，含认证 + 模型发现） |
| `get model` / `thinkingLevel` | state 的快捷 getter（注意 `model` 运行时可能为 `undefined`） |
| `get isStreaming` | 返回 `_isAgentRunActive`（非 `state.isStreaming`；持续到 `agent_settled` 而非 `agent_end`） |
| `sendCustomMessage(message, options?)` | 发送自定义类型消息（扩展场景常用） |
| `sendUserMessage(content, options?)` | 发送用户消息（扩展中常用的 prompt 替代 API） |
| `clearQueue()` | 清空 steer/followUp 队列 |
| `waitForIdle()` | 等待 agent 空闲（retry/compaction/queue 全部处理完） |
| `getContextUsage()` | 返回上下文使用情况（token 占用等） |
| `navigateTree(...)` | 会话树导航（分叉/回溯） |

> 上表只列高频成员。完整公开成员清单（含 `setSessionName` / `getActiveToolNames` / `abortBash` / `abortCompaction` / `abortRetry` / `recordBashResult` / `exportToHtml` / `exportToJsonl` / `getLastAssistantText` / `hasExtensionHandlers` / `cycleThinkingLevel` / `getAvailableThinkingLevels` / `supportsThinking` / `setScopedModels` 等约 60+ 成员）见 `agent-session.ts` class `AgentSession`。

> 切换会话 / 分叉 / 重启等运行时操作不在 `AgentSession` 本身，而在 `AgentSessionRuntime` —— 详见 [03-agent-session-runtime.md](03-agent-session-runtime.md)。
