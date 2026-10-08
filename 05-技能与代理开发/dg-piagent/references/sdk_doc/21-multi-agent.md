# Multi-Agent -- 多智能体架构模式

## 概述

Multi-Agent 是在 pi-agent 上构建**多智能体系统**的架构模式集合。pi-agent 本身不提供内置的 multi-agent 抽象，而是通过其灵活的 session、fork、extension 和 RPC 机制，支持多种多 agent 协作模式。

核心价值场景：

- **任务分解**：将大型任务拆分为多个子任务，交由不同 agent 处理
- **专业化分工**：不同 agent 使用不同的 system prompt、tools、model 配置
- **并行加速**：多个独立任务同时执行，缩短总体时间
- **审查流水线**：一个 agent 生成代码，另一个 agent 审查，第三个 agent 修复
- **上下文隔离**：每个 agent 有独立的上下文窗口，避免上下文污染

---

## 模式一：多 Session 实例

### 原理

在同一个 Node.js 进程中创建多个独立的 `AgentSession` 实例。每个 session 拥有独立的模型、工具、系统提示词和消息历史。

### 适用场景

- 需要并行处理多个独立任务
- 主控程序需要同时运行多个 agent
- 共享进程资源（内存、文件句柄）

### 示例

```ts
import { createAgentSessionRuntime } from "@earendil-works/pi-coding-agent"

// 创建两个独立的 runtime
const runtime1 = await createAgentSessionRuntime(factory, {
  cwd: "/project/frontend",
  agentDir: agentDir,
  sessionManager: SessionManager.create("/project/frontend"),
})

const runtime2 = await createAgentSessionRuntime(factory, {
  cwd: "/project/backend",
  agentDir: agentDir,
  sessionManager: SessionManager.create("/project/backend"),
})

// 两个 agent 独立工作
await Promise.all([
  runtime1.session.prompt("优化 React 组件性能"),
  runtime2.session.prompt("优化 API 端点性能"),
])
```

> **`factory` 是什么？** `createAgentSessionRuntime` 的第一个参数是一个工厂函数 `CreateAgentSessionRuntimeFactory`，签名为 `(options: { cwd, agentDir, sessionManager, sessionStartEvent?, projectTrustContext? }) => Promise<CreateAgentSessionRuntimeResult>`。它负责根据 cwd + sessionManager 创建完整的 AgentSession。通常在应用启动时创建一次，然后传给所有 runtime 调用。完整流程见 [01-create-agent-session.md](01-create-agent-session.md) 和 [03-agent-session-runtime.md](03-agent-session-runtime.md)。

### 注意事项

- 每个 session 的 `sessionManager` 管理独立的会话树
- 需要手动管理 session 生命周期和清理
- 不能在同一进程中让两个 session 共享同一个 session 文件

---

## 模式二：Session Runtime 切换

### 原理

利用 `AgentSessionRuntime` 的 `switchSession()`、`newSession()`、`fork()` 方法，在**同一进程内**切换不同的 Agent Session 上下文。

### 适用场景

- 用户手动在不同会话之间切换
- 从当前会话分叉到新分支继续工作
- 创建新会话并从旧会话带上下文（handoff）

### AgentSessionRuntime 核心方法

```ts
class AgentSessionRuntime {
  // 切换到指定会话文件
  switchSession(sessionPath: string, options?: {
    cwdOverride?: string
    withSession?: (ctx: ReplacedSessionContext) => Promise<void>
    projectTrustContextFactory?: (cwd: string) => ProjectTrustContext
  }): Promise<{ cancelled: boolean }>

  // 创建新会话
  newSession(options?: {
    parentSession?: string
    setup?: (sessionManager: SessionManager) => Promise<void>
    withSession?: (ctx: ReplacedSessionContext) => Promise<void>
  }): Promise<{ cancelled: boolean }>

  // 从指定 message fork 新分支
  fork(entryId: string, options?: {
    position?: "before" | "at"
    withSession?: (ctx: ReplacedSessionContext) => Promise<void>
  }): Promise<{ cancelled: boolean; selectedText?: string }>

  // 导入 JSONL 会话文件
  importFromJsonl(inputPath: string, cwdOverride?: string): Promise<{ cancelled: boolean }>
}
```

### Server 集成关键方法

在 server 集成场景（如 SSE 流式服务）中，session 切换后需要重新绑定订阅。`AgentSessionRuntime` 提供两个方法：

```ts
// session 替换后自动重新绑定（如 subscribe / extension 事件）
runtime.setRebindSession(async (newSession) => {
  newSession.subscribe(handleEvent)
})

// session 销毁前的同步清理（如卸载 TUI 组件）
runtime.setBeforeSessionInvalidate(() => {
  cleanupUI()
})
```

