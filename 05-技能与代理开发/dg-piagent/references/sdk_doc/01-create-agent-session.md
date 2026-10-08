# createAgentSession

## 这是什么

`createAgentSession` 是 pi-agent SDK 的**主入口函数**。它接受全部配置，返回一个可用的 `AgentSession` 实例。所有 Agent 创建都从这里开始。

## 函数签名

```ts
function createAgentSession(options?: CreateAgentSessionOptions): Promise<CreateAgentSessionResult>;
```

## 参数表：CreateAgentSessionOptions

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `cwd` | `string` | `options.sessionManager?.getCwd()` ?? `process.cwd()` | 项目根目录，用于发现 `.pi/` 配置和扩展；若未传 `cwd` 但传了 `sessionManager`，先用其 `getCwd()` |
| `agentDir` | `string` | `~/.pi/agent`（来自 `getAgentDir()`） | 全局配置目录 |
| `modelRuntime` | `ModelRuntime` | `await ModelRuntime.create({ authPath, modelsPath })` | 模型/认证运行时（统一 `authStorage` + `modelRegistry`，二者已不在 Options 上） |
| `model` | `Model<any>` | 从 settings 读取，否则选第一个可用 | 指定使用的模型 |
| `thinkingLevel` | `ThinkingLevel` | 从 settings 读取，否则 `"medium"` | 推理深度：`"off"` / `"minimal"` / `"low"` / `"medium"` / `"high"` / `"xhigh"` / `"max"`（7 档，按模型能力 `clampThinkingLevel()` 自动降级） |
| `scopedModels` | `Array<{model, thinkingLevel?}>` | `undefined` | 候选模型列表；TUI 下用 Ctrl+P 切换，SDK 下用 `session.cycleModel("forward" \| "backward")` 编程切换 |
| `noTools` | `"all"` / `"builtin"` | `undefined` | 抑制工具：`"all"` 禁用全部，`"builtin"` 禁用内置但保留扩展工具 |
| `tools` | `string[]` | 默认内置工具 | 工具名白名单（只在列表中的工具可用） |
| `excludeTools` | `string[]` | `undefined` | 工具名黑名单，在 `tools` 之后生效（两者同传时，先白名单再黑名单过滤） |
| `customTools` | `ToolDefinition[]` | `undefined` | 自定义工具列表，使用 `defineTool()` 定义 |
| `resourceLoader` | `ResourceLoader` | `DefaultResourceLoader` | 资源加载器：控制 prompt/skills/themes/扩展 的发现与加载 |
| `sessionManager` | `SessionManager` | `SessionManager.create(cwd, getDefaultSessionDir(cwd, agentDir))` | 会话管理器：控制会话的持久化方式；默认持久化到 `~/.pi/agent/sessions/<编码后的-cwd>/`（cwd 被编码成形如 `--path-with-dashes--` 的安全目录名，**不在项目目录内**），传 `SessionManager.inMemory(cwd)` 可走内存 |
| `settingsManager` | `SettingsManager` | `SettingsManager.create(cwd, agentDir)` | 设置管理器 |
| `sessionStartEvent` | `SessionStartEvent` | `undefined` | 扩展运行时的会话启动事件元数据 |

## 返回值：CreateAgentSessionResult

```ts
interface CreateAgentSessionResult {
  session: AgentSession;           // 创建的会话实例
  extensionsResult: LoadExtensionsResult;  // 扩展加载结果
  modelFallbackMessage?: string;   // 模型回退警告，详见下方「会话恢复行为」
}
```

`modelFallbackMessage` 在两种场景下会赋值：

1. **会话保存的模型无法恢复**——比如上次用了 `anthropic/claude-opus-4-5`，但本次启动后该模型未配置 API key。此时若还能找到其他可用模型，消息形如 `Could not restore model anthropic/claude-opus-4-5. Using <provider>/<modelId>`；若彻底找不到可用模型，`modelFallbackMessage` 会被**覆盖**为 `formatNoModelsAvailableMessage()` 的输出（形如 `No models available. Use /login to log into a provider via OAuth or API key. See: ...`），原 `"Could not restore..."` 消息不保留。
2. **完全无可用模型**——`modelRuntime` 里没有任何已配置 API key 的模型（`hasConfiguredAuth` 全部失败），消息由 `formatNoModelsAvailableMessage()` 生成。注意：此时 `session` 仍会返回，**不会 throw**；真正的报错发生在 `session.prompt()` 时（`!this.model` → throw `formatNoModelSelectedMessage()`）。

