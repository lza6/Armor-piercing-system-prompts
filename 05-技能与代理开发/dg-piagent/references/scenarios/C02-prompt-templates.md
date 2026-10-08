# 场景：定义 Prompt 模板 (C02)

## 什么时候用 / 不用会怎样

**适合**：当你有一段**会反复用到、但每次参数略不同**的提示词流程时——例如「/deploy production」「/review PR-123」「/test --filter=auth」。把流程封装成 `/command` 形式，用户输入 `/deploy production` 即可触发预定义的 prompt 内容并自动替换参数占位符。

**不适合**：
- 简单单条指令 → 用 [A03](A03-system-prompt.md) 的 `systemPrompt` / `appendSystemPrompt` 直接写一行就够
- 需要在运行时动态生成 prompt → 用 [E05-input-transform](E05-input-transform.md) 拦截用户输入改写
- 大段静态参考数据（schema、API 文档） → 用 [G01-context-injection](G01-context-injection.md) 按需注入

**不用会怎样**：用户每次得手敲一长串提示词，容易遗漏步骤；或者你得在每个项目里写 wrapper 脚本，丢失 pi-agent 的 steer/abort/compaction 等会话能力。

## 默认行为（不传 `promptsOverride` 时）

`createAgentSession` 不显式传 `resourceLoader` 时，SDK 内部 `new DefaultResourceLoader(...)` 会自动发现 Prompt 模板。完整的扫描来源（`resource-loader.ts:482-484` 合并 `cliEnabledPrompts + enabledPrompts + additionalPromptTemplatePaths`，其中 `enabledPrompts` 来自 `packageManager.resolve()`）：

1. **全局目录** `~/.pi/agent/prompts/`（user scope）
2. **项目目录** `<cwd>/.pi/prompts/`（project scope，**受项目信任门槛控制**——见下方「陷阱 3」）
3. **包源**（package scope）：已安装扩展/依赖包里的 `prompts/` 目录，或 `package.json` 的 `pi.prompts` 字段声明的路径（`package-manager.ts:2126-2174` `collectPackageResources`）
4. **Settings 源**：`settings.json` 的 `prompts` 数组（可放文件或目录路径，`package-manager.ts:2353,2359`）——可被 `overrides` 启用/禁用
5. **显式路径** `additionalPromptTemplatePaths` 中的文件或目录

> CLI 还有 `--prompt-template <path>`（可重复，注入 `cliEnabledPrompts`）和 `--no-prompt-templates`（关闭默认目录扫描）两个开关；SDK 不走这条路，但 SDK 的 `noPromptTemplates` / `additionalPromptTemplatePaths` 选项对应同样效果。

每个目录的发现规则：
- **非递归**扫描目录下直接 `.md` 子文件（与 Skill 的递归 + SKILL.md 不同）
- 文件名（去 `.md`）即为模板 `name`，即 `/command` 的命令名
- frontmatter 解析 `description` 和 `argument-hint`（YAML 键名是 kebab-case）
- 未提供 `description` 时取正文第一行非空文本（截断到 60 字符）
- 符号链接会跟随，但断裂链接跳过（仅 coding-agent 同步版行为，`prompt-templates.ts:152-161`）

> **路径可被环境变量覆盖**：全局目录 `~/.pi/agent/` 实际由 `getAgentDir()`（`config.ts:515-521`）决定，优先读 `PI_CODING_AGENT_DIR` 环境变量；项目目录里的 `.pi` 段由 `CONFIG_DIR_NAME`（`config.ts:491`）决定，可被 `package.json` 的 `piConfig.configDir` 覆盖。测试/部署场景常靠这两个旋钮切换配置目录。

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `DefaultResourceLoader` 的 `promptsOverride` | 过滤/替换/追加模板列表 | [sdk_doc/08-resource-loader.md](../sdk_doc/08-resource-loader.md) |
| `additionalPromptTemplatePaths` 选项 | 不改写列表，只加路径 | [sdk_doc/08-resource-loader.md](../sdk_doc/08-resource-loader.md) |
| `noPromptTemplates: true` | 跳过默认目录扫描（但 `additionalPromptTemplatePaths` 和 CLI `--prompt-template` 仍生效） | [sdk_doc/08-resource-loader.md](../sdk_doc/08-resource-loader.md) |
| `PromptTemplate` / `createSyntheticSourceInfo` | 创建虚拟模板对象（不落地文件） | [sdk_doc/10-prompt-templates.md](../sdk_doc/10-prompt-templates.md) |
| `loader.getPrompts()` | 查询已加载模板 + diagnostics | [sdk_doc/10-prompt-templates.md](../sdk_doc/10-prompt-templates.md) |
| `session.promptTemplates` getter | 运行时访问会话内模板 | [sdk_doc/02-agent-session.md](../sdk_doc/02-agent-session.md) |

