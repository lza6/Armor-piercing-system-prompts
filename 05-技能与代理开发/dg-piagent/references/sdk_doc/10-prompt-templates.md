# 10 - 提示词模板 (Prompt Templates)

## 这是什么

Prompt Templates 是 pi-agent 的**可复用提示词模板系统**。每个模板是一个 Markdown 文件，存储在 `.pi/prompts/` 目录中。用户在交互模式下通过 `/templatename` 语法调用，Agent 会自动展开模板内容并替换参数占位符（`$1`, `$2`, `$@` 等）。

**什么时候用**：当某些提示词是固定流程且需要参数化复用时——典型的如部署（`/deploy v2.0.0 production`）、代码审查（`/review src/foo.ts`）、测试生成等。把长 prompt 固化成模板的好处是：复用（团队共享同一份）、参数化（同一模板接不同入参）、减少手敲长文本的出错率。如果只是一次性短 prompt，直接 `session.prompt("...")` 即可，不必做成模板。

## PromptTemplate 类型

> 以下为 SDK 导出版（`@earendil-works/pi-coding-agent`，含 6 字段）。底层 `pi-agent-core` 的 `PromptTemplate`（`packages/agent/src/harness/types.ts`）仅含 `name`/`description?`/`content` 三字段，没有 `argumentHint`/`sourceInfo`/`filePath`。SDK 用户从 coding-agent 导入，只接触以下 6 字段版。

```ts
interface PromptTemplate {
  name: string;            // 模板名，用户通过 /name 调用
  description: string;     // 功能说明
  argumentHint?: string;   // 参数提示（帮助用户了解需要传什么参数）
  content: string;         // 模板正文（可含 $1, $2, $@ 等占位符）
  sourceInfo: SourceInfo;  // 来源信息
  filePath: string;        // 模板文件路径
}
```

## 核心 API

### loadPromptTemplates() -- 加载模板

```ts
function loadPromptTemplates(options: LoadPromptTemplatesOptions): PromptTemplate[]

interface LoadPromptTemplatesOptions {
  cwd: string;              // 工作目录
  agentDir: string;         // 全局配置目录
  promptPaths: string[];    // 显式模板路径
  includeDefaults: boolean; // 是否包含默认目录
}
```

**加载顺序**：
1. 全局目录：`agentDir/prompts/`
2. 项目目录：`cwd/.pi/prompts/`
3. 显式路径：`promptPaths`

> 此顺序描述的是**直接调用 `loadPromptTemplates({ includeDefaults: true })`** 时的行为。经 `DefaultResourceLoader` 加载时，默认目录的发现由 `PackageManager` 统一注入到 `promptPaths`（顺序是全局 → 项目，由 settings 合并），再叠 CLI `--prompt-template` 路径和 `additionalPromptTemplatePaths`，且内部调用始终传 `includeDefaults: false`。所以经 `DefaultResourceLoader` 的真实顺序是"全局+项目（settings 合并）→ CLI → additional"。

### expandPromptTemplate() -- 展开模板

```ts
function expandPromptTemplate(text: string, templates: PromptTemplate[]): string
```

检查输入文本是否以模板名开头，如果是则展开模板内容。如果不匹配，原样返回——分两种情况：① 文本不以 `/` 开头，直接返回原文；② 以 `/` 开头但模板名不存在，也返回原文（此时开头的 `/` 会保留在文本中发给 LLM）。需要把 `/something` 原样传给 LLM 时可借此实现，或用 `session.prompt(text, { expandPromptTemplates: false })` 显式关闭。

### substituteArgs() -- 参数替换

```ts
function substituteArgs(content: string, args: string[]): string
```

支持以下占位符语法：
- `$1`, `$2`, ... -- 位置参数
- `${N:-default}` -- 位置参数带默认值（如 `${1:-production}` 表示"用第 1 个参数，没有时默认为 production"）
- `$@` 和 `$ARGUMENTS` -- 所有参数
- `${@:-default}` -- 所有参数为空时用默认值（如 `${@:-production}` 表示"没传任何参数时默认为 production"）
- `${ARGUMENTS:-default}` -- 同上，`$ARGUMENTS` 的默认值变体
- `${@:N}` -- 从第 N 个参数开始（bash 风格切片）
- `${@:N:L}` -- 从第 N 个开始取 L 个参数

> 替换只在模板字符串上执行，参数值中包含的 `$1`, `$@`, `$ARGUMENTS` 等模式不会递归替换。

### parseCommandArgs() -- 解析命令参数

