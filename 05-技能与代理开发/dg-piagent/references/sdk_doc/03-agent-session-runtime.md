# 03 - 会话运行时 (AgentSessionRuntime)

## 这是什么

当你需要**替换活跃会话**（新建/切换/分叉/导入）时，用 `AgentSessionRuntime`。它管理 `AgentSession` 的完整生命周期——创建、替换、销毁——并在每次替换时重建 cwd 绑定的服务（settings、resourceLoader、modelRuntime 等）。

**如果只需要 prompt / subscribe / steer / abort**，直接用 `createAgentSession()` 返回的 `AgentSession` 就够了，不需要 Runtime。详见 [02-agent-session.md](02-agent-session.md)。

## 创建运行时

```ts
async function createAgentSessionRuntime(
  createRuntime: CreateAgentSessionRuntimeFactory,
  options: {
    cwd: string;            // 项目根目录，所有工具在此目录下执行
    agentDir: string;       // Pi 全局配置目录（~/.pi/agent），存 auth.json、models.json、skills 等
    sessionManager: SessionManager;  // 会话管理器，决定持久化策略（文件/inMemory）
    sessionStartEvent?: SessionStartEvent;  // 扩展层的 session_start 事件元数据
  },
): Promise<AgentSessionRuntime>
```

`createRuntime` 是工厂函数，**会被存储复用**——每次 `newSession` / `switchSession` / `fork` / `importFromJsonl` 都会重新调用它来重建 cwd 绑定服务：

```ts
type CreateAgentSessionRuntimeFactory = (options: {
  cwd: string;
  agentDir: string;
  sessionManager: SessionManager;
  sessionStartEvent?: SessionStartEvent;
  projectTrustContext?: ProjectTrustContext;  // switchSession 内部传入，用户一般不直接传
}) => Promise<CreateAgentSessionRuntimeResult>;
```

返回值：

```ts
interface CreateAgentSessionRuntimeResult extends CreateAgentSessionResult {
  services: AgentSessionServices;                      // cwd 绑定服务（见底部 AgentSessionServices 结构）
  diagnostics: AgentSessionRuntimeDiagnostic[];        // 启动过程中收集的非致命问题（info/warning/error），由调用方决定是否展示
}
```

### 典型创建

工厂内部通常用 `createAgentSessionServices()` + `createAgentSessionFromServices()` 两步走：
先创建 cwd 绑定服务，再基于服务创建 session。这样可以在创建 session 前对 services 做额外配置。

```ts
import {
  createAgentSessionRuntime,
  createAgentSessionServices,
  createAgentSessionFromServices,
  SessionManager,
} from "@earendil-works/pi-coding-agent";

const runtime = await createAgentSessionRuntime(
  async ({ cwd, agentDir, sessionManager, sessionStartEvent }) => {
    const services = await createAgentSessionServices({ cwd, agentDir });
    const result = await createAgentSessionFromServices({
      services,
      sessionManager,
      sessionStartEvent,
      customTools: [myCustomTool],
    });
    return { ...result, services };
  },
  {
    cwd: process.cwd(),
    agentDir: "/home/user/.pi/agent",
    sessionManager: SessionManager.continueRecent(process.cwd()),
  },
);
```

## 实例属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `runtime.session` | `AgentSession` | 当前活跃的 session。**注意：`newSession`/`switchSession`/`fork`/`importFromJsonl` 后这个引用会变**，旧 session 已销毁 |
| `runtime.services` | `AgentSessionServices` | 当前 cwd 绑定的基础设施服务（auth、settings、model、resourceLoader） |
| `runtime.cwd` | `string` | 当前工作目录（即 `services.cwd`） |
| `runtime.diagnostics` | `readonly AgentSessionRuntimeDiagnostic[]` | 启动/切换过程中收集的非致命问题，`{ type: "info"|"warning"|"error", message: string }` |
| `runtime.modelFallbackMessage` | `string \| undefined` | 模型不可用时的回退提示信息，如 `"Could not restore model anthropic/claude-opus-4-5. Using openai/gpt-5"` |

## 实例方法

### `runtime.newSession(options?)`

创建全新会话，替换当前 session。

```ts
async newSession(options?: {
  parentSession?: string;   // 父会话文件路径（从已有会话继承上下文）
  setup?: (sessionManager: SessionManager) => Promise<void>;  // 新 session 创建后、rebind 前调用，可在此追加初始消息
  withSession?: (ctx: ReplacedSessionContext) => Promise<void>;  // rebind 之后调用；ctx 携带 sendMessage/sendUserMessage，可在新 session 上立即触发对话（types.ts:394-404）
}): Promise<{ cancelled: boolean }>
```

