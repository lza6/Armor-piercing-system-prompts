# 场景：多 Agent 协作 (H06)

## 这是什么 / 不是什么

**是**：在同一个 Node.js 进程里用 `createAgentSession` / `AgentSessionRuntime` / 扩展机制让**多个 Agent 实例协同工作**的工程模式集合。本场景覆盖 5 种已被社区验证的协作模式：

1. **多 Session 并行**（任务级隔离）
2. **Session Runtime 切换**（顺序协作，共享 runtime）
3. **Subagent 工具**（主 Agent 调度子 Agent）
4. **Handoff 上下文转移**（生成自包含 prompt 接力）
5. **Fork 分支探索**（在同一会话树里探索替代方案）

**不是**：
- **不是分布式多进程**：本场景所有 Agent 都在**同一个 Node.js 进程**里。要跨进程协调（如 worker / cluster）请用消息总线（Redis Pub/Sub / NATS）+ 每个 worker 内独立 session。
- **不是改 Agent loop**：每个 session 内部仍然是 SDK 固定的 `LLM 调用 → 工具执行 → 循环`。要改 loop 行为用 [扩展层 hooks](../sdk_doc/07-extensions-api.md)（`before_agent_start` / `context` / `tool_call`）。
- **不是 RPC 多用户服务**：每个用户一个 session 的服务架构见 [sdk_doc/21 多 Agent 架构](../sdk_doc/21-multi-agent.md)，本场景聚焦"一个逻辑任务用多个 Agent 完成"。
- **不是无状态并行**：每个 session 都有内存开销（messages 历史在 AgentState 里），不调 `dispose()` 会**进程级泄漏**（agent 内部 subscribe 不解绑，见陷阱 #4）。

## 什么时候用 / 不用会怎样

| 触发场景 | 用什么模式 | 不用会怎样 |
|---------|-----------|-----------|
| 多个独立子任务要并行（如批量翻译 10 个文件） | 模式 1 多 Session 并行 | 顺序执行总时长 = N × 单任务时长 |
| 主 Agent 运行中需要专业子 Agent（如审查、规划） | 模式 3 Subagent 工具 | 把所有职责塞一个 system prompt，模型在多任务间漂移 |
| 用户主动切换会话（CLI `/resume`、`/new`、`/fork`） | 模式 2 Session Runtime | 自己拼 dispose → 重建 → bindExtensions，漏一步就泄漏 |
| 任务太大需"重新启动"一个干净的上下文 | 模式 4 Handoff | 老对话历史越长 token 越贵，模型还会被旧上下文误导 |
| 在某个决策点想试两种方案 | 模式 5 Fork | 只能跑一个方案，跑完再回溯——浪费时间 |
| 简单问答 / 单轮任务 | **不需要本场景**，直接用 [A01](A01-minimal-startup.md) | 引入多 Agent 抽象 = 增加复杂度无收益 |
| 跨用户隔离 | **不需要本场景**，每个用户独立进程 / 容器 | 多 session 共享 agentDir，会读到对方的 auth.json（见陷阱 #1）|

## ⚠️ 最大陷阱：默认共享 agentDir，不是"完全独立"

**事实**：`createAgentSession()` 不传 `agentDir` 时，所有 session **默认共享 `~/.pi/agent` 目录**——也就是**共享同一个 `auth.json` + `models.json` + `agents/` 配置**。sdk 写得很清楚：

```ts
const agentDir = options.agentDir ? resolvePath(options.agentDir) : getDefaultAgentDir();
//                                                                      ↑ ~/.pi/agent
```

后续组件都从这个 agentDir 读（签名）：

```ts
const modelRuntime = options.modelRuntime ?? (await ModelRuntime.create({ authPath, modelsPath }));
// ← 共享 auth.json + models.json（ModelRuntime 内部封装了 AuthStorage + ModelRegistry）
// ⚠️ 注意：未传 agentDir 时 authPath/modelsPath 为 undefined，
//    ModelRuntime.create 内部 fallback 到 getAgentDir()（即 ~/.pi/agent）读默认文件——
//    所以"不传 agentDir"等价于"共享 ~/.pi/agent 配置"。
//    但实例本身仍是每次 new 一个（见下表 modelRuntime 行）。
```

**这意味着**：