## 核心数据模型

### PromptTemplate 接口

```ts
interface PromptTemplate {
  name: string;            // 模板名（即 /name 的命令名）；自动发现时由文件名决定
  description: string;     // 功能说明，用于 UI 列表（slash command 菜单）
  argumentHint?: string;   // 参数提示（如 "<environment>"），可选；来自 frontmatter "argument-hint"。
                           // UI 约定：<angle brackets> 表示必填，[square brackets] 表示可选，
                           // 渲染到 slash command 自动补全下拉（见官方 docs/prompt-templates.md:37-53）
  content: string;         // 模板正文，可含 $1, $@, ${N:-default} 等占位符
  sourceInfo: SourceInfo;  // 来源元信息
  filePath: string;        // 模板文件绝对路径
}
```

**字段约束与来源**：
- `name`：自动发现时由文件名（去 `.md`）决定；通过 `promptsOverride` 注入时可自定义，但**不要含空格，且建议避开 `/`**——`expandPromptTemplate` 用正则 `^/([^\s]+)` 解析命令名（按**空白符**切分，所以不能含空格）；`/` 虽不被正则拒绝，但会与 `/skill:name`、extension command 的命名空间撞，且 `expandPromptTemplate` 按 `name === templateName` **精确匹配**（`prompt-templates.ts:278`），建议避免
- `description`：未提供时取正文第一行非空文本（截断到 60 字符）
- `argumentHint`：来自 frontmatter `argument-hint`（**kebab-case**，不是 camelCase）
- `content`：模板正文，支持参数占位符（见下方「参数占位符语法」节）

### createSyntheticSourceInfo 签名

```ts
function createSyntheticSourceInfo(
  path: string,
  options: {
    source: string;                                // 来源标识，如 "sdk" / "local" / 自定义字符串
    scope?: "user" | "project" | "temporary";      // 默认 "temporary"
    origin?: "package" | "top-level";              // 默认 "top-level"
    baseDir?: string;
  }
): SourceInfo
```

**注意**：第二参数是 `options` 对象，**不是字符串、不是文件内容**。`source` 字段必填。

## 参数占位符语法

模板 `content` 支持以下占位符：

| 占位符 | 含义 | 示例 |
|--------|------|------|
| `$1`, `$2`, ... | 位置参数（1-indexed） | `/deploy production` → `$1` = `production` |
| `${N:-default}` | 位置参数带默认值 | `${1:-production}`：缺省时替换为 `production` |
| `$@` / `$ARGUMENTS` | 所有参数的原始文本（空格连接） | `/deploy v1 prod` → `v1 prod` |
| `${@:-default}` / `${ARGUMENTS:-default}` | 所有参数为空时使用默认值（三者等价，正则统一匹配 `\|@` 与 `ARGUMENTS`） | `${@:-staging}`：无参数时替换为 `staging` |
| `${@:N}` | 从第 N 个参数开始（bash 风格切片） | `${@:2}` 跳过第 1 个 |
| `${@:N:L}` | 从第 N 个开始取 L 个参数 | `${@:1:2}` 取前两个 |

**关键细节**：替换只在模板字符串上执行一次，**参数值中包含的 `$1`、`$@` 等模式不会被递归替换**。这是为了避免注入攻击。

