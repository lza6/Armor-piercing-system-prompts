# 场景：会话运行时切换与恢复 (F02)

## 什么时候用 / 不用会怎样

**用 `AgentSessionRuntime` 的前提**：你需要在运行时**替换活跃会话**——新建、切换历史会话、分叉、导入外部 JSONL。如果你的程序只是"一次性对话到结束"，直接用 `createAgentSession()` 就够了（见 [A01](A01-minimal-startup.md)），不需要 Runtime 这一层。

| 你的场景 | 用什么 | 为什么 |
|---------|--------|--------|
| 一次性问答 / 单条 prompt 流 | `createAgentSession()` | 简单，无额外抽象层 |
| 需要在同一进程内多次切换会话（如 CLI `/resume`、`/new`、`/fork`） | `AgentSessionRuntime` | runtime 内部统一管理 dispose → 重建 services → 创建新 session → 触发 `session_shutdown` 事件的完整流程，避免手工拼装遗留状态 |
| 不需要持久化、只想用内存会话测试 | `SessionManager.inMemory()` + `createAgentSession` | 见 [F01](F01-session-persistence.md) |

**不用 Runtime 自己手写切换会怎样**：①忘了 dispose 旧 session 会导致扩展 `session_shutdown` 事件不触发、abort 不执行；②忘了重建 cwd 绑定服务（settings/resourceLoader/modelRegistry）会让工具仍然在旧 cwd 下执行；③跨会话的事件订阅不会自动迁移，需要手工 unsubscribe + 重新 subscribe。

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `createAgentSessionRuntime(createRuntime, options)` | 创建 runtime（工厂函数存储复用） | [sdk_doc/03-agent-session-runtime.md](../sdk_doc/03-agent-session-runtime.md) |
| `createAgentSessionServices(options)` | 工厂内创建 cwd 绑定服务 | [sdk_doc/03-agent-session-runtime.md](../sdk_doc/03-agent-session-runtime.md) |
| `createAgentSessionFromServices(options)` | 基于服务创建 session | [sdk_doc/03-agent-session-runtime.md](../sdk_doc/03-agent-session-runtime.md) |
| `runtime.switchSession(path, options?)` | 切换到指定历史会话（含 `cwdOverride` / `withSession` / `projectTrustContextFactory`） | 见下方方法说明 |
| `runtime.newSession(options?)` | 新建会话（含 `parentSession` / `setup` / `withSession`） | 见下方方法说明 |
| `runtime.fork(entryId, options?)` | 从指定 entry 分叉或克隆（`position: "before"\|"at"`，默认 `before`） | 见下方方法说明 |
| `runtime.importFromJsonl(path, cwdOverride?)` | 从外部 JSONL 导入（复制到 session 目录后 open） | 见下方方法说明 |
| `runtime.dispose()` | 销毁 runtime（**async**，需 await） | 见下方方法说明 |
| `runtime.setRebindSession(callback?)` | 注册 rebind 回调，由 runtime 在每次替换后自动调用 | 见下方 rebind 机制 |

> 所有方法的完整签名以 `@earendil-works/pi-coding-agent` 的 `.d.ts` 为准。

## 三个核心机制（先理解，再写代码）

### 1. 工厂函数的"存储复用"

`createAgentSessionRuntime` 的第一个参数是一个**工厂函数**，会被**存储**并在后续每次 `newSession` / `switchSession` / `fork` / `importFromJsonl` 中**重新调用**。工厂内部负责重建 cwd 绑定服务（因为 cwd 可能随会话切换而变化）。

工厂收到的参数：

```ts
type CreateAgentSessionRuntimeFactory = (options: {
  cwd: string;                    // 当前有效 cwd（switchSession 可能带来新 cwd）
  agentDir: string;               // Pi 配置目录（~/.pi/agent）
  sessionManager: SessionManager; // 本次目标 SessionManager
  sessionStartEvent?: SessionStartEvent;   // 传给扩展的 session_start 元数据
  projectTrustContext?: ProjectTrustContext; // switchSession 时由 options.projectTrustContextFactory 产生
}) => Promise<CreateAgentSessionRuntimeResult>;
```

