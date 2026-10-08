# 场景：管理 Settings 设置 (B02)

## 什么时候用

你需要**在代码里读写 pi-agent 的进程配置**——例如：

- 启动前预置压缩参数（`reserveTokens` / `keepRecentTokens`）
- 根据用户偏好切换默认 provider / model / thinkingLevel
- 让测试用例使用内存配置（不污染磁盘）
- 在多租户服务中为每个租户隔离一份 settings

**不该用本场景的情况**：

- 只是想改一个工具白名单 → 那是 `createAgentSession({ tools: [...] })` 的事，见 [A04](A04-tool-whitelist.md)。**`Settings` 里没有 `tools` 字段**，工具白名单不走 SettingsManager。
- 想动态注册/注销工具 → 见 [D02](D02-dynamic-tools.md)
- 只是想覆盖一次模型/thinkingLevel → 直接传 `createAgentSession({ model, thinkingLevel })`，不需要 SettingsManager

## 默认行为（★ 最常用路径，无需手动创建）

**绝大多数场景不需要手动创建 SettingsManager**。`createAgentSession` 不传 `settingsManager` 时，SDK 自动创建：

```ts
// 自动创建等价于：
const settingsManager = SettingsManager.create(cwd, agentDir);
//                                      ↑     ↑
//                                  项目根  全局目录（默认 ~/.pi/agent，受 PI_CODING_AGENT_DIR 覆盖）
```

**自动加载的两级配置**：

| 作用域 | 路径 | 说明 |
|--------|------|------|
| global | `~/.pi/agent/settings.json` | 用户级默认，跨项目共享 |
| project | `<cwd>/.pi/settings.json` | 项目级覆盖（**需项目受信任**，见下方「项目信任」） |

合并规则：浅层合并——第一层嵌套对象做属性级 shallow merge，2 层及以上整体替换。

> ⚠️ **源码命名陷阱**：实现这个逻辑的函数名叫 `deepMergeSettings`、注释写 "merge recursively"，但**实现并没有递归**——只做单层 `{ ...baseValue, ...overrideValue }`。此处按**实际行为**（shallow merge）描述，翻源码时不要被名字/注释误导。

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `SettingsManager` | 读写 / 合并 / 持久化 pi-agent 进程配置 | [sdk_doc/13-settings-manager.md](../sdk_doc/13-settings-manager.md) |

## 前置条件

- 已安装 `@earendil-works/pi-coding-agent`
- 若用 `SettingsManager.create()`（文件模式）：`<cwd>/.pi/` 和 `<agentDir>/` 目录可写。注意 `FileSettingsStorage` 的 `mkdirSync` **只在真正写入时触发**（`withLock` 内 `next !== undefined` 分支）——读时文件不存在不会建目录
- 若要加载项目级配置（`.pi/settings.json`）：项目必须受信任（见「项目信任」节）

## 创建方式（三种）

### 方式一：绑定项目目录（文件模式，生产环境）

```ts
import { SettingsManager } from "@earendil-works/pi-coding-agent";

const sm = SettingsManager.create("/path/to/project");
// 自动读取：
//   ~/.pi/agent/settings.json       (global)
//   /path/to/project/.pi/settings.json  (project，需项目受信任)
// 并做浅层合并（第一层嵌套对象 shallow merge，2 层以上整体替换；源码函数名虽叫 deepMergeSettings，实现实为单层）
```

`agentDir` 可选，默认 `getAgentDir()`（受 **`PI_CODING_AGENT_DIR`** 环境变量覆盖——即 `${APP_NAME.toUpperCase()}_CODING_AGENT_DIR`，APP_NAME 为 `pi`；见 `config.ts` 的 `ENV_AGENT_DIR` / `getAgentDir()`）：

```ts
// 显式指定 agentDir（例如多用户服务为每个用户分一个目录）
const sm = SettingsManager.create("/path/to/project", "/data/user42/pi-agent");
```

### 方式二：内存模式（测试用，不写磁盘）

```ts
const smMem = SettingsManager.inMemory({
  defaultProvider: "openai",
  compaction: { enabled: false },
});
// 所有 I/O 走内存，不触碰任何文件
```

### 方式三：自定义存储后端

适合需要把配置存到 KMS / Vault / DB 等非文件系统的场景：

```ts
import { SettingsManager } from "@earendil-works/pi-coding-agent";

// SettingsStorage 和 SettingsScope 类型未从包入口 re-export，需内联定义
type SettingsScope = "global" | "project";
interface SettingsStorage {
  withLock(scope: SettingsScope, fn: (current: string | undefined) => string | undefined): void;
}

const myStorage: SettingsStorage = {
  withLock(scope: SettingsScope, fn) {
    const current = myReadConfig(scope);      // 你自己的读
    const next = fn(current);                 // SDK 给出新的 JSON 字符串
    if (next !== undefined) myWriteConfig(scope, next);  // 你自己的写
  },
};

const sm = SettingsManager.fromStorage(myStorage);
```