| 项 | 默认共享？ | 后果 |
|----|-----------|------|
| `auth.json` / `models.json`（**文件**） | ✅ 共享 | 多 session 读同一份文件 → quota 算同一账号、API key 互相覆盖。未传 `agentDir` 时 `authPath/modelsPath` 为 `undefined`，`ModelRuntime.create` 内部 fallback 到 `getAgentDir()` 读 `~/.pi/agent` 默认文件 |
| `modelRuntime`（**内存实例**） | ❌ 独立 | 每次 `createAgentSession` 默认 new 一个独立实例（`options.modelRuntime ?? await ModelRuntime.create(...)`）。**内存态 `registerProvider` / `registerModel` 不跨实例可见**——session A 注册的 provider，session B 的实例看不到（除非 B 重新 reload 读文件，或显式传同一实例）。要让多 session 共享实例，必须显式传同一 `options.modelRuntime` |
| `settingsManager`（settings.json） | ❌ 独立（cwd 不同） | 每 cwd 一份，但都从 agentDir 读全局 settings |
| `sessionManager`（消息历史） | ❌ 独立 | 不同 cwd 编码到不同 sessionDir |
| `AgentState`（messages、tools） | ❌ 独立 | 各自的 AgentLoop，互不干扰 |
| `subscribe` 监听器 | ❌ 独立（绑在 AgentSession 实例上） | 切换 session 后旧订阅不会自动迁移（见陷阱 #2）|
| 扩展实例（`pi.registerTool` 注册的工具） | ❌ 独立（每 session bindExtensions 一次） | 注册的工具不跨 session 共享 |

**正确做法**：要真正隔离，传独立 `agentDir`（或更彻底地传独立 `modelRuntime` 实例）：

```ts
import { join } from "node:path";
import { homedir } from "node:os";

const { session: agent1 } = await createAgentSession({
  cwd: "/project/frontend",
  agentDir: join(homedir(), ".pi", "agent-frontend"),  // ← 独立 agentDir
});
const { session: agent2 } = await createAgentSession({
  cwd: "/project/backend",
  agentDir: join(homedir(), ".pi", "agent-backend"),  // ← 独立 agentDir
});
```

如果只想共享配置但隔离 cwd / 消息，**不传 agentDir 即可**（默认共享）。多数应用场景（一个进程内多 Agent 协作同一项目）共享 agentDir 是**期望行为**——避免每个 session 重复加载 auth/models。

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `createAgentSession(options)` | 创建独立 session 实例。不传 `agentDir` 默认共享 `~/.pi/agent` | [sdk_doc/01-create-agent-session.md](../sdk_doc/01-create-agent-session.md) |
| `createAgentSessionRuntime(factory, options)` | 创建 runtime（工厂存储复用，每次切换重调用） | [sdk_doc/03-agent-session-runtime.md](../sdk_doc/03-agent-session-runtime.md) |
| `runtime.switchSession(path, options?)` | 切换到指定历史会话文件 | [sdk_doc/03-agent-session-runtime.md](../sdk_doc/03-agent-session-runtime.md) |
| `runtime.newSession(options?)` | 新建会话（含 `parentSession` / `setup` / `withSession`） | [sdk_doc/03-agent-session-runtime.md](../sdk_doc/03-agent-session-runtime.md) |
| `runtime.fork(entryId, options?)` | 从 entry 分叉新分支（`position: "before" \| "at"`） | [sdk_doc/03-agent-session-runtime.md](../sdk_doc/03-agent-session-runtime.md) |
| `runtime.setRebindSession(cb?)` | 注册 rebind 回调，session 替换后自动调用 | [sdk_doc/03-agent-session-runtime.md](../sdk_doc/03-agent-session-runtime.md) |
| `pi.registerTool(tool)` | 注册自定义工具（含 subagent） | [sdk_doc/06-tools.md](../sdk_doc/06-tools.md) |
| `pi.registerCommand(name, options)` | 注册命令（如 handoff）。命令 handler 收到的是 `ExtensionCommandContext`，**比 tool execute 多 `newSession` / `fork` / `switchSession` / `navigateTree` / `reload` 等方法** | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `session.subscribe(listener)` | 订阅 session 事件。返回 unsubscribe 函数。**绑在具体实例上，切换 session 后不会自动迁移** | [sdk_doc/04-events.md](../sdk_doc/04-events.md) |
| `session.sendUserMessage(content, options?)` | 给 session 发消息。`options.deliverAs: "steer" \| "followUp"` 控制 streaming 中的排队策略 | 本场景 §通信 |
| `session.steer(text)` / `session.followUp(text)` | 单向干预：steer 入队等下一轮消费、followUp 直接追加（不打断） | 本场景 §通信 |

