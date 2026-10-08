# 场景：实时打印 Agent 工作进度（SSE/HTTP 集成）(E11)

## 什么时候用 / 不用会怎样

**适合**：
- Web 聊天前端（Express + SSE / WebSocket 服务器端推送）
- CLI 长任务进度条（多轮工具调用、不想干等到最终答复）
- 第三方监控仪表盘（实时观察 Agent 状态）

**不用会怎样**：多轮工具调用时前端干等无任何信号（10-30s），出问题也无法判断是卡住还是慢。

**不适合**：
- 一次性 RPC 调用（如短信验证码、单轮问答）—— 直接 `await session.prompt()` 等 Promise 即可
- 同步 REST API（`request → response` 严格模式）—— SSE 是单向流式推送，与同步语义冲突
- 需要拦截/变换工具调用的场景 → 走 [E01-tool-intercept.md](E01-tool-intercept.md)（扩展层 `pi.on("tool_call")`）

## 涉及 SDK

### API（唯一的集成入口）

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `session.subscribe(listener)` | 订阅 Agent 事件流，**SSE 集成的唯一入口** | [sdk_doc/02-agent-session.md §subscribe](../sdk_doc/02-agent-session.md) |
| `session.prompt(text, options)` | 触发 Agent 循环（在 try 内调用，finally 兜底发完成信号） | [sdk_doc/02-agent-session.md §prompt](../sdk_doc/02-agent-session.md) |
| unsubscribe 函数（`subscribe` 返回值）| 长连接断开时**必须调用**，否则监听器泄漏 | 同上 |

### 事件（subscribe 监听这些 type）

| 事件 type | 何时触发 | E11 用途 | 详细文档 |
|----------|---------|---------|---------|
| `turn_start` / `turn_end` | 一轮 LLM 响应边界（含工具链执行） | turn 边界感知 / 需要时读取本轮完整 message | [sdk_doc/04-events.md](../sdk_doc/04-events.md) |
| `message_update` + `assistantMessageEvent` | 文本/思考/工具调用的流式增量 | text_delta 直接转发 | 同上 |
| `tool_execution_start` | 工具开始执行（携带 toolName） | status 进度信号 | 同上 |
| `agent_settled`（B 类，推荐）/ `agent_end`（备选，需带 `!willRetry`）| Agent 主循环真正结束 / agent 单轮结束 | 完成信号（与 prompt finally 双保险，见「完成信号机制」节）| [sdk_doc/04-events.md 坑 1](../sdk_doc/04-events.md) |

> ⚠️ **subscribe 静默收不到的 6 个事件**：`context` / `tool_call` / `tool_result` / `before_agent_start` / `input` / `model_select` —— 这些只在扩展层 `pi.on` 派发，**subscribe 监听这些 type 永远不会命中，且不报错**。详见 [sdk_doc/04-events.md 坑 4](../sdk_doc/04-events.md)。

> **本场景是外部宿主集成视角**（subscribe 转发到外部客户端），与 E03/E06/E08 的「扩展内部主动推送」不同。需要拦截/变换工具调用 → 走扩展层（[E01](E01-tool-intercept.md) / [E06](E06-streaming-transform.md)）。

## 默认行为

- `session.subscribe()` 不需要任何配置，每个 session 实例自带事件流
- `session.prompt(text)` **不传 `tools`** 时启用全部内置工具（bash / read / edit / write / grep / find / ls 等），与 A04 / D01 / D04 / D05 / E01 / E06 一致
- SSE 集成时通常**不需要**传 `tools`，让 Agent 用默认工具集；如需禁用部分工具，在 session 创建时配 `tools` 白名单（跨链 A04）
- `prompt()` 在 streaming 模式下（`isStreaming === true`）必须传 `streamingBehavior: "steer" | "followUp"`，决定新消息入队方式。普通 SSE 集成一般用 `"followUp"`（排队）；CLI 交互场景用 `"steer"`（中断）

## 实现思路

### 1. 进度信息的来源

| 来源 | 自然度 | 可靠性 | 推荐策略 |
|------|-------|-------|---------|
| 模型在 text 通道的输出（说明、引导、最终答复） | 高（自然语言、承上启下） | 高（流式转发，用户实时可见） | 直接转发为 content |
| `tool_execution_start` 携带的 `toolName` | 低（机械、用户难懂） | 高（每个工具必触发） | 转发为 status |
| bash 工具的实时输出（`tool_execution_update` 的 `partialResult` / B 类 `bash_execution_update` 的 `delta`）| 高（真实命令输出，用户最想看的进度）| 仅 bash 工具触发（其他工具默认不发 update）| bash 场景的强进度线（跨链 [E06](E06-streaming-transform.md)）|