⚠️ 自定义存储**必须实现锁语义**（`withLock` 同步互斥）——否则并发写入会丢数据。

## 修改设置

### 常用 set 方法（写入 global scope）

```ts
// 思考级别
sm.setDefaultThinkingLevel("high");
// 取值："off" | "minimal" | "low" | "medium" | "high" | "xhigh" | "max"

// 默认 provider + model（一次性设两个）
sm.setDefaultModelAndProvider("anthropic", "claude-opus-4-5");

// 压缩开关（仅切换 enabled；reserveTokens/keepRecentTokens 是只读 getter，无 setter）
sm.setCompactionEnabled(false);
// 想改 reserveTokens/keepRecentTokens？没有 setter——只能经 inMemory({...}) 预置
// 或 applyOverrides({ compaction: { reserveTokens: 8192 } }) 运行时叠加（见下文）

// 重试开关
sm.setRetryEnabled(false);

// Steering 模式（消息队列模式）
sm.setSteeringMode("all");  // "all" | "one-at-a-time"

// 主题（影响 TUI，但服务端集成一般用不到）
sm.setTheme("dracula");
```

⚠️ **所有 `set*()` 方法写入 global scope（`~/.pi/agent/settings.json`）**，不是项目级。如需写项目级，使用对应的 `setProjectXxx()`（仅 5 个字段支持，方法名映射：`packages`→`setProjectPackages` / `extensions`→`setProjectExtensionPaths` / `skills`→`setProjectSkillPaths` / `prompts`→`setProjectPromptTemplatePaths` / `themes`→`setProjectThemePaths`）。

### 运行时叠加（不持久化，优先级最高）

```ts
sm.applyOverrides({
  compaction: { reserveTokens: 8192 },
  defaultThinkingLevel: "low",
});
// 仅在内存中叠加，不会触发写入
// 常用于 CLI 参数覆盖（--model 等）
```

`applyOverrides` **不是 `Settings` 字段的任意值**——只能传 `Settings` 接口定义的字段。例如：

- ✅ `compaction: { enabled, reserveTokens, keepRecentTokens }`
- ✅ `retry: { enabled, maxRetries, baseDelayMs, provider }`
- ❌ `compaction.threshold`（不存在这个字段）
- ❌ `tools: { enabled: [...] }`（`Settings` 里根本没有 `tools`）

## 持久化与错误处理

### flush()：等待写入队列清空

```ts
await sm.flush();
```

**关键细节**：`flush()` 本身**不触发写入**——写入在每个 `set*()` 方法中经 `enqueueWrite()` 排入队列（`flush()` 仅 `await this.writeQueue`，确保前面的写入任务全部完成）。

所以 `setDefaultThinkingLevel()` 后不调用 `flush()` 也能生效（异步会写），但**进程退出前不 await 可能丢写入**——服务端集成建议在退出点 / 周期性调用 `flush()`。

### drainErrors()：获取 I/O 错误（★ 写入失败不抛异常）

```ts
const errs = sm.drainErrors();
if (errs.length > 0) {
  for (const e of errs) {
    console.error(`[${e.scope}]`, e.error.message);
  }
}
```

**写入失败不抛异常**，而是由 `recordError()` 记录到 `this.errors` 数组。**不要用 try/catch 指望捕获 I/O 错误**——必须主动 `drainErrors()`。

`drainErrors` 会**清空**错误列表，下次调用只返回新增的。

## 优先级层级（★ 易踩坑）

pi-agent 的最终生效配置由 **4 层叠加**，从低到高：

```
global (~/.pi/agent/settings.json)
   ↓ 浅层合并
project (<cwd>/.pi/settings.json，需受信任)
   ↓ 浅层合并
applyOverrides()    ← 运行时叠加，不持久化
   ↓
createAgentSession({ model, thinkingLevel, ... })  ← 编程式参数，最高优先级
```

**验证方式**：`sm.getCompactionEnabled()` 等 getter 返回的是**已合并**的最终值（读 `this.settings`，而非 `this.globalSettings`）。

## 项目信任（★ 服务端集成必读）

**项目级配置 `.pi/settings.json` 只在 `projectTrusted === true` 时才加载**（`loadFromStorage()` 内 `if (scope === "project" && !projectTrusted) return {}`）。这是为了防止执行未知仓库时被恶意 `.pi/settings.json` 接管。

