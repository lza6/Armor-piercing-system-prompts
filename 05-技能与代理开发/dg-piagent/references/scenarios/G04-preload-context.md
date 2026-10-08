# 场景：会话启动时预加载项目上下文 (G04)

## 这是什么

**预加载**不是 pi-agent 的内置功能——SDK 不提供"配置一下就自动读 package.json / git status"的机制。本场景讲的是**用扩展层 hook 在会话启动或每次 prompt 时读取项目元数据**，然后通过 contextFiles / context hook / appendEntry 等机制注入到 Agent 上下文。

SDK 提供的素材只有事件 hook（`before_agent_start` / `turn_start` / `context`）、扩展 API（`pi.appendEntry` / `pi.sendMessage` / `pi.sendUserMessage`）、以及 contextFiles 加载机制（`DefaultResourceLoader`）。具体读什么文件、什么时候读、读到后注入到哪一层，都是扩展自己实现的。

> **G01 vs G04 分工**：[G01](G01-context-injection.md) 聚焦「怎么注入」（四种机制对比），G04 聚焦「读什么 + 何时读」（项目元数据 / Git 状态 / 依赖树等动态数据的读取时机与缓存策略）。两场景互补，G04 假设读者已了解 G01 的四种注入机制。

## 什么时候用 / 不用会怎样