模型 text 直接以 content 流式推给用户（打字机效果）；工具调用用 `tool_execution_start` 的 toolName 发一条 status 提示。bash 工具调用时还可叠加实时输出，是 SSE 进度推送的最佳信号源。

### 2. SSE 事件转发决策表

> 子事件 type 在 `event.assistantMessageEvent.type` 上（不是顶层 event.type）。顶层事件 type 是 `message_update`，下面「子事件」列指 assistantMessageEvent 的 type 值。

| 顶层事件 | 子事件（assistantMessageEvent.type）| SSE 类型 | 何时转发 |
|---------|------------------------------------|---------|---------|
| `message_update` | `text_delta` | `content` | 每个 delta 到达即转发（打字机效果）|
| `message_update` | `thinking_delta` / `thinking_start` / `thinking_end` | **不转发**（见避坑 1）| — |
| `tool_execution_start` | — | `status` | 到达即转发 toolName（可选携带 args）|
| `tool_execution_update`（A 类）/ `bash_execution_update`（B 类）| — | `status`（bash 实时输出） | bash 工具执行时，转发 `partialResult`/`delta` 作为实时进度（跨链 [E06](E06-streaming-transform.md)；其他工具默认不发 update）|
| `turn_end` | — | — | 不需要转发；需要本轮完整结果时读 `event.message`（见避坑 4）|
| `agent_settled` + `prompt()` finally | — | `done` | 必须双保险（见「完成信号机制」节）；推荐用 `agent_settled`，`agent_end` 在 retry 场景会提前触发（备选：`agent_end` && `!willRetry`）|

## 核心代码

唯一需要的状态变量是完成信号去重标志 `doneSent`（防 `agent_settled` + finally 重复发 done）。下面是 minimum viable pattern（覆盖 text / 工具 / 完成信号三条路径）：

```ts
// 外部宿主侧（Express route / WebSocket handler / CLI wrapper 等）
let doneSent = false;
const sendDone = () => { if (!doneSent) { doneSent = true; sendSSE("done", ""); } };

const unsubscribe = session.subscribe((event) => {
  // 文本增量：直接转发，打字机效果；思考增量忽略（用户不可见 + token 碎片）
  if (event.type === "message_update") {
    const ae = event.assistantMessageEvent;
    if (ae?.type === "text_delta") {
      sendSSE("content", ae.delta);
    }
    // 不处理 thinking_* —— 见避坑第 1 条
  }

  // 工具开始执行：转发为 status 进度信号
  if (event.type === "tool_execution_start") {
    sendSSE("status", `执行中：${event.toolName}`);
  }

  // 完成信号：推荐用 agent_settled（retry/compaction/steer 队列全消费完才派发，
  // 不会在 willRetry=true 时提前关 SSE；agent_end 在 retry 场景会提前触发）。
  // 详见下文「完成信号机制」节。
  if (event.type === "agent_settled" && !doneSent) {
    doneSent = true;
    sendSSE("done", "");
  }
});

// 完成信号双保险：prompt finally 兜底（详见「完成信号机制」节）

try {
  await session.prompt(message);
} finally {
  sendDone();      // 即便 agent_settled 未触发也兜底
  unsubscribe();   // ★ 长连接断开时务必调用，防止监听器泄漏
}
```

**并行多工具调用**（一个 turn 内多个 `toolCall`）：text 通道仍是单一流，不受影响；若要按工具分别跟踪执行进度，用 `event.toolCallId` 分桶。

## AgentSessionEvent 清单（subscribe 能收到的事件）

`AgentSessionEvent` 共 **22 种 distinct type**（源自 agent-session.ts:141-183，经 `coding-agent/src/index.ts:18` 对 SDK 使用者导出——类型签名可见且运行时会真实派发。源码里 `auto_retry_end` 在 167/182 行重复声明、`summarization_retry_attempt_start` 有 2 个 variant，TS 合并后各算 1 种）：

**A 类：来自 AgentEvent（10 种，扩展层 `pi.on` 也派发）**