内部流程：emit `session_before_switch`（可被取消）→ 创建新 SessionManager → 销毁旧 session → 调 factory 创建新 runtime → 调 setup → rebind（重绑扩展 + 事件订阅）。

```ts
const result = await runtime.newSession({
  setup: async (sm) => {
    sm.appendMessage(/* 初始系统消息 */);
  },
});
if (result.cancelled) {
  // 扩展拒绝了切换
}
```

### `runtime.switchSession(sessionPath, options?)`

切换到指定 JSONL 会话文件。

```ts
async switchSession(
  sessionPath: string,       // JSONL 会话文件路径
  options?: {
    cwdOverride?: string;    // 目标会话的 cwd 不存在时，用此路径替代
    withSession?: (ctx: ReplacedSessionContext) => Promise<void>;  // ctx 支持 sendMessage/sendUserMessage（见 types.ts:394-404）
    projectTrustContextFactory?: (cwd: string) => ProjectTrustContext;
  },
): Promise<{ cancelled: boolean }>
```

- 触发 `session_before_switch`（reason: `"resume"`），可被取消
- 通过 `SessionManager.open(sessionPath)` 加载目标会话
- 断言 cwd 存在（不存在且无 `cwdOverride` 时抛 `MissingSessionCwdError`）

### `runtime.fork(entryId, options?)`

从指定 entry 分叉出新会话分支。

```ts
async fork(
  entryId: string,
  options?: {
    position?: "before" | "at";   // 默认 "before"
    withSession?: (ctx: ReplacedSessionContext) => Promise<void>;  // ctx 支持 sendMessage/sendUserMessage（见 types.ts:394-404）
  },
): Promise<{ cancelled: boolean; selectedText?: string }>
```

- `position: "before"`（默认）— 在该 entry 的**父节点**处分叉，不包含该 entry 本身。这样你可以重新编辑这条用户消息。**限制：仅对 user message 有效**，其他类型 entry 会 throw
- `position: "at"` — 在该 entry **处**分叉，**包含**该 entry。无法编辑，只能从该点继续
- 触发 `session_before_fork` 事件，可被取消
- 返回的 `selectedText` 仅在 `"before"` 时有值，是该用户消息的文本内容

### `runtime.importFromJsonl(inputPath, cwdOverride?)`

从外部 JSONL 文件导入会话并切换。

```ts
async importFromJsonl(inputPath: string, cwdOverride?: string): Promise<{ cancelled: boolean }>
```

- 若源文件不在 session 目录中，先复制进去再打开
- 触发 `session_before_switch`（reason: `"resume"`）
- 源文件不存在时 throw `SessionImportFileNotFoundError`
- 目标会话 cwd 无法解析且无 `cwdOverride` 时 throw `MissingSessionCwdError`

### `runtime.dispose()`

清理运行时：触发 `session_shutdown`（reason: `"quit"`）→ 执行 `beforeSessionInvalidate` 回调 → 销毁 session。

```ts
async dispose(): Promise<void>
```

### 其他方法

```ts
// 设置 session 替换后的重绑回调。每次 newSession/switchSession/fork/import 后自动调用。
// 典型用途：重新绑定扩展、重新订阅事件（见下方"订阅陷阱"）。
setRebindSession(rebindSession?: (session: AgentSession) => Promise<void>): void;

// 设置同步回调：在 session_shutdown 事件处理完毕后、session.dispose() 前执行。
// 用于 UI 清理等不可让出事件循环的操作。
setBeforeSessionInvalidate(beforeSessionInvalidate?: () => void): void;
```

## ★ 订阅陷阱：session 替换后必须重订阅 + 重绑扩展

`session.subscribe()` 的监听器绑定在**特定** `AgentSession` 实例上。`newSession` / `switchSession` / `fork` / `importFromJsonl` 会销毁旧 session、创建新 session——旧监听器随之失效，**不会自动迁移**。

**正确模式**：用 `setRebindSession` 统一管理重订阅 + 重绑扩展。参考官方示例 `examples/sdk/13-session-runtime.ts` 的 `bindSession()` 模式——先 `bindExtensions({})` 再 `subscribe(...)`：

```ts
let session = runtime.session;
let unsubscribe: (() => void) | undefined;

async function bindSession() {
  // 旧订阅必须取消
  unsubscribe?.();
  // 绑定到新 session
  session = runtime.session;
  // ★ 必须重新 bindExtensions：它会设置 UI 上下文、重发 session_start 事件、触发 resources_discover
  await session.bindExtensions({});
  unsubscribe = session.subscribe(handleEvent);
}

runtime.setRebindSession(async () => {
  await bindSession();
});

// 首次也要显式 bind 一次
await bindSession();

// 之后任意切换都会自动重绑
await runtime.newSession();
```

