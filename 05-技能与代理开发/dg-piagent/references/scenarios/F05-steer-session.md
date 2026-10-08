# 场景：向会话队列注入 steer 消息与历史读取 (F05)

## 目标

掌握 `session.steer()` 的入队机制、理解它在 Agent loop 中的消费时机、区分 steer / followUp / nextTurn 三种投递模式，以及如何读取会话历史。

## 什么时候用 / 不用会怎样

**用 steer 的前提**：Agent 正在处理一个多轮任务（典型：长链工具调用、代码重构、多文件分析），你想在**不打断当前 LLM 调用**的前提下，给 Agent 补充一句指示（如"记得先写单元测试"、"切到另一个分支再改"、"跳过测试文件"）。steer 的消息会**插队**——在当前 turn 结束后立刻注入并触发下一轮 turn，让 Agent 在继续工作前先看到你的补充。

| 你的场景 | 用什么 | 为什么 |
|---------|--------|--------|
| Agent 在多轮工具调用中，你想补充一句指示 | `session.steer(text)` | 插队注入，下轮 turn 立刻看到，不打断当前 LLM 调用 |
| Agent 在工作中，你想塞一句话但**不希望插队**（让 Agent 自然完成当前思路） | `session.followUp(text)` | 等 Agent 想停下来时（无 tool call 且无 steer）再投递 |
| Agent 已停止，你想直接给它发新消息 | `session.prompt(text)` | steer 只入队，loop 退出后 steer 不触发新 turn，需等下次 prompt() 才被消费 |
| 想在不发新 user message 的情况下，修改下一轮 LLM 看到的 context | 扩展层 `pi.on("context")` 事件（内部经 sdk.ts 接入 agent 层 `transformContext`） | steer 加的是 user message，context hook 改的是 LLM 输入快照，层次不同 |
| 想中止当前进行中的 LLM 调用 | `session.abort()`（[F04](F04-abort-session.md)） | steer 是入队，abort 是打断——两者完全不同 |

**核心区别一句话**：`steer` 是"等当前 turn 结束后把消息塞进下一轮、并触发新一轮 turn"；`followUp` 是"等 Agent 想停下来时再投递"；`abort` 是"打断进行中的 LLM/工具/retry"。

## 涉及 SDK

| 能力 | 签名 | 用途 | 详细文档 |
|------|------|------|---------|
| `session.steer(text, images?)` | `async steer(text: string, images?: ImageContent[]): Promise<void>` | 入队 steering 消息（当前 turn 结束后注入） | [sdk_doc/02-agent-session.md](../sdk_doc/02-agent-session.md) |
| `session.followUp(text, images?)` | `async followUp(text: string, images?: ImageContent[]): Promise<void>` | 入队 follow-up 消息（Agent 想停止时才投递） | [sdk_doc/02-agent-session.md](../sdk_doc/02-agent-session.md) |
| `session.clearQueue()` | `clearQueue(): { steering: string[]; followUp: string[] }` | 清空两个队列并返回被清空的内容 | [sdk_doc/02-agent-session.md](../sdk_doc/02-agent-session.md) |
| `session.getSteeringMessages()` | `getSteeringMessages(): readonly string[]` | 只读查询当前 steering 队列内容 | 见下方「队列管理 API」 |
| `session.pendingMessageCount` | `get pendingMessageCount(): number` | 两个队列的总待处理数（steering + followUp） | 见下方 |
| `session.steeringMode` | `get steeringMode: "all" \| "one-at-a-time"` | drain 模式（一次取全部 vs 一次取一条） | 见下方 |
| `session.setSteeringMode(mode)` | `setSteeringMode(mode: "all" \| "one-at-a-time"): void` | 设置 drain 模式（steeringMode 是 getter-only，必须用此方法设置） | 见下方 |
| `session.state.messages` | `get state(): AgentState`（含 `messages: AgentMessage[]`） | 读取对话历史（返回 `agent.state.messages` 的同一引用，非冻结副本；streaming 中随 `message_end` 累积增长，勿直接写入该数组） | [sdk_doc/02-agent-session.md](../sdk_doc/02-agent-session.md) |
| `pi.sendUserMessage(content, options?)` | `sendUserMessage(content: string \| (TextContent \| ImageContent)[], options?: { deliverAs?: "steer" \| "followUp" }): void` | 扩展中投递 user 消息（流式时按 deliverAs 入队） | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `pi.sendMessage(message, options?)` | `sendMessage<T>(message: Pick<CustomMessage<T>, "customType" \| "content" \| "display" \| "details">, options?: { triggerTurn?: boolean; deliverAs?: "steer" \| "followUp" \| "nextTurn" }): void` | 扩展中投递 custom 消息 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |

## 三个核心机制（先理解，再写代码）

### 1. steer 是入队、不打断当前 LLM 调用——但会"插队"触发新一轮 turn


```ts
steer(message: AgentMessage): void {
    this.steeringQueue.enqueue(message);
}
```

底层 `agent.steer()` **同步**入队，不干扰当前正在进行的 LLM 调用或工具执行。但入队之后，agent-loop 在**每轮 turn 结束时**会调用 `getSteeringMessages` 检查队列。如果队列非空，loop 不会退出，而是把这些消息作为新一轮 turn 的开头注入，然后立刻调 LLM 生成响应。

**换句话说**：steer 不打断"当前 LLM 调用"，但会**打断"Agent 想停下来"的意图**——原本 Agent 这一轮 turn 做完就该 emit agent_end，因为 steer 的存在被强行续命了一轮。

这与 `followUp` 形成对照：followUp 入队后，只有当 Agent **真的想停下来**（无 tool call 且 steering 队列空）时，outer loop 才检查 follow-up 队列。所以 followUp 才是真正的"等 Agent 自然完成再投递"。

### 2. 消费时机——在每轮 turn 结束后、turn_start 发出后、下一轮 LLM 调用前

的关键顺序：

```
emit turn_end
  ↓
（如果 shouldStopAfterTurn 返回 true）emit agent_end + return
  ↓
pendingMessages = await getSteeringMessages()   ← 这里 drain steering 队列
  ↓
回到 inner while 循环判断：hasMoreToolCalls || pendingMessages.length > 0
  ↓
如果有 pendingMessages，进入新一轮 turn
  ↓
turn_start emit（仅在非首轮）
  ↓
注入 pendingMessages 到 messages（emit message_start/end）
  ↓
streamAssistantResponse   ← 下一轮 LLM 调用
```

**关键点**：注入发生在 **`turn_start` 之后**、下一轮 `streamAssistantResponse` 之前。源码顺序（agent-loop.ts:174-193）：inner while 判断继续 → emit `turn_start`（仅非首轮）→ 注入 pendingMessages（emit message_start/end + push 到 messages）→ `streamAssistantResponse`。即 steering 消息是在新一轮 turn **已经启动（`turn_start` 已发）之后**才注入的——所以如果想在 `turn_start` 里立刻读到注入后的 messages，会踩空（此时还没注入）。

> 特别地，loop 启动时也会先 drain 一次——"user may have typed while waiting"。注意：`agent.continue()`（retry 续跑场景）的首次 drain 会被 `skipInitialSteeringPoll` 跳过（agent.ts:471-475），避免重复消费刚 drain 的消息；`prompt()` 主路径不跳过。

### 3. steer 的生效前提——loop 必须还在跑

steer 只入队，不主动启动 agent loop。如果调用 `steer()` 时 agent loop 已经退出（emit agent_end 之后），消息会留在队列里**不会触发新 turn**——需等下一次 `prompt()` 启动新 loop 时，loop 启动会先 drain 队列（agent-loop.ts:167），届时才被消费。

**典型失败场景**：
- GLM 等响应快的模型 + 简单 prompt（如"列出当前目录"）→ agent loop 可能在你的 `setTimeout(steer, 1000)` 触发前就全部完成退出
- 此时 steer 消息留在队列，下一次 `prompt()` 时被消费——但这次 prompt 又是新的 user message，相当于 steer 的指示被"延迟到了不相关的对话"