| 事件 | 关键字段 | E11 是否用 |
|------|---------|----------|
| `agent_start` | — | 一般不用 |
| `agent_end` | `messages` / `willRetry`（subscribe 独有）| ⚠️ 完成信号备选（retry 时提前触发，需带 `!willRetry`；推荐改用 B 类 `agent_settled`，见「完成信号机制」节）|
| `turn_start` | **无字段**（subscribe 层；扩展层 `pi.on` 有 `turnIndex` + `timestamp`） | 可选（UI 上的轮次分隔）|
| `turn_end` | `message`（本轮 assistant message）/ `toolResults`（工具结果数组，独立字段）| ✅ 需要时读取本轮完整 message |
| `message_start` / `message_end` | `message` | message_end 不当完成信号（见坑 2）|
| `message_update` | `message` / `assistantMessageEvent` | ✅ 主力事件 |
| `tool_execution_start` | `toolCallId` / `toolName` / `args` | ✅ status 进度 |
| `tool_execution_update` | `toolCallId` / `toolName` / `args` / `partialResult` | 一般不用（跨链 E06）|
| `tool_execution_end` | `toolCallId` / `toolName` / `result` / `isError` | 可选 |

**B 类：session 独有（12 种，扩展层 `pi.on` 收不到）**

| 事件 | 触发时机 | E11 是否用 |
|------|---------|----------|
| `queue_update` | `steer()` 入队消息变化 | 可选（观察 steer 队列）|
| `compaction_start` / `compaction_end` | 上下文压缩 | 一般不用 |
| `session_info_changed` | 会话名字变更 | 一般不用 |
| `thinking_level_changed` | 思考等级变化 | 一般不用 |
| `auto_retry_start` / `auto_retry_end` | agent turn 自动重试（transient provider 错误） | 长跑场景可监听 |
| `summarization_retry_scheduled` / `summarization_retry_attempt_start` / `summarization_retry_finished` | compaction / branchSummary 的摘要子调用重试（共用 retry budget）| 一般不用（监听 `compaction_*` 时会遇到这些伴随事件）|
| `bash_execution_update` | bash 工具执行的 stdout/stderr 增量（`delta`）| ✅ 进度信号源（实时输出流，见「进度信息的来源」）|
| `agent_settled` | Agent 主循环完全结束（含 retry / compaction / steer 队列消费完毕）| ✅ 推荐替代 agent_end 做完成信号（见下文「完成信号机制」节）|
| `entry_appended` | Session 追加自定义条目 | 一般不用 |

> 完整字段表跨链 [sdk_doc/02-agent-session.md §subscribe](../sdk_doc/02-agent-session.md)。

## 完成信号机制（双保险）

### 为什么需要双保险

**坑 1（来自 04-events）**：偶发情况下所有 `message_end` 已正常触发，但 `agent_end` **始终不发出**。外层 `await session.prompt()` 不 resolve，HTTP/SSE 连接挂住。

**对策**：完成事件订阅（推荐 `agent_settled`，备选 `agent_end` 带 `!willRetry`）+ `prompt()` finally 双保险。

> **推荐**：用 `agent_settled` 事件（agent-session.ts）替代 `agent_end` 做完成信号。`agent_end` 在 retry 场景下会提前触发（`willRetry: true` 时不是真完成），而 `agent_settled` 只在 retry / compaction / steer 队列全部消费完毕后才派发（两层都派发），能避免 SSE 提前关闭的问题。

### agent_end 与 prompt() 的时序关系

源码证据：agent-session.ts:628 在 `_emitExtensionEvent` 之后调用 `_emit({ type: "agent_end", ... })`。`prompt()` 的 Promise 在 agent 主循环（含 agent_end 派发）全部完成后才 resolve。

因此 `try { await session.prompt(...) } finally { sendDone() }` 的执行顺序是：

```
agent_end emit → subscribe listener 同步执行 → prompt() resolve → finally 块跑
```

finally 必在 agent_end 之后跑，所以双保险的本质是：**兜底防 listener 异常、防 agent_end 漏触发（坑 1）**，不是"竞争关系"。

### agent_end.willRetry 字段（subscribe 独有）

subscribe 层的 `agent_end` 多一个 `willRetry: boolean` 字段（agent-session.ts:143-147）：

- `willRetry: false` —— 真正完成，可以安全发 `done`
- `willRetry: true` —— Agent 即将自动重试（如工具错误恢复），**不是真完成**