```ts
// 创建时指定初始信任状态
const sm = SettingsManager.create("/path/to/project", undefined, {
  projectTrusted: false,  // 默认 true；设为 false 会跳过加载 .pi/settings.json
});

// 运行时切换
sm.setProjectTrusted(true);   // 重新加载项目配置
sm.setProjectTrusted(false);  // 丢弃当前项目配置
```

**陷阱**：不受信任时调用 `setProjectPackages()` 等 `setProjectXxx` 方法会**直接抛异常**（`assertProjectTrustedForWrite()` 守卫）：

```
Error: Project is not trusted; refusing to write project settings
```

**全局默认信任策略**（仅 global setting）：

```ts
sm.getDefaultProjectTrust();  // "ask" | "always" | "never"，默认 "ask"
sm.setDefaultProjectTrust("always");
```

非交互模式（`-p` / `--mode json`）不弹信任提示，由 `defaultProjectTrust` 决定行为。

## 完整示例

```ts
import { createAgentSession, SettingsManager } from "@earendil-works/pi-coding-agent";

// 1. 自定义 settings（绑定项目 + 关闭压缩）
const sm = SettingsManager.create("/app/workspace");
sm.setCompactionEnabled(false);
sm.setDefaultThinkingLevel("medium");

// 2. 检查写入错误
const errs = sm.drainErrors();
if (errs.length > 0) {
  console.error("Settings 写入失败:", errs);
  // 决定是否中止启动
}

// 3. 确认落盘
await sm.flush();

// 4. 传给 createAgentSession（替代默认的 SettingsManager）
const { session } = await createAgentSession({
  cwd: "/app/workspace",
  settingsManager: sm,
});

try {
  await session.prompt("hello");
} finally {
  session.dispose();
}
```

测试场景（内存模式）：

```ts
import { createAgentSession, SettingsManager, SessionManager } from "@earendil-works/pi-coding-agent";
//                                        ↑ 与 SettingsManager 同包 re-export（见 index.ts）

const sm = SettingsManager.inMemory({
  defaultProvider: "openai",
  compaction: { enabled: false, reserveTokens: 8192 },
});

const { session } = await createAgentSession({
  settingsManager: sm,
  sessionManager: SessionManager.inMemory(),  // 同样走内存
});
```

## 陷阱

### 陷阱 1：写入失败不抛异常

见上文「drainErrors」。**生产环境必须在启动点 + 周期性调用 `drainErrors()`**，否则写入静默失败时你完全不知道。

### 陷阱 2：`applyOverrides` 不持久化

```ts
sm.applyOverrides({ defaultThinkingLevel: "high" });
// ⚠️ 重启后失效——它只在内存叠加
// 要持久化请用 setDefaultThinkingLevel("high")
```

### 陷阱 3：项目不受信任时 `setProjectXxx` 抛异常

见「项目信任」节。不受信任的项目连 `setProjectPackages()` 都不能调用。

### 陷阱 4：settings.json 损坏时不崩溃，但停止写入该 scope

如果 `settings.json` 文件 JSON 解析失败，`SettingsManager` **不会崩溃**——`tryLoadFromStorage()` 捕获异常、把该 scope 视为空 `{}` 并记录到 `globalSettingsLoadError` / `projectSettingsLoadError`。后续对该 scope 的写入会被**跳过**（`save()` / `saveProjectSettings()` 开头检查对应 `*LoadError`，非空则直接 return）——所以损坏文件不会被覆写修复，需要手动删除。

### 陷阱 5：脏标记保护外部编辑

`SettingsManager` 用脏标记机制追踪本次会话修改的字段，写入时**重新读取磁盘**并只覆盖修改过的字段。这意味着 pi 运行时外部修改 `settings.json` **不会被覆写**——但也意味着你用 `setTheme("dark")` 修改的字段，如果外部同时改了 `theme`，最终你的写入会覆盖外部的修改（这是预期行为）。

## 变体与延伸

- 自定义压缩算法 / 压缩回调 → 见 [G02](G02-custom-compaction.md)（压缩参数 `reserveTokens` / `keepRecentTokens` 是 SettingsManager 的事，压缩**策略**是扩展的事）
- 工具白名单 → 见 [A04](A04-tool-whitelist.md)（**不是** SettingsManager 的职责，`Settings` 里没有 `tools` 字段）
- `createAgentSession` 中传入 settingsManager → 见 [sdk_doc/01](../sdk_doc/01-create-agent-session.md)
- 完整 API 清单（80+ getter/setter）→ 见 [sdk_doc/13-settings-manager.md](../sdk_doc/13-settings-manager.md)