> ⚠️ **关键边界**：扩展 `pi.registerTool` 的 execute 收到的是 `ExtensionContext`（types）——**没有** `newSession` / `fork` / `switchSession` / `waitForIdle`。只有 `pi.registerCommand` 的 handler 收到的是 `ExtensionCommandContext`（types）——才有这些方法。在 tool execute 里调 `ctx.newSession` 会编译报错或运行时 `undefined is not a function`。

## 五种模式

### 模式 1：多 Session 并行

**原理**：直接创建多个 `createAgentSession` 实例，用 `Promise.all` 并行。每个 session 有自己的 AgentState（messages/tools）和 subscribe 监听器，互不干扰。

**适用场景**：批量任务（多文件翻译、多 PR 审查）、主控程序同时跑多个独立 Agent。

```ts
import { createAgentSession, SessionManager, getAgentDir } from "@earendil-works/pi-coding-agent";

// 1. 显式定义 cwd——避免双 cwd 陷阱（详见 H01）
const frontendCwd = "/project/frontend";
const backendCwd = "/project/backend";

// 2. 共享 agentDir（共享 auth/models 配置）——多数场景的期望行为
const agentDir = getAgentDir();  // ← ~/.pi/agent

// 3. 分别创建 session——每个 cwd 独立编码到 sessionDir
const { session: agent1 } = await createAgentSession({
  cwd: frontendCwd,
  agentDir,                                              // 共享配置
  sessionManager: SessionManager.create(frontendCwd),    // ← 显式传 cwd，避免 inMemory 默认 process.cwd()
});
const { session: agent2 } = await createAgentSession({
  cwd: backendCwd,
  agentDir,
  sessionManager: SessionManager.create(backendCwd),
});

// 4. 各自独立 subscribe——监听器绑在具体 session 上
const unsubs = [
  agent1.subscribe((e) => { if (e.type === "message_update") process.stdout.write("[A1] "); }),
  agent2.subscribe((e) => { if (e.type === "message_update") process.stdout.write("[A2] "); }),
];

// 5. 并行执行——每个 session 的 AgentLoop 独立
try {
  await Promise.all([
    agent1.prompt("优化 React 组件性能"),
    agent2.prompt("优化 API 端点性能"),
  ]);
} finally {
  // 6. 显式 dispose——agent.subscribe 内部没解绑会进程挂起
  agent1.dispose();
  agent2.dispose();
  unsubs.forEach((u) => u());
}
```

**横向陷阱**：
- **双 cwd 陷阱**（见 [H01](H01-full-control.md)）：`SessionManager.create(cwd)` 必须显式传 cwd，否则默认 `process.cwd()`。inMemory 同理（`SessionManager.inMemory(cwd?)`）。
- **默认 agentDir 共享**：两个 session 都读同一份 `auth.json`。要完全隔离见上方"最大陷阱"节。
- **subscribe 不自动迁移**：两个 session 的订阅互不影响，但如果你在运行中切换 session，**旧 session 的 subscribe 不会迁移到新 session**（见模式 2）。

### 模式 2：Session Runtime 切换

**原理**：用 `AgentSessionRuntime` 在**同一进程内**切换不同 session——runtime 内部统一管理 dispose → 重建服务 → 创建新 session → 触发 `session_shutdown` 事件的完整流程，避免手工拼装遗留状态。

**适用场景**：用户主动切换会话（CLI `/resume`、`/new`、`/fork`）、handoff 后的接力。

