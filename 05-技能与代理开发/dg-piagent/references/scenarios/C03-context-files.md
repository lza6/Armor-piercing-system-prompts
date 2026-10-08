# 场景：注入项目上下文文件 (C03)

## 什么时候用 / 不用会怎样

**适合**：当你需要让 Agent 遵守一套**项目级的规则、约定、领域知识**时——例如「本项目所有 API 走 /v2 前缀」「部署需 2 人审核」「代码风格用 TypeScript strict 模式」。Context files 把这些指令以 Markdown 形式注入系统提示词，每次会话启动自动加载。

**三种典型用法**：
1. **物理文件自动发现**：在项目根放 `AGENTS.md`，SDK 自动扫描注入（零代码）
2. **运行时虚拟注入**：通过 `agentsFilesOverride` 在内存中构造内容，不落盘（CI/CD、动态规则）
3. **过滤/关闭**：二次开发场景下排除宿主环境的 `CLAUDE.md`，避免污染运行时 Agent

**不适合**：
- 单条简单指令 → 用 [A03](A03-system-prompt.md) 的 `systemPrompt` / `appendSystemPrompt` 直接写一行就够
- 需要正文渐进式披露（只注入索引，模型按需 read 正文） → 用 [C01](C01-custom-skill.md) 的 Skill
- 需要在运行时动态切换的规则 → Context files 在会话启动时加载，不能中途换；考虑 [E05-input-transform](E05-input-transform.md)
- 大段参考数据（schema、API 文档） → 用 [G01-context-injection](G01-context-injection.md) 按需注入

**不用会怎样**：Agent 只能凭通用知识 + 当前对话上下文猜项目约定，容易绕过规范、反复用工具探查已知信息。一次简单查询可能多消耗 5+ 次工具调用。

## 默认行为（不传 `agentsFilesOverride` 时）

`createAgentSession` 不显式传 `resourceLoader` 时，SDK 内部 `new DefaultResourceLoader(...)` 会自动发现 context files。

**发现流程**（`loadProjectContextFiles`）：

1. **全局 context file**：先看 `<agentDir>/AGENTS.md` 或 `<agentDir>/CLAUDE.md`（`agentDir` 默认 `~/.pi/agent/`）——找到就排第一
2. **从 `cwd` 向文件系统根逐级扫描**：在每级目录中按 `AGENTS.md` > `AGENTS.MD` > `CLAUDE.md` > `CLAUDE.MD` 的优先级查找
3. **顺序**：全局 → 父级目录 → 子级目录（即"从外到内"注入，最具体的规则排在最后，但 LLM 的注意力通常在首条最强——实际效果因模型而异）

**每个目录的检测优先级**：
- 同一目录下 `AGENTS.md` 优先于 `AGENTS.MD` 优先于 `CLAUDE.md` 优先于 `CLAUDE.MD`
- 只取第一个匹配的，**不会同时加载同目录的 AGENTS.md 和 CLAUDE.md**
- 大小写不敏感（`.MD` 也会被检测，但优先级低）

**去重**：按绝对路径去重（`seenPaths` Set），同一物理文件不会重复注入。此外，当 cwd 在嵌套 git worktree 内时，`findShadowedContextFile` 会检测 main repo 中被 worktree 副本覆盖的同一个 tracked AGENTS.md/CLAUDE.md 并跳过，避免重复加载。

## ⚠️ 关键事实：不受项目信任门槛控制（与 `.pi/` 资源不同）

这是 context files 与 `.pi/skills/` / `.pi/SYSTEM.md` 等 `.pi/` 资源的**关键差异**：

- `.pi/` 下所有资源（extensions / skills / prompts / themes / SYSTEM.md / APPEND_SYSTEM.md）受 `projectTrusted` 控制——项目不受信时全部跳过（见 [A06](A06-load-extensions.md) / [B04](B04-project-trust.md)）
- **Context files 不受此门槛控制**——`loadProjectContextFiles` 在 `reload()` 内是**无条件调用**的，独立于 `resolveProjectTrust` 流程

**原因**：context files 的语义是"用户/项目的全局规则"，扫描范围是 `cwd` 向上 + `agentDir`，不限于 `<cwd>/.pi/`，所以不走信任门槛。**这意味着任何能写入 `cwd` 或其祖先目录的人都能影响 Agent 行为**——部署时要注意 `cwd` 选择。

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| 物理文件 `AGENTS.md` / `CLAUDE.md` | 零代码自动发现 | [sdk_doc/11-context-files.md](../sdk_doc/11-context-files.md) |
| `DefaultResourceLoader` 的 `agentsFilesOverride` | 过滤/替换/追加 context files 列表 | [sdk_doc/08-resource-loader.md](../sdk_doc/08-resource-loader.md) |
| `noContextFiles: true` | 完全跳过 context files 发现 | [sdk_doc/08-resource-loader.md](../sdk_doc/08-resource-loader.md) |
| `loadProjectContextFiles()` | 不走 ResourceLoader，独立探测加载 | [sdk_doc/11-context-files.md](../sdk_doc/11-context-files.md) |
| `loader.getAgentsFiles()` | 查询已加载的文件列表 | [sdk_doc/08-resource-loader.md](../sdk_doc/08-resource-loader.md) |