> ⚠️ **跨包差异**：上表是 `@earendil-works/pi-coding-agent` 的 `substituteArgs` 行为（`coding-agent/src/core/prompt-templates.ts:70-102`，支持默认值语法）。若你直接用 `@earendil-works/pi-agent` 的 harness（`agent/src/harness/prompt-templates.ts:249-262`），**`${N:-default}` / `${@:-default}` / `${ARGUMENTS:-default}` 不支持**——那个版本只认 `$N` / `$@` / `$ARGUMENTS` / `${@:N}` / `${@:N:L}`，遇到默认值语法会原样保留不替换。本 skill 的所有场景都基于 coding-agent 的 `agent-session`，默认值语法可用。

参数解析用 bash 风格，支持引号包裹：

```ts
// 内部行为（loadPromptTemplates / expandPromptTemplate / parseCommandArgs / substituteArgs
// 在 core/prompt-templates.ts 里是 export 的，但未被 SDK 主入口
// @earendil-works/pi-coding-agent re-export——想直接用需 deep import，不保证跨版本稳定）
parseCommandArgs('hello "world test" foo');
// => ["hello", "world test", "foo"]
```

## 实现思路

1. 选择路径：**自动发现**（把 `.md` 文件放进 `.pi/prompts/`）**或** **虚拟注入**（通过 `promptsOverride`）
2. 自动发现：文件名即命令名，frontmatter 写 description 和 argument-hint
3. 虚拟注入：创建 `PromptTemplate` 对象，通过 `promptsOverride` 回调追加到 `base.prompts`
4. `await loader.reload()` 后模板生效
5. 用户输入 `/deploy production` 即触发模板展开 + 参数替换

## 核心代码：自动发现方式（推荐）

最小侵入——只需把文件放进 `<cwd>/.pi/prompts/deploy.md`：

```markdown
---
description: Deploy the application to an environment
argument-hint: <environment>
---

# Deploy Instructions

Deploy the application to **$1** environment.

## Steps
1. Build: `npm run build`
2. Test: `npm test`
3. Deploy: `npm run deploy -- --env $1`
4. Verify: Check `$1.example.com` is responding

Extra args: $@
```

> 项目 `.pi/prompts/` 受项目信任门槛控制——但**SDK 路径下默认就是受信任的**（见下方「陷阱 3」）；只有 CLI 交互路径才会首次询问用户。

无需任何 TS 代码，默认 `createAgentSession` 即自动加载。`session.prompt("/deploy production")` 会把 `$1` 替换为 `production`。

## 核心代码：虚拟注入方式

适合「模板内容需要运行时动态生成」「不想落地文件」的场景：

```ts
import {
  createAgentSession,
  createSyntheticSourceInfo,
  DefaultResourceLoader,
  getAgentDir,
  SessionManager,
  type PromptTemplate,  // ★ type-only import，运行时不存在
} from "@earendil-works/pi-coding-agent";

const cwd = process.cwd();
const agentDir = getAgentDir();

// 1. 创建虚拟模板
//    ★ createSyntheticSourceInfo 第二参数是 options 对象，不是字符串
const deployTemplate: PromptTemplate = {
  name: "deploy",
  description: "Deploy the project to production",
  argumentHint: "<env>",
  filePath: "/virtual/prompts/deploy.md",
  sourceInfo: createSyntheticSourceInfo(
    "/virtual/prompts/deploy.md",
    { source: "sdk", scope: "temporary" },  // ★ 对象，不是字符串
  ),
  content: [
    "# Deploy Instructions for $1",
    "",
    "1. Run tests to verify everything passes",
    "2. Build the project",
    `3. Push to $1 server`,
    "4. Verify deployment health",
    "",
    "All args: $@",
  ].join("\n"),
};

// 2. 用 promptsOverride 追加（保留默认 + 加自定义）
//    回调签名：(base: { prompts, diagnostics }) => { prompts, diagnostics }
const loader = new DefaultResourceLoader({
  cwd,
  agentDir,
  promptsOverride: (base) => ({
    prompts: [...base.prompts, deployTemplate],
    diagnostics: base.diagnostics,
  }),
});
await loader.reload();

// 3. 查看已加载模板
//    ★ getPrompts() 返回 { prompts, diagnostics }，不是 PromptTemplate[]
const { prompts, diagnostics } = loader.getPrompts();
console.log("Loaded templates:", prompts.map(p => `/${p.name}`));
if (diagnostics.length > 0) {
  console.log("Diagnostics:", diagnostics);
}

const { session } = await createAgentSession({
  resourceLoader: loader,
  sessionManager: SessionManager.inMemory(),
});

// 4. 触发模板：输入 "/deploy production" 时自动展开
//    $1 替换为 production，$@ 替换为 "production"
await session.prompt("/deploy production");
```