```ts
import {
  createAgentSessionRuntime,
  createAgentSessionServices,
  createAgentSessionFromServices,
  SessionManager,
  getAgentDir,
  type AgentSession,
} from "@earendil-works/pi-coding-agent";

// 1. 工厂函数——会被存储，每次 newSession/switchSession/fork 都重新调用
const runtime = await createAgentSessionRuntime(
  async ({ cwd, agentDir, sessionManager, sessionStartEvent }) => {
    const services = await createAgentSessionServices({ cwd, agentDir });
    const result = await createAgentSessionFromServices({
      services,
      sessionManager,
      sessionStartEvent,
    });
    return { ...result, services, diagnostics: services.diagnostics };
  },
  {
    cwd: process.cwd(),
    agentDir: getAgentDir(),
    sessionManager: SessionManager.continueRecent(process.cwd()),
  },
);

// 2. ★ 关键：setRebindSession 注册回调，每次 session 替换后自动调用
//    扩展不会自动迁移——必须在这里 rebind + 重新 subscribe
let unsubscribe: (() => void) | undefined;
const extensionBindings = { /* 你的扩展绑定 */ } as any;

const rebind = async (session: AgentSession) => {
  await session.bindExtensions(extensionBindings);
  unsubscribe?.();                                          // 退订旧的
  unsubscribe = session.subscribe((e) => {
    if (e.type === "message_update" && e.assistantMessageEvent?.type === "text_delta") {
      process.stdout.write(e.assistantMessageEvent.delta);
    }
  });
};
runtime.setRebindSession(rebind);
await rebind(runtime.session);  // 首次手动触发一次

// 3. 各种切换操作
await runtime.newSession();                                    // 新建
await runtime.switchSession("/path/to/session.jsonl");         // 切换
await runtime.fork("entry-uuid", { position: "before" });      // 分叉
await runtime.importFromJsonl("/external/path.jsonl");         // 导入

// 4. 清理——dispose 是 async，必须 await
await runtime.dispose();
unsubscribe?.();
```

> 完整方法签名、事件列表、陷阱清单见 [F02](F02-session-runtime.md)。本场景只列协作时最关键的三件事：**rebind 机制、dispose 是 async、subscribe 不迁移**。

### 模式 3：Subagent 工具

**两种实现路径**，根据子 Agent 是否需要独立进程选择：

#### 3a：官方 subagent 机制（独立子进程）

**原理**：通过 `~/.pi/agent/agents/*.md` 配置文件声明专业子 Agent，主 Agent 在对话中 spawn 子进程 `pi --mode json -p --no-session` 执行任务。

**适用场景**：需要完全的上下文隔离、不希望子 Agent 共享主 Agent 的 cwd / 消息历史。

完整配置与示例见 [I05](I05-subagent.md) 和 [sdk_doc/21-multi-agent.md](../sdk_doc/21-multi-agent.md)。本场景不重复。

#### 3b：扩展内 `createAgentSession`（同进程子 Agent）

**原理**：在 `pi.registerTool` 的 execute 中**直接** `createAgentSession` 创建一个临时子 session，执行完任务后立即 dispose。

**适用场景**：不需要独立进程开销、主 Agent 与子 Agent 共享 cwd、子任务结果可直接聚合到主上下文。

```ts
import { defineTool, createAgentSession, SessionManager } from "@earendil-works/pi-coding-agent";
import { getBuiltinModel } from "@earendil-works/pi-ai/providers/all";
import { Type } from "@earendil-works/pi-ai";

export default (pi) => {
  pi.registerTool(defineTool({
    name: "delegate",
    description: "Delegate a self-contained task to a sub-agent (same process)",
    parameters: Type.Object({
      task: Type.String({ description: "Self-contained task description" }),
    }),
    async execute(toolCallId, params, signal, onUpdate, ctx) {
      // 1. 用父 Agent 的 cwd / agentDir / authStorage（共享配置）
      //    想完全隔离：传独立 agentDir + 独立 authStorage（见"最大陷阱"）
      const { session } = await createAgentSession({
        cwd: ctx.cwd,
        agentDir: undefined!,     // 共享默认 ~/.pi/agent
        model: ctx.model,         // 继承父 Agent 的模型
        tools: ["read", "bash"],  // 子 Agent 工具白名单
        sessionManager: SessionManager.inMemory(ctx.cwd),  // ← 显式传 cwd！
      });

      try {
        await session.prompt(params.task);
        // 2. 正确读取最后一条 assistant 消息——content 一定是数组
        const messages = session.state.messages;
        const lastAssistant = [...messages].reverse().find((m) => m.role === "assistant");
        const text = lastAssistant
          ? (lastAssistant.content as Array<{ type: string; text?: string }>)
              .filter((c) => c.type === "text").map((c) => c.text ?? "").join("\n")
          : "(no output)";

        return {
          content: [{ type: "text", text }],
          details: { callCount: messages.length },
        };
      } finally {
        // 3. 必须显式 dispose——agent.subscribe 内部订阅不释放会进程挂起
        session.dispose();
      }
    },
  }));
};
```

