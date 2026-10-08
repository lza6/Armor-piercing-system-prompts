# 13. SettingsManager -- 配置管理器

> **定位**：本文档是 `SettingsManager` 类的 **API 参考**（面向 SDK 开发者）。配置项的含义和完整 JSON 格式见官方文档 [settings.md](https://github.com/earendil-works/pi-coding-agent/blob/main/packages/coding-agent/docs/settings.md)。

## 概述

**什么时候需要直接操作 SettingsManager？** 当你在写 pi-agent 的独立脚本或框架内部代码，需要读写 pi-agent 配置时——比如调整模型参数、切换主题、修改压缩策略。在扩展中，SettingsManager 由框架内部管理，你通过扩展 API 间接影响设置；在独立脚本中，通过 `SettingsManager.create()` 直接创建实例。

`SettingsManager` 是 pi-agent 的配置中枢，负责管理全局配置（`~/.pi/agent/settings.json`）和项目级配置（`.pi/settings.json`），并提供合并、覆盖、持久化等能力。所有配置读写都通过 `SettingsManager`，确保操作安全（文件锁）且可追踪（脏标记增量写入，防止外部编辑被内存快照覆盖）。

## 核心概念

### 两级配置存储

| 作用域 | 路径 | 用途 |
|--------|------|------|
| global | `~/.pi/agent/settings.json` | 用户级默认配置，跨项目共享 |
| project | `<cwd>/.pi/settings.json` | 项目级配置，覆盖 global 同名字段 |

**合并规则**：global 和 project 做**单层深度合并**（spread merge）：一级嵌套对象（如 `compaction`、`retry`、`terminal`、`images`）按字段逐个合并；但再深一层（如 `retry.provider`）则 project 的整对象**替换** global 的对应对象。基本类型和数组由 project 覆盖 global。

```json
// ~/.pi/agent/settings.json (global)
{ "theme": "dark", "compaction": { "enabled": true, "reserveTokens": 16384 } }

// .pi/settings.json (project)
{ "compaction": { "reserveTokens": 8192 } }

// 合并结果（一级嵌套按字段合并）
{ "theme": "dark", "compaction": { "enabled": true, "reserveTokens": 8192 } }
```

> ⚠️ **合并陷阱**：`retry.provider` 这类**二级嵌套**是整对象替换，不是按字段合并。
> ```json
// global
{ "retry": { "provider": { "timeoutMs": 10000, "maxRetries": 3 } } }

// project
{ "retry": { "provider": { "maxRetries": 5 } } }

// 合并结果——timeoutMs 丢失！project 的 retry.provider 整对象替换了 global 的
{ "retry": { "provider": { "maxRetries": 5 } } }
> ```
> 需要在 project 级别覆盖 `retry.provider` 的某个字段时，必须在 project 中显式写全 `retry.provider` 的所有字段。

> 💡 **源码命名提示**：实现这个合并的函数叫 `deepMergeSettings`（`settings-manager.ts:137`），注释里也写着 "merge recursively"，但**实现其实是单层 shallow merge**（`:157` 只做 `{ ...baseValue, ...overrideValue }`）。翻源码时以实际行为为准，别被函数名/注释误导——这就是为什么上面说二级嵌套是整对象替换。

### 项目信任机制

项目级配置（`.pi/settings.json`）只有在该项目的**目录被用户信任后**才会加载。信任决策存储在 `~/.pi/agent/trust.json`。

相关 API：

- `isProjectTrusted(): boolean` — 当前项目是否受信任
- `setProjectTrusted(trusted: boolean): void` — 设置信任状态（受信任时重新加载项目配置，不受信任时丢弃）
- `getDefaultProjectTrust(): "ask" | "always" | "never"` — 全局默认信任策略（仅读 global settings）
- `setDefaultProjectTrust(value): void` — 设置全局默认信任策略

非交互模式（`-p`、`--mode json`、`--mode rpc`）不弹信任提示，由 `defaultProjectTrust` 决定行为。用 `--approve` / `--no-approve` 可临时覆盖。

### 惰性写入 + 脏标记

`SettingsManager` 在每个 `set*()` 方法中会触发写入，但写入是**异步排队串行化**的：
1. 修改内存中的设置
2. 记录哪些字段被修改（脏标记，支持顶层字段和嵌套字段）
3. 调用内部 `save()` 将写入任务排入队列（串行化 Promise 链）
4. 仅在需要确认落盘时调用 `flush()` 等待队列清空

**为什么需要脏标记？** 核心原因是解决"外部编辑被内存快照覆盖"的 bug：当写入时，`persistScopedSettings` 会重新读取当前磁盘内容，只覆盖本次会话中修改过的字段，未修改的字段（包括外部新增的）不受影响。这样在 pi 运行时外部修改 `settings.json` 不会被意外覆盖。

## API 签名

### 静态工厂方法

```ts
// 从文件加载（生产环境）
static create(
  cwd: string,
  agentDir?: string,            // 默认 getAgentDir()，通常为 ~/.pi/agent
  options?: SettingsManagerCreateOptions  // { projectTrusted?: boolean }
): SettingsManager
```

- `cwd`：项目根目录，项目级配置读自 `<cwd>/.pi/settings.json`
- `agentDir`：agent 数据目录，全局配置读自 `<agentDir>/settings.json`
- `options.projectTrusted`：初始信任状态，默认 `true`。设为 `false` 时跳过项目配置
- 返回的 `SettingsManager` 已加载并合并了两级配置

```ts
// 纯内存实例（测试用）
static inMemory(
  settings?: Partial<Settings>,          // 作为 global 设置写入内存存储
  options?: SettingsManagerCreateOptions
): SettingsManager
```

- 不涉及任何文件 I/O
- 适合单元测试或临时场景

```ts
// 从自定义存储后端创建
static fromStorage(
  storage: SettingsStorage,
  options?: SettingsManagerCreateOptions
): SettingsManager
```

- `SettingsStorage` 是实现 `withLock(scope, fn)` 接口的任意对象
- 内置实现：`FileSettingsStorage`（文件）、`InMemorySettingsStorage`（内存）

### 实例方法

```ts
// 在已有合并设置之上叠加覆盖（最高优先级，不持久化）
applyOverrides(overrides: Partial<Settings>): void

// 一次性同时设置默认 provider 和 model
setDefaultModelAndProvider(provider: string, modelId: string): void

// 设置默认思考级别
setDefaultThinkingLevel(level: ThinkingLevel): void

// 等待所有待写入队列完成，确保数据落盘
flush(): Promise<void>

// 获取并清空 I/O 错误列表
drainErrors(): SettingsError[]

// 重新从磁盘加载配置（丢弃当前内存中的未保存修改）
reload(): Promise<void>
```

### 常用 getter / setter 方法

按功能分组，`set*()` 默认写入 **global scope**，`setProject*()` 写入项目级。

**模型与推理：**

| 方法 | 说明 |
|------|------|
| `getDefaultProvider()` / `setDefaultProvider(p)` | 默认 AI provider |
| `getDefaultModel()` / `setDefaultModel(m)` | 默认模型 |
| `setDefaultModelAndProvider(p, m)` | 一次性同时设置 provider + model |
| `getDefaultThinkingLevel()` / `setDefaultThinkingLevel(l)` | 思考级别（`"off"` \| `"minimal"` \| `"low"` \| `"medium"` \| `"high"` \| `"xhigh"` \| `"max"`） |
| `getHideThinkingBlock()` / `setHideThinkingBlock(h)` | 是否隐藏 thinking 块 |
| `getThinkingBudgets()` | 自定义各思考级别的 token 预算 |

**消息与传输：**

| 方法 | 说明 |
|------|------|
| `getSteeringMode()` / `setSteeringMode(m)` | steering 模式（`"all"` \| `"one-at-a-time"`，默认 `"one-at-a-time"`） |
| `getFollowUpMode()` / `setFollowUpMode(m)` | 跟进消息模式（`"all"` \| `"one-at-a-time"`，默认 `"one-at-a-time"`） |
| `getTransport()` / `setTransport(t)` | 传输层（`"auto"` \| `"sse"` \| `"websocket"` \| `"websocket-cached"`） |
| `getHttpIdleTimeoutMs()` / `setHttpIdleTimeoutMs(t)` | HTTP 空闲超时（毫秒，默认 300000） |
| `getWebSocketConnectTimeoutMs()` | WebSocket 连接超时 |

**压缩与重试：**

| 方法 | 说明 |
|------|------|
| `getCompactionEnabled()` / `setCompactionEnabled(e)` | 是否启用上下文压缩 |
| `getCompactionSettings()` | 压缩完整配置（enabled + reserveTokens + keepRecentTokens） |
| `getCompactionReserveTokens()` / `getCompactionKeepRecentTokens()` | 压缩 token 参数 |
| `getRetryEnabled()` / `setRetryEnabled(e)` | 是否启用重试 |
| `getRetrySettings()` | 重试完整配置 |
| `getProviderRetrySettings()` | provider 级重试配置 |
| `getBranchSummarySettings()` | 分支摘要配置 |
| `getBranchSummarySkipPrompt()` | 是否跳过摘要提示 |

**终端与图片：**

| 方法 | 说明 |
|------|------|
| `getShowImages()` / `setShowImages(s)` | 终端是否显示图片 |
| `getImageAutoResize()` / `setImageAutoResize(e)` | 图片自动缩放（最大 2000x2000） |
| `getBlockImages()` / `setBlockImages(b)` | 阻止所有图片发给 LLM |
| `getImageWidthCells()` / `setImageWidthCells(w)` | 终端内联图片宽度（单元格） |
| `getClearOnShrink()` / `setClearOnShrink(c)` | 内容缩小时清除空行 |
| `getShowTerminalProgress()` / `setShowTerminalProgress(p)` | OSC 9;4 终端进度指示器 |
| `getEditorPaddingX()` / `setEditorPaddingX(n)` | 输入编辑器水平内边距（0-3，默认 0） |
| `getAutocompleteMaxVisible()` / `setAutocompleteMaxVisible(n)` | 自动补全最大可见项（3-20，默认 5） |
| `getShowHardwareCursor()` / `setShowHardwareCursor(h)` | 显示终端光标（IME 支持） |
| `getShowCacheMissNotices()` / `setShowCacheMissNotices(s)` | 是否显示缓存未命中通知 |
| `getFullscreenScrollbar()` / `setFullscreenScrollbar(f)` | 全屏滚动条设置 |

**Shell 环境：**

| 方法 | 说明 |
|------|------|
| `getShellPath()` / `setShellPath(p)` | 自定义 shell 路径（如 Cygwin） |
| `getShellCommandPrefix()` / `setShellCommandPrefix(p)` | 每条 bash 命令的前缀 |
| `getNpmCommand()` / `setNpmCommand(cmd)` | npm 命令 argv（用于包管理操作） |
| `getExternalEditorCommand()` | 外部编辑器命令（Ctrl+G） |

**UI 与行为：**

| 方法 | 说明 |
|------|------|
| `getTheme()` / `setTheme(t)` | 当前主题名（主题名含 `/` 时视为自定义路径，`getTheme()` 返回 undefined） |
| `getThemeSetting()` | 返回原始 theme 字符串（与 `getTheme()` 同源读 `settings.theme`，但不做 `/` 过滤） |
| `getQuietStartup()` / `setQuietStartup(q)` | 隐藏启动头部 |
| `getCollapseChangelog()` / `setCollapseChangelog(c)` | 更新后折叠 changelog |
| `getEnabledModels()` / `setEnabledModels(p)` | 模型循环列表（Ctrl+P） |
| `getUiMode()` / `setUiMode(m)` | UI 模式设置 |
| `getOutputPad()` / `setOutputPad(p)` | 输出内边距设置 |
| `getDoubleEscapeAction()` / `setDoubleEscapeAction(a)` | 双击 Escape 行为（`"tree"` \| `"fork"` \| `"none"`） |
| `getTreeFilterMode()` / `setTreeFilterMode(m)` | /tree 默认过滤器 |
| `getEnableSkillCommands()` / `setEnableSkillCommands(e)` | 注册 skills 为 `/skill:name` 命令 |
| `getWarnings()` / `setWarnings(w)` | 警告设置（如 anthropicExtraUsage） |

**扩展与资源（全局 + 项目级）：**

| 方法 | 说明 |
|------|------|
| `getPackages()` / `setPackages(p)` / `setProjectPackages(p)` | npm/git 包列表 |
| `getExtensionPaths()` / `setExtensionPaths(p)` / `setProjectExtensionPaths(p)` | 扩展路径 |
| `getSkillPaths()` / `setSkillPaths(p)` / `setProjectSkillPaths(p)` | skill 路径 |
| `getPromptTemplatePaths()` / `setPromptTemplatePaths(p)` / `setProjectPromptTemplatePaths(p)` | prompt 模板路径 |
| `getThemePaths()` / `setThemePaths(p)` / `setProjectThemePaths(p)` | 主题路径 |

**项目信任与会话：**

| 方法 | 说明 |
|------|------|
| `isProjectTrusted()` / `setProjectTrusted(t)` | 项目信任状态 |
| `getDefaultProjectTrust()` / `setDefaultProjectTrust(t)` | 全局默认信任策略（global setting only） |
| `getSessionDir()` | 自定义会话存储目录 |

**遥测与分析：**

| 方法 | 说明 |
|------|------|
| `getEnableInstallTelemetry()` / `setEnableInstallTelemetry(e)` | 安装/更新版本 ping |
| `getEnableAnalytics()` / `setEnableAnalytics(e)` | 数据共享（生成 trackingId） |
| `getTrackingId()` | 分析追踪标识符 |

**内部工具：**

| 方法 | 说明 |
|------|------|
| `getGlobalSettings()` / `getProjectSettings()` | 获取原始配置副本（`structuredClone`） |
| `getCodeBlockIndent()` | 代码块缩进字符串 |
| `getLastChangelogVersion()` / `setLastChangelogVersion(v)` | changelog 版本追踪 |

## 关键类型定义

> **哪些类型能从包 import？** `@earendil-works/pi-coding-agent` 的入口（`index.ts`）仅 re-export 以下 8 个符号：`CompactionSettings`、`DefaultProjectTrust`、`ImageSettings`、`PackageSource`、`RetrySettings`、`SettingsManager`、`SettingsManagerCreateOptions`、`UiMode`。
>
> 下列类型中 **`SettingsStorage`、`SettingsScope`、`SettingsError`、`BranchSummarySettings`、`ProviderRetrySettings`、`TerminalSettings`、`ThinkingBudgetsSettings`、`MarkdownSettings`、`WarningSettings`** 均**未从包导出**。需要在自己的脚本中使用时，请照抄下面的内联定义，不要写 `import type { ... } from "@earendil-works/pi-coding-agent"`（会得到 TS 报错 `Module has no exported member ...`）。

```ts
interface SettingsManagerCreateOptions {
  projectTrusted?: boolean;  // 初始信任状态，默认 true
}

interface SettingsStorage {
  withLock(scope: SettingsScope, fn: (current: string | undefined) => string | undefined): void;
}

interface SettingsError {
  scope: SettingsScope;  // "global" | "project"
  error: Error;
}

// 压缩设置
interface CompactionSettings {
  enabled?: boolean;       // 默认 true
  reserveTokens?: number;  // 默认 16384（预留给 LLM 回复的 token）
  keepRecentTokens?: number; // 默认 20000（保留不压缩的最近 token）
}

// 分支摘要设置
interface BranchSummarySettings {
  reserveTokens?: number;  // 默认 16384
  skipPrompt?: boolean;    // 默认 false
}

// 图片设置
interface ImageSettings {
  autoResize?: boolean;  // 默认 true（缩放到 2000x2000）
  blockImages?: boolean; // 默认 false（true = 阻止所有图片发给 LLM）
}

// 终端设置
interface TerminalSettings {
  showImages?: boolean;          // 默认 true
  imageWidthCells?: number;      // 默认 60
  clearOnShrink?: boolean;       // 默认 false
  showTerminalProgress?: boolean; // 默认 false
}

// 包源（新格式支持按资源类型过滤）
type PackageSource = string | {
  source: string;
  extensions?: string[];
  skills?: string[];
  prompts?: string[];
  themes?: string[];
  autoload?: boolean;       // 是否自动加载资源（默认 true）
};

// 重试设置
interface RetrySettings {
  enabled?: boolean;       // 默认 true
  maxRetries?: number;     // 默认 3
  baseDelayMs?: number;    // 默认 2000（指数退避：2s, 4s, 8s）
  provider?: ProviderRetrySettings;
}

interface ProviderRetrySettings {
  timeoutMs?: number;
  maxRetries?: number;
  maxRetryDelayMs?: number; // 默认 60000
}
```

其余低频类型见源码 `settings-manager.ts`：
- **已从包导出**：`DefaultProjectTrust`（`"ask" | "always" | "never"`，可直接 import）
- **未导出**：`ThinkingBudgetsSettings`、`MarkdownSettings`、`WarningSettings`（如需使用请内联定义）

## 使用示例

### 独立脚本

```ts
import { SettingsManager } from "@earendil-works/pi-coding-agent";

// 创建并加载（自动合并 global + project）
const mgr = SettingsManager.create(process.cwd());

// 获取当前合并后的值
console.log(mgr.getDefaultProvider());     // "anthropic"
console.log(mgr.getCompactionEnabled());   // true
console.log(mgr.getSteeringMode());        // "one-at-a-time"

// 修改并持久化（写入 ~/.pi/agent/settings.json）
mgr.setTheme("dracula");
mgr.setDefaultThinkingLevel("high");
mgr.setDefaultModelAndProvider("anthropic", "claude-sonnet-4-20250514");

// 确认落盘（等待写入队列清空）
await mgr.flush();

// 运行时叠加覆盖（不持久化，优先级最高）
mgr.applyOverrides({ transport: "sse" });

// 检查 I/O 错误
const errors = mgr.drainErrors();
if (errors.length > 0) {
  console.error("设置写入失败:", errors);
}

// 操作项目信任
if (!mgr.isProjectTrusted()) {
  mgr.setProjectTrusted(true);  // 信任后自动重载项目配置
}

// 查看各层配置
console.log(mgr.getGlobalSettings());   // structuredClone 副本
console.log(mgr.getProjectSettings());
```

### 测试场景

```ts
// 纯内存，无文件 I/O
const testMgr = SettingsManager.inMemory({
  defaultProvider: "openai",
  compaction: { enabled: false },
});

// 也可控制项目信任
const untrustedMgr = SettingsManager.inMemory(
  { defaultProvider: "anthropic" },
  { projectTrusted: false }
);
```

### 自定义存储后端

> ⚠️ **注意类型来源**：`SettingsStorage` 和 `SettingsScope` **未从 `@earendil-works/pi-coding-agent` re-export**（包入口只导出 8 个符号，详见下方"关键类型定义"开头的导出说明）。下面示例把这两个类型**内联定义**，这样脚本可直接编译运行，无需额外依赖包内未导出的类型。

```ts
import { SettingsManager } from "@earendil-works/pi-coding-agent";

// 这两个类型未从包导出，需自行内联定义
type SettingsScope = "global" | "project";
interface SettingsStorage {
  withLock(
    scope: SettingsScope,
    fn: (current: string | undefined) => string | undefined,
  ): void;
}

const myStorage: SettingsStorage = {
  withLock(scope, fn) {
    // 实现你自己的加锁 + 读写逻辑（current 是磁盘上现有的 JSON 字符串）
    const current = myReadConfig(scope);
    const next = fn(current); // fn 返回 undefined 表示不写回
    if (next !== undefined) myWriteConfig(scope, next);
  },
};

const mgr = SettingsManager.fromStorage(myStorage);
```

## 关键细节与注意事项

### 1. 写入作用域

所有 `setXxx()` 方法写入 **global scope**（`~/.pi/agent/settings.json`）。如需写入项目级设置，使用对应的 `setProjectXxx()` 方法（如 `setProjectPackages()`、`setProjectSkillPaths()`）。这是设计决策：交互式修改（如 `/theme` 命令）默认影响全局配置。

### 2. 文件锁机制

`FileSettingsStorage` 使用 `proper-lockfile` 进行文件锁操作。写入时会尝试获取锁（最多重试 10 次，间隔 20ms），防止并发写入损坏配置文件。

### 3. 旧格式迁移

`SettingsManager` 内置迁移逻辑，自动处理：
- `queueMode` -> `steeringMode`
- `websockets: boolean` -> `transport: "websocket" | "sse"`
- 旧版 `skills` 对象格式 -> 新版数组格式
- `retry.maxDelayMs` -> `retry.provider.maxRetryDelayMs`

### 4. 写入失败不抛异常

写入操作通过 Promise 链串行化，失败时记录到 `this.errors` 数组而非抛出。调用方应使用 `drainErrors()` 主动检查 I/O 错误（不要用 try-catch 指望捕获异常）。

### 5. `applyOverrides` 不持久化

`applyOverrides()` 仅在内存层叠加，不会触发写入。它常用于 CLI 参数覆盖（`--model` 等），优先级高于持久化设置。

### 6. 解析错误处理

如果 `settings.json` 文件损坏（JSON 解析失败），`SettingsManager` 不会崩溃。它记录错误到对应 scope（`globalSettingsLoadError` 或 `projectSettingsLoadError`），并将该 scope 视为空配置 `{}`，同时阻止向损坏的文件写入（调用 `drainErrors()` 获取错误详情）。

### 7. 项目信任与配置加载

项目级配置（`.pi/settings.json`）只有在 `projectTrusted === true` 时才加载。非交互模式不弹信任提示，由 `defaultProjectTrust` 全局设置决定行为：
- `"ask"`（默认）：忽略项目配置
- `"always"`：自动信任
- `"never"`：忽略项目配置

写入项目配置前会检查 `assertProjectTrustedForWrite()`，不受信任时抛异常。

> ⚠️ **`setProjectTrusted()` 只改内存标志**：调用后该实例的 `projectTrusted` 标志被更新并重载项目配置，但**不写入 `trust.json`**。持久化信任由 CLI 层的 `ProjectTrustStore` 负责（`/login`、`--approve` 等）。SDK 直调 `setProjectTrusted()` 后重启进程，信任状态不会保留。

### 8. ThinkingLevel 类型

```ts
type ThinkingLevel = "off" | "minimal" | "low" | "medium" | "high" | "xhigh" | "max";
```

注意：两个包对 `ThinkingLevel` 的定义**不同**，容易混淆：

- **`pi-agent-core`**（`packages/agent/src/types.ts:294`，`SettingsManager` 用的是这个版本）= `"off" | "minimal" | "low" | "medium" | "high" | "xhigh" | "max"`（含 `"off"`，共 7 值）
- **`pi-ai`**（`packages/ai/src/types.ts:80`）= `"minimal" | "low" | "medium" | "high" | "xhigh" | "max"`（**不含** `"off"`，共 6 值）

⚠️ **易踩坑点**：`pi-ai` 另有 `ModelThinkingLevel = "off" | ThinkingLevel`（`types.ts:81`），它**是含 `"off"` 的**。所以如果你在 `pi-ai` 侧看到带 `"off"` 的类型，那大概率是 `ModelThinkingLevel` 而不是 `ThinkingLevel`。`SettingsManager.setDefaultThinkingLevel()` 的参数类型走的是 agent-core 版本，可以合法传 `"off"`。