- `setRebindSession(fn)`: 每次 `switchSession`/`newSession`/`fork` 成功后调用，传入新 session
- `setBeforeSessionInvalidate(fn)`: 在 `session_shutdown` 事件之后、旧 session dispose 之前同步调用

### 示例

```ts
// 在扩展中使用 session 切换
pi.registerCommand("handoff", {
  handler: async (args, ctx) => {
    // 创建新会话，parentSession 可追溯来源
    const result = await ctx.newSession({
      parentSession: ctx.sessionManager.getSessionFile(),
      withSession: async (newCtx) => {
        newCtx.ui.setEditorText(generatedPrompt)
      },
    })
  },
})
```

### 注意事项

- `switchSession` 会触发 `session_before_switch` 扩展事件（reason: `"resume"`，可被扩展取消）
- `newSession` 也会触发 `session_before_switch` 扩展事件（reason: `"new"`，可被扩展取消）
- `fork` 会触发 `session_before_fork` 扩展事件
- 切换前后会调用 `session_shutdown` -> `session_start` 生命周期事件
- 所有切换操作会先 teardown 当前 session（dispose），再 apply 新 session

---

## 模式三：Subagent 工具

### 原理

通过扩展注册 `subagent` 工具，让主 agent 在对话中调用子 agent 执行子任务。子 agent 是**独立的 `pi` 进程** (`pi --mode json -p --no-session`)。

### 三种执行模式

| 模式 | 参数 | 说明 |
|------|------|------|
| Single | `{ agent, task }` | 一个 agent 执行一个任务 |
| Parallel | `{ tasks: [...] }` | 多个 agent 并行执行（最多 8 个，4 并发） |
| Chain | `{ chain: [...] }` | 顺序执行，`{previous}` 占位符引用上一步输出 |

### Agent 配置文件格式

```markdown
---
name: scout
description: Fast codebase reconnaissance
tools: read, grep, find, ls, bash
model: claude-haiku-4-5
---

你是一个代码库侦察员。快速找到相关文件，返回结构化的发现。
```

### 查找位置

- `~/.pi/agent/agents/*.md` -- 用户级 agent（默认启用）
- `.pi/agents/*.md` -- 项目级 agent（需显式启用 `agentScope: "project"` 或 `"both"`）
- `agentScope: "both"` 时，同名的 project agent 会覆盖 user agent

### 示例

```ts
// 通过 tool call 使用 subagent
// 单 agent
{ agent: "scout", task: "Find all authentication-related code" }

// 并行
{
  tasks: [
    { agent: "scout", task: "Find all API route definitions" },
    { agent: "scout", task: "Find all database model definitions" },
  ]
}

// 链式
{
  chain: [
    { agent: "scout", task: "Find the authentication module" },
    { agent: "planner", task: "Based on:\n{previous}\n\nSuggest improvements" },
  ]
}
```

### Subagent 实现细节

源码参考：`packages/coding-agent/examples/extensions/subagent/`

核心机制：
1. 主 agent 调用 `subagent` 工具
2. 扩展中 `spawn("pi", ["--mode", "json", "-p", "--no-session", ...])` 启动子进程
3. 子进程通过 `--append-system-prompt` 加载 agent 的 system prompt
4. 通过 JSONL stdout 流式收集子 agent 的消息和工具调用
5. 聚合所有结果返回给主 agent

安全模型：
- 项目级 agent 可读取文件、执行 bash 命令，仅限受信任的仓库使用
- `agentScope` 默认为 `"user"`，不加载项目 agent
- 交互模式下，`confirmProjectAgents: true`（默认）会在运行项目 agent 前弹出确认

---

## 模式四：Handoff 上下文转移

### 原理

Handoff 不是简单的上下文压缩，而是**生成一个自包含的新提示词**，在新会话中继续工作。它比 compaction 更彻底：不是摘要旧历史嵌入上下文，而是重新构造一个清晰的、面向新任务的提示词。

### 与 Compaction 的区别

| 维度 | Compaction | Handoff |
|------|-----------|---------|
| 方式 | 摘要旧消息插入当前上下文 | 生成新 prompt 创建新 session |
| 上下文 | 仍然有上下文窗口压力 | 全新上下文窗口 |
| 可编辑性 | 不可编辑（自动） | 用户可编辑生成的 prompt |
| 适用性 | 长对话自动压缩 | 任务切换、重新聚焦 |

### 示例

