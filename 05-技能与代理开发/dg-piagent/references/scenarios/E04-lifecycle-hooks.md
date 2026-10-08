# 场景：生命周期 Hook (E04)

## 什么时候用 / 不用会怎样

**该用本场景**：

- **资源初始化 / 清理**：会话开始时建立数据库连接、读入配置文件、设置 UI 状态栏；会话结束时释放资源、生成摘要报告
- **轮次 / 消息级跟踪**：每轮对话开始 / 结束时记日志、统计 token、做审核
- **改系统提示词**：每轮开始前基于动态上下文修改 systemPrompt（如注入最新用户偏好、业务规则）
- **改最终消息**：assistant 完成响应后，落库前替换 / 加工消息内容

**不用会怎样**：

- 想做副作用（初始化、日志、清理）只能写在 prompt 调用前后——丢失粒度（轮次级、消息级都拿不到）
- 想改 systemPrompt 只能整体重写 `createAgentSession({ systemPrompt })`——无法基于运行时动态信息
- 想统计工具调用次数只能事后扫消息列表——拿不到逐轮的精确时序

**不适合本场景**：

- 拦截 / 改写工具调用参数 → 见 [场景 E01](E01-tool-intercept.md)（tool_call 钩子，能 block）
- 改写工具返回结果 → 见 [场景 E05](E05-input-transform.md) / [D05](D05-tool-result-render.md)（tool_result 钩子）
- 注册自定义工具 / 用户命令 → 见 [场景 E02](E02-extension-basics.md)（扩展三件套）
- 流式渲染 assistant 输出 → 见 [场景 E06](E06-streaming-transform.md)（message_update 流式变换）
- 会话持久化 / 恢复 → 见 [场景 F01](F01-session-persistence.md)

---

## 范围（★ 先看这个）

本场景聚焦 **Agent 生命周期事件**（`before_agent_start` / `agent_start` / `turn_start` / `turn_end` / `agent_end`）+ **消息生命周期事件**（`message_start` / `message_end`）。这 7 个事件覆盖「会话级 + 轮次级 + 消息级」三层粒度。

不在本场景展开：
- `message_update`（逐 token 流式）→ [E06](E06-streaming-transform.md)，需要分桶 / 增量协议知识
- `tool_call` / `tool_result` → [E01](E01-tool-intercept.md) / [D05](D05-tool-result-render.md)
- `session_*` 9 种事件（多 CLI 专属）→ [04-events.md](../sdk_doc/04-events.md)

---

## 涉及 SDK

| 事件 | 触发时机 | handler 可改 | 详细文档 |
|------|---------|------------|---------|
| `before_agent_start` ⭐ | 用户提交 prompt 后、agent loop 前 | **systemPrompt / 注入消息** | [sdk_doc/04-events.md](../sdk_doc/04-events.md) |
| `agent_start` | agent loop 开始（每 trace 1 次） | 否 | [sdk_doc/04-events.md](../sdk_doc/04-events.md) |
| `agent_end` | agent loop 结束（每 trace 1 次） | 否 | [sdk_doc/04-events.md](../sdk_doc/04-events.md) |
| `turn_start` | 每个 turn 开始 | 否 | [sdk_doc/04-events.md](../sdk_doc/04-events.md) |
| `turn_end` | 每个 turn 结束 | 否 | [sdk_doc/04-events.md](../sdk_doc/04-events.md) |
| `message_start` | user / assistant / toolResult 消息开始 | 否 | [sdk_doc/04-events.md](../sdk_doc/04-events.md) |
| `message_end` | 消息结束 | **替换 message（role 必须一致）** | [sdk_doc/04-events.md](../sdk_doc/04-events.md) |

