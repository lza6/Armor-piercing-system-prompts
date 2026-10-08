# 场景：完全手动组装所有组件 (H01)

## 这是什么

**完全手动组装** = 绕过 `createAgentSession()` 的"缺省即自动创建"行为，自己 `new` 出每一个组件（`AuthStorage` / `ModelRegistry` / `SettingsManager` / `SessionManager` / `ResourceLoader`），然后一股脑传给 `createAgentSession()`。

适合：

- 想完全控制系统提示词、扩展、技能、提示词模板的来源（比如从数据库或远程拉取，而不是从 `~/.pi/agent/` 读文件）
- 想完全控制会话持久化路径（比如把会话写到加密存储，而不是默认 JSONL 文件）
- 想注入测试用的 in-memory 组件，避免任何文件 I/O

**不是什么**：

- **不改 Agent loop 的执行逻辑**——`Agent` 内部的"LLM 调用 → 工具执行 → 循环"流程由 SDK 固定，本场景只换"喂给 Agent 的资源"，不换"Agent 怎么用资源"。要改 loop 行为得用 [扩展层 hooks](../sdk_doc/07-extensions-api.md)（`before_agent_start` / `context` / `tool_call` 等）
- **不换 Agent 类本身**——`createAgentSession()` 内部 `new Agent(...)`，用户没有改 Agent 类的入口。要换 Agent 类只能用更底层的 `agent-core` 包，绕开 `createAgentSession`

## 什么时候用 / 不用会怎样

| 场景 | 用 H01 | 不用会怎样 |
|------|-------|----------|
| 系统提示词要从数据库读，不读 `~/.pi/agent/AGENTS.md` | ✅ 手写 `ResourceLoader.getSystemPrompt()` | 默认 `DefaultResourceLoader` 会按"全局 → 父目录 → 当前目录"顺序扫 `AGENTS.md` / `CLAUDE.md` |
| 会话要写到加密存储 / 自定义目录 | ✅ 手写 `SessionManager` 子类或用 `SessionManager.create(cwd, customDir)` | 默认 `~/.pi/agent/sessions/<encoded-cwd>/` |
| 单元测试，不能有文件 I/O | ✅ 全部用 `.inMemory()` 工厂 | 默认会创建文件，污染测试环境 |
| 只想关掉某个具体资源（如不要扩展） | ❌ 用 `DefaultResourceLoader({ noExtensions: true })` 更简单 | 本场景是"全手写"，杀鸡用牛刀 |
| 只想换模型 provider | ❌ 见 [H02 自定义 Provider](H02-custom-provider.md) | H01 是"换基础设施"，不是"换模型" |

## ⚠️ 最大陷阱：双 cwd 不一致

**`createAgentSession` 不回填 cwd 给你传入的自定义组件**。

```ts
// 错误：sessionManager 的 cwd 永远是 process.cwd()
const sessionManager = SessionManager.inMemory();  // ← 内部 cwd = process.cwd()
const { session } = await createAgentSession({
  cwd: "/my/project",          // ← 这个 cwd 只给 AgentSession 自己
  sessionManager,               // ← sessionManager.getCwd() 仍是 process.cwd()
});
// 后果：sessionManager 写会话头里的 cwd 字段 = process.cwd()，不是 /my/project
// session.sessionManager.getCwd() ≠ session 实际工作的 cwd
```

`createAgentSession` 内部逻辑（sdk.ts 的 `createAgentSession`）：

- `cwd = resolvePath(options.cwd ?? options.sessionManager?.getCwd() ?? process.cwd())` —— 先看你传的 `cwd`，没有再问 `sessionManager.getCwd()`，最后 fallback 到 `process.cwd()`
- 但**只读取，不写入**——拿到的 cwd 不会反向 set 给 sessionManager

**正确做法**：自定义组件创建时就传 cwd，让两边的 cwd 一致。