```ts
import type { AgentMessage } from "@earendil-works/pi-agent-core"
import type { SessionEntry } from "@earendil-works/pi-coding-agent"
import { convertToLlm, serializeConversation } from "@earendil-works/pi-coding-agent"

// 参考 handoff.ts 扩展实现（packages/coding-agent/examples/extensions/handoff.ts）
// 注意：真实实现会检查 ctx.mode !== "tui" 并报错退出——仅限交互模式用

// ⚠️ 关键：getBranch() 返回 SessionEntry[]（联合类型，含 message/compaction/modelChange
//    等多种条目），不能直接喂 convertToLlm()（形参是 AgentMessage[]）。
//    必须先把 SessionEntry 映射成 AgentMessage，并处理 compaction 重组。
function entryToMessage(entry: SessionEntry): AgentMessage | undefined {
  if (entry.type === "message") return entry.message
  if (entry.type === "compaction") {
    return {
      role: "compactionSummary",
      summary: entry.summary,
      tokensBefore: entry.tokensBefore,
      timestamp: new Date(entry.timestamp).getTime(),
    }
  }
  return undefined // modelChange 等非消息条目跳过
}

// 若分支被 compact 过，直接把整条 branch 喂给 LLM 会带上已被压缩的旧消息，
// 反而漏掉 compaction summary。正确做法：找到最后一个 compaction，
// 重组为 [compactionSummary, ...(firstKept..compaction), ...(compaction..end)]。
function getHandoffMessages(branch: SessionEntry[]): AgentMessage[] {
  let compactionIndex = -1
  for (let i = branch.length - 1; i >= 0; i--) {
    if (branch[i].type === "compaction") { compactionIndex = i; break }
  }
  if (compactionIndex < 0) {
    return branch.map(entryToMessage).filter((m): m is AgentMessage => m !== undefined)
  }
  const compaction = branch[compactionIndex]
  const firstKeptIndex =
    compaction.type === "compaction"
      ? branch.findIndex((e) => e.id === compaction.firstKeptEntryId)
      : -1
  const compactedBranch = [
    compaction,
    ...(firstKeptIndex >= 0 ? branch.slice(firstKeptIndex, compactionIndex) : []),
    ...branch.slice(compactionIndex + 1),
  ]
  return compactedBranch.map(entryToMessage).filter((m): m is AgentMessage => m !== undefined)
}

pi.registerCommand("handoff", {
  handler: async (args, ctx) => {
    const goal = args.trim()

    // 1. 收集当前分支的对话历史（经 compaction 重组），转成 LLM 格式
    const messages = getHandoffMessages(ctx.sessionManager.getBranch())
    const llmMessages = convertToLlm(messages)
    const conversationText = serializeConversation(llmMessages)

    // 2. 通过 ctx.modelRegistry（兼容包装器）获取认证信息，再调 complete()
    // 注：ctx.modelRegistry 是 ModelRegistry（兼容包装器），内部委托 ModelRuntime
    // 声明位置：extensions/types.ts:319；实现：model-registry.ts:31-32
    const auth = await ctx.modelRegistry.getApiKeyAndHeaders(ctx.model!)
    if (!auth.ok) {
      ctx.ui.notify("Failed to get API key", "error")
      return
    }
    const response = await ctx.modelRegistry.complete(ctx.model!, {
      systemPrompt: "你是一个上下文转移助手...",
      messages: [{
        role: "user",
        content: [{ type: "text", text: `## 对话历史\n\n${conversationText}\n\n## 新任务目标\n\n${goal}` }],
        timestamp: Date.now(),
      }],
    }, { apiKey: auth.apiKey, headers: auth.headers, env: auth.env })
    const prompt = response.content.filter(c => c.type === "text").map(c => c.text).join("\n")

    // 3. 让用户编辑
    const edited = await ctx.ui.editor("Edit handoff prompt", prompt)

    // 4. 创建新 session 并设置 editor 内容
    await ctx.newSession({
      parentSession: ctx.sessionManager.getSessionFile(),
      withSession: async (newCtx) => {
        newCtx.ui.setEditorText(edited)
      },
    })
  },
})
```

> 完整实现见 `packages/coding-agent/examples/extensions/handoff.ts`，包含加载器 UI、错误处理等。上面 `getHandoffMessages()` / `entryToMessage()` 即取自该文件（handoff.ts:42-78）。

---

## 模式五：Fork 分支探索

### 原理

`AgentSessionRuntime.fork()` 允许从会话历史中的某个 message entry 分叉出一条新分支，在新 session 中探索替代方案。

### 位置参数

- `"before"`：从选中消息**之前**开始新分支（默认）。注意：选中的 entry 必须是 user message，否则会抛错
- `"at"`：从选中消息的位置开始（可用于以当前状态为基础）

### 示例

```ts
// 从某个 user message 之前分叉
const result = await runtime.fork("entry-id-123", {
  position: "before",
  withSession: async (ctx) => {
    ctx.ui.notify("Forked new branch. Try a different approach.", "info")
  },
})
// result.selectedText -- 选中消息的文本内容
```

---

## 同进程多 Session 通信方式

> ⚠️ **适用范围**：本节方法适用于**同一进程内的多个 `AgentSession`**（即「模式一：多 Session 实例」和「模式二：Runtime 切换」的 session 之间）。
>
> **不适用于 subagent**：subagent 是 `spawn` 出的独立 `pi` 子进程（见「模式三」），只能通过 task 输入/输出交互、通过 `AbortSignal` 中止——**无法**对子进程 steer / followUp / subscribe。

选择指南：

| 方式 | 适用场景 | 特点 |
|------|---------|------|
| 消息传递 (`sendUserMessage`) | 单向通知——Agent A 完成工作后告知 Agent B | 简单直接，但无返回值 |
| 自定义消息 (`sendCustomMessage`) | 结构化通信——传递非文本消息（自定义类型 + 内容） | 支持 `deliverAs: "steer" \| "followUp" \| "nextTurn"` |
| Event Bus (`pi.on` / `session.subscribe`) | 松耦合协调——Agent A 完成后自动触发 Agent B 动作 | 解耦好，但事件不跨进程（同进程内多 session） |
| Steering / FollowUp | **同进程多 session 的实时干预**——Agent A 运行中，Agent B/主控向其消息队列插队 | 可中断/重定向正在运行的 session |

> **跨层方法名差异**：`sendCustomMessage` 是 **session 外部层**（`AgentSession`）的方法；在**扩展层**（`ExtensionContext` / `ReplacedSessionContext`）上，等价方法叫 `sendMessage`（`agent-session.ts:3317` 把 `ctx.sendMessage` 委托给 `session.sendCustomMessage`）。同理 `sendUserMessage` 在两层同名。在扩展里写 `ctx.sendCustomMessage` 会找不到方法——应写 `ctx.sendMessage`。

Server 集成场景更常用 `session.subscribe`（外部层）而非扩展层的 `pi.on`。

### 通过消息传递

```ts
// 在一个 session 中通过 sendUserMessage 发送消息
// 签名: sendUserMessage(content, options?)
// - content: string | (TextContent | ImageContent)[]
// - options.deliverAs: "steer" | "followUp"  —— agent 正在 streaming 时必需
runtime1.session.sendUserMessage("Agent 2 has finished analyzing the database schema.")
// agent 正在运行时可指定排队策略：
runtime1.session.sendUserMessage("Urgent: stop and check this", { deliverAs: "steer" })
```

### 通过 Event Bus

```ts
// 扩展可以监听 session 事件并协调多个 session
// 推荐 agent_settled（v0.80.4+）：所有 retry/compaction/queue 处理完才触发，
// 比 agent_end 更可靠——agent_end 在每轮对话结束时都会触发，
// 而 agent_settled 表示 agent 真正"安定"下来，不会再继续行动。
// 注：session 外部层（subscribe）的 agent_end 比 core/扩展层多一个
// `willRetry: boolean` 字段（agent-session.ts:144-147），用于判断本轮是否
// 会自动重试；若不想在 retry 间隙触发动作，就用 agent_settled 或检查 willRetry。
pi.on("agent_settled", (event) => {
  // Agent 1 彻底完成了（含所有 retry/compaction），通知 Agent 2
  runtime2.session.steer("Agent 1 completed. Here is the result: ...")
})
```

### 通过 Steering / FollowUp

```ts
// 同进程内：主控程序向某个 session 的消息队列插队
// （steer：打断当前 streaming；followUp：等 agent 空下来再投递）
await session1.steer("Stop current work, switch to task B")
await session2.followUp("Agent 1 says: the fix is in src/auth.ts")
```

> `steer()`/`followUp()` 操作的是**当前 `AgentSession` 内部 agent 的消息队列**（`agent-session.ts:1334/1354`，最终委托 `this.agent.steer()`），只对**同进程内**的 session 有效。**subagent 是独立子进程，不能用这两个方法干预**——只能通过传入的 `AbortSignal` 中止（`subagent/index.ts:399-408`）。

---

## 关键细节与陷阱

### 1. Session 隔离性

每个 session 是完全独立的：
- 独立的模型配置
- 独立的工具注册
- 独立的 system prompt
- 独立的会话树（session tree）
- 独立的消息历史

它们不共享任何状态，除非你显式地在代码中传递数据。

### 2. getBranch() 返回 SessionEntry[]，不能直接传 convertToLlm()

这是 handoff / 自定义上下文转移场景最容易踩的坑（H06 类教训）：

- `ctx.sessionManager.getBranch()` 返回 **`SessionEntry[]`**（`session-manager.ts:1260`）。`SessionEntry` 是**联合类型**，除了 `message` 条目，还含 `compaction` / `modelChange` 等非消息条目。
- `convertToLlm()` 的形参是 **`AgentMessage[]`**（`messages.ts:148`），二者类型不匹配，直接 `convertToLlm(branch)` 编译不过。
- 正确做法：先 `SessionEntry → AgentMessage`（`entry.type === "message"` 取 `entry.message`，`compaction` 转 `compactionSummary`，其余跳过），再 `convertToLlm`。
- **若分支被 compact 过**，还要按最后一个 compaction 的 `firstKeptEntryId` 重组为 `[compactionSummary, ...(firstKept..compaction), ...(compaction..end)]`，否则会带上已被压缩的旧消息、漏掉 compaction 摘要。
- 现成实现见 `examples/extensions/handoff.ts:42-78` 的 `entryToMessage()` + `getHandoffMessages()`。

### 3. 资源清理

创建的 session 需要手动 dispose。`AgentSessionRuntime` 在切换时自动 dispose 旧 session，但如果创建了不被 runtime 管理的独立 session，需要显式调用 `session.dispose()`。

`teardownCurrent` 的完整序列：先调用 `session.abort()` settle 活跃 response（确保 tool results 等持久化到旧 session），然后 emit `session_shutdown`，再调用 `beforeSessionInvalidate` 回调，最后 `session.dispose()`。

### 4. Fork 前提条件

`fork()` 在持久化和非持久化 session 上都能工作，但行为不同：

- **持久化 session**（`isPersisted()` 返回 `true`）：通过 `SessionManager.open()` 打开会话文件，用 `createBranchedSession()` 创建物理分支。fork 出的分支存储在同一 session 文件中。
- **非持久化 session**（内存模式）：直接在当前 `SessionManager` 上调用 `newSession()` 或 `createBranchedSession()`，不涉及文件 I/O。

两种路径都在同一 session 树内创建分支。非持久化 session 可用 `SessionManager.inMemory(cwd)` 构造（`session-manager.ts:1568`，不落盘）。

### 5. Subagent 的限制

- 并行模式最多 8 个任务，4 个并发
- 每个任务的输出上限 50KB（超出部分截断，但完整结果保留在 tool details 中）
- Subagent 不继承主 session 的上下文，只接收 task 描述
- Subagent 使用 `--no-session` 标志，不保存到磁盘
- Subagent 是独立进程，**不能**对它 steer / followUp / subscribe——只能通过 task 输入/输出交互、通过 `AbortSignal` 中止（见上方「同进程多 Session 通信方式」的适用范围说明）

### 6. Subagent 的 `cwd` 字段与失败语义

- **`cwd` 字段**：single / parallel（`tasks[]`）/ chain（`chain[]`）的每一项都支持可选 `cwd`（子进程工作目录），不指定时默认用主 `ctx.cwd`（`subagent/index.ts:335-336`、schema `subagent/index.ts:434/440/457`）。
- **失败语义**：
  - **parallel**：单个任务失败**不影响**其他任务，所有任务的结果都会返回，失败项标注 `failed (stopReason)`（`subagent/index.ts:647-664`）。
  - **chain**：某步失败则**整链停止**，返回已完成的步骤结果并标注 `Chain stopped at step N`（`subagent/index.ts:566-573`）。

### 7. Chain 模式的 `{previous}` 占位符

chain 中每个步骤的 task 字符串中的 `{previous}` 会被替换为上一步 assistant 的文本输出。如果上一步失败，chain 会停止并报告失败步骤。

### 8. 项目级 agent 安全

项目级 agent (`.pi/agents/*.md`) 是 repo 控制的 prompt，可能指示模型读取文件、执行命令。只对受信任的仓库启用 `agentScope: "project"` 或 `"both"`。

> **`agentScope` 三档**：`"user"`（默认，仅用户级）、`"project"`（仅项目级，不含用户级）、`"both"`（两者都加载，同名 project 覆盖 user）。加载逻辑见 `subagent/agents.ts:97-116`。