**两步都不能省**：

- **漏掉 `subscribe`**：切换后的 session 事件你全都收不到，表现为"切换后 agent 不响应了"
- **漏掉 `bindExtensions`**：新 session 没有 UI 上下文、扩展收不到 `session_start`、`resources_discover` 不触发——表现为"切换后扩展静默失效"（skill/prompt/theme 等扩展资源不加载、扩展命令不响应）

## 完整生命周期

```
createAgentSessionRuntime()
  |
  v
[活跃 Session] ---newSession()-------> [旧 session 销毁] ---factory---> [新 Session]
  |                                                                     |
  +---switchSession()---> [旧 session 销毁] ---factory---> [新 Session]  |
  |                                                                     |
  +---fork()-----------> [旧 session 销毁] ---factory---> [新 Session]   |
  |                                                                     |
  +---importFromJsonl()-> [旧 session 销毁] ---factory---> [新 Session]  |
  |                                                                     |
  +---dispose()--------> [session_shutdown + dispose]                    |
                                                                        |
  每次替换流程：                                                         |
  session_before_* → 创建新 SessionManager → 销毁旧 session              |
    → factory 创建新 runtime → setup(仅 newSession) → rebind             |
```

## AgentSessionServices 结构

```ts
interface AgentSessionServices {
  cwd: string;                       // 项目根目录
  agentDir: string;                  // Pi 全局配置目录
  modelRuntime: ModelRuntime;        // 模型/认证运行时（统一 authStorage + modelRegistry）
  settingsManager: SettingsManager;  // 设置管理器（.pi/settings.json）
  resourceLoader: ResourceLoader;    // 资源加载器（skills/prompts/themes/扩展）
  diagnostics: AgentSessionRuntimeDiagnostic[];  // 创建过程中收集的诊断信息
}
```

通过 `createAgentSessionServices(options)` 创建，常用参数：

| 参数 | 说明 |
|------|------|
| `cwd` | 项目根目录 |
| `agentDir` | Pi 全局配置目录，默认 `~/.pi/agent` |
| `modelRuntime` | 自定义模型/认证运行时（默认从 `<agentDir>/auth.json` + `<agentDir>/models.json` 构造） |
| `settingsManager` | 自定义设置管理器 |
| `modelRuntimeSignal` | 创建时网络模型刷新的取消信号 |
| `resourceLoaderOptions` | 传给 `DefaultResourceLoader` 的选项，可设 `additionalSkillPaths`、`extensionFactories` 等 |
| `resourceLoaderReloadOptions` | 传给 `resourceLoader.reload()` 的选项 |
| `extensionFlagValues` | 扩展 flag 值映射（已注册 flag 才生效，未知 flag 会作为 diagnostic 报错） |

> 💡 **迁移提示**：旧的 `authStorage` + `modelRegistry` 两个字段被 `modelRuntime` 整体替换。`AgentSessionServices` 接口里只剩 `modelRuntime`，扩展内部访问 API Key 解析、模型发现都通过 `modelRuntime` 或 `ctx.modelRegistry`（仍可用的薄包装器，内部委托 `modelRuntime`）。

## 关键注意事项

1. **Factory 被多次调用**：传给 `createAgentSessionRuntime()` 的 factory 会在初始创建及后续每次 `newSession` / `switchSession` / `fork` / `importFromJsonl` 时重新调用。每次都会重建 cwd 绑定服务，消耗不小。factory 内部不要做重量级一次性操作
2. **Factory 抛错会让 runtime 进入不可用状态**（重要陷阱）：切换流程是「先 `teardownCurrent` 销毁旧 session → 再 `apply(createRuntime(...))`」。如果 factory 抛错，旧 session 已销毁、新 session 未创建，runtime 持有的是**已销毁的旧 session 引用**。调用方必须 try/catch 并自行重建（例如重新调 `createAgentSessionRuntime`）——否则后续访问 `runtime.session` 会得到失效引用
3. **`session_before_switch` 可取消**：扩展 handler 返回 `{ cancel: true }` 会阻止切换，此时旧 session 保持活跃
4. **`fork` 对持久化/非持久化行为不同**：持久化 session 会创建新的 JSONL 文件；inMemory session 在内存中分叉
5. **`cwdOverride`**：`switchSession` / `importFromJsonl` 时若目标会话的 cwd 不存在，可传入替代路径。不传且 cwd 不存在则抛 `MissingSessionCwdError`
6. **dispose 后内部 session 已 dispose**：再次调用 runtime 方法行为未定义（源码未对 dispose 做幂等保护，"不可再用"是逻辑后果而非硬约束）