## 核心数据模型

### ContextFile 结构（内联类型，无独立导出）

```ts
// 没有 export type ContextFile ——API 中使用内联结构
type ContextFile = {
  path: string;     // 文件路径或虚拟路径，任意字符串
  content: string;  // Markdown 内容，原样注入系统提示词
};
```

**字段语义**：
- `path`：**任意字符串**，不必是真实文件路径。自动发现时是绝对路径（如 `/home/user/project/AGENTS.md`），虚拟注入时可以是 `/virtual/RULES.md`、`/ci/generated-rules.md` 等。**这个值会原样写入系统提示词的 `<project_instructions path="...">` 标签**，LLM 能看到——所以建议取语义化路径名（不要泄露敏感绝对路径）
- `content`：**原样注入**的 Markdown 文本，不做任何解析或转换。frontmatter、代码块、链接都按字面字符进入 prompt

**注意**：`ContextFile` **没有导出为独立类型名**（[sdk_doc/11-context-files.md L18](../sdk_doc/11-context-files.md)），API 中以 `Array<{path: string; content: string}>` 形式内联出现。

### agentsFilesOverride 回调签名

```ts
agentsFilesOverride?: (
  base: { agentsFiles: Array<{ path: string; content: string }> }
) => {
  agentsFiles: Array<{ path: string; content: string }>;
};
```

**关键约束**：
- 回调在默认 `loadProjectContextFiles` 完成**之后**、最终赋值**之前**执行
- 输入 `base.agentsFiles` 是去重后的默认发现结果（可能为空数组，例如 `noContextFiles: true` 时）
- **必须返回完整的 `{ agentsFiles: [...] }` 对象**——不返回字段就真的没了（不是"保留默认"）

## 系统提示词组装机制（★ A03 横向直接适用）

**Context files 不是独立 prompt 源——它们是 `buildSystemPrompt()` 组装链的一环**：

```
final_system_prompt = buildSystemPrompt({
  customPrompt,        // ← 来自 systemPromptOverride 或 SYSTEM.md
  appendSystemPrompt,  // ← 来自 appendSystemPromptOverride 或 APPEND_SYSTEM.md
  contextFiles,        // ← 来自 loader.getAgentsFiles().agentsFiles ★ C03 关键
  skills,              // ← 来自 loader.getSkills().skills
  selectedTools,       // ← 工具白名单
  toolSnippets,        // ← 工具片段
  promptGuidelines,    // ← 工具附加规则
  cwd,
})
```

`contextFiles` 在最终 prompt 中的呈现：

```xml
<project_context>

Project-specific instructions and guidelines:

<project_instructions path="/virtual/AGENTS.md">
# 用户在 AGENTS.md 写的内容
- Use TypeScript strict mode
- ...
</project_instructions>

<project_instructions path="/home/user/project/CLAUDE.md">
...其他 context file 内容...
</project_instructions>

</project_context>
```

**两个关键事实**：

1. **即便设置了 `customPrompt`（systemPromptOverride 返回非空），contextFiles 仍会被追加到末尾**（`if (customPrompt)` 分支同样追加 contextFiles）。这是反直觉的——你以为完全替换了 system prompt，实际没有。**如果想完全屏蔽默认 context files，必须用 `noContextFiles: true` 或 `agentsFilesOverride: () => ({ agentsFiles: [] })`**
2. **调试时**：可以在系统提示词中搜索 `<project_context>` 定位所有注入内容；每个文件是独立的 `<project_instructions path="...">` 标签

更多 system prompt 组装链的细节见 [A03](A03-system-prompt.md)。

## 核心代码

### 方式一：物理文件自动发现（最常用，零代码）

把 `AGENTS.md` 放在项目根：

```
my-project/
├── AGENTS.md          # ← 项目级规则
├── src/
└── package.json
```

`AGENTS.md` 内容示例：

```markdown
# Project Rules

- All new code must be in TypeScript
- Use ESLint with the project config
- Never push directly to main
- Commits must include a description
```

SDK 调用（**无需任何 context files 相关代码**）：