> ⚠️ **关键警示（扩展独有事件）**：`before_agent_start` 是 6 个**只在扩展层派发**的事件之一（另五个是 `context` / `tool_call` / `tool_result` / `input` / `model_select`）。在 server 层用 `session.subscribe("before_agent_start", ...)` 会**静默失败**——handler 注册了但永远不被调用，不报错不警告。想在 server 层感知 agent 启动只能用 `agent_start`（两层都派发，但拿不到 systemPrompt）。证据：types.ts、agent-session.ts。详见 [04-events 坑 4](../sdk_doc/04-events.md#坑-46-个事件-subscribe-静默收不到-最大集成坑)。

---

## 事件 payload 速查（★ 必读）

handler 第一个参数 `event` 的字段（types.ts）：

| 事件 | payload 字段 | 关键说明 |
|------|------------|---------|
| `before_agent_start` | `{ type, prompt: string, images?: ImageContent[], systemPrompt: string, systemPromptOptions }` | `prompt` 是用户原始输入（扩展后）；`systemPrompt` 是**完整组装好的字符串**（能直接读 / 改）；`systemPromptOptions` 是构建系统提示词的结构化选项（含加载的文件、技能等资源信息），可用于了解 Pi 加载了哪些资源 |
| `agent_start` | `{ type: "agent_start" }` | 无额外字段 |
| `agent_end` | `{ type, messages: AgentMessage[] }` | ⚠️ 扩展层 payload **没有 `willRetry`**（subscribe 层才有，见陷阱 5） |
| `turn_start` | `{ type, turnIndex: number, timestamp: number }` | ⚠️ 扩展层比 session 层多 `turnIndex` / `timestamp` |
| `turn_end` | `{ type, turnIndex: number, message: AgentMessage, toolResults: ToolResultMessage[] }` | ⚠️ 扩展层比 session 层多 `turnIndex`；**没有 `toolCalls` 字段**——工具调用信息在 `message.content` 里筛 `type === "toolCall"` |
| `message_start` | `{ type, message: AgentMessage }` | `message` 是联合类型（UserMessage / AssistantMessage / ToolResultMessage） |
| `message_end` | `{ type, message: AgentMessage }` | handler 返回 `{ message: newMsg }` 可替换，但 **role 必须一致**（见陷阱 4） |

> **三层 payload 差异**（v0.80.2 源码核对）：`turn_start` / `turn_end` / `agent_end` 在扩展层与 session 层字段不同。若集成时跨层转发事件，**不能假设 payload 同构**。详见 [04-events.md 三层差异速查](../sdk_doc/04-events.md#三层-payload-差异速查)。

#### handler 返回值

| 事件 | 返回值类型 | 作用 |
|------|-----------|------|
| `before_agent_start` | `{ message?: CustomMessage, systemPrompt?: string }` | 返回 `systemPrompt` 替换本轮系统提示词，**多扩展链式覆盖**（后覆盖前）；返回 `message` 注入自定义消息 |
| `message_end` | `{ message?: AgentMessage }` | 替换最终消息（**role 必须一致**，否则被丢弃） |
| 其他 5 个 | `void` | 返回值被忽略 |

证据：types.ts。

---

## ExtensionContext（事件 handler 的 ctx）

每个事件 handler 接收 `(event, ctx)`，其中 `ctx: ExtensionContext` 提供运行时上下文。**关键字段**：

| 字段 | 类型 | 含义 |
|------|------|------|
| `ctx.ui` | `ExtensionUIContext` | UI 方法（setStatus / notify / select / confirm / input 等） |
| `ctx.mode` | `"tui" \| "rpc" \| "json" \| "print"` | 当前运行模式，**默认 "print"**（见陷阱 2） |
| `ctx.hasUI` | `boolean` | 是否有可交互 UI（tui / rpc 为 true） |
| `ctx.cwd` | `string` | 当前会话工作目录（**不是 `process.cwd()`**） |
| `ctx.sessionManager` | `ReadonlySessionManager` | 只读会话管理器 |
| `ctx.model` | `Model<any> \| undefined` | 当前模型 |
| `ctx.signal` | `AbortSignal \| undefined` | 当前流式操作的 abort signal |
| `ctx.abort()` | `() => void` | 中断当前 agent 操作 |
| `ctx.shutdown()` | `() => void` | 优雅关闭 pi 并退出 |
| `ctx.compact()` | `(options?) => void` | 触发压缩 |
| `ctx.getSystemPrompt()` | `() => string` | 获取当前系统提示词 |

> ⚠️ **`ExtensionContext` 没有 `session` 字段、没有 `waitForIdle` 方法**。证据：types.ts。`waitForIdle` 只在 `ExtensionCommandContext`（命令 handler 的 ctx）上——见 [E02 ExtensionCommandContext](E02-extension-basics.md#extensioncommandcontext命令-handler-的-ctx)。
>
> `ExtensionContext` 还有 `modelRegistry`、`scopedModels`、`thinkingLevel`、`isIdle()`、`isProjectTrusted()`、`hasPendingMessages()`、`getContextUsage()` 等字段，上表仅列出生命周期钩子常用的关键字段。完整接口见 types.ts。

---

## 触发频次（★ 高频 vs 低频）

| 频次 | 事件 | handler 性能要求 |
|------|------|----------------|
| **每 trace 1 次** | `before_agent_start` / `agent_start` / `agent_end` | 低频，但仍被派发方 `await`——可做轻量启动（建连接、读小文件），**不可 `await` 秒级 LLM 调用或慢 DB 写**，那些必须 fire-and-forget |
| **每 prompt 1 次** | `agent_settled` | 低频，但仍被派发方 `await`（低频≠可阻塞）。在所有 retry / compaction / steer 队列消费完后才派发，两层都派发——是坑 1（agent_end 不可靠）的官方解药。同样**不可 `await` 秒级重活** |
| **每 turn 1 次** | `turn_start` / `turn_end` | 中频，handler 应百毫秒内完成 |
| **每轮 LLM 调用 1 次** | `message_end`（assistant） | 中频，一轮多次触发 |
| **每次工具调用 1 次** | `message_end`（toolResult） | 看工具数量 |
| **逐 token** | `message_update` | **高频**——不在本场景，见 [E06](E06-streaming-transform.md) |

**关键提示**：`message_update` 才是真正的高频事件（每个 token 一次），`turn_*` 频率并不算高。但**所有 pi.on handler 都被派发方 `await`**，不论高频低频，里面都不该 `await` 慢 I/O——那会卡住 agent loop、延长 `prompt()` 的 resolve。低频≠可阻塞：`agent_settled` 虽每 prompt 只发一次，handler 里 `await` DB 写仍会让 `await session.prompt()` 多挂这几百毫秒。落库 / 二次 LLM 调用这类重活要么 fire-and-forget（推队列后台写），要么改用 `session.subscribe` 落库（不 await listener，天然不阻塞）。证据：[04-events.md 事件触发频次](../sdk_doc/04-events.md#事件触发频次)、[04-events.md 关键细节](../sdk_doc/04-events.md#关键细节)。

---

## 核心代码

完整的「初始化 → 跟踪 → 改 prompt → 改消息 → 收尾」示例：

```ts
// my-lifecycle-extension.ts
export default (pi) => {
  const stats = { turns: 0, toolCalls: 0, startTime: 0 };

  // ① before_agent_start：每 trace 1 次，能改 systemPrompt
  pi.on("before_agent_start", (event, ctx) => {
    // event.systemPrompt 是完整组装好的字符串（可读可改）
    console.log("[Lifecycle] systemPrompt 长度:", event.systemPrompt.length);
    // 返回 { systemPrompt } 替换本轮系统提示词（多扩展链式覆盖）
    // return { systemPrompt: event.systemPrompt + "\n额外规则: ..." };
  });

  // ② agent_start：每 trace 1 次，初始化资源 + 设置 UI 状态
  pi.on("agent_start", (event, ctx) => {
    stats.startTime = Date.now();
    // ⚠️ ctx.ui.* 默认是 no-op（mode="print"），生产代码加 hasUI 守卫
    if (ctx.hasUI) {
      ctx.ui.setStatus("lifecycle", "会话进行中");
    }
  });

  // ③ turn_start：每 turn 1 次，预加载数据
  pi.on("turn_start", (event, ctx) => {
    stats.turns++;
    // event.turnIndex / event.timestamp 可用（扩展层独有字段）
    console.log(`[Lifecycle] 第 ${event.turnIndex + 1} 轮开始`);
  });

  // ④ turn_end：每 turn 1 次，统计工具调用
  pi.on("turn_end", (event, ctx) => {
    // ⚠️ 没有 event.toolCalls 字段！工具调用信息在 message.content 里
    const toolCallsInTurn = event.message.content.filter(
      (c: any) => c.type === "toolCall"
    ).length;
    stats.toolCalls += toolCallsInTurn;
    console.log(
      `[Lifecycle] 第 ${event.turnIndex + 1} 轮结束，` +
        `本轮 ${toolCallsInTurn} 次工具调用，累计 ${stats.toolCalls}`
    );
  });

  // ⑤ message_end：替换最终消息（role 必须一致）
  pi.on("message_end", (event, ctx) => {
    const msg = event.message;
    if (msg.role === "assistant") {
      // 可以在这里加工消息内容
      // ⚠️ 返回的 message 的 role 必须与原 message 相同，否则 runner 会丢弃并 emitError
      // return { message: { ...msg, content: processedContent } };
    }
  });

  // ⑥ agent_end：每 trace 1 次，清理 + 摘要
  pi.on("agent_end", (event, ctx) => {
    const duration = Date.now() - stats.startTime;
    // ⚠️ 扩展层 payload 没有 willRetry！判断重试要自己跑 SDK 的正则（见陷阱 5）
    if (ctx.hasUI) {
      ctx.ui.setStatus("lifecycle", undefined); // 清除状态栏
      ctx.ui.notify(
        `会话结束。${stats.turns} 轮, ${stats.toolCalls} 次工具调用, ` +
          `耗时 ${(duration / 1000).toFixed(1)}s`,
        "info"
      );
    }
    console.log("[Lifecycle] Agent 结束");
  });
};
```

> **代码块之间有呼吸**：上方示例展示了「初始化 + 跟踪 + 改 prompt + 改消息 + 收尾」五类模式。下面分别展开两类最易踩坑的（改 systemPrompt + 改消息）。

---

## 模式一：用 `before_agent_start` 改系统提示词

**场景**：每轮开始前，基于动态信息修改 systemPrompt（如注入最新用户偏好、业务规则、运行时配置）。

```ts
pi.on("before_agent_start", (event, ctx) => {
  // event.systemPrompt 是完整字符串，能直接读
  const basePrompt = event.systemPrompt;

  // 基于运行时信息追加规则
  const dynamicRules = loadDynamicRules(ctx.cwd);
  return {
    systemPrompt: `${basePrompt}\n\n## 运行时规则\n${dynamicRules}`,
  };
});
```

**链式覆盖机制**（runner.ts）：

- 多个扩展都返回 `systemPrompt` 时，**后覆盖前**（runner 把 `currentSystemPrompt` 传给下一个 handler，下一次 emit 用最新值）
- 顺序由 `extensionFactories` 数组顺序决定——对顺序敏感的逻辑必须显式控制加载顺序
- 同时返回 `message` 会作为自定义消息注入对话（见 [02-agent-session.md CustomMessageEntry](../sdk_doc/02-agent-session.md)）

> ⚠️ **不要用 `ctx.getSystemPrompt()` 在 handler 里读"改后"的 prompt**——`ctx.getSystemPrompt()` 返回闭包变量 `currentSystemPrompt`，而该变量的更新发生在 handler 返回之后（runner.ts）。所以 handler 执行期间调 `getSystemPrompt()` 看到的是"上一个 handler 的覆盖结果（或初始值）"，不是自己刚 return 的值。多个 handler 链式覆盖时，每个 handler 拿到的都是「进入本 handler 前」的快照。

---

## 模式二：用 `message_end` 替换最终消息

**场景**：assistant 完成响应后、落库前，对消息做加工（脱敏、加水印、修正格式）。

```ts
pi.on("message_end", (event, ctx) => {
  const msg = event.message;
  if (msg.role !== "assistant") return;  // 只处理 assistant

  // ⚠️ 返回的 message 的 role 必须与原 message 相同
  // runner.ts 会校验 role 一致性，不一致就丢弃 + emitError
  return {
    message: {
      ...msg,
      content: sanitizeContent(msg.content),  // 自定义加工
    },
  };
});
```

**关键限制**（runner.ts）：

1. **role 必须一致**——不能把 assistant 消息改成 user 消息（runner 校验失败后丢弃，并 emitError）
2. **多 handler 累积替换**——前一个 handler 返回的 message 会作为下一个 handler 的输入（`{ ...event, message: currentMessage }`）
3. **粒度是「每次 LLM 调用」**——单轮对话可能触发多次（如调用工具前的预文本 + 工具返回后的最终回答），不是「一条最终消息一次」（见 [04-events 坑 2](../sdk_doc/04-events.md#坑-2message_end-单轮多次触发粒度是每次-llm-调用非每条消息)）

---

## 关键陷阱（6 条）

### 陷阱 1：`ctx.ui.*` 默认是 no-op（mode 守卫）

`ExtensionMode` 默认是 `"print"`（SDK 集成 / `--print` 模式），此时 `ctx.ui` 是 `noOpUIContext`——所有 UI 方法都是空操作（`setStatus` 不报错但不显示、`confirm` 返回 false、`select` 返回 undefined）。证据：runner.ts、runner.ts（默认 mode）、runner.ts（setUIContext 默认值也是 "print"）。

**生产代码必须加守卫**：

```ts
pi.on("agent_start", (event, ctx) => {
  if (ctx.hasUI) {           // 或 ctx.mode === "tui"
    ctx.ui.setStatus("my-ext", "已就绪");
  }
});
```

### 陷阱 2：生命周期钩子异常处理（与 tool_call 不同）

**生命周期钩子（`agent_start` / `agent_end` / `turn_*` / `message_start`）** 走通用 `emit` 方法（runner.ts），**有 try/catch + emitError**——handler 抛异常会被捕获，记录到扩展错误流，**不影响 agent**。

`before_agent_start`（runner.ts）和 `message_end`（runner.ts）也都有 try/catch。

**对比 `tool_call`**：`emitToolCall`（runner.ts）**内部没有 try/catch**——handler 抛异常会冒泡到外层 `beforeToolCall`（agent-session.ts），转成 "Extension failed, blocking execution"，**等同 block 但 reason 不可控**。

| 事件 | try/catch？ | handler 异常后果 |
|------|-----------|----------------|
| `before_agent_start` / `message_end` / `tool_result` / `user_bash` | ✅ 有 | emitError，**不影响 agent** |
| `agent_start` / `agent_end` / `turn_*` / `message_start` 等通用事件 | ✅ 有 | emitError，**不影响 agent** |
| `tool_call` | ❌ **无** | 冒泡到外层 → 等同 block（reason 不可控） |

**最佳实践**：即便生命周期钩子的异常不会中断 agent，也建议在 handler 内部包 try/catch——方便自定义日志、避免 emitError 刷屏。

### 陷阱 3：`before_agent_start` 是扩展独有事件（subscribe 静默失败）

server 层 `session.subscribe("before_agent_start", ...)` **永远不会被触发**，且不报错不警告。想在 server 层感知 agent 启动只能订阅 `agent_start`（两层都派发，但 payload 只有 `{ type }`，拿不到 systemPrompt / prompt / images）。

证据：agent-session.ts、types.ts。

6 个扩展独有事件：`before_agent_start` / `context` / `tool_call` / `tool_result` / `input` / `model_select`。详见 [04-events.md 坑 4](../sdk_doc/04-events.md#坑-46-个事件-subscribe-静默收不到-最大集成坑)。

### 陷阱 4：`message_end` 替换消息 role 必须一致

`emitMessageEnd`（runner.ts）会校验 `handlerResult.message.role !== currentMessage.role`——不一致就 emitError 并丢弃，原消息不变。

```ts
// ❌ 错：role 不一致，会被丢弃
pi.on("message_end", (event, ctx) => {
  if (event.message.role === "assistant") {
    return { message: { role: "user", content: "..." } };  // ← 会 emitError
  }
});

// ✅ 对：保持 role
pi.on("message_end", (event, ctx) => {
  if (event.message.role === "assistant") {
    return { message: { ...event.message, content: processed } };
  }
});
```

### 陷阱 5：`agent_end` 扩展层无 `willRetry`，重试感知盲区

SDK 默认开启自动重试（5xx 如 500/502/503/504/524、429 / overloaded / rate limit / too many requests、各类网络与流中断错误如 connection refused / fetch failed / socket hang up / stream 提前结束 / WebSocket 关闭，以及 gRPC `ResourceExhausted` 等触发，完整清单见 `RETRYABLE_PROVIDER_ERROR_PATTERN`，指数退避 `baseDelayMs * 2^(attempt-1)`，默认 2s→4s→8s）。重试时会 emit 一个**全新的 `agent_start`**，扩展层完全感知不到这是重试。

- **subscribe 层**：`agent_end` payload 含 `willRetry: boolean`，并有 `auto_retry_start` / `auto_retry_end` 事件
- **扩展层**：`agent_end` payload 被 `_emitExtensionEvent` 重新构造为 `{ type, messages }`，**主动丢弃 `willRetry`**；`auto_retry_*` 事件也**只在 subscribe 层派发**

**后果**：任何按 agent_start / end 建账的扩展逻辑（如 trace 表、计费表）会被一次用户提问的重试拆成 N 份。

**对策**：扩展层只能从 `agent_end.messages` 里找最后一条 assistant，**自己跑一遍 SDK 的可重试正则判定**（与 `_isRetryableError` 对齐）。详见 [04-events.md 坑 5](../sdk_doc/04-events.md#坑-5扩展层感知不到重试)。

### 陷阱 6：不要把 `agent_end` 当成流程结束的唯一信号

在某些执行路径下（诱因未完全定位），所有 `message_end` 已正常触发，但 `agent_end` 事件**始终不发出**——外层 `await session.prompt(message)` 也不 resolve，HTTP/SSE 连接挂住。

> **推荐**：使用 `agent_settled` 事件（agent-session.ts）作为「Agent 完全稳定」的信号。`agent_end` 在 retry 场景下会提前触发，而 `agent_settled` 只在所有 retry / compaction / steer 队列消费完毕后才派发（两层都派发），是坑 1 的官方解药。

**对策**：

```ts
// 方案 A（推荐）：扩展层直接用 agent_settled 作为完成信号
// ⚠️ agent_settled 被派发方 await，handler 里只能做轻量收尾；
//    重活（落库 / 二次 LLM 调用）要么 fire-and-forget，要么改用 session.subscribe 落库
pi.on("agent_settled", (event, ctx) => {
  // 此时所有 retry / compaction / steer 队列都已消费完毕
  sendDone();  // 轻量收尾
});

// 方案 B（subscribe 层兜底）：不依赖 agent_end / agent_settled 任一事件，
// 而是用 try/finally + subscribe 双保险，适合 server 层无法写扩展的场景
let doneSent = false;
const unsubscribe = session.subscribe((event) => {
  if (event.type === "agent_end" && !doneSent) {
    doneSent = true;
    sendDone();
  }
});
try {
  await session.prompt(message);
} finally {
  if (!doneSent) {
    doneSent = true;
    sendDone();  // ← 即便 agent_end 未触发也要结束
  }
  unsubscribe();
}
```

证据与案例见 [04-events.md 坑 1](../sdk_doc/04-events.md#坑-1agent_end-不可靠不一定触发-retry-时多次触发)。

---

## 变体与延伸

- **改系统提示词（注入运行时规则）** → 见上方「模式一」
- **替换最终消息（脱敏 / 加水印）** → 见上方「模式二」
- **基于 session 状态的动态规则** → `session_start` 事件初始化闭包变量，`turn_start` / `turn_end` 里使用（详见 [E01 变体 F](E01-tool-intercept.md#变体-f基于-session-状态的动态规则)）
- **配合 `session.steer()` 在 turn 启动时注入外部消息** → 见 [F05-steer-session.md](F05-steer-session.md)
- **会话结束时自动 git commit** → git hook 专属，本 skill 不展开
- **UI 状态栏展示生命周期** → CLI 专属，需加 `ctx.hasUI` 守卫（见陷阱 1）
- **`message_update` 流式渲染 / 变换** → [E06-streaming-transform.md](E06-streaming-transform.md)（逐 token 高频事件）
- **工具执行过程跟踪** → 用 `tool_execution_start` / `tool_execution_end`，不是 `turn_end`（粒度更细）

---

## 横向联动

- [E01 工具拦截](E01-tool-intercept.md)：`tool_call` 钩子（block / 改参）
- [E02 扩展基础](E02-extension-basics.md)：扩展三件套（hook + tool + command）
- [E05 输入变换](E05-input-transform.md)：`input` 事件（变换 / 拦截用户输入）
- [E06 流式变换](E06-streaming-transform.md)：`message_update` 流式逐 token 处理
- [F01 会话持久化](F01-session-persistence.md)：`session_shutdown` + 持久化策略
- [F05 会话 steer](F05-steer-session.md)：配合 `turn_start` 注入外部消息
- [sdk_doc/04-events.md](../sdk_doc/04-events.md)：完整事件清单 + 扩展独有 vs 双层派发分类 + 三层 payload 差异
- [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md)：ExtensionAPI 完整接口
- [sdk_doc/02-agent-session.md §subscribe](../sdk_doc/02-agent-session.md)：外部层事件订阅（17 种事件清单）