```ts
const cwd = "/my/project";
const sessionManager = SessionManager.inMemory(cwd);  // ← 显式传
const { session } = await createAgentSession({ cwd, sessionManager });
```

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `getBuiltinModel(provider, id)` | 从内置 catalog 读取模型定义（不查 ModelRuntime） | [sdk_doc/05-auth-model-registry.md](../sdk_doc/05-auth-model-registry.md) |
| `ModelRuntime.create(options?)` | 推荐入口：创建模型 + 认证运行时。默认 `~/.pi/agent/auth.json` + `~/.pi/agent/models.json` | [sdk_doc/05-auth-model-registry.md](../sdk_doc/05-auth-model-registry.md) |
| `SettingsManager.inMemory(settings)` | 创建无文件 I/O 的设置管理器 | [sdk_doc/13-settings-manager.md](../sdk_doc/13-settings-manager.md) |
| `SessionManager.inMemory(cwd?)` | 创建无文件持久化的会话管理器。**cwd 必须显式传，否则固定为 `process.cwd()`** | [sdk_doc/12-session-manager.md](../sdk_doc/12-session-manager.md) |
| `ResourceLoader` 接口 | 手写资源加载器（签名见下方） | [sdk_doc/08-resource-loader.md](../sdk_doc/08-resource-loader.md) |
| `createAgentSession(options)` | 手动传所有组件。传入自定义组件就替代默认，不会"禁用"——缺省组件仍会自动创建 | [sdk_doc/01-create-agent-session.md](../sdk_doc/01-create-agent-session.md) |

> ⚠️ `AuthStorage` 不再从包根导出，`ModelRegistry.create()` 静态工厂已删除（但 `ModelRegistry` 类本身仍从包根导出，作为扩展层同步兼容门面），`createAgentSession` 的 options 只接 `modelRuntime`，不接 `authStorage` / `modelRegistry`。H01 场景里的「自底向上创建组件」从「先建 AuthStorage → 再建 ModelRegistry」简化为「一步 `ModelRuntime.create()`」。

> **`createAgentSessionServices` 不适合本场景**：它内部强制 `new DefaultResourceLoader(...)`（agent-session-services.ts 的 `createAgentSessionServices`），不支持传入自定义 `ResourceLoader`。本场景的核心就是手写 ResourceLoader，所以必须直接用 `createAgentSession`。

## 组件依赖关系

```
ModelRuntime（含认证 + 模型列表）
                   │
                   ├── settingsManager ── DefaultResourceLoader
                   │                    （createAgentSession 内部自动创建）
                   │
createAgentSession ─┤
                   │
                   ├── sessionManager    ← 你提供就用你的，不提供就 SessionManager.create(cwd)
                   │
                   └── resourceLoader    ← 你提供就用你的，不提供就 DefaultResourceLoader
                                          （会自动调 .reload()）
```

**关键点**：`ModelRuntime` / `SettingsManager` / `SessionManager` / `ResourceLoader` 之间通过 `createAgentSession` 的 options 组装，**互相不直接引用**（ModelRuntime 自带 credentials，不再依赖外部 AuthStorage；`DefaultResourceLoader` 需要 `SettingsManager`，但构造时 `settingsManager` 可选——缺省自动 `SettingsManager.create(cwd, agentDir)` 会创建文件 I/O；如想要无 I/O，必须显式传 `SettingsManager.inMemory()`）。手写 ResourceLoader 时不必注入 SettingsManager——你要什么自己读。

## ResourceLoader 接口签名

完整签名（`ResourceLoader` 接口，含 11 个方法）。⚠️ 其中 `getSystemPromptSource` / `getAppendSystemPromptSources` 是 v0.83.0 新增方法，当前发布版（v0.80.x）的 d.ts 只有 9 个方法——v0.80.x 用户去掉这两行即可编译通过：