| 你的场景 | 用什么 | 为什么 |
|---------|--------|--------|
| 每次 prompt 时让 Agent 看到当前依赖树、Git 分支等"每次会话开始加载一次就够"的元数据 | **`pi.on("before_agent_start")`** + 缓存 | 每次 prompt 只触发一次，无死循环风险，适合相对稳定的项目元数据 |
| 每轮 LLM 调用前注入最新 Git status / 未提交改动列表（真"每轮都变"的数据） | **`pi.on("context")`** hook | 改本轮 LLM 输入快照，不进 messages 历史，不触发新 turn |
| 启动时一次性加载长期规则（代码风格、命名约定） | **Context Files**（`AGENTS.md` / `CLAUDE.md`） | 注入 system prompt，零运行时开销，跨会话持久 |
| 把读到的项目元数据**持久化**到会话树（跨 session 可读，不发给 LLM） | **`pi.appendEntry(customType, data)`** | 写 CustomEntry 到 `~/.pi/agent/sessions/*.jsonl`（`getSessionsDir()`，`agentDir` 默认 `~/.pi/agent`），不触发 turn，不影响上下文窗口 |
| 想在 Agent 工作中补充一句用户指示 | **`session.steer()`** / **`pi.sendUserMessage({ deliverAs: "steer" })`** | 加 user message 进对话流，触发新一轮 turn（★ 不要在 turn_start 里调用，会死循环，见 [陷阱 1](#陷阱-1turn_start--sendusermessage--steer-死循环)） |

**一句话区分**：稳定数据走 contextFiles（启动时注入 system prompt）；动态数据走 `context` hook（每轮临时改 LLM 输入）；元数据持久化走 `appendEntry`（不进 LLM 视野）；补充用户指示走 `steer`（触发新 turn）。

## ⚠️ 最大陷阱：不要在 turn_start 里预加载

**事实**：`turn_start` 在每个 turn 开始时触发（agent-loop.ts:110,176），扩展层 payload 含 `turnIndex / timestamp`（types.ts:728-732，`packages/coding-agent/src/core/extensions/types.ts`）。

**在 turn_start 里 `sendUserMessage + deliverAs:"steer"`** 会触发死循环：

```
turn_start → sendUserMessage(steer) → 消息入 queue
→ 当前 turn 结束 → getSteeringMessages 消费 queue → 触发新一轮 turn
→ 新一轮 turn 又触发 turn_start → 又 sendUserMessage(steer) → 无限循环
```

证据链：`getSteeringMessages` 在 `turn_end` 之后被消费（agent-loop.ts:259），回到循环顶部第 176 行又 emit `turn_start`。详见 [G01 陷阱 1](G01-context-injection.md#陷阱-1turn_start--sendusermessage--steer-死循环) / [G03 陷阱 5](G03-auto-summarize.md#陷阱-5用-turn_start-加载历史--死循环)。

**修复**：
- 想每次 prompt 加载一次 → 用 `pi.on("before_agent_start")`（每次 prompt 只触发一次）
- 想每轮 LLM 调用前改消息 → 用 `pi.on("context")` hook（不触发新 turn）
- 想持久化元数据 → 用 `pi.appendEntry`（完全不触发 turn）
- 非要在 turn_start 注入 → 加去重条件（如 `event.turnIndex === 0` 或基于时间戳的缓存）

## ⚠️ 第二大陷阱：sendMessage 在 streaming 期间默认走 steer

**事实**：`pi.sendMessage({ customType, content, display })` 在 streaming 期间**不带 `deliverAs` 时默认走 steer**（agent-session.ts:1443-1448）。`sendMessage` 没有 `streamingBehavior` 参数——该参数是 `session.prompt()` 和 `pi.sendUserMessage()` 的。在 streaming 期间调 `sendUserMessage` 不带 `deliverAs` 会**抛错**（agent-session.ts:1157-1164），因为 sendUserMessage 内部调 `prompt()`。

**后果**：在 `turn_start` / `context` 等 streaming 期间的 hook 里调 `sendMessage` 不带 `deliverAs` → 消息默认 steer 入队 → 当前 turn 结束后触发新一轮 turn → 在 hook 里反复调用会死循环（同陷阱 1）。

**修复**：
- 想完全不触发 turn → 用 `pi.appendEntry(customType, data)`（写入 CustomEntry，不进 messages，不触发 turn）
- streaming 期间追加但不 steer → 用 `deliverAs: "nextTurn"`（push 到 pendingNextTurnMessages，下一轮 prompt 时注入）
- 非 streaming 期间追加且不触发 turn → 用 `sendMessage(msg, { triggerTurn: false })`（默认行为，追加到 state + session 持久化，不发 LLM）

## 涉及 SDK

| 能力 | 签名 / 真实字段 | 用途 | 详细文档 |
|------|----------------|------|---------|
| `pi.on("before_agent_start")` | `(event: BeforeAgentStartEvent, ctx) => void` | **每次 prompt 后**、agent loop 前触发（扩展独有，subscribe 收不到）。适合"每次会话加载一次"的项目元数据读取 | [04-events.md](../sdk_doc/04-events.md) |
| `pi.on("context")` | `(event: { messages: AgentMessage[] }, ctx) => { messages?: AgentMessage[] } \| void` | **每轮 LLM 调用前**修改发给模型的消息列表（扩展独有）。适合"每轮都变"的动态数据 | [04-events.md](../sdk_doc/04-events.md) |
| `pi.on("turn_start")` | `(event: { type, turnIndex, timestamp }, ctx) => void` | 每个 turn 开始时触发。**不适合**做 preload（会死循环） | [04-events.md](../sdk_doc/04-events.md) |
| `pi.appendEntry()` | `appendEntry<T = unknown>(customType: string, data?: T): void` | 写入 CustomEntry 到会话树（`~/.pi/agent/sessions/*.jsonl`），**不触发 turn，不发给 LLM**（同步返回；sendMessage/sendUserMessage 为 fire-and-forget，错误经 emitError 上报不抛出，不要 try/catch 期待捕获） | [07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `pi.sendMessage()` | `sendMessage<T>(message: { customType, content, display, details? }, options?: { triggerTurn?, deliverAs? }): void` | 注入 CustomMessage 到会话。**streaming 期间不带 options 默认走 steer**（陷阱 2） | [07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `pi.sendUserMessage()` | `sendUserMessage(content: string \| (TextContent \| ImageContent)[], options?: { deliverAs?: "steer" \| "followUp" }): void` | 注入 user 消息，**总是触发 turn**（deliverAs 仅控制排队时机） | [07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| Context Files | `loadProjectContextFiles({ cwd, agentDir })` → `Array<{path, content}>` | 启动时发现 `AGENTS.md` / `CLAUDE.md`，注入 system prompt（★ 不受 projectTrusted 门槛，无条件加载） | [11-context-files.md](../sdk_doc/11-context-files.md) |

> ⚠️ **派发层次差异**（★ E02 横向陷阱）：`context` 和 `before_agent_start` 是**扩展独有事件**——只在扩展层 `pi.on` 派发，`session.subscribe` 静默收不到。详见 [04-events.md 坑 4](../sdk_doc/04-events.md#坑-46-个扩展独有事件sessionsubscribe-静默收不到-最大集成坑)。

## 核心代码

### 模式 A：用 `before_agent_start` 每次 prompt 加载一次（推荐，安全）

```ts
import { readFile } from "node:fs/promises";
import { exec } from "node:child_process";
import { promisify } from "node:util";

const execAsync = promisify(exec);

export default (pi) => {
  let lastLoadedAt = 0;
  let cachedContext: string | null = null;
  const CACHE_TTL_MS = 60_000;  // 1 分钟缓存，避免每次 prompt 都读盘

  pi.on("before_agent_start", async (event, ctx) => {
    // 每次 prompt 触发一次（不是每轮 turn），无死循环风险
    if (Date.now() - lastLoadedAt < CACHE_TTL_MS) return;  // 缓存命中

    try {
      // 1. 读取项目元数据（用 async IO，不要用 execSync 阻塞）
      const pkgRaw = await readFile(`${ctx.cwd}/package.json`, "utf-8");
      const pkg = JSON.parse(pkgRaw);
      const deps = Object.keys(pkg.dependencies || {}).slice(0, 20).join(", ");

      const { stdout: gitStatus } = await execAsync("git status --short", {
        cwd: ctx.cwd,
        maxBuffer: 1024 * 1024,
      });

      // 2. 拼接上下文文本
      cachedContext = [
        `[Project Context]`,
        `- Dependencies (top 20): ${deps}`,
        `- Git status:\n${gitStatus.slice(0, 500)}`,
      ].join("\n");

      lastLoadedAt = Date.now();
      console.log(`[Preload] Loaded project context (${cachedContext.length} chars)`);
    } catch (err) {
      console.warn("[Preload] Failed to load context:", err);
    }

    // 注意：before_agent_start 的 event.systemPrompt 属性是只读的（mutation 不生效）
    // 修改 systemPrompt 要通过 handler 的返回值：return { systemPrompt: "new prompt" }
    // （BeforeAgentStartEventResult，runner 会链式生效多个扩展的 systemPrompt 返回值：后注册的扩展覆盖前者，通过 ctx.getSystemPrompt() 可见累积值——不是字符串拼接）
    // 这里只做"读取 + 缓存"，注入走下面的 context hook 或 contextFiles
  });

  // 配合：用 context hook 把缓存的内容注入每轮 LLM 输入
  pi.on("context", async (event) => {
    if (!cachedContext) return;  // 还没加载过就跳过
    return {
      messages: [
        ...event.messages,
        { role: "user" as const, content: cachedContext },
      ],
    };
  });
};
```

**为什么用 `before_agent_start` 而不是 `turn_start`**：
- `before_agent_start` 每次 prompt 只触发一次（agent-session.ts:1224，emitBeforeAgentStart 在 `_runAgentPrompt` 调用前），不会死循环
- `turn_start` 每轮都触发，在其中 steer / sendMessage 会无限循环（陷阱 1 / 陷阱 2）

> ⚠️ **首次 LLM 调用与 context 事件**（[04-events.md 坑 3](../sdk_doc/04-events.md#坑-3context-事件在首次-llm-调用时不触发)）：04-events.md 坑 3 报告"首次 LLM 调用不走 context 事件"。但从源码看 `transformContext`（sdk.ts:350-353）在每次 `streamAssistantResponse` 调用时都执行，包括首次 turn（agent-loop.ts:193），无跳过逻辑。此行为待进一步确认。

### 模式 B：用 `context` hook 每轮注入最新数据（推荐，避免死循环）

```ts
import { exec } from "node:child_process";
import { promisify } from "node:util";

const execAsync = promisify(exec);

export default (pi) => {
  pi.on("context", async (event, ctx) => {
    // 每轮 LLM 调用前触发（扩展独有，subscribe 收不到）
    // event.messages 是即将发给 LLM 的完整消息列表
    try {
      const { stdout: gitStatus } = await execAsync("git status --short", {
        cwd: ctx.cwd,
        maxBuffer: 1024 * 1024,
      });

      if (!gitStatus.trim()) return;  // 没改动就不改 messages

      // 追加一条临时 user message（不进 messages 历史，只本轮 LLM 看到）
      return {
        messages: [
          ...event.messages,
          {
            role: "user" as const,
            content: `[Runtime Context]\nUncommitted changes:\n${gitStatus.slice(0, 500)}`,
          },
        ],
      };
    } catch (err) {
      console.warn("[Context] Failed to read git status:", err);
      return;  // 出错时不改 messages
    }
  });
};
```

**关键点**：
- `context` hook 修改的是**本轮 LLM 调用输入快照**（不进 messages 历史，不触发新 turn，不会死循环）
- handler 返回 `{ messages }` 时替换；返回 `void / undefined` 时保持原 messages
- 派发位置：在 `streamAssistantResponse` 内、`convertToLlm` 之前（agent-loop.ts:290-295，transformContext 在 290-292，convertToLlm 在 295）

### 模式 C：用 `appendEntry` 持久化项目元数据（不发给 LLM）

```ts
import { readFile } from "node:fs/promises";

export default (pi) => {
  pi.on("before_agent_start", async (event, ctx) => {
    try {
      const pkgRaw = await readFile(`${ctx.cwd}/package.json`, "utf-8");
      const pkg = JSON.parse(pkgRaw);

      // ✅ 写入会话树（CustomEntry 类型），跨 session 持久化到 ~/.pi/agent/sessions/*.jsonl
      // 注意：appendEntry 不会发给 LLM，也不会触发 turn
      pi.appendEntry("project_meta", {
        name: pkg.name,
        version: pkg.version,
        dependencyCount: Object.keys(pkg.dependencies || {}).length,
        timestamp: Date.now(),
      });
    } catch (err) {
      console.warn("[Preload] Failed:", err);
    }
  });
};
```

**用途**：当你想让宿主 / UI 能读到"这次会话开始时的项目状态"（用于展示、调试、统计），但**不想让 LLM 看到**（避免污染上下文窗口）时，用 `appendEntry`。CustomEntry 存在 `~/.pi/agent/sessions/*.jsonl`，跨 session 可读。

### 模式 D：用 Context Files 启动时注入静态规则

```ts
import {
  createAgentSession,
  DefaultResourceLoader,
  getAgentDir,
  SessionManager,
} from "@earendil-works/pi-coding-agent";

const loader = new DefaultResourceLoader({
  cwd: process.cwd(),
  agentDir: getAgentDir(),
  // 追加一条运行时的项目元数据（无需写文件）
  agentsFilesOverride: (base) => ({
    agentsFiles: [
      ...base.agentsFiles,
      {
        path: "/virtual/PROJECT_META.md",
        content: `# Project Metadata
- Name: my-agent
- Branch: main
- Loaded at: ${new Date().toISOString()}`,
      },
    ],
  }),
});

await loader.reload();

const { session } = await createAgentSession({
  resourceLoader: loader,
  sessionManager: SessionManager.inMemory(),
});

try {
  // Agent 的 system prompt 末尾会包含 <project_context> 块
  await session.prompt("Help me refactor src/utils.ts");
} finally {
  session.dispose();
}
```

**适合**：项目级长期规则（代码风格、命名约定、Git 工作流）。零运行时开销，跨会话持久。

> ⚠️ **★ A03 陷阱**：即使设置 `systemPrompt: "my custom prompt"`，**context files / skills / cwd 仍会被自动追加**（system-prompt.ts 的 customPrompt 分支，无 date 注入）。详见 [G01 陷阱 2](G01-context-injection.md#陷阱-2customprompt-就是最终-system-prompt)。

> ⚠️ **★ C03 横向陷阱**：context files 加载**不受 `projectTrusted` 门槛**（resource-loader.ts）。即使项目未被信任，AGENTS.md / CLAUDE.md 也会被无条件读取注入。详见 [G01 陷阱 3](G01-context-injection.md#陷阱-3context-files-受-projecttrusted-保护)。

## 常见误期待与陷阱

### 陷阱 1：`turn_start` + `sendUserMessage` + `steer` 死循环

**现象**：在 `turn_start` hook 里无条件调用 `pi.sendUserMessage(text, { deliverAs: "steer" })`，期望"每轮注入上下文"。实际：每个 steer 触发新一轮 turn → 新一轮 turn 触发 turn_start → 又 steer → **无限循环**直到 token 耗尽。

**修复**：
- 想每轮 LLM 调用前改消息 → 用 `pi.on("context")` hook（不触发新 turn）
- 想每次 prompt 加载一次 → 用 `pi.on("before_agent_start")`（每次 prompt 只触发一次）
- 非要在 `turn_start` 注入 → 加去重条件（如"只在首轮" `event.turnIndex === 0`，或基于时间戳的缓存）

详见 [G01 陷阱 1](G01-context-injection.md#陷阱-1turn_start--sendusermessage--steer-死循环)。

### 陷阱 2：sendMessage 在 streaming 期间默认走 steer

**现象**：在 `turn_start` / `context` 等 streaming 期间的 hook 里调 `pi.sendMessage({ customType, content, display })` 不带 `deliverAs`，以为消息不会被处理。

**实际**：streaming 期间 `sendMessage` 不带 `deliverAs` 时**默认走 steer**（agent-session.ts:1443-1448），消息会入队并触发新一轮 turn。`sendMessage` 没有 `streamingBehavior` 参数（那是 `prompt()` / `sendUserMessage()` 的）。在 streaming hook 里反复调用会死循环。

**修复**：
- 想完全不触发 turn → 用 `pi.appendEntry(customType, data)`（写入 CustomEntry）
- streaming 期间追加但不 steer → 用 `deliverAs: "nextTurn"`（push 到 pendingNextTurnMessages）
- 非 streaming 期间追加且不触发 turn → 用 `sendMessage(msg, { triggerTurn: false })`

### 陷阱 3：「sendMessage 的 content 能传任意对象」

**现象**：写 `pi.sendMessage({ customType: "context", content: { deps, gitStatus } })` 期望存结构化数据。

**实际**：`CustomMessage.content` 类型是 `string | (TextContent | ImageContent)[]`（messages.ts），**不是 object**。`display` 类型是 `boolean`（是否在 UI 显示），**不是 string**。

**修复**：
- 想存结构化数据 → 用 `pi.appendEntry(customType, data)`（data 是泛型 `T = unknown`，可存任意对象）
- 必须用 sendMessage → content 序列化为 JSON 字符串：`content: JSON.stringify({ deps, gitStatus })`

### 陷阱 4：「turn_start 拿不到 ctx」

**现象**：以为 `TurnStartEvent` 字段只有 `{ type, turnIndex, timestamp }`（types.ts:728-732），所以拿不到 cwd。

**实际**：cwd 在 **handler 的第二个参数 `ctx`**（`ExtensionContext`）里，不在 event 里。所有事件 handler 都能拿到同一个 ctx（types.ts:307-347）：

```ts
pi.on("turn_start", (event, ctx) => {
  console.log(ctx.cwd);         // ✓ 当前工作目录
  console.log(ctx.model);       // ✓ 当前模型
  console.log(ctx.signal);      // ✓ AbortSignal
  console.log(event.turnIndex); // ✓ turn 序号（扩展层独有字段）
});
```

### 陷阱 5：每个 turn 都读盘（性能反模式）

**现象**：在 hook 里无缓存地读 package.json / 跑 git status，导致高频重复 IO。两种 hook 的频率不同：
- **`context` hook 每轮 LLM 调用都触发**——无缓存则每轮读盘（真正的高频路径）
- **`before_agent_start` 每次 prompt 触发**——无缓存则每次 prompt 读盘（不是每 turn，频率低于 context）

**修复**：
- 加时间戳缓存（如模式 A 的 `lastLoadedAt + CACHE_TTL_MS`）
- 对真正高频变更的数据（如 git status），考虑只在 `context` hook 里读（每轮 LLM 调用一次，不是每个 turn）
- 用 `async` IO（`readFile` / `exec` 的 promisify），不要用 `execSync` 阻塞事件循环

## 变体与延伸

- 注入外部记忆与上下文的四种机制对比 → [场景 G01](G01-context-injection.md)（聚焦"怎么注入"，G04 聚焦"读什么 + 何时读"）
- 自动总结并持久化记忆 → [场景 G03](G03-auto-summarize.md)（提取要点到外部存储）
- steer 的入队机制详解（steeringMode、queue_update、生效前提）→ [场景 F05](F05-steer-session.md)
- lifecycle hooks 完整参考（before_agent_start / turn_start / context 等）→ [场景 E04](E04-lifecycle-hooks.md)
- context files 完整 API（loadProjectContextFiles / agentsFilesOverride / noContextFiles）→ [11-context-files.md](../sdk_doc/11-context-files.md)
- 扩展层事件完整清单（含 context / before_agent_start 等 6 个扩展独有事件）→ [04-events.md](../sdk_doc/04-events.md)