**修复模式**：`setTimeout` 必须在 `session.prompt()` **之前**启动，且延迟设短（500ms-1s）。这样保证 timer 触发时 loop 还在跑：

```ts
// ✅ 正确：setTimeout 在 prompt 之前启动
setTimeout(() => {
  session.steer("补充指示");
}, 500);
const promise = session.prompt("原始指令");
await promise;
```

## 三种 deliverAs 模式对比

扩展层 `pi.sendUserMessage` / `pi.sendMessage` 在流式时支持 `deliverAs` 选项。`session.prompt(text, { streamingBehavior })` 也走相同机制。

| 模式 | 投递时机 | 是否插队 | 典型场景 |
|------|---------|---------|---------|
| `"steer"` | 当前 turn 结束后立即注入，触发新一轮 turn | ✅ 强行续命一轮 | Agent 正在工作中补充指示 |
| `"followUp"` | Agent 想停止时（无 tool call 且 steering 空）才投递 | ❌ 不插队 | 想追加问题但不想打断 Agent 思路 |
| `"nextTurn"` | 下次 `prompt()` 启动新 loop 时作为 context 附加（仅 custom message 支持） | ❌ 不启动新 turn | 持久化附加上下文（如本次会话的环境变量），不主动触发对话 |

`"nextTurn"` 只有 `pi.sendMessage`（custom message）支持；`pi.sendUserMessage` 只支持 `"steer"` / `"followUp"`。

> **触发新 turn 的条件**：`"steer"` 和 `"followUp"` 都会让 agent loop 继续（差别在于"何时继续"）；`"nextTurn"` **不会**让当前 loop 继续，只是把消息塞进 `_pendingNextTurnMessages` 数组，等下次 `prompt()` 时被注入。

## steeringMode：drain 模式（`all` vs `one-at-a-time`）


| 模式 | drain 行为 | 默认 | 典型场景 |
|------|-----------|------|---------|
| `"all"` | 一次取出队列里所有消息，全部注入后调一次 LLM | ❌ | 用户连发多条补充，希望合并成一轮处理 |
| `"one-at-a-time"` | 一次只取一条，每条触发一轮独立的 turn | ✅ | 默认行为；每条补充都有自己的 LLM 响应 |

```ts
// 修改 steeringMode（必须用 setSteeringMode() 方法，steeringMode 是 getter-only）
session.setSteeringMode("all");  // 之后入队的 steer 消息会一次全部 drain
```

也可通过 settings 配置（`settings.steeringMode`）。旧字段名 `queueMode` 已迁移为 `steeringMode`。

> **内部数据结构**：steering 消息在内部维护双层结构——`_steeringMessages`（session 层，UI 跟踪用 string[]）和 `steeringQueue`（agent 层，agent-loop 实际 drain 源 PendingMessageQueue）。`getSteeringMessages()` 返回的是 session 层的 `_steeringMessages`。

## followUpMode：followUp 队列的对称 API

与 `steeringMode` 对称，followUp 队列也有自己的管理 API：

| 方法 / getter | 签名 | 用途 |
|--------------|------|------|
| `session.followUpMode` | `get: "all" \| "one-at-a-time"` | followUp 队列的 drain 模式（与 steeringMode 语义对称） |
| `session.setFollowUpMode(mode)` | `(mode: "all" \| "one-at-a-time") => void` | 设置 followUp drain 模式 |
| `session.getFollowUpMessages()` | `(): readonly string[]` | 只读查询当前 followUp 队列内容 |

签名完全对称：`steeringMode` / `setSteeringMode` / `getSteeringMessages` 对应 `followUpMode` / `setFollowUpMode` / `getFollowUpMessages`。

## queue_update 事件：观察队列状态

当 steer / followUp 入队或被消费时，session 会派发 `queue_update` 事件：

```ts
session.subscribe((event) => {
  if (event.type === "queue_update") {
    console.log("steering 队列:", event.steering);  // readonly string[]
    console.log("followUp 队列:", event.followUp);  // readonly string[]
  }
});
```