```ts
function parseCommandArgs(argsString: string): string[]
```

按 bash 风格解析命令参数，支持引号包裹：

```ts
parseCommandArgs('hello "world test" foo');
// => ["hello", "world test", "foo"]
```

## 使用方式

### 方式一：通过 DefaultResourceLoader 的 promptsOverride

```ts
import {
  createAgentSession,
  createSyntheticSourceInfo,
  DefaultResourceLoader,
  getAgentDir,
  type PromptTemplate,
  SessionManager,
} from "@earendil-works/pi-coding-agent";

// 创建虚拟模板
const deployTemplate: PromptTemplate = {
  name: "deploy",
  description: "Deploy the application",
  argumentHint: "<environment>",
  filePath: "/virtual/prompts/deploy.md",
  sourceInfo: createSyntheticSourceInfo("/virtual/prompts/deploy.md", { source: "sdk" }),
  content: `# Deploy Instructions for $1

1. Build: npm run build
2. Test: npm test
3. Deploy to $1: npm run deploy -- --env $1`,
};

const loader = new DefaultResourceLoader({
  cwd: process.cwd(),
  agentDir: getAgentDir(),
  promptsOverride: (current) => ({
    prompts: [...current.prompts, deployTemplate],
    diagnostics: current.diagnostics,
  }),
});

await loader.reload();

// 查看所有模板
const discovered = loader.getPrompts().prompts;
for (const t of discovered) {
  console.log(`  /${t.name}: ${t.description}`);
}

const { session } = await createAgentSession({
  resourceLoader: loader,
  sessionManager: SessionManager.inMemory(),
});
```

### 方式二：通过 additionalPromptTemplatePaths

```ts
const loader = new DefaultResourceLoader({
  cwd: process.cwd(),
  agentDir: getAgentDir(),
  additionalPromptTemplatePaths: ["/extra/prompts/dir"],
});

await loader.reload();
const { session } = await createAgentSession({ resourceLoader: loader });
```

### 方式三：关闭模板自动发现

```ts
const loader = new DefaultResourceLoader({
  cwd: process.cwd(),
  agentDir: getAgentDir(),
  noPromptTemplates: true,  // 跳过 settings/扩展声明的默认 prompt 路径（含 .pi/prompts/ 和全局 prompts 目录的自动发现）；CLI --prompt-template 路径和 additionalPromptTemplatePaths 仍生效
});
```

### 方式四：运行时访问已加载的模板

通过 `session.promptTemplates` 可以在运行时查看所有已加载的模板：

```ts
const { session } = await createAgentSession({ resourceLoader: loader });

// 查看所有模板
for (const t of session.promptTemplates) {
  console.log(`  /${t.name}: ${t.description}`);
}
```

> `loadPromptTemplates` / `expandPromptTemplate` / `substituteArgs` / `parseCommandArgs` 是内部函数，未从 SDK 公开导出。如需手动展开模板，建议使用方式一的 `promptsOverride` 集成到 ResourceLoader，或直接调用 `session.prompt("/deploy production")` 由 Agent 自动展开。

## 模板文件格式

### 目录结构

```
.pi/
  prompts/
    deploy.md
    review.md
    test.md
```

### 文件内容示例

**deploy.md**:

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

$@
```

> 模板的 `name` 由文件名（不含扩展名）决定。frontmatter 中 `description` 和 `argument-hint` 可选（注意 YAML 键名是 kebab-case）。如果未提供 `description`，会取正文第一行非空文本截断到 60 字符作为描述；当该行实际长度超过 60 字符时，会在末尾追加 `...`（即最终最长 63 字符）。

## 交互模式调用

在 pi 交互模式中，用户输入以 `/` 开头时自动匹配模板：

```
/deploy production
```

Agent 会将上述输入展开为模板内容，并把 `production` 替换到 `$1` 位置。

匹配流程：
1. 提取 `/` 后的第一个词作为模板名
2. 在已加载的 `PromptTemplate[]` 中按 `name` 匹配
3. 剩余部分作为参数，传递给 `substituteArgs()` 进行替换
4. 将展开后的内容发送给 LLM

## 完整示例：带参数的部署模板

```ts
import {
  createAgentSession,
  createSyntheticSourceInfo,
  DefaultResourceLoader,
  getAgentDir,
  type PromptTemplate,
  SessionManager,
} from "@earendil-works/pi-coding-agent";