```ts
import { createAgentSession, SessionManager } from "@earendil-works/pi-coding-agent";

const { session } = await createAgentSession({
  sessionManager: SessionManager.inMemory(),
});

// Agent 的系统提示词已自动包含 <cwd>/AGENTS.md 内容
try {
  await session.prompt("What are the project rules?");
} finally {
  session.dispose();
}
```

### 方式二：通过 `agentsFilesOverride` 注入虚拟文件（运行时动态规则）

不修改文件系统，在内存中构造规则：

```ts
import {
  createAgentSession,
  DefaultResourceLoader,
  getAgentDir,
  SessionManager,
} from "@earendil-works/pi-coding-agent";

const cwd = process.cwd();
const agentDir = getAgentDir();

const loader = new DefaultResourceLoader({
  cwd,
  agentDir,
  agentsFilesOverride: (base) => ({
    agentsFiles: [
      ...base.agentsFiles,  // 保留自动发现的（AGENTS.md / CLAUDE.md）
      {
        path: "/virtual/RULES.md",
        content: [
          "## Project Rules",
          "- Always use TypeScript for new files",
          "- Follow ESLint rules strictly",
          "- Never push to main directly",
          "",
          "## Code Style",
          "- Use async/await, never raw promises",
          "- Prefer functional components in React",
        ].join("\n"),
      },
      // 可以注入多个文件
      {
        path: "/virtual/DEPLOY.md",
        content: "Deployment must be approved by 2 reviewers.",
      },
    ],
  }),
});

await loader.reload();

// 验证加载结果（★ 官方示例的核心步骤，必做）
const { agentsFiles } = loader.getAgentsFiles();
console.log(`Loaded ${agentsFiles.length} context files:`);
for (const file of agentsFiles) {
  console.log(`  ${file.path}: ${file.content.length} chars`);
}

const { session } = await createAgentSession({
  resourceLoader: loader,
  sessionManager: SessionManager.inMemory(),
});

try {
  await session.prompt("What are the project rules for code style?");
} finally {
  session.dispose();
}
```

## 变体

### 变体 A：过滤已有的 context files

只保留全局规则，排除项目级：

```ts
import { DefaultResourceLoader, getAgentDir } from "@earendil-works/pi-coding-agent";

const loader = new DefaultResourceLoader({
  cwd: process.cwd(),
  agentDir: getAgentDir(),
  agentsFilesOverride: (base) => ({
    agentsFiles: base.agentsFiles.filter(
      (f) => !f.path.includes(process.cwd()),  // 排除 cwd 及子目录的
    ),
  }),
});

await loader.reload();
```

### 变体 B：完全关闭 context files 发现（★ 二次开发关键）

用 pi-agent 开发第三方 Agent 时，宿主环境的 `CLAUDE.md`（给 Claude Code 开发者看的项目文档）会被误注入给运行时 Agent——必须关闭：

```ts
import { DefaultResourceLoader } from "@earendil-works/pi-coding-agent";

const loader = new DefaultResourceLoader({
  cwd: process.cwd(),
  agentDir: getAgentDir(),
  noContextFiles: true,  // ← 完全不加载 context files
});

await loader.reload();
const { agentsFiles } = loader.getAgentsFiles();
console.log(agentsFiles.length);  // 0
```