**横向陷阱**：
- ❌ **`SessionManager.inMemory()` 不传 cwd** → 双 cwd 陷阱（[H01](H01-full-control.md)），多 Agent 协作时 cwd 串到 process.cwd。必须 `SessionManager.inMemory(ctx.cwd)`。
- ❌ **不调 `session.dispose()`** → 进程级泄漏。tool execute 是 agent loop 同步等待的——子 session 不释放，主 Agent 会持续累积监听器。
- ❌ **`typeof lastMsg?.content === "string"`** → 类型不对。`AssistantMessage.content` 一定是数组（ai/types），UserMessage.content 也常是数组。要从数组里 filter text 块。
- ❌ **execute 里调 `ctx.newSession`** → 编译报错。`ExtensionContext`（types）没这个方法，只有 `ExtensionCommandContext` 才有。tool execute 收到的是前者。

#### Subagent 限制（仅适用于模式 3a 官方机制）

| 限制 | 值 | 说明 |
|------|------|------|
| 并行任务上限 | 8 | Parallel 模式最多同时调度 8 个 |
| 并发上限 | 4 | 同时执行不超过 4 个（其余排队） |
| 单任务输出上限 | 50KB | 超出部分截断（完整结果保留在 tool details） |
| 上下文继承 | 不继承 | 只接收 task 描述，不带主 session 历史 |
| 持久化 | `--no-session` | 子进程不写磁盘 |

> ⚠️ **这些值来自示例代码，不是 SDK 硬限**：`MAX_PARALLEL_TASKS=8` / `MAX_CONCURRENCY=4` / `PER_TASK_OUTPUT_CAP=50KB` 都是 `examples/extensions/subagent/index.ts` 的常量（`:33` / `:34` / `:36`），`--no-session` 也是该示例传给子进程的参数（`:294`）。**自建 subagent 工具可自由调整**这些阈值——SDK 并不强制。50KB 的截断阈值在 SDK 侧另有 `tools/truncate.ts:12` 的 `DEFAULT_MAX_BYTES`，是工具层默认值同样可改。

### 模式 4：Handoff 上下文转移

**原理**：Handoff **不是摘要**——而是用 LLM 生成一个**自包含的新 prompt**，在新会话中继续工作。它比 compaction 更彻底：不是把旧历史压缩嵌入当前上下文，而是重新构造一个清晰、面向新任务的 prompt。

**与 Compaction 的区别**（[sdk_doc/21](../sdk_doc/21-multi-agent.md)）：

| 维度 | Compaction | Handoff |
|------|-----------|---------|
| 方式 | 摘要旧消息插入当前上下文 | 生成新 prompt 创建新 session |
| 上下文窗口压力 | 仍有（虽然减小） | 全新窗口 |
| 可编辑性 | 不可编辑（自动） | 用户可编辑生成的 prompt |
| 适用 | 长对话自动压缩 | 任务切换、重新聚焦 |

**核心代码**（必须用 `pi.registerCommand`，handler 收到 `ExtensionCommandContext`）：