返回值必须**同时**返回 `session` 和 `services`（否则后续替换无法更新 `_services`）。

### 2. 替换流程（每次 newSession / switchSession / fork / importFromJsonl 都跑）

统一走以下流程：

```
emitBeforeSwitch / emitBeforeFork
   ↓ cancelled=true → 直接返回 { cancelled: true }，不做任何改动
teardownCurrent(reason, targetSessionFile)
   ├─ session.abort()  ← 让进行中的 turn（含工具结果）持久化到旧 session 后再替换
   ├─ emit session_shutdown 事件（扩展可清理状态）
   ├─ beforeSessionInvalidate?.() 回调（宿主 UI 拆除）
   └─ session.dispose()  ← 取消进行中的 retry/compaction/agent
apply(await createRuntime({...}))
   ├─ 重建 cwd 绑定服务
   └─ 创建新 AgentSession
finishSessionReplacement(withSession?)
   ├─ rebindSession?.(this.session)  ← 你注册的 rebind 回调
   └─ withSession?.(replacedCtx)    ← options 传入的回调
```

**关键点**：
- `session.dispose()` 在 `teardownCurrent` 内部调用，所以切换前**不需要**自己 dispose
- 扩展不会自动迁移到新 session——必须在 rebind 回调里 `runtime.session.bindExtensions(...)` 重新绑定
- 事件订阅**完全不会迁移**——subscribe 是绑在具体 `AgentSession` 实例上的

### 3. Rebind 机制（不是 SDK 自动，是"你注册 + runtime 调用"）

`setRebindSession(callback)` **只是注册**回调。runtime 在 `finishSessionReplacement` 内部**自动调用**这个回调，传入新的 `session`。

回调内通常做两件事：
1. **重新绑定扩展**：`session.bindExtensions(...)`（扩展上下文需要指向新 session）
2. **重新订阅事件**：先 `unsubscribe()` 旧 session 的订阅，再对新 session 调 `session.subscribe(...)`

> 注意：`setRebindSession` **不直接管事件订阅**——它只是个钩子。事件订阅的迁移是调用方在回调内的责任。详见 sdk.md。

## 核心代码