```ts
interface ResourceLoader {
  // 扩展：返回 LoadExtensionsResult（不是数组！）
  getExtensions(): LoadExtensionsResult;
  // { extensions: Extension[]; errors: Array<{path;error}>; runtime: ExtensionRuntime }

  // 技能/提示词/主题：都返回带 diagnostics 的对象
  getSkills(): { skills: Skill[]; diagnostics: ResourceDiagnostic[] };
  getPrompts(): { prompts: PromptTemplate[]; diagnostics: ResourceDiagnostic[] };
  getThemes(): { themes: Theme[]; diagnostics: ResourceDiagnostic[] };

  // agents.md 文件
  getAgentsFiles(): { agentsFiles: Array<{ path: string; content: string }> };

  // 系统提示词
  getSystemPrompt(): string | undefined;
  getSystemPromptSource(): { path: string } | undefined;   // ⚠️ v0.83.0 新增，v0.80.x 发布版不含此方法
  getAppendSystemPrompt(): string[];  // 追加到系统提示词末尾
  getAppendSystemPromptSources(): Array<{ path: string }>; // ⚠️ v0.83.0 新增，v0.80.x 发布版不含此方法

  // 路径扩展（运行时添加扩展/技能/提示词路径）
  extendResources(paths: ResourceExtensionPaths): void;

  // 重新加载（createAgentSession 不会调，需要你自己调或在适当时机调）
  reload(options?: ResourceLoaderReloadOptions): Promise<void>;
}
```

**注意**：`getExtensions` / `getSkills` / `getPrompts` / `getThemes` / `getAgentsFiles` 必须返回带正确形状的对象（不能返回 `[]`）；`getAppendSystemPrompt` / `getAppendSystemPromptSources` 返回数组类型，可返回空数组 `[]`；`getSystemPrompt` 返回 `string | undefined`，`extendResources` 返回 `void`。下面是最小可用模板。

## 核心代码

> ⚠️ 以下代码中 `getSystemPromptSource` / `getAppendSystemPromptSources` 需要 v0.83.0+。v0.80.x 用户去掉这两个方法即可编译通过。

```ts
import { getBuiltinModel } from "@earendil-works/pi-ai/providers/all";
import {
  ModelRuntime,
  createAgentSession,
  createExtensionRuntime,
  type ResourceLoader,
  SessionManager,
  SettingsManager,
} from "@earendil-works/pi-coding-agent";

// 1. 显式定 cwd——后面所有组件都用它
const cwd = process.cwd();

// 2. 手写 ResourceLoader——完全控制所有资源
const myLoader: ResourceLoader = {
  getExtensions: () => ({
    extensions: [],            // 不加载任何扩展
    errors: [],
    runtime: createExtensionRuntime(),  // 最小可用空 runtime；实际需要扩展功能时见 sdk_doc/08
  }),
  getSkills: () => ({ skills: [], diagnostics: [] }),
  getPrompts: () => ({ prompts: [], diagnostics: [] }),
  getThemes: () => ({ themes: [], diagnostics: [] }),
  getAgentsFiles: () => ({ agentsFiles: [] }),
  getSystemPrompt: () => "You are a custom assistant.",
  getSystemPromptSource: () => undefined,              // ⚠️ v0.83.0 新增必需方法，v0.80.x 删除此行
  getAppendSystemPrompt: () => [],
  getAppendSystemPromptSources: () => [],              // ⚠️ v0.83.0 新增必需方法，v0.80.x 删除此行
  extendResources: () => {},
  reload: async () => {},
};

// 3. 创建 ModelRuntime（一步到位，不再分 AuthStorage + ModelRegistry）
//    ⚠️ 无参数调用 ModelRuntime.create() 默认读 ~/.pi/agent/auth.json + ~/.pi/agent/models.json
//    如果下面 createAgentSession 传了 agentDir（如 "/my/.pi"），两边的路径会不一致！
//    解决：要么也给 ModelRuntime.create 传对应的 authPath/modelsPath，要么统一不传 agentDir
const modelRuntime = await ModelRuntime.create();

// 4. 把所有组件传给 createAgentSession（只接 modelRuntime，不接 authStorage/modelRegistry）
const { session } = await createAgentSession({
  cwd,                                               // ← 显式传
  model: getBuiltinModel("anthropic", "claude-sonnet-4-20250514"),
  resourceLoader: myLoader,
  tools: ["read", "bash"],
  sessionManager: SessionManager.inMemory(cwd),      // ← 显式传 cwd
  settingsManager: SettingsManager.inMemory({}),
  modelRuntime,
});

// 5. 用 try/finally 保证 dispose
// dispose 会取消所有进行中的 LLM 调用、abort 重试/压缩/分支总结
// 不调会导致进程挂起（agent 内部的 subscribe 没解绑）
try {
  await session.prompt("hello");
} finally {
  session.dispose();
}
```