## 变体

### 变体 A：不改写列表，只加路径

适合「我就想多加载一个目录的模板，别的不动」的场景。无需 `promptsOverride`：

```ts
const loader = new DefaultResourceLoader({
  cwd,
  agentDir,
  additionalPromptTemplatePaths: ["/abs/path/to/extra-prompts"],
});
await loader.reload();
```

`additionalPromptTemplatePaths` 接受文件或目录。**不存在的本地路径不会 throw**，会写入 `promptDiagnostics`（type: `"error"`，`resource-loader.ts:488-499`，检查包在 `if (isLocalPath(p))` 内）。远程/npm 源路径走 `packageManager.resolveExtensionSources` 另一套解析，不在此处报 error。目录扫描非递归，只读直接 `.md` 子文件。

### 变体 B：关闭模板自动发现

完全不用模板系统（例如你要自己实现命令解析）：

```ts
const loader = new DefaultResourceLoader({
  cwd,
  agentDir,
  noPromptTemplates: true,
});
```

注意：`noPromptTemplates: true` 仍然允许 `additionalPromptTemplatePaths` 加载。

### 变体 C：运行时访问已加载模板

通过 `session.promptTemplates` getter：

```ts
const { session } = await createAgentSession({ resourceLoader: loader });

for (const t of session.promptTemplates) {
  console.log(`/${t.name}: ${t.description}`);
  if (t.argumentHint) console.log(`  args: ${t.argumentHint}`);
}
```

## 关键陷阱

### 陷阱 1：`createSyntheticSourceInfo` 第二参数是对象不是字符串

签名是 `(path, options: { source, scope?, origin?, baseDir? })`。**`source` 字段必填**，`scope` / `origin` / `baseDir` 可选但有默认值。

```ts
// ❌ 错误：第二参数不是字符串（曾是早期文档的写法，已废止）
createSyntheticSourceInfo("/virtual/deploy.md", "Deploy the app to production...")

// ❌ 错误：source 是必填字段
createSyntheticSourceInfo("/virtual/deploy.md", { scope: "user" })

// ✅ 正确
createSyntheticSourceInfo("/virtual/deploy.md", { source: "sdk", scope: "temporary" })
```

### 陷阱 2：`loader.getPrompts()` 返回对象不是数组

```ts
// ❌ 错误：getPrompts() 返回 { prompts, diagnostics }，直接 .map 会报错
const prompts = loader.getPrompts();
prompts.map(p => p.name);  // TypeError: prompts.map is not a function

// ✅ 正确
const { prompts, diagnostics } = loader.getPrompts();
prompts.map(p => p.name);  // OK
```

`diagnostics` 可能包含 `type: "collision"`（同名冲突）、`type: "error"`（路径不存在）等记录，建议检查。

### 陷阱 3：项目级模板受「信任门槛」控制（SDK 与 CLI 默认相反）

`<cwd>/.pi/prompts/` 下的模板受 `projectTrusted` 控制。**关键：SDK 路径和 CLI 路径的默认值相反**——

- **SDK 路径（本 skill 的场景）默认受信任**。`createAgentSession` 内部调 `SettingsManager.create(cwd, agentDir)` **不传 `projectTrusted`**，而 `fromStorage` 默认 `projectTrusted ?? true`（`settings-manager.ts:325`）。即纯 SDK 调用下，`.pi/prompts/` **直接生效**，无需任何额外设置。
- **CLI 交互路径默认不信任**。`main.ts:543` 显式传 `{ projectTrusted: false }`，然后通过 `resolveProjectTrust` 询问用户；用户确认后才信任。文档里常说的「首次打开项目要确认信任」是 **CLI 行为**，不是 SDK 行为。