**陷阱**：如果 SSE 集成在 `willRetry: true` 时就发 `done`，前端会提前关闭，下一轮重试的进度用户完全看不到。

**正确做法**：

```ts
if (event.type === "agent_end" && !event.willRetry && !doneSent) {
  doneSent = true;
  sendSSE("done", "");
}
// willRetry=true 时不发 done，让重试继续走，subscribe 继续转发后续事件
```

## 超时与中断（跨链 F04）

SSE 集成是长连接场景，**必须**做超时与中断设计。详见 [F04-abort-session.md §超时设计（Web/SSE 集成必读）](F04-abort-session.md)，核心要点：

1. **双层 timeout**：单 turn timeout（如 60s）+ 总时长 timeout（如 5min）。单层总时长会让单个工具卡死拖垮整个会话
2. **abort 后处理**：超时触发 `session.abort()` 后，agent_end 会带 `willRetry: false` 派发，subscribe 收到后正常发 done
3. **客户端断开检测**：Express 中监听 `req.on("close", ...)`，客户端断开时调用 `session.abort()` + `unsubscribe()`，防止 server 端继续跑无用计算

## 长连接断开（unsubscribe 必要性）

`session.subscribe()` 返回一个 unsubscribe 函数。**长连接场景（SSE / WebSocket）断开时必须调用**，否则：

- 每次连接断开后 listener 仍在 session 的事件监听器列表里
- 下次连接再 subscribe，监听器数量持续增长（内存泄漏）
- 旧 listener 仍会触发，向已关闭的 socket 写数据可能抛异常

正确模式：

```ts
const unsubscribe = session.subscribe(listener);

req.on("close", () => {
  unsubscribe();
  // 可选：session.abort() 如果想让 server 端停止计算
});

try {
  await session.prompt(message);
} finally {
  unsubscribe();  // 双保险，即便 req.on("close") 没触发
}
```

## 避坑

1. **不转发 `thinking_delta`**：token 级碎片（1-3 字符/次），一次响应上百条，且 thinking 内容用户本就不可见。

2. **不用 `message_end` 当完成信号**：粒度是「每次 LLM 调用」，多轮工具调用一次提问触发 5+ 次。用 `agent_end` + `prompt()` finally 双保险。详见 [04-events 坑 1/2](../sdk_doc/04-events.md)。

3. **不假设模型每 turn 都输出 text**：模型可能只在 thinking 里说进度而 text 通道为空，必须靠 `tool_execution_start` 兜底，否则用户什么都看不到。

4. **`turn_end.message` 是本轮完整 response**：text/thinking/toolCall 都在 `message.content`，工具结果在独立的 `toolResults` 字段（不在 content 里）。想一次拿全本轮 → 用 `turn_end` 而非拼 `message_end`。

5. **`agent_end.willRetry: true` 不是真完成**：subscribe 独有字段，true 时 Agent 即将自动重试（错误恢复），别发 done。

6. **subscribe 收不到 6 个扩展独有事件**（`context`/`tool_call`/`tool_result`/`before_agent_start`/`input`/`model_select`），写 type 分支永不命中且不报错。需要它们 → 走扩展层 `pi.on`。详见 [04-events 坑 4](../sdk_doc/04-events.md)。

---

## 变体与延伸

- 工具内部主动 `onUpdate()` 推送进度（依赖工具作者配合，**只有 bash 工具真正触发**）→ [场景 E06](E06-streaming-transform.md) §内置工具 onUpdate 支持表
- 扩展主动用 `sendUserMessage` / `notify` 推送给用户（CLI 专属，本 skill 不展开）
- 终端 UI 定制（statusBar / footer）（CLI 专属，本 skill 不展开）
- 完整的 12 个 `assistantMessageEvent` 子事件枚举与时序性质 → [sdk_doc/04-events.md](../sdk_doc/04-events.md) 「assistantMessageEvent 子事件类型」节
- 超时 / abort / 错误处理 → [F04-abort-session.md §超时设计](F04-abort-session.md)
- 扩展层（`pi.on`）视角的事件系统 → [sdk_doc/04-events.md](../sdk_doc/04-events.md)
- AgentSession 完整方法清单 → [sdk_doc/02-agent-session.md](../sdk_doc/02-agent-session.md)
- 自定义工具返回值结构 → [D01-custom-tool.md](D01-custom-tool.md)
- 扩展基础（ExtensionContext / ExtensionMode）→ [E02-extension-basics.md](E02-extension-basics.md)