## 变体与延伸

| 变体 | 怎么改 | 参考 |
|------|-------|------|
| 自定义 model provider | 实现 `Provider` 接口 + `ModelRegistry.registerProvider(provider: Provider)` 或 `ModelRegistry.registerProvider(name: string, config: ProviderConfigInput)` | [H02 自定义 Provider](H02-custom-provider.md) |
| 把会话写到自定义目录 | `SessionManager.create(cwd, "/my/custom/dir")` | [sdk_doc/12-session-manager.md](../sdk_doc/12-session-manager.md) |
| 完全用文件持久化（不用 inMemory） | 全部用 `.create()` 工厂，不传任何组件给 `createAgentSession`——这就是默认行为，不需要 H01 | [sdk_doc/01-create-agent-session.md](../sdk_doc/01-create-agent-session.md) |
| 创建 AgentSessionRuntime 管理生命周期（cwd 切换、服务热重载） | 用 `createAgentSessionRuntime()` | [场景 F02](F02-session-runtime.md) |
| 同时注入自定义工具 | `createAgentSession({ customTools: [...] })` | [sdk_doc/06-tools.md](../sdk_doc/06-tools.md) |
| 关掉某些默认工具 | `createAgentSession({ excludeTools: ["bash"] })` | [sdk_doc/01-create-agent-session.md](../sdk_doc/01-create-agent-session.md) |
| 自定义 `~/.pi/agent` 目录位置 | `createAgentSession({ agentDir: "/my/.pi" })`，所有默认路径都会基于它 | [sdk_doc/01-create-agent-session.md](../sdk_doc/01-create-agent-session.md) |

## 常见误期待与陷阱

1. **"我传了 cwd 给 createAgentSession，sessionManager 应该也用这个 cwd"** —— 错。createAgentSession 只读不写，自定义 sessionManager 必须自己传 cwd。详见上方"双 cwd 陷阱"。
2. **"我手写了 ResourceLoader 就能改变 Agent loop"** —— 错。ResourceLoader 只决定"喂什么资源"，Agent loop 的执行逻辑（stream → tool → 流式回放）由 SDK 固定。要改 loop 行为用扩展层 hooks。
3. **"getExtensions 返回 `[]` 就行"** —— 类型错。返回的是 `LoadExtensionsResult` 对象，含 `extensions` / `errors` / `runtime` 三字段。其他 get* 方法同理（见上方签名）。
4. **"用 `createAgentSessionServices` 更高级，我手写 ResourceLoader 也传给它"** —— 错。该函数不支持自定义 ResourceLoader，强制 `DefaultResourceLoader`。
5. **"`getModel()` 就是查 ModelRegistry"** —— 错。`getModel`（compat.ts）是已废弃的别名，指向 `getBuiltinModel`，从内置 catalog 读静态模型定义。要查运行时用 `modelRuntime.getModel(provider, id)`（实例方法，非导出函数）。
6. **"dispose 可调可不调"** —— 错。`agent.subscribe` 内部订阅没解绑，进程会挂起。`try/finally` 必备。