> **层次提醒**：`queue_update` 是 `AgentSessionEvent`（subscribe 层）独有，**不在扩展层 `pi.on` 的事件清单中**。扩展层（`pi`）**没有**获取队列内容的方法，只有 `pi.hasPendingMessages(): boolean`（仅布尔，无内容，types.ts:338）；若需队列内容，只能在 session 层（subscribe）用 `session.getSteeringMessages()`。详见 [04-events.md](../sdk_doc/04-events.md)。

## 队列管理 API（B 档）

除了 `steer` / `followUp` 入队，session 还提供以下管理 API：

| 方法 | 签名 | 用途 |
|------|------|------|
| `clearQueue()` | `(): { steering: string[]; followUp: string[] }` | 清空两个队列，返回被清空的内容 |
| `getSteeringMessages()` | `(): readonly string[]` | 只读查询当前 steering 队列内容 |
| `pendingMessageCount` | `get: number` | 两个队列的总待处理数 |

> **层次提醒**：以上 `getSteeringMessages()` / `getFollowUpMessages()` / `pendingMessageCount` / `clearQueue()` 均为 **session 层** API（在 `AgentSession` 上）。扩展层（`pi`）不暴露这些方法，队列查询仅有 `pi.hasPendingMessages(): boolean`（仅布尔，无内容）。

> **F02 横向陷阱**：如果使用 `AgentSessionRuntime` 切换会话，**新 session 上的队列是空的**——旧 session 的 steer 队列不会迁移。详见 [F02](F02-session-runtime.md)。

## 核心代码

```ts
import { createAgentSession, SessionManager } from "@earendil-works/pi-coding-agent";

const { session } = await createAgentSession({
  sessionManager: SessionManager.inMemory(),
});

// 订阅 queue_update 观察队列状态（subscribe 层事件）
const unsubscribe = session.subscribe((event) => {
  if (event.type === "queue_update") {
    console.log("[queue]", event.steering);
  }
});

try {
  // ✅ 模式 1：setTimeout 在 prompt 之前启动，确保 timer 触发时 loop 还在跑
  setTimeout(() => {
    session.steer("记得先写单元测试再重构");
  }, 500);

  const promise = session.prompt("重构 src/utils.ts");
  await promise;

  // ✅ 模式 2：在 turn_end hook 里读取当前 messages（loop 还在跑时也可读，
  //    但要等 agent_end 后才完整）
  for (const msg of session.state.messages) {
    const preview = typeof msg.content === "string"
      ? msg.content.slice(0, 80)
      : "(structured content)";
    console.log(`[${msg.role}]`, preview);
  }
} finally {
  unsubscribe();
  session.dispose();
}
```

**关键点**：
- `setTimeout(..., 500)` **必须在 `session.prompt(...)` 之前**调用——否则快模型可能在 timer 触发前就完成 loop，steer 消息永远不被消费
- `session.state.messages` 返回的是 `agent.state.messages` 的**同一引用**（非冻结副本），streaming 中会随 `message_end` 累积增长；勿直接写入该数组。**安全读取时机是 `agent_end` 事件之后**（推荐用 `agent_settled` 作为更可靠的稳定信号——retry 场景下 `agent_end` 会多次 emit，`agent_settled` 只在所有 retry / steer 消费完毕后 emit 一次）
- `session.dispose()` 是同步的（[F04](F04-abort-session.md)）；用 `AgentSessionRuntime` 时改用 `await runtime.dispose()`

## 扩展中注入 steer 消息

```ts
// ✅ 安全模式：只在 before_agent_start 注入一次（避免死循环）
export default (pi) => {
  pi.on("before_agent_start", async (event, ctx) => {
    // 从外部数据源加载上下文，一次性注入
    const recentChanges = await getRecentGitCommits(ctx.cwd);
    pi.sendUserMessage(
      `[Recent Changes]\n${recentChanges}`,
      { deliverAs: "steer" },  // 当前 turn 结束后注入并触发新一轮
    );
  });
};
```