```ts
import { convertToLlm, serializeConversation } from "@earendil-works/pi-coding-agent";

export default (pi) => {
  pi.registerCommand("handoff", {
    handler: async (args, ctx) => {
      // 1. 收集当前分支的对话历史
      //    getBranch() 返回 SessionEntry[]，convertToLlm 需要 AgentMessage[]
      //    必须 filter+map 转换（参考官方 handoff.ts 的 getHandoffMessages）
      const branch = ctx.sessionManager.getBranch();
      const messages = branch
        .filter((e) => e.type === "message")
        .map((e) => e.message);
      // ⚠️ 上方简化版仅适用【未压缩】分支——只保留 message entry。
      //    分支被 compaction 过时，filter 会【静默丢弃所有 compaction 摘要】，
      //    导致丢失压缩上下文。生产代码必须用官方 getHandoffMessages
      //    （把 compaction entry 转成 compactionSummary 消息 + 保留 firstKeptEntryId 之后的 entry）。
      const llmMessages = convertToLlm(messages);
      const conversationText = serializeConversation(llmMessages);

      // 2. 用 LLM 生成自包含的 handoff prompt
      //    ：扩展内通过 ctx.modelRegistry 薄包装器调用 complete()
      //    证据：examples/extensions/handoff, 120-147
      const goal = args.trim();
      const auth = await ctx.modelRegistry.getApiKeyAndHeaders(ctx.model!);
      if (!auth.ok || !auth.apiKey) {
        throw new Error(auth.ok ? `No API key for ${ctx.model!.provider}` : auth.error);
      }
      const response = await ctx.modelRegistry.complete(ctx.model!, {
        systemPrompt: "你是一个上下文转移助手...",
        messages: [{
          role: "user",
          content: [{ type: "text", text: `## 对话历史\n\n${conversationText}\n\n## 新任务目标\n\n${goal}` }],
          timestamp: Date.now(),
        }],
      }, { apiKey: auth.apiKey, headers: auth.headers, env: auth.env });
      const prompt = response.content
        .filter((c) => c.type === "text").map((c) => c.text).join("\n");

      // 3. 让用户编辑（可选）
      const edited = ctx.hasUI ? await ctx.ui.editor("Edit handoff prompt", prompt) : prompt;

      // 4. 创建新 session，parentSession 可追溯来源
      //    ★ ctx.newSession 只在 ExtensionCommandContext 中存在
      await ctx.newSession({
        parentSession: ctx.sessionManager.getSessionFile(),  // ← 持久化 session 才有值；inMemory 是 undefined
        withSession: async (newCtx) => {
          newCtx.ui.setEditorText(edited ?? prompt);
        },
      });
    },
  });
};
```

**横向陷阱**：
- **`ctx.newSession` 只在 command handler 里能用**——`pi.registerTool` 的 execute 收到的是 `ExtensionContext`，没这个方法。要在 tool 里做 handoff，从 tool execute 调 `pi.registerCommand` 注册的命令（间接），或者直接 `createAgentSession`（同进程子 Agent，模式 3b）。
- **`ctx.sessionManager.getSessionFile()` 在 inMemory session 是 undefined**——`parentSession` 字段会写 undefined。要追溯来源必须用持久化 session。
- **`ctx.ui.editor` 只在 hasUI 时可用**——非交互模式（自动化测试 / RPC server）调用会失败。先用 `ctx.hasUI` 守卫。

> 完整实现参考 `packages/coding-agent/examples/extensions/handoff.ts`，包含 compaction 摘要、加载器 UI、错误处理。

### 模式 5：Fork 分支探索

**原理**：`AgentSessionRuntime.fork(entryId, options)` 从会话历史中的某个 entry 分叉出新分支，在同一会话树内探索替代方案。

**适用场景**：在某个决策点想试两种方案（如两种重构思路），保留主线继续探索。

```ts
// position: "before"（默认）—— 从该 user message 之前分叉出新分支
//   要求 entryId 必须是 user message（否则抛 "Invalid entry ID for forking"）
const result1 = await runtime.fork("entry-uuid", {
  position: "before",
  withSession: async (ctx) => {
    ctx.ui.notify("Forked new branch. Try a different approach.", "info");
  },
});
// result1.selectedText 包含被分叉 user message 的文本（仅 position="before" 时）

// position: "at" —— 克隆整条会话到该 entry 处（对任何 entry 类型都可用）
const result2 = await runtime.fork("entry-uuid", { position: "at" });
```

**Fork 的两种路径**（根据持久化模式）：
- **持久化 session**（`isPersisted()` 返回 true）：通过 `SessionManager.open()` 打开文件，用 `createBranchedSession()` 创建物理分支，存在同一文件中。
- **非持久化 session**（inMemory）：直接在当前 SessionManager 上 `newSession()` / `createBranchedSession()`，无文件 I/O。

完整 fork 细节见 [F02](F02-session-runtime.md)。

## 多 Agent 通信方式

多 Agent 协作最大的工程问题：**Agent 之间怎么通信？**——`AgentSession` 不提供内置的跨 session 总线。三种常见模式：

| 方式 | 适用场景 | 特点 |
|------|---------|------|
| 消息传递（`sendUserMessage`） | 单向通知——Agent A 完成后告知 Agent B | 简单直接，无返回值 |
| Event Bus（`session.subscribe` + 触发器） | 松耦合协调——A 完成后自动触发 B | 解耦好，但事件不跨进程（同进程内多 session） |
| Steering / FollowUp | 实时干预——父进程在子 Agent 运行中主动介入 | 可中断/重定向正在运行的 agent |

Server 集成场景更常用 `session.subscribe`（外部层）而非扩展层的 `pi.on`——subscribe 能拿到所有事件，`pi.on` 只在扩展层。

### 消息传递

```ts
// 签名：session.sendUserMessage(content, options?)
// - content: string | (TextContent | ImageContent)[]
// - options.deliverAs: "steer" | "followUp"  —— agent 正在 streaming 时必需
//   源码：agent-session，内部走 prompt() + deliverAs 参数