> 💡 **迁移提示**：`CreateAgentSessionOptions` 上的 `authStorage` / `modelRegistry` 参数已被 `modelRuntime` 整体替换。旧代码 `{ authStorage, modelRegistry }` → 新代码 `{ modelRuntime: await ModelRuntime.create({ authPath, modelsPath }) }`。若不传 `modelRuntime`，`createAgentSession` 会用默认路径 `<agentDir>/auth.json` + `<agentDir>/models.json` 自动构造一个。`AuthStorage` 类不再从包根导出（`ModelRegistry` 类仍从包根导出，可用）；一次性读取 auth.json 用 `readStoredCredential()`（见 [sdk_doc/05](05-auth-model-registry.md)）。

## 用法示例

### 最简启动（全默认）

```ts
import { createAgentSession } from "@earendil-works/pi-coding-agent";

const { session } = await createAgentSession();
try {
  await session.prompt("Hello!");
} finally {
  session.dispose();  // 必须释放：清理监听器、中止重试/压缩/分支摘要/bash 任务、使扩展上下文失效、断开 agent 连接
}
```

### 指定模型 + 思考等级

```ts
import { getModel } from "@earendil-works/pi-ai/compat"; // 注意：getModel 已标记 @deprecated，新代码建议用 getBuiltinModel from "@earendil-works/pi-ai/providers/all" 或 Models.getModel()
import { createAgentSession } from "@earendil-works/pi-coding-agent";

const { session } = await createAgentSession({
  model: getModel("anthropic", "claude-opus-4-5"),
  thinkingLevel: "high",
});
```

### 工具白名单（只读模式）

```ts
const { session } = await createAgentSession({
  tools: ["read", "grep", "find", "ls"],  // 只允许读操作
});
```

### 完整自定义

```ts
const modelRuntime = await ModelRuntime.create({ authPath, modelsPath });
const { session } = await createAgentSession({
  cwd: "/path/to/project",
  model: getModel("openai", "gpt-5"),
  thinkingLevel: "medium",
  tools: ["read", "bash", "my_tool"],
  customTools: [myToolDefinition],
  resourceLoader: myResourceLoader,
  sessionManager: SessionManager.inMemory(),
  modelRuntime, // 替代 authStorage + modelRegistry
  settingsManager: mySettingsManager,
});
```

## 关键细节

### 工具白名单/黑名单/抑制三者的优先级（★ 易踩坑）

三者的实际求值顺序（源码 `sdk.ts` 的 `initialActiveToolNames` 求值式）：

```text
initialActiveToolNames =
  (tools 传了？ → 用 tools
   否则 noTools 传了？ → 用 []
   否则 → 默认 [read, bash, edit, write])
  .filter(name => !excludeTools?.includes(name))
```

> ⚠️ **`noTools: "all"` 与 `"builtin"` 的关键差异**：上面代码块把两者都画成 `→ []`（初始激活列表确实都清空），但还有一个 `allowedToolNames` 变量控制**扩展工具是否受影响**——`noTools: "all"` 会把 `allowedToolNames` 也置为 `[]`（连扩展工具一并清空），`noTools: "builtin"` 则保持 `allowedToolNames = undefined`（扩展工具仍可用）。

要点：

- **`tools` 优先于 `noTools`**。若同时传 `tools: ["read"]` 和 `noTools: "all"`，`noTools` 被**完全忽略**，结果是 `["read"]`。源码注释原话：`noTools` 是"当未提供 allowlist 时的默认抑制模式"。
- `noTools: "builtin"` 表示禁用内置工具但**保留扩展/自定义工具**；`noTools: "all"` 则把初始激活列表清空。
- `excludeTools` 在最后一步生效，无论上面怎么算，最终都会再过滤一遍黑名单。
- **`tools` 白名单会统一过滤内置 + 扩展 + 自定义工具**。这是最容易踩的坑：很多人以为扩展工具不受 `tools` 限制，**并非如此**。源码 `_refreshToolRegistry` 里有一道 `isAllowedTool` 滤网，对**所有**工具（内置、扩展注册的、`customTools` 传入的）一视同仁地按 `allowedToolNames`（即 `options.tools`）过滤。所以传 `tools: ["read"]` 时，不在列表里的扩展工具**进不了 registry、也进不了 active 列表**。

  对应的三种"禁用扩展工具"姿势：

  | 目标 | 写法 | 结果 |
  |------|------|------|
  | 只留指定工具（连扩展工具一起挡掉） | `tools: ["read"]` | 只有 `read` 可用，扩展工具全被挡 |
  | 禁内置、留扩展 | `noTools: "builtin"` | 内置清空，扩展/自定义工具仍可用（`allowedToolNames` 保持 `undefined`） |
  | 全禁（含扩展） | `noTools: "all"` 或 `tools: []` | `allowedToolNames` 被置为 `[]`，所有工具清空 |

  若想从根上不加载某扩展，用自定义 `resourceLoader` 跳过它（这是另一条独立路径，与 `tools`/`noTools` 无关）。

