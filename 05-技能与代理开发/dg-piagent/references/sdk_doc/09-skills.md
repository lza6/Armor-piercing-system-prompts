# 09 - 技能系统 (Skills)

## 这是什么

Skills 是 pi-agent 的**专业知识注入系统**。每个 Skill 是一个 Markdown 文件（格式遵循 [Agent Skills 标准](https://agentskills.io/integrate-skills)），包含特定领域的指令，在会话启动时被加载到系统提示词中。Skill 使得 Agent 可以在不同场景下获得专门的行为指令。

**不用 Skills 会怎样**：模型缺少领域专属指令，每次都得手动把规则塞进 prompt（既冗长又难以复用），也无法享受按需加载、`/skill:name` 显式调用、碰撞优先级等机制。

> **本文档针对 `@earendil-works/pi-coding-agent` 包**。`@earendil-works/pi-agent`（agent-core）包也有 `loadSkills` / `formatSkillsForPrompt` / `loadSkillsFromDir` 同名 API，但存在差异：agent 包的 `loadSkills` 是 **async**（返回 `Promise`）、签名不同（接收 `ExecutionEnv` + 目录列表）、`Skill` 接口字段也不同（见下方说明）。跨包使用请先核对签名。

## Skill 类型

```ts
interface Skill {
  name: string;                    // Skill 名称，用于引用和 /skill:name 显式调用
  description: string;             // 功能说明
  filePath: string;                // SKILL.md 文件路径
  baseDir: string;                 // Skill 所在基础目录
  sourceInfo: SourceInfo;          // 来源信息（路径、来源类型、作用域）
  disableModelInvocation: boolean; // true 时不会自动注入提示词，只能通过 /skill:name 显式调用
}
```

> **⚠ 跨包陷阱（C01 教训复现点）**：以上是 `@earendil-works/pi-coding-agent` 包的 `Skill` 接口。`@earendil-works/pi-agent`（agent-core）包的 `Skill` 接口**完全不同**（`packages/agent/src/harness/types.ts:64-75`）：
> - 有 `content: string`（正文内联在对象里，而非靠 read tool 读取 filePath）
> - **没有** `baseDir` 和 `sourceInfo` 字段
> - `disableModelInvocation` 是**可选**（`disableModelInvocation?: boolean`），coding-agent 包则是必填
>
> 跨包传递 Skill 时需手动映射字段，否则 TypeScript 会报类型错误（或更糟——运行时静默丢失 `content`）。

### SkillFrontmatter

SKILL.md 文件可以包含 YAML frontmatter 元数据：

```ts
interface SkillFrontmatter {
  name?: string;                        // Skill 名称
  description?: string;                 // Skill 说明
  "disable-model-invocation"?: boolean; // 禁止模型自动调用
  [key: string]: unknown;               // 允许自定义扩展字段
}
```

虽然 `[key: string]: unknown` 在 TS 层面允许任意字段，但 pi **会识别**以下官方 frontmatter 字段（详见 [Agent Skills spec](https://agentskills.io/specification#frontmatter-required) 与 `repo/packages/coding-agent/docs/skills.md:141-149`）：

| 字段 | 是否必填 | 说明 |
|------|---------|------|
| `name` | 是 | 最长 64 字符，仅 a-z / 0-9 / `-`。coding-agent 包**不要求**与父目录同名（与官方标准不同） |
| `description` | 是 | 最长 1024 字符；为空则 Skill 不加载 |
| `disable-model-invocation` | 否 | `true` 时从系统提示词隐藏，仅 `/skill:name` 可调 |
| `license` | 否 | 许可证名称或指向包内文件的引用 |
| `compatibility` | 否 | 最长 500 字符，环境要求说明 |
| `metadata` | 否 | 任意键值对 |
| `allowed-tools` | 否（实验性） | 空格分隔的预批准工具列表 |

> 注意：`license` / `compatibility` / `metadata` / `allowed-tools` 这几个字段 coding-agent 包**不会在加载时强制校验或消费**（它们属于 spec 约定，由其他工具链或未来的 pi 版本使用），但写入 frontmatter 不会触发 diagnostic。

## 核心 API

### loadSkills() -- 从所有配置位置加载

```ts
function loadSkills(options: LoadSkillsOptions): LoadSkillsResult

interface LoadSkillsOptions {
  cwd: string;              // 工作目录（项目级 skills）
  agentDir: string;         // 全局配置目录（全局 skills）
  skillPaths: string[];     // 显式 Skill 路径（文件或目录）
  includeDefaults: boolean; // 是否包含默认 skills 目录
}

interface LoadSkillsResult {
  skills: Skill[];
  diagnostics: ResourceDiagnostic[];  // 冲突、格式错误等诊断信息
}
```

**默认加载顺序**：
1. 全局目录：`agentDir/skills/`
2. 项目目录：`cwd/.pi/skills/`
3. 显式路径：`skillPaths` 中的路径

### loadSkillsFromDir() -- 从单个目录加载

```ts
function loadSkillsFromDir(options: LoadSkillsFromDirOptions): LoadSkillsResult

interface LoadSkillsFromDirOptions {
  dir: string;    // 目标目录
  source: string; // 来源标识
}
```

**`source` 参数的魔法字符串**：`source` 不是自由文本——`createSkillSourceInfo`（`skills.ts:136-158`）会对几个特定值做映射，决定生成的 `SourceInfo.scope`：

| `source` 传入值 | 生成的 `scope` | 生成的 `source` 字段 |
|----------------|---------------|---------------------|
| `"user"` | `user` | `local` |
| `"project"` | `project` | `local` |
| `"path"` | `temporary`（默认） | `local` |
| 其他任意字符串 | `temporary`（默认） | 原样传入 |

> 影响：`scope` 会作用到碰撞优先级排序（见下方 [关键细节](#关键细节)）。如果你想让目录加载出来的 skills 在 `DefaultResourceLoader` 集成时被识别为 project 级，就必须传 `"project"`，而不是随手传一个自定义字符串。

**发现规则**：
- 如果目录包含 `SKILL.md` 文件，将其作为 Skill 根目录，**不再递归**
- 否则，加载根目录下的直接 `.md` 子文件
- 递归进入子目录查找 `SKILL.md`

### formatSkillsForPrompt() -- 格式化注入

```ts
function formatSkillsForPrompt(skills: Skill[]): string
```

将 Skill 列表格式化为符合 Agent Skills 标准的 XML 格式，用于注入系统提示词。`disableModelInvocation: true` 的 Skill 会被排除（只能显式调用）。

**实际输出示例**（假设有两个 Skill：`data-schema` 和 `api-helper`）：

```xml
The following skills provide specialized instructions for specific tasks.
Use the read tool to load a skill's file when the task matches its description.
When a skill file references a relative path, resolve it against the skill directory (parent of SKILL.md / dirname of the path) and use that absolute path in tool commands.

<available_skills>
  <skill>
    <name>data-schema</name>
    <description>Provides database table schemas and field descriptions</description>
    <location>/my-project/.pi/skills/data-schema/SKILL.md</location>
  </skill>
  <skill>
    <name>api-helper</name>
    <description>Internal API endpoint reference and request patterns</description>
    <location>/my-project/.pi/skills/api-helper/SKILL.md</location>
  </skill>
</available_skills>
```

> **重要**：正文（SKILL.md 内容）**不在此输出中**——这就是渐进式披露机制。模型看到索引后按需用 read tool 读取正文。详见下方 [集成踩坑](#集成踩坑实测skill-正文不自动注入需-read-tool-按需读取) 节。

### createSyntheticSourceInfo() -- 创建虚拟 Skill

```ts
function createSyntheticSourceInfo(path: string, options: {
  source: string;          // 来源标识
  scope?: "user" | "project" | "temporary";
  origin?: "package" | "top-level";
  baseDir?: string;
}): SourceInfo
```

用于创建不来自文件系统的虚拟 Skill：

```ts
import { createSyntheticSourceInfo, type Skill } from "@earendil-works/pi-coding-agent";

const virtualSkill: Skill = {
  name: "my-custom-skill",
  description: "Custom project-specific instructions",
  filePath: "/virtual/skills/custom/SKILL.md",
  baseDir: "/virtual/skills/custom",
  sourceInfo: createSyntheticSourceInfo("/virtual/skills/custom/SKILL.md", {
    source: "sdk",
    scope: "temporary",
  }),
  disableModelInvocation: false,
};
```

## 使用方式

### 方式一：通过 DefaultResourceLoader 的 skillsOverride

这是推荐方式，在现有 Skill 基础上过滤或追加：

```ts
import {
  createAgentSession,
  createSyntheticSourceInfo,
  DefaultResourceLoader,
  getAgentDir,
  SessionManager,
  type Skill,
} from "@earendil-works/pi-coding-agent";

const customSkill: Skill = {
  name: "my-skill",
  description: "My custom skill",
  filePath: "/virtual/skills/my-skill/SKILL.md",
  baseDir: "/virtual/skills/my-skill",
  sourceInfo: createSyntheticSourceInfo("/virtual/skills/my-skill/SKILL.md", { source: "sdk" }),
  disableModelInvocation: false,
};

const loader = new DefaultResourceLoader({
  cwd: process.cwd(),
  agentDir: getAgentDir(),
  skillsOverride: (current) => ({
    skills: [
      // 只保留特定 skills + 添加自定义
      ...current.skills.filter((s) => s.name.includes("browser") || s.name.includes("search")),
      customSkill,
    ],
    diagnostics: current.diagnostics,
  }),
});

await loader.reload();

// 查看所有已加载的 Skill
const { skills, diagnostics } = loader.getSkills();
console.log("Skills:", skills.map((s) => s.name));
if (diagnostics.length > 0) {
  console.log("Diagnostics:", diagnostics);
}

const { session } = await createAgentSession({
  resourceLoader: loader,
  sessionManager: SessionManager.inMemory(),
});
```

### 方式二：通过 createAgentSession 的 resourceLoader 传入

```ts
const loader = new DefaultResourceLoader({
  cwd: process.cwd(),
  agentDir: getAgentDir(),
  additionalSkillPaths: ["/extra/skills/dir"],  // 额外 Skill 路径
});

await loader.reload();
const { session } = await createAgentSession({ resourceLoader: loader });
```

### 方式三：关闭 Skill 自动发现

```ts
const loader = new DefaultResourceLoader({
  cwd: process.cwd(),
  agentDir: getAgentDir(),
  noSkills: true,  // 跳过自动发现；CLI 路径（--skill）和 additionalSkillPaths 仍会加载
});
```

## Skill 文件结构

一个典型的 Skill 目录结构：

```
.pi/
  skills/
    browser/
      SKILL.md          # Skill 内容文件
    search/
      SKILL.md
    deprecated/
      SKILL.md          # 可设置 disableModelInvocation: true
```

SKILL.md 文件格式：

```markdown
---
name: my-skill
description: Does something useful
---

# My Skill

## When to Use
- ...

## Instructions
- ...
```

## 在交互模式下使用 Skill

在交互式 pi 会话中，用户可以通过 `/skill:name` 显式调用一个 Skill：

```
/skill:browser
```

即使 `disableModelInvocation: true`，显式调用依然有效（`_expandSkillCommand` 从全量 skills 中 find，不过滤 `disableModelInvocation`）。

**显式调用后会发生什么**：`_expandSkillCommand`（`agent-session.ts:1300-1324`）读取 SKILL.md 全文，**strip 掉 frontmatter** 后包成下面的块注入用户消息：

```
<skill name="browser" location="/path/to/SKILL.md">
References are relative to /path/to.

（SKILL.md 正文）
</skill>
```

`/skill:foo` 后面跟的 args 会追加在块之后（如 `/skill:pdf-tools extract` 中的 `extract`）。SDK 直接调 `session.prompt("/skill:foo args")` 也会触发此展开——这是 skill 正文进入模型上下文的另一条路径（区别于 read tool 按需读取）。

## 完整工作流示例

```ts
import {
  createAgentSession,
  createSyntheticSourceInfo,
  DefaultResourceLoader,
  formatSkillsForPrompt,
  getAgentDir,
  loadSkills,
  SessionManager,
  type Skill,
} from "@earendil-works/pi-coding-agent";

// 1. 直接从目录加载
const result = loadSkills({
  cwd: "/my-project",
  agentDir: getAgentDir(),
  skillPaths: ["/extra/skills"],
  includeDefaults: true,
});

console.log(`Loaded ${result.skills.length} skills`);
console.log(`Diagnostics: ${result.diagnostics.length}`);

// 2. 格式化为 Prompt
const promptText = formatSkillsForPrompt(result.skills);
console.log("Prompt injection preview:", promptText.substring(0, 200));

// 3. 通过 ResourceLoader 集成到 Session
const loader = new DefaultResourceLoader({
  cwd: "/my-project",
  agentDir: getAgentDir(),
  skillsOverride: (current) => ({
    skills: [...current.skills],
    diagnostics: current.diagnostics,
  }),
});

await loader.reload();
const { session } = await createAgentSession({
  resourceLoader: loader,
  sessionManager: SessionManager.inMemory(),
});

try {
  await session.prompt("Use your skills to help me with this task.");
} finally {
  session.dispose();
}
```

## 关键细节

- Skill 文件自动发现的来源（按加载顺序）：
  1. `~/.pi/agent/skills/`（pi 用户级）
  2. `.pi/skills/`（pi 项目级，**仅扫描 `cwd/.pi/skills` 单目录**，不做祖先遍历）
  3. `~/.agents/skills/`（agents 用户级）
  4. `.agents/skills/`（agents 项目级，**含祖先遍历**：从 cwd 向上走到 git root，在每一层收集 `.agents/skills`）
  5. package-based skills（npm 安装的包内置 skills）：来自 npm 包的 `skills/` 目录或 `package.json` 的 `pi.skills` 字段；`DefaultResourceLoader.mapSkillPath`（`resource-loader.ts:635-655`）会把指向目录的 package skill 解析到其下的 `SKILL.md`
  6. CLI 路径（`--skill` 参数）
  7. `additionalSkillPaths` option
  > **pi 模式与 agents 模式的发现规则不同**（见 `collectSkillEntries()` 的 `mode` 分支）：pi 模式在 skills 根目录会加载**散装 `.md` 文件**（每个 `.md` 视为一个独立 skill）；agents 模式**只认 `SKILL.md`**（不加载散装 `.md`）。祖先遍历只针对 `.agents/skills`，`.pi/skills` 不做
- `disableModelInvocation: true` 的 Skill 不会出现在系统提示词中，只能通过 `/skill:name` 显式调用
- 同名 Skill 的冲突会被记录为 `ResourceDiagnostic`。**碰撞优先级取决于调用路径**：经 `DefaultResourceLoader` 集成（SDK 推荐路径）时，资源按 `resourcePrecedenceRank` 排序，**project（rank 0-1）优先于 user（rank 2-3）优先于 package（rank 4）**，CLI `--skill` 路径最先处理；直接调 `loadSkills({ includeDefaults: true })` 时则相反（先扫 user 目录再扫 project 目录，user 优先）
- `formatSkillsForPrompt()` 生成的 XML 格式符合 Agent Skills 标准
- `skillsOverride` 在默认加载后、赋值前执行，是对 Skill 列表进行定制的最佳时机
- `createSyntheticSourceInfo` 创建虚拟 Skill 时，`filePath` 和 `baseDir` 需要匹配并提供虚拟路径
- `loadSkillsFromDir()` 的递归规则确保每个 Skill 只从一个 SKILL.md 文件加载
- `SourceInfo.scope` 含义：`"user"` 全局级、`"project"` 项目级、`"temporary"` 临时级
- **Skill 名称校验**：必须仅含小写字母 `a-z`、数字 `0-9`、连字符 `-`，最长 64 字符；不能以连字符开头/结尾，不能含连续连字符（如 `--`）。不符合任一规则 → `ResourceDiagnostic`（warning），Skill 仍会加载但行为不可预期
- **描述字数限制**：最大 1024 字符，超出触发 diagnostic 警告；描述为空（`""` 或纯空白）则 Skill **不会加载**（返回 `null`）
- **ignore 文件过滤**：skill 发现过程支持 `.gitignore` / `.ignore` / `.fdignore` 规则过滤。**两条加载路径各自有独立的 `addIgnoreRules` 实现**：直接 `loadSkills` 路径见 `coding-agent/src/core/skills.ts:47-65`；`DefaultResourceLoader` 路径见 `package-manager.ts:245-263`（由 `collectSkillEntries` 调用）。被 ignore 规则匹配的 skill 文件会被跳过

## 集成踩坑（实测）：Skill 正文不自动注入，需 read tool 按需读取

**现象**：SKILL.md 已加载（`getSkills()` 返回了），但模型像"完全不知道 schema"——反复用工具探表结构，一次查询多消耗 5+ 次工具调用。

**根因**：pi-agent 的 skill 加载采用**渐进式披露**（Agent Skills 标准）——`formatSkillsForPrompt()` 只把 skill 的**索引**（`name` + `description` + `location`）注入系统提示词，正文 SKILL.md 从不自动注入。索引末尾会附一行说明：

```
Use the read tool to load a skill's file when the task matches its description.
```

模型看到相关 description 后，应主动用 **read tool** 读 `<location>` 指向的 SKILL.md 全文。但如果 `createAgentSession({ tools: [...] })` 的工具白名单**不含 `read`**（例如只配了业务工具 `["execute_sql"]`），模型永远拿不到正文，skill 形同虚设。

**对策**：使用 skill 的项目，tools 白名单必须包含 `read`：

```ts
const { session } = await createAgentSession({
  tools: ["execute_sql", "read"],  // ← read 用于让模型按需加载 skill 正文
  customTools: [executeSqlTool],
});
```

**安全提示**：内置 read tool 不支持路径白名单（详见 [06-tools.md](06-tools.md) 的「内置工具的安全边界」节）。若项目含 `.env`、源码等敏感文件，建议用 `createReadToolDefinition()` 包装一层路径校验，只允许读 `.pi/skills/` 目录。

**验证方法**：若怀疑 skill 未生效，打印 `formatSkillsForPrompt(loader.getSkills().skills)` 看实际注入内容——应该只看到索引 XML，看不到正文。确认正文是否进入模型上下文，查 trace 里 read 工具的调用次数。