> **⚠️ 不要在 `turn_start` 里无条件 `sendUserMessage` + `deliverAs: "steer"`**！每个 steer 都会触发新一轮 turn，新一轮 turn 又会触发 turn_start，形成**死循环**。如果非要在 `turn_start` 注入，必须加去重条件（如"只在首轮"或"满足某条件才注入"）。`before_agent_start` 是更安全的钩子（每次 prompt 只触发一次）。

## steer 与扩展层 context hook 的区别

两者都用于"给 Agent 补充信息"，但层次完全不同：

| 维度 | `session.steer()` | 扩展层 `pi.on("context")` hook |
|------|-------------------|------------------------------|
| 层次 | 加 user message 到对话流 | 修改 LLM 调用前的 context 快照 |
| 进 messages 历史 | ✅ 进（user role） | ❌ 不进（只是临时修改 LLM 输入） |
| 触发新一轮 turn | ✅ 触发 | ❌ 不触发（只在当前 turn 内生效） |
| 典型场景 | 用户指示、补充要求 | 注入环境信息（git log、文件列表）到 LLM 输入 |
| 文档 | 本文档 | [G01](G01-context-injection.md) / [G04](G04-preload-context.md) |

详见 [G01-context-injection.md](G01-context-injection.md)。

## 常见误期待与陷阱

1. **「loop 结束后 steer 仍生效」**——错。steer 只入队，loop 已退出则消息永远不被消费（除非下次 prompt）。修复：setTimeout 在 prompt 之前启动。
2. **「在 turn_start hook 里无条件 sendUserMessage + steer」**——错，会死循环。每个 steer 触发新一轮 turn → 新一轮 turn_start → 又 steer。修复：用 `before_agent_start`（每次 prompt 只触发一次）或加去重条件。
3. **「steer 是温和的不打扰」**——半真。steer 不打断当前 LLM 调用，但**会插队**强行触发新一轮 turn——原本 Agent 想停的意图被打断了。真"不打扰"的是 `followUp`（等 Agent 想停时再投递）。
4. **「streaming 中读 messages 拿到的是完整历史」**——错。`session.state.messages` 返回的是 `agent.state.messages` 的**同一引用**（非冻结副本），streaming 中正在变动（streamingMessage 不在其中，但已完成的消息会被 push）。安全读取时机是 `agent_end` 事件之后；**推荐用 `agent_settled`**——`agent_end` 在 retry 场景下会提前触发，`agent_settled` 确保所有 retry / steer 队列消费完毕后才派发。
5. **「扩展层能监听 queue_update 事件」**——错。`queue_update` 是 subscribe 层（AgentSessionEvent）独有，扩展层 `pi.on` 收不到。扩展层**没有**获取队列内容的方法（`pi.getSteeringMessages()` 不存在），只能用 `pi.hasPendingMessages(): boolean` 查询"是否有待处理消息"（仅布尔）；若需队列内容，必须在 session 层（subscribe）用 `session.getSteeringMessages()`。
6. **「steeringMode 无所谓」**——看场景。默认 `one-at-a-time` 每条 steer 触发独立一轮 turn；如果用户连发多条补充希望合并处理，应该设为 `all`。
7. **「AgentSessionRuntime 切换会话后 steer 队列会迁移」**——错。新 session 的队列是空的。详见 [F02](F02-session-runtime.md)。

## 变体与延伸

- steer 与 abort 的区别（打断 vs 入队） → 见 [场景 F04](F04-abort-session.md)
- context injection 的完整方案（transformContext / pi.context hook） → 见 [场景 G01](G01-context-injection.md)
- 扩展中 prefill 上下文（`before_agent_start` 模式） → 见 [场景 G04](G04-preload-context.md)
- 队列管理与 runtime 切换的关系 → 见 [场景 F02](F02-session-runtime.md)
- session 完整 API 参考 → 见 [sdk_doc/02-agent-session.md](../sdk_doc/02-agent-session.md)