agent1.session.sendUserMessage("Agent 2 has finished analyzing the database schema.");
// agent 正在运行时可指定排队策略：
agent1.session.sendUserMessage("Urgent: stop and check this", { deliverAs: "steer" });
```

### Event Bus（同进程多 session 协调）

```ts
// 扩展可以监听 session 事件并协调多个 session
pi.on("agent_settled", (event) => {
  // Agent 1 完全稳定了（retry/compaction/steer 全部消费完毕），通知 Agent 2
  // ：agent_settled 比 agent_end 更可靠（agent_end 在 retry 场景下会提前触发）
  // ★ 注意：扩展层事件不跨 session 迁移（见 F02 陷阱 #1）
  agent2.session.steer("Agent 1 completed. Result: " + extractSummary(event));
});

// 或者用 subscribe 监听（外部层，能收到所有事件）
const unsub = agent1.subscribe((event) => {
  if (event.type === "agent_settled") {
    agent2.sendUserMessage("Agent 1 done: " + extractSummary(event));
  }
});
```

### Steering / FollowUp

```ts
// session.steer(text, images?) —— 入队等下一轮 turn 消费
//   源码：agent-session
//   时机：只在 agent loop 每轮 turn_end 之后被消费
//   常见失败：agent loop 在 steer 触发前就完成了（GLM 快、prompt 简单）
//   修复：setTimeout 要在 session.prompt() 之前启动，延迟短（500ms-1s）
await session1.steer("Stop current work, switch to task B");

// session.followUp(text, images?) —— 直接追加（不打断当前 turn）
//   源码：agent-session
await session2.followUp("Agent 1 says: the fix is in src/auth.ts");

// followUp 队列的对称管理 API（，agent-session, 1530-1532, 1767-1770）：
//   session.followUpMode         — get/set: "all" | "one-at-a-time"
//   session.setFollowUpMode(mode) — 设置 followUp drain 模式
//   session.getFollowUpMessages() — 只读查询 followUp 队列内容
```

完整 steer 时序见 [F05](F05-steer-session.md)。

## ReplacedSessionContext：替换后的"新 ctx"

`runtime.newSession` / `fork` / `switchSession` 的 `withSession` 回调收到的是 `ReplacedSessionContext`（types）——它继承自 `ExtensionCommandContext`，**额外**有：

```ts
export interface ReplacedSessionContext extends ExtensionCommandContext {
  sendMessage<T = unknown>(
    message: Pick<CustomMessage<T>, "customType" | "content" | "display" | "details">,
    options?: { triggerTurn?: boolean; deliverAs?: "steer" | "followUp" | "nextTurn" },
  ): Promise<void>;

  sendUserMessage(
    content: string | (TextContent | ImageContent)[],
    options?: { deliverAs?: "steer" | "followUp" },
  ): Promise<void>;
}
```

**关键**：`withSession` 回调内拿到的 ctx **是新的**——指向替换后的 session。**不要**用闭包捕获的旧 ctx（旧 ctx 在 dispose 后会 invalidate，types 的报错信息明确说明）。

## 完整生命周期：每个 session 独立 dispose

`AgentSession.dispose()` **是同步的**（agent-session），不返回 Promise。但 `AgentSessionRuntime.dispose()` 是 **async**，必须 await（agent-session-runtime）。

**对比表**：

| 操作 | 同步还是 async | 后果 |
|------|--------------|------|
| `session.dispose()` | **同步** | 调完立即返回，但 `agent.abort()` 内部的 abort hook 可能还没跑完——同步代码不会触发 `session_shutdown` 事件 |
| `runtime.dispose()` | **async** | 必须 await！不 await 会让 `session_shutdown` 扩展事件来不及完成，扩展拿不到清理机会 |

**多 session 清理模板**：

```ts
// 同步 dispose 多个独立 session——可以批量调
const sessions = [agent1, agent2, agent3];
sessions.forEach((s) => s.dispose());