**详细踩坑见** [sdk_doc/11-context-files.md 集成踩坑一节](../sdk_doc/11-context-files.md#集成踩坑实测用-pi-agent-二次开发第三方-agent-时必须隔离宿主-claudemd)。

### 变体 C：`loadProjectContextFiles()` 独立探测（不走 ResourceLoader）

只想知道会扫描到哪些文件，不需要创建 session：

```ts
import { loadProjectContextFiles } from "@earendil-works/pi-coding-agent";

const files = loadProjectContextFiles({
  cwd: "/my-project/src",
  agentDir: "/home/user/.pi/agent",
});

console.log(`Found ${files.length} context files:`);
for (const file of files) {
  console.log(`  ${file.path}: ${file.content.length} chars`);
}
```

**返回类型**：`Array<{ path: string; content: string }>`——**注意是数组，不是 `{ agentsFiles }` 对象**（这是 `loadProjectContextFiles` 的返回，与 `loader.getAgentsFiles()` 不同）。

### 变体 D：运行时查询已加载文件

```ts
// loader.reload() 之后任何时刻都能查询
const { agentsFiles } = loader.getAgentsFiles();

for (const file of agentsFiles) {
  console.log(`  ${file.path}`);
  console.log(`    -> ${file.content.split("\n")[0]}`);  // 首行预览
}
```

**注意**：`getAgentsFiles()` 返回 `{ agentsFiles: ContextFile[] }` 对象，**不是数组**——必须解构使用。

## 陷阱

### 1. `cwd` 向上扫描可能误抓宿主环境的 CLAUDE.md（★ 最常踩坑）

`loadProjectContextFiles` 从 `cwd` 一路向上扫到文件系统根。如果你的项目嵌套在某个父目录下，父目录（甚至磁盘根）的 `CLAUDE.md` 会被一起注入，污染源难以追踪。

**正确做法**：
- 二次开发场景 → `noContextFiles: true`
- 需要自动发现但限定范围 → `agentsFilesOverride` 中 `filter`

### 2. `customPrompt`（systemPromptOverride）**不能**屏蔽默认 context files（A03 横向）

你以为完全替换了 system prompt，实际 `buildSystemPrompt()` 在 `if (customPrompt)` 分支**仍会追加** contextFiles。

```ts
// ❌ 以为这样能完全自定义 system prompt，实际 context files 仍会追加
const loader = new DefaultResourceLoader({
  cwd, agentDir,
  systemPromptOverride: () => "You are a custom agent.",
});
// Agent 的 prompt 仍是 "You are a custom agent.\n\n<project_context>..."

// ✅ 完全屏蔽需要这样
const loader = new DefaultResourceLoader({
  cwd, agentDir,
  systemPromptOverride: () => "You are a custom agent.",
  noContextFiles: true,  // ← 关键
});
```

### 3. 不受项目信任门槛控制（与 `.pi/` 资源不同，横向反例）

`.pi/skills/` / `.pi/SYSTEM.md` 等受 `projectTrusted` 控制（[A06](A06-load-extensions.md) / [B04](B04-project-trust.md) 横向），**但 context files 不受此门槛**（`loadProjectContextFiles` 无条件调用）。

**含义**：
- `resolveProjectTrust` 返回 `false` 也不会跳过 context files
- 任何能写入 `cwd` 或其祖先目录的人都能影响 Agent 行为（安全敏感场景要注意）
- 与 `.pi/skills/` 的"项目不受信就跳过"语义不同——这是横向提示 A06 的**精确边界**

### 4. `agentsFilesOverride` 是「最终赋值前」拦截点（不返回就真没了）

回调输入是默认 `loadProjectContextFiles` 的结果，输出是**最终使用的值**。

```ts
// ❌ 不返回字段，运行时会抛 TypeError：resolvedAgentsFiles 是 undefined，取 .agentsFiles 直接崩溃
agentsFilesOverride: (base) => {
  base.agentsFiles.push({ path: "/x.md", content: "..." });  // 直接改 base
  // 没返回 { agentsFiles: ... }，结果会是 undefined
}

// ✅ 必须返回完整对象
agentsFilesOverride: (base) => ({
  agentsFiles: [...base.agentsFiles, { path: "/x.md", content: "..." }],
}),
```

### 5. 同目录只加载一个 context file

`loadContextFileFromDir` 按 `AGENTS.md` > `AGENTS.MD` > `CLAUDE.md` > `CLAUDE.MD` 顺序检测，**找到第一个就返回**。如果你想"AGENTS.md 放主规则、CLAUDE.md 放补充规则"，**做不到**——必须合并到一个文件，或用 `agentsFilesOverride` 显式追加。

### 6. `path` 字段会出现在系统提示词中

`<project_instructions path="...">` 的 `path` 值是 LLM 可见的。虚拟注入时**不要泄露敏感绝对路径**（如 `/home/username/secret-project/...`），用语义化路径（`/virtual/RULES.md`）。

### 7. 必须调用 `reload()` 后 getter 才有值

`new DefaultResourceLoader(...)` 只是构造，**不会触发加载**。必须 `await loader.reload()` 之后 `loader.getAgentsFiles()` 才返回正确结果。`createAgentSession` 内部会自动调 `reload`，自己手动构造时别忘。

## 横向联动

- **[A03](A03-system-prompt.md)**：context files 是 system prompt 组装链一环——本文档的核心机制说明
- **[A06](A06-load-extensions.md)**：信任门槛横向——**注意 C03 是反例**（context files 不受门槛控制）
- **[B04](B04-project-trust.md)**：信任机制详解
- **[C01](C01-custom-skill.md)** / **[C02](C02-prompt-templates.md)**：并列机制（Skill / Prompt 模板）
- **[E05](E05-input-transform.md)**：运行时动态改写（context files 是会话启动时加载，不能动态切换）
- **[G01](G01-context-injection.md)**：按需注入大量参考数据
- **[sdk_doc/08-resource-loader.md](../sdk_doc/08-resource-loader.md)**：ResourceLoader 总览
- **[sdk_doc/11-context-files.md](../sdk_doc/11-context-files.md)**：context files 详细规则 + 集成踩坑（二次开发场景必读）