// 1. 定义带有多个参数的模板
const deployTemplate: PromptTemplate = {
  name: "deploy",
  description: "Deploy application",
  argumentHint: "<env> <version>",
  filePath: "/virtual/prompts/deploy.md",
  sourceInfo: createSyntheticSourceInfo("/virtual/prompts/deploy.md", { source: "sdk" }),
  content: `# Deploy

Deploy version **$1** to the **$2** environment.

All arguments: $@

## Steps
1. Tag release v$1
2. Build Docker image
3. Push to $2 registry
4. Apply Terraform for $2`,
};

// 2. 集成到 ResourceLoader
const loader = new DefaultResourceLoader({
  cwd: process.cwd(),
  agentDir: getAgentDir(),
  promptsOverride: (current) => ({
    prompts: [...current.prompts, deployTemplate],
    diagnostics: current.diagnostics,
  }),
});

await loader.reload();
const { session } = await createAgentSession({
  resourceLoader: loader,
  sessionManager: SessionManager.inMemory(),
});

// 3. 通过 session.prompt() 触发模板 —— 输入 "/deploy v2.0.0 production" 时自动展开
//    或者通过 session.promptTemplates 查看所有已加载模板
for (const t of session.promptTemplates) {
  console.log(`  /${t.name}: ${t.description}`);
}

try {
  await session.prompt("/deploy v2.0.0 production");
} finally {
  session.dispose();
}
```

## 冲突与优先级（重要陷阱）

### `/` 命令的内部优先级链

在 `session.prompt()` 中，当输入以 `/` 开头且 `expandPromptTemplates` 未关闭（默认 true）时，按以下顺序依次尝试，**前者命中即返回，后者被截胡**：

1. **扩展命令**（`pi.registerCommand` 注册的 `/xxx`）——命中则直接执行并返回，不展开模板
2. **`/skill:name` 技能命令**——匹配 `_expandSkillCommand`
3. **提示词模板**——匹配 `expandPromptTemplate`

> 含义：若自定义模板名与扩展命令撞名（如 `/help`），模板永远无法触发；若模板名以 `skill:` 开头，会被 skill 命令截胡。`session.steer()` / `session.followUp()` 略有不同——它们对扩展命令直接抛错（不可排队），但仍走 `_expandSkillCommand` → `expandPromptTemplate`，所以 skill 命令 > 模板的优先级同样适用。

### 同名模板冲突

多个来源声明同名模板时**先到先得**（按加载顺序，首个胜出），后续同名的模板被丢弃并记录一条 `type: "collision"` 诊断。通过 `loader.getPrompts().diagnostics` 可读取冲突详情：

```ts
const { prompts, diagnostics } = loader.getPrompts();
for (const d of diagnostics.filter((x) => x.type === "collision")) {
  // d.collision: { resourceType, name, winnerPath, loserPath }
  console.log(`模板 /${d.collision!.name} 冲突：胜出 ${d.collision!.winnerPath}，丢弃 ${d.collision!.loserPath}`);
}
```

> 实践建议：`promptsOverride` 在默认加载之后执行，可在这里做最终裁决（改名、过滤、合并），避免默认模板覆盖你自定义的同名模板而不自知。

## 关键细节

- 模板名由文件名决定（不含 `.md` 扩展名），frontmatter 中的 `description` 用于 UI 列表
- 模板文件从 `.pi/prompts/`（项目级）和 `~/.pi/agent/prompts/`（全局级）自动发现
- 交互模式中 `/templatename` 只匹配第一个词作为模板名，后续内容作为参数
- `$@` 和 `$ARGUMENTS` 替换为所有参数的原始文本
- `${@:N:L}` 语法支持参数切片，与 bash 行为一致；`${@:-default}` / `${ARGUMENTS:-default}` 支持所有参数为空时的默认值
- `expandPromptTemplate()` 不匹配时原样返回（含两种情况：不以 `/` 开头、或以 `/` 开头但模板名不存在——后者 `/` 会保留发给 LLM）
- `promptsOverride` 在默认加载后执行，是定制模板列表的推荐方式
- 同名模板冲突：先到先得，冲突记录在 `loader.getPrompts().diagnostics`（`type: "collision"`）；详见上文「冲突与优先级」一节
- `session.prompt()` 内部 `/` 命令优先级：扩展命令 > `/skill:name` > 模板（详见上文）
- `session.prompt()` 默认展开模板，可通过 `session.prompt(text, { expandPromptTemplates: false })` 禁用——需要把 `/something` 原样传给 LLM 时使用
- 模板展开同样作用于 `session.steer()` 和 `session.followUp()`（不受 `expandPromptTemplates` 选项控制，始终展开）