### 其他细节

- `model` 从 `@earendil-works/pi-ai/compat` 的 `getModel()` 获取（注意是 `/compat` 子路径，不是包根；源码 JSDoc 里写的 `@earendil-works/pi-ai` 是其自身历史残留），不是随便传字符串
- `thinkingLevel` 会被 `clampThinkingLevel()` 按模型能力自动降级（如传 `"high"` 但模型不支持，会回退到模型支持的最高档）；另外，若 model 解析失败（`model === undefined`），thinkingLevel 会被强制设为 `"off"`
- **自定义 `resourceLoader` 必须自己先 `await loader.reload()`**——只有未传 `resourceLoader` 时 `createAgentSession` 才会自动 reload（源码 `sdk.ts`）
- 记得在 `try/finally` 中调用 `session.dispose()`
- **项目级配置的信任开关（`projectTrusted`）不在 `createAgentSession` 参数上**——它属于传入的 `settingsManager`。默认 `SettingsManager.create()` 视项目为可信（`projectTrusted=true`），会正常加载 `<cwd>/.pi/settings.json`。但若自定义 `settingsManager` 时误传 `projectTrusted=false`，项目级配置会**静默失效**（源码 `loadFromStorage`：`scope === "project" && !projectTrusted` 直接返回 `{}`）。这是 SDK 集成的隐性陷阱：配置没生效但没有任何报错。

## 会话恢复行为（★ 重要：第二次调用会"接着上次"）

只要 `sessionManager` 默认走的是持久化模式（非 `inMemory`），且会话文件里已有 messages，`createAgentSession()` 就会**自动恢复**——这不是显式参数，而是隐式行为。源码逻辑在 `sdk.ts`：

1. **触发判定**：`sessionManager.buildSessionContext().messages.length > 0` 即认为有历史会话。注意：`existingSession.model` 可能为 `null`（会话无保存的模型信息），此时跳过模型恢复分支，直接走 `findInitialModel()` 兜底。
2. **模型恢复**：若未显式传 `options.model`，按以下顺序回填：
   - 从会话保存的模型信息查 `modelRuntime.getModel(provider, modelId)`；
   - 若该模型未配置 API key（`modelRuntime.hasConfiguredAuth(provider)` 失败），触发 `modelFallbackMessage`，再走 `findInitialModel()` 兜底；
   - 若兜底也找不到，仅赋告警文案，`session.model` 实际为 `undefined`，待 `prompt()` 时 throw。
3. **thinkingLevel 恢复**：会话分支里有 `thinking_level_change` 条目就按那个走，否则用 settings 默认值，再经 `clampThinkingLevel()` 按模型能力降级。
4. **消息历史恢复**：`agent.state.messages` 直接被覆写为 `existingSession.messages`，相当于"接着上次对话"。

**想强制开新会话**有几种方式：

```ts
// 方式 1：用内存 sessionManager（不持久化）
import { SessionManager } from "@earendil-works/pi-coding-agent";
const { session } = await createAgentSession({
  sessionManager: SessionManager.inMemory(process.cwd()),
});

// 方式 2：默认持久化，但传一个全新的空目录作为 sessionDir
import { SessionManager, getAgentDir } from "@earendil-works/pi-coding-agent";
import { tmpdir } from "node:os";
import { join } from "node:path";
const { session } = await createAgentSession({
  // 每次都用一个新的临时目录，等同"新会话"；也可换成你自己的固定空目录
  sessionManager: SessionManager.create(process.cwd(), join(tmpdir(), `pi-session-${Date.now()}`)),
});

// 方式 3：默认持久化，但调用前清空旧的 session 目录（要复用默认路径时用这个）
//   注意：默认 session 目录是 ~/.pi/agent/sessions/<编码后的-cwd>/，路径由 getDefaultSessionDir 内部生成、未公开导出。
//   想清空它，最稳妥是用 SessionManager.create(cwd, <你选的目录>) 自己掌控路径，然后 fs.rmSync 清空：
import { rmSync } from "node:fs";
const mySessionDir = join(getAgentDir(), "sessions", "my-fixed-session");
rmSync(mySessionDir, { recursive: true, force: true });
const { session: s2 } = await createAgentSession({
  sessionManager: SessionManager.create(process.cwd(), mySessionDir),
});
```

**想做"continue 上次会话"**：直接用默认参数调用 `createAgentSession()` 即可，无需任何 `continue` 标志——源码 JSDoc 里残留的 `continueSession: true` 是历史残留参数，**当前 Options 类型里已不存在**。