// runtime.dispose 必须 await
await runtime.dispose();
```

## 常见误期待与陷阱

1. **「多个 session 共享 cwd / 消息」** → **错**。每个 session 有独立的 AgentState（messages、tools）、独立的 sessionManager、独立的 sessionFile（除非 inMemory）。要共享状态必须**显式**在代码中传递。
2. **「session 完全独立」** → **错**。默认共享 `~/.pi/agent` 目录 → 共享 `auth.json` + `models.json`（通过 `modelRuntime`）。要完全隔离传独立 `agentDir`，或更彻底地传独立 `modelRuntime` 实例。详见上方"最大陷阱"。
3. **「subscribe 会自动迁移到新 session」** → **错**。subscribe 绑在具体 AgentSession 实例上，切换 session 后旧订阅对新 session 无效。必须在 `setRebindSession` 回调里退订旧的 + 订阅新的。
4. **「dispose 可调可不调」** → **错**。`agent.subscribe` 内部订阅没解绑，进程会挂起。`try/finally` 必备，每个独立 session 都要 dispose。
5. **「`runtime.dispose()` 是同步」** → **错**。它是 async，必须 await。不 await 会让 `session_shutdown` 扩展事件来不及完成。
6. **「tool execute 里能调 `ctx.newSession`」** → **错**。tool execute 收到的是 `ExtensionContext`（types），**没有** `newSession`。只有 `pi.registerCommand` 的 handler 收到 `ExtensionCommandContext` 才有。
7. **「`SessionManager.inMemory()` 默认 cwd 就是当前项目」** → **错**。默认 `process.cwd()`——你启动 Node 时所在的目录，不一定是项目根。多 Agent 协作时每个 session 都默认串到 process.cwd，会互相污染。必须 `SessionManager.inMemory(explicitCwd)`。
8. **「Handoff 等于 Compaction」** → **错**。Handoff 生成全新 prompt 接力，上下文窗口全新；Compaction 是摘要嵌入当前上下文，仍受上下文窗口压力。
9. **「Subagent 工具自动持久化」** → **错**（仅 3a 官方机制）。子进程用 `--no-session` 标志，不写磁盘。3b 同进程子 Agent 是否持久化取决于你传的 sessionManager。
10. **「Fork 只能在持久化 session 上」** → **错**。Fork 在持久化和非持久化 session 上都能工作，但行为不同：持久化用 `SessionManager.open()` + `createBranchedSession()`（物理分支），非持久化直接内存操作。
11. **「`ctx.sessionManager.getSessionFile()` 总有值」** → **错**。inMemory session 返回 `undefined`。要作为 `parentSession` 必须用持久化 session。
12. **「多个 session 各自注册的扩展会自动共享」** → **错**。每个 session 独立 `bindExtensions`，扩展实例不跨 session。**注册的 provider / model** 也不自动跨 session——每个 session 默认有独立的 `ModelRuntime`（内含独立 `ModelRegistry`），`registerProvider` 写在实例内存里，另一个 session 的实例看不到。只有显式传**同一个** `options.modelRuntime` 实例时，provider/model 才跨 session 共享（见上方"最大陷阱"表 modelRuntime 行）。
13. **「`session.state.messages.at(-1).content === string`」** → **错**。`AssistantMessage.content` 一定是数组（ai/types），UserMessage.content 也常是数组。要从数组 filter text 块。
14. **「AgentSessionRuntime 是多 session 并行的最佳选择」** → **看场景**。Runtime 主要为顺序切换设计（每次只有一个 active session）。要真正并行还是用多个 `createAgentSession` + `Promise.all`（模式 1）。

## 变体与延伸

| 变体 | 怎么改 | 参考 |
|------|-------|------|
| 完全手动组装多 Agent（自定义 ResourceLoader / ModelRegistry） | 用 [H01](H01-full-control.md) 的方式给每个 session 传独立组件 | [H01](H01-full-control.md) |
| 多 Agent 用同一 Provider 但不同模型 | 共享 agentDir / authStorage；每个 session 传不同 `model` | [H02](H02-custom-provider.md) |
| 用 Faux Provider 测试多 Agent 协调 | 每个 session 独立 `registerFauxProvider` 或共享同一 handle | [H03](H03-faux-provider.md) |
| Session Runtime 完整方法 / rebind 机制 / 事件列表 | 详见 F02 | [F02](F02-session-runtime.md) |
| Abort 行为 / dispose 内部机制 | 详见 F04 | [F04](F04-abort-session.md) |
| Steer 时序问题 / steer 失效场景 | 详见 F05 | [F05](F05-steer-session.md) |
| 持久化策略 / SessionManager 基础 | 详见 F01 | [F01](F01-session-persistence.md) |
| 官方 Subagent 配置 / 三种执行模式 | 详见 I05 | [I05](I05-subagent.md) |
| 多 Agent 完整架构文档（含通信 / Fork / 限制） | 官方文档 | [sdk_doc/21-multi-agent.md](../sdk_doc/21-multi-agent.md) |
| ExtensionContext vs ExtensionCommandContext 边界 | 完整字段对比 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