```ts
import {
  createAgentSessionRuntime,
  createAgentSessionServices,
  createAgentSessionFromServices,
  SessionManager,
  getAgentDir,  // ← 用 getAgentDir() 取代硬编码 "~/.pi/agent"（避免绕过 PI_CODING_AGENT_DIR/piConfig 配置）
} from "@earendil-works/pi-coding-agent";

// 1. 创建 Runtime
//    factory 会被存储，在每次 newSession / switchSession / fork / import 中重新调用
const runtime = await createAgentSessionRuntime(
  async ({ cwd, agentDir, sessionManager, sessionStartEvent }) => {
    const services = await createAgentSessionServices({ cwd, agentDir });
    const result = await createAgentSessionFromServices({
      services,
      sessionManager,
      sessionStartEvent,
    });
    // 必须返回 session + services + diagnostics + modelFallbackMessage（runtime 在 apply() 时会更新这四个字段）
    return { ...result, services, diagnostics: services.diagnostics };
  },
  {
    cwd: process.cwd(),
    agentDir: getAgentDir(),
    // 初始 session：有历史就续最近的，没有就新建（等同 create）
    sessionManager: SessionManager.continueRecent(process.cwd()),
  },
);

// 2. 注册 rebind 回调（在每次替换后被自动调用）
let unsubscribe: (() => void) | undefined;
const extensionBindings = { /* 你的扩展绑定 */ } as any;

const rebind = async (session: AgentSession) => {
  // 2a. 重新绑定扩展
  await session.bindExtensions(extensionBindings);
  // 2b. 重新订阅事件（先退订旧的，再订新的）
  unsubscribe?.();
  unsubscribe = session.subscribe((event) => {
    if (event.type === "message_update" && event.assistantMessageEvent.type === "text_delta") {
      process.stdout.write(event.assistantMessageEvent.delta);
    }
  });
};
runtime.setRebindSession(rebind);
await rebind(runtime.session);  // 首次手动触发一次

// 3. 切换会话——cwdOverride 可选（用于跨项目导入同事的 session）
await runtime.switchSession("/path/to/other-session.jsonl", {
  cwdOverride: "/my/project",           // 可选：覆盖 session header 里的 cwd
  withSession: async (ctx) => {         // 可选：替换完成后的回调（rebind 之后）
    await ctx.sendUserMessage("切换完成");
  },
});

// 4. 新建会话——parentSession 可选，setup 在新 session 创建后执行
const result = await runtime.newSession({
  parentSession: runtime.session.sessionFile,  // 可选：把当前 session 作为父分支
  setup: async (sm) => {                        // 可选：新 session 初始化（如注入 system message）
    // sm 是新的 SessionManager，可以读 sm.buildSessionContext()
  },
});
if (result.cancelled) {
  console.log("切换被 session_before_switch 扩展取消");
}

// 5. 分叉——position 默认 "before"，传 "at" 时是"克隆当前 entry"
await runtime.fork("entry-uuid", { position: "before" });  // 从这条 user message 之前分叉
await runtime.fork("entry-uuid", { position: "at" });      // 克隆整条会话到 entry 处

// 6. 导入外部 JSONL（会被复制到当前 sessionDir 下）
await runtime.importFromJsonl("/external/path.jsonl");

// 7. 清理——dispose 是 async，必须 await！
//    不 await 会让 session_shutdown 事件来不及触发，扩展拿不到清理机会
await runtime.dispose();
unsubscribe?.();
```

## 方法详解

### `runtime.switchSession(sessionPath, options?)`


| 参数 | 类型 | 含义 |
|------|------|------|
| `sessionPath` | `string` | 目标 JSONL 路径（绝对或相对均可，内部 resolvePath） |
| `options.cwdOverride?` | `string` | 覆盖 session header 里的 cwd（典型场景：把同事的 session 导入自己项目，强制用你的 cwd） |
| `options.withSession?` | `(ctx: ReplacedSessionContext) => Promise<void>` | 替换完成**之后**（rebind 之后）触发的回调。`ctx` 除 `sendUserMessage()` 外，还能 `sendMessage(message, options?)`（发 custom 消息，支持 `triggerTurn` / `deliverAs: steer\|followUp\|nextTurn`），并继承 `ExtensionCommandContext`（可访问 `pi` / `cwd` / `sessionManager` 等扩展上下文）。详见源码 `extensions/types.ts` 的 `ReplacedSessionContext` 定义 |
| `options.projectTrustContextFactory?` | `(cwd: string) => ProjectTrustContext` | 当目标 cwd 不在当前信任列表里时，按需构造信任上下文。一般场景用不到，详见 [B04-project-trust.md](B04-project-trust.md) |

**内部流程**：emitBeforeSwitch("resume") → `SessionManager.open(sessionPath, undefined, cwdOverride)` → `assertSessionCwdExists`（cwd 不存在时抛 `MissingSessionCwdError`） → teardownCurrent → apply(createRuntime) → finishSessionReplacement。

**返回**：`{ cancelled: boolean }`。`cancelled=true` 表示扩展 `session_before_switch` 返回了 `{ cancel: true }`。

### `runtime.newSession(options?)`


| 参数 | 类型 | 含义 |
|------|------|------|
| `options.parentSession?` | `string` | 把当前 session 文件路径写入新 session header 的 `parentSession` 字段，形成父子关系（不复制内容） |
| `options.setup?` | `(sessionManager: SessionManager) => Promise<void>` | 新 session 创建后、rebind 前执行。典型用法：读取 `sessionManager.buildSessionContext()` 获取初始 messages，或注入自定义 system message |
| `options.withSession?` | `(ctx: ReplacedSessionContext) => Promise<void>` | 同 switchSession |