若你在 SDK 里想**模拟** CLI 的安全语义（比如做一个需要用户确认的宿主），可以：

```ts
import { SettingsManager, DefaultResourceLoader } from "@earendil-works/pi-coding-agent";

const settingsManager = SettingsManager.create(cwd, agentDir, { projectTrusted: false });
const loader = new DefaultResourceLoader({ cwd, agentDir, settingsManager });
// 先用不信任状态 bootstrap，再根据你的确认逻辑决定是否切到信任：
await loader.reload({
  resolveProjectTrust: async ({ extensionsResult }) => {
    // 返回 true = 信任，返回 false = 不信任
    return await myConfirmDialog("Trust this project?");
  },
});
```

这与 Skill（[C01 陷阱 3](C01-custom-skill.md)）、`.pi/SYSTEM.md`、`.pi/skills/` 的行为完全一致。详见 [B04-project-trust.md](B04-project-trust.md)。

### 陷阱 4：`session.prompt()` 的命令解析优先级

当用户输入 `/foo bar` 时，pi-agent 按以下顺序匹配：

1. **Extension command**（扩展通过 `pi.registerCommand` 注册）— 立即执行，不走 LLM
2. **Skill command**（`/skill:name` 形式）— 展开为 skill 调用
3. **Prompt template**（`/name` 形式）— 展开模板内容
4. 都不匹配 → 原样传给 LLM

如果你的模板名和某个 extension command 撞了，**extension 优先**，模板永远不触发。命名时建议避开扩展命令的命名空间。

### 陷阱 5：`expandPromptTemplates: false` 可禁用模板展开

需要把 `/something` **原样**传给 LLM（不让 pi 展开）时：

```ts
await session.prompt("/something", { expandPromptTemplates: false });
```

默认值是 `true`。注意：此选项**只作用于 `session.prompt()`**——`session.steer()` 和 `session.followUp()` **始终展开模板**，无法关闭。

### 陷阱 6：同名模板冲突「先到先得」

`dedupePrompts` 按加载顺序去重，第一个出现的胜出，后续同名模板被丢弃并写入 `diagnostics`（type: `"collision"`）。加载顺序：global → project → additional paths → override 追加。

注意：`promptsOverride` 回调看到的是**去重后**的列表，所以你在回调里追加的同名模板不会被去重掉。

### 陷阱 7：`promptsOverride` 是「最终赋值前」的拦截点

`promptsOverride` 拦截的是 `loadPromptTemplates()` + `dedupePrompts()` 的最终输出。**回调里不返回某模板，它就真的没了**——即便它在默认目录里存在。如果只想追加、不想过滤，记得把 `base.prompts` 完整传出去：

```ts
promptsOverride: (base) => ({
  prompts: [...base.prompts, customTemplate],   // 保留默认 + 追加
  diagnostics: base.diagnostics,
})
```

## 横向联动

- Prompt Templates 系统详解（占位符、文件格式、内部函数） → [sdk_doc/10-prompt-templates.md](../sdk_doc/10-prompt-templates.md)
- ResourceLoader 完整选项（`additionalPromptTemplatePaths` / `noPromptTemplates` / 其他 override） → [sdk_doc/08-resource-loader.md](../sdk_doc/08-resource-loader.md)
- 同时加载 Skill → [场景 C01](C01-custom-skill.md)（模板和 skill 可共用 ResourceLoader）
- 同时加载 Context Files（AGENTS.md / CLAUDE.md） → [场景 C03](C03-context-files.md)
- 系统提示词的合成链（模板不直接进系统提示词，但和 SYSTEM.md 共存） → [场景 A03](A03-system-prompt.md)
- 项目信任门槛（`.pi/prompts/` 生效前提） → [场景 B04](B04-project-trust.md)
- 在运行时拦截/改写用户输入（替代或增强模板） → [场景 E05](E05-input-transform.md)