**SessionManager 选择**：当前 session 持久化 → `SessionManager.create(cwd, sessionDir)`；当前 inMemory → `SessionManager.inMemory(cwd)`。

> **A05 横向陷阱**：`SessionManager.inMemory(cwd)` 不会传染 process.cwd，但前提是上层 runtime 的 `this.cwd` 是调用方显式传入的。如果 `createAgentSessionRuntime` 时 cwd 写了 `process.cwd()`，后续 newSession 会一直沿用这个值，自定义组件 SDK 不会回填 cwd。详见 [A05](A05-custom-cwd.md)。

### `runtime.fork(entryId, options?)`


| 参数 | 类型 | 含义 |
|------|------|------|
| `entryId` | `string` | 分叉点 entry 的 id |
| `options.position?` | `"before" \| "at"` | **默认 `before`**。`before` 要求 entry 是 user message，从该 user message 之前分叉出新分支（保留该 user message 作为新分支起点）；`at` 是"克隆"语义——从该 entry 处复制整条会话 |
| `options.withSession?` | 同上 | 同上 |

**触发事件**：`session_before_fork`（不是 `session_before_switch`）。扩展返回 `{ cancel: true }` 可取消。

> **注意**：`SessionBeforeForkResult` 类型里还有一个 `skipConversationRestore?: boolean` 字段（types.ts:1107-1110），但它当前是**保留位（Reserved for future）**——runtime 的 `emitBeforeFork` 只读 `result?.cancel`，**完全忽略** `skipConversationRestore`（agent-session-runtime.ts:150-165）。官方 `docs/extensions.md:444` 也明确标注 "Reserved for future conversation restore control"。写了 `{ skipConversationRestore: true }` **没有任何效果**（静默失败），不要当成可用功能。

**返回**：`{ cancelled: boolean; selectedText?: string }`。`selectedText` 只在 `position="before"` 时有值，是被分叉 user message 的文本（用于 UI 回填输入框）。

### `runtime.importFromJsonl(inputPath, cwdOverride?)`


| 参数 | 类型 | 含义 |
|------|------|------|
| `inputPath` | `string` | 外部 JSONL 路径 |
| `cwdOverride?` | `string` | 覆盖目标 session 的 cwd |

**内部流程**：
1. `resolvePath(inputPath)` + `existsSync` 校验 → 不存在抛 `SessionImportFileNotFoundError`
2. 如果 sessionDir 不存在：`mkdirSync(sessionDir, { recursive: true })` 创建
3. 计算 `destinationPath = join(sessionDir, basename(resolvedPath))`
4. emitBeforeSwitch("resume", destinationPath) —— **可被取消**
5. 如果 `resolve(destinationPath) !== resolvedPath`（即源文件不在 sessionDir 内）：`copyFileSync` 复制
6. `SessionManager.open(destinationPath, sessionDir, cwdOverride)`
7. `assertSessionCwdExists` + teardown + apply + finishSessionReplacement

> **陷阱**：源文件名冲突时会被**直接覆盖**（`copyFileSync` 无 `flag: "wx"`）。如果 sessionDir 下已有同名文件，原始内容会丢失。命名建议带时间戳或 uuid。

### `runtime.dispose()`


**签名**：`async dispose(): Promise<void>`（**必须 await**）

**流程**：emit `session_shutdown` reason="quit" → `beforeSessionInvalidate?.()` → `session.dispose()`。

> **F04 横向陷阱**：`session.dispose()` 内部会调 `agent.abort()`（见 [F04](F04-abort-session.md)）。**不 await `runtime.dispose()` 会让扩展 `session_shutdown` 处理器来不及完成**——返回的 Promise 被 caller 忽略时，emit 的 await chain 会被截断。

## 完整签名速查表

```ts
// 工厂类型
type CreateAgentSessionRuntimeFactory = (options: {
  cwd: string;
  agentDir: string;
  sessionManager: SessionManager;
  sessionStartEvent?: SessionStartEvent;
  projectTrustContext?: ProjectTrustContext;
}) => Promise<CreateAgentSessionRuntimeResult>;

interface CreateAgentSessionRuntimeResult extends CreateAgentSessionResult {
  services: AgentSessionServices;
  diagnostics: AgentSessionRuntimeDiagnostic[];
}

// 创建函数
function createAgentSessionRuntime(
  createRuntime: CreateAgentSessionRuntimeFactory,
  options: {
    cwd: string;
    agentDir: string;
    sessionManager: SessionManager;
    sessionStartEvent?: SessionStartEvent;
  },
): Promise<AgentSessionRuntime>;

// AgentSessionRuntime 关键成员
class AgentSessionRuntime {
  get session(): AgentSession;          // 当前 session（每次替换后变化）
  get services(): AgentSessionServices; // 当前 cwd 绑定服务
  get cwd(): string;                    // 当前有效 cwd（= services.cwd）
  get diagnostics(): readonly AgentSessionRuntimeDiagnostic[];
  get modelFallbackMessage(): string | undefined;

  setRebindSession(cb?: (session: AgentSession) => Promise<void>): void;
  setBeforeSessionInvalidate(cb?: () => void): void;  // UI 拆除用，同步（主要供 TUI/CLI 宿主拆除扩展组件，Web/纯 SDK 集成一般用不到）

  switchSession(path, options?): Promise<{ cancelled: boolean }>;
  newSession(options?): Promise<{ cancelled: boolean }>;
  fork(entryId, options?): Promise<{ cancelled: boolean; selectedText?: string }>;
  importFromJsonl(inputPath, cwdOverride?): Promise<{ cancelled: boolean }>;
  dispose(): Promise<void>;  // ★ async
}
```

> **`diagnostics` 和 `modelFallbackMessage` 读了干嘛**：`diagnostics` 是 `AgentSessionRuntimeDiagnostic[]`（含 `type: "info"\|"warning"\|"error"` + `message`），services 创建时的非致命问题（未知 flag、provider 注册失败等）都进这里；`modelFallbackMessage` 是模型回退提示（如 "Could not restore model X/Y. Using Z/W"）。**宿主应在每次创建/切换后读这两个字段**，向用户展示 warning/error 和模型回退提示——它们不会自动上报，你不读用户就看不到。

## Session 相关事件（扩展层）


| 事件 | 触发时机 | payload 关键字段 | 可取消 |
|------|---------|-----------------|--------|
| `session_start` | 每次 session 创建（startup/reload/new/resume/fork） | `reason`、`previousSessionFile?` | 否 |
| `session_before_switch` | newSession / switchSession / importFromJsonl 之前 | `reason: "new"\|"resume"`、`targetSessionFile?` | **是**（返回 `{ cancel: true }`） |
| `session_before_fork` | fork 之前 | `entryId`、`position` | **是** |
| `session_shutdown` | teardownCurrent / dispose 时 | `reason: "quit"\|"reload"\|"new"\|"resume"\|"fork"`、`targetSessionFile?` | 否 |

> 详见 [04-events.md](../sdk_doc/04-events.md)。

**派发层归属（重要）**：上述四个 session 事件（`session_start` / `session_before_switch` / `session_before_fork` / `session_shutdown`）**全部只在扩展层 `pi.on(...)` 派发**，`session.subscribe(...)` 一个都收不到。`session_start` 由 `extensionRunner.emit(this._sessionStartEvent)` 派发（agent-session.ts:2253），从未走 `this._emit()`；subscribe 层的 `AgentSessionEvent` 类型联合（agent-session.ts:141-183）里根本没有这四个事件类型。

**subscribe 层怎么感知 session 切换**：只能靠 rebind 回调——在新 session 上重新 `session.subscribe(...)` 订阅 agent 类事件（如 `agent_settled`、`message_*`）。但 session_* 本身永远拿不到。这是 SDK 多层架构的静默失败坑：在 subscribe 回调里写 `if (e.type === "session_start") ...` 不会报错，只是永远不命中。详见 [04-events.md](../sdk_doc/04-events.md) 的同名陷阱。

## 常见误期待与陷阱

1. **"切换后我的 subscribe 还有效"**——错。subscribe 绑在具体 AgentSession 实例上，切换后旧订阅对新 session 无效，且旧 session dispose 后不会再触发任何事件。必须在 rebind 回调里退订旧的 + 订阅新的。推荐在新 session 的 subscribe 中监听 `agent_settled` 作为 session 完全稳定的信号——比 `agent_end` 更可靠（retry 场景下 `agent_end` 会提前触发）。
2. **"dispose 是同步的"**——错。`dispose()` 返回 Promise，必须 await。不 await 会让 `session_shutdown` 扩展事件来不及完成。
3. **"agentDir 必须用 getAgentDir()，不能写字面 `~/.pi/agent`"**——部分对，但**原因和直觉相反**。Node 的 fs 确实不展开 `~`，但 SDK 全程走 `resolvePath` → `normalizePath`，后者默认 `expandTilde ?? true`（paths.ts:66-72）**会展开 `~`**，所以传 `~/.pi/agent` 给 `createAgentSessionServices`（agent-session-services.ts:139）能正常工作，**不会失败**。真正该用 `getAgentDir()` 的原因是：它会读 `PI_CODING_AGENT_DIR` 环境变量覆盖（config.ts:516），且 `CONFIG_DIR_NAME` 可被 `pkg.piConfig.configDir` 改成非 `.pi`（config.ts:491）。**硬编码 `~/.pi/agent` 会绕过这两层配置**——用户改了环境变量或 piConfig，你的代码还指向旧路径。
4. **"fork 的 position 参数无所谓"**——错。`before` 要求 entry 是 user message（否则抛 "Invalid entry ID for forking"）；`at` 是克隆语义，对任何 entry 类型都可用。
5. **"importFromJsonl 会校验目标文件是否已存在"**——错。`copyFileSync` 默认覆盖，同名文件会被冲掉。
6. **"扩展会自动迁移到新 session"**——错。必须在 rebind 回调里 `session.bindExtensions(...)` 重新绑定。
7. **"newSession 不会触发 session_before_switch"**——错。newSession 经由 `emitBeforeSwitch("new")`，**会**触发 `session_before_switch` 事件，扩展可返回 `{ cancel: true }` 取消。
8. **"runtime.cwd 会随 switchSession 自动变化"**——对。但前提是 `cwdOverride` 或目标 session header 里有 cwd；都没有时会降级到 `process.cwd()`。
9. **"`agent_end` 是 session 完全稳定的信号"**——不完全对。`agent_end` 在 retry 场景下会提前触发（`willRetry: true`）。**`agent_settled`** 才是「所有 retry / compaction / steer 队列消费完毕」的真正稳定信号，两层都派发。

## 变体与延伸

- 持久化策略 / SessionManager 基础 → 见 [F01](F01-session-persistence.md)
- abort 行为与 dispose 内部机制 → 见 [F04](F04-abort-session.md)
- steer 与 session 状态查询 → 见 [F05](F05-steer-session.md)
- 多 Agent 协作中的 session 切换 → 见 [H06](H06-multi-agent.md)
- 子 Agent 调度 → 见 [I05](I05-subagent.md)
- 自定义 cwd 与 inMemory 默认值陷阱 → 见 [A05](A05-custom-cwd.md)
- 完整 API 参考 → 见 [sdk_doc/03-agent-session-runtime.md](../sdk_doc/03-agent-session-runtime.md)
