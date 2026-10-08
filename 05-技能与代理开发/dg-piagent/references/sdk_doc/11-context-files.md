# 11 - 上下文文件 (Context Files)

## 这是什么

Context Files 是 pi-agent 的**项目级指令注入机制**。它们是指定项目行为的 Markdown 文件（`AGENTS.md` 或 `CLAUDE.md`），在会话启动时被自动发现并加载到系统提示词中。每个上下文文件包含该项目特有的规则、约定和指导。

## 结构

每个上下文文件是一个简单对象：

```ts
{
  path: string;     // 文件的完整路径或虚拟路径
  content: string;  // 文件的 Markdown 内容
}
```

> 注意：`ContextFile` 没有导出为独立类型名。API 中使用内联的 `Array<{path: string; content: string}>` 形式。每个上下文文件由路径和内容两部分组成。

## 核心 API

### loadProjectContextFiles() -- 从目录发现

```ts
function loadProjectContextFiles(options: {
  cwd: string;
  agentDir: string;
}): Array<{ path: string; content: string }>
```

**发现逻辑**（从高优先级到低）：

1. **全局代理目录**：`agentDir/AGENTS.md` 或 `agentDir/CLAUDE.md`（约 `~/.pi/agent/`）
2. **从 cwd 向根目录逐级递归**：在每级目录中按 `AGENTS.md` > `AGENTS.MD` > `CLAUDE.md` > `CLAUDE.MD` 的优先级查找
3. 找到的文件会按 **从父级到子级** 的顺序排列注入到提示词中

```ts
import { loadProjectContextFiles } from "@earendil-works/pi-coding-agent";

const files = loadProjectContextFiles({
  cwd: "/my-project",
  agentDir: "/home/user/.pi/agent",
});

console.log(`Found ${files.length} context files:`);
for (const file of files) {
  console.log(`  ${file.path}: ${file.content.length} chars`);
}
```

### DefaultResourceLoader 的 getAgentsFiles()

```ts
interface ResourceLoader {
  getAgentsFiles(): {
    agentsFiles: Array<{ path: string; content: string }>;
  };
  // ...
}
```

`DefaultResourceLoader` 在 `reload()` 时自动调用 `loadProjectContextFiles()`，并将结果存储在 `agentsFiles` 中。

```ts
const loader = new DefaultResourceLoader({ cwd, agentDir });
await loader.reload();

const { agentsFiles } = loader.getAgentsFiles();
console.log("Context files loaded:", agentsFiles.length);
```

## 使用方式

### 方式一：通过 DefaultResourceLoader 自动发现

这是标准用法，无需额外代码：

```ts
import {
  createAgentSession,
  DefaultResourceLoader,
  getAgentDir,
} from "@earendil-works/pi-coding-agent";

const loader = new DefaultResourceLoader({
  cwd: process.cwd(),
  agentDir: getAgentDir(),
  // 默认自动发现 AGENTS.md / CLAUDE.md
});

await loader.reload();

// 查看发现的上下文文件
const { agentsFiles } = loader.getAgentsFiles();
for (const file of agentsFiles) {
  console.log(`Context file: ${file.path}`);
  console.log(`Content preview: ${file.content.substring(0, 100)}...`);
}

const { session } = await createAgentSession({ resourceLoader: loader });
```

### 方式二：通过 agentsFilesOverride 注入虚拟文件

在不修改文件系统的情况下注入额外的上下文规则：

```ts
import {
  createAgentSession,
  DefaultResourceLoader,
  getAgentDir,
  SessionManager,
} from "@earendil-works/pi-coding-agent";

const loader = new DefaultResourceLoader({
  cwd: process.cwd(),
  agentDir: getAgentDir(),
  agentsFilesOverride: (base) => ({
    agentsFiles: [
      ...base.agentsFiles,  // 保留自动发现的
      {
        path: "/virtual/AGENTS.md",
        content: `# Project Guidelines

## Code Style
- Use TypeScript strict mode
- No \`any\` types
- Prefer \`const\` over \`let\`

## Git Workflow
- Branch naming: feature/xxx, fix/xxx
- Squash merge to main`,
      },
    ],
  }),
});

await loader.reload();
const { session } = await createAgentSession({
  resourceLoader: loader,
  sessionManager: SessionManager.inMemory(),
});
```

### 方式三：过滤已有的上下文文件

```ts
const loader = new DefaultResourceLoader({
  cwd: process.cwd(),
  agentDir: getAgentDir(),
  agentsFilesOverride: (base) => ({
    // 只保留全局 context file，排除项目级的
    agentsFiles: base.agentsFiles.filter((f) =>
      f.path.startsWith(getAgentDir()),
    ),
  }),
});

await loader.reload();
```

### 方式四：关闭上下文文件发现

```ts
const loader = new DefaultResourceLoader({
  cwd: process.cwd(),
  agentDir: getAgentDir(),
  noContextFiles: true,  // 完全不加载上下文文件
});

await loader.reload();
const { agentsFiles } = loader.getAgentsFiles();
console.log(agentsFiles.length);  // 0
```

## 文件发现示例

假设文件系统结构如下：

```
/home/user/.pi/agent/
  AGENTS.md                    # 全局规则（优先级最高，注入最先）

/my-project/
  parent/
    project/
      AGENTS.md                # 项目级规则
      src/
        CLAUDE.md              # 子目录规则（优先级最低，注入最后）
```

`cwd` 设为 `/my-project/parent/project/src/` 时，`loadProjectContextFiles()` 的发现顺序为：

1. `/home/user/.pi/agent/AGENTS.md` （全局）
2. `/my-project/parent/project/AGENTS.md` （项目级）
3. `/my-project/parent/project/src/CLAUDE.md` （子目录）

注入到系统提示词的顺序同样是从全局到局部（父级 -> 子级）。

## 完整示例：项目规则注入

```ts
import {
  createAgentSession,
  DefaultResourceLoader,
  getAgentDir,
  loadProjectContextFiles,
  SessionManager,
} from "@earendil-works/pi-coding-agent";

// 1. 直接使用底层 API 发现
const cwd = "/my-project/src";
const agentDir = getAgentDir();

const rawFiles = loadProjectContextFiles({ cwd, agentDir });
console.log(`Direct discovery found ${rawFiles.length} files`);

// 2. 通过 DefaultResourceLoader 集成（推荐）
const loader = new DefaultResourceLoader({
  cwd,
  agentDir,
  agentsFilesOverride: (base) => {
    // 追加一条运行时规则
    return {
      agentsFiles: [
        ...base.agentsFiles,
        {
          path: "/virtual/RUNTIME.md",
          content: `# Runtime Context
- Current date: ${new Date().toISOString().split("T")[0]}
- Git branch: main
- Environment: ${process.env.NODE_ENV || "development"}`,
        },
      ],
    };
  },
});

await loader.reload();

// 3. 验证加载结果
const { agentsFiles } = loader.getAgentsFiles();
for (const file of agentsFiles) {
  console.log(`  ${file.path}`);
  console.log(`    -> ${file.content.split("\n")[0]}`);
}

// 4. 创建会话
const { session } = await createAgentSession({
  resourceLoader: loader,
  sessionManager: SessionManager.inMemory(),
});

// Agent 的系统提示词中将包含所有 agentsFiles 的内容
try {
  await session.prompt("What are the project rules for code style?");
} finally {
  session.dispose();
}
```

## 关键细节

- **文件优先级**：`AGENTS.md` > `AGENTS.MD` > `CLAUDE.md` > `CLAUDE.MD`
- **注入顺序**：从父级到子级、从全局到项目级逐级注入
- **大小写不敏感**：`AGENTS.md` 和 `AGENTS.MD` 都会被检测
- **`noContextFiles: true`**：跳过 `AGENTS.md` / `CLAUDE.md` 的自动发现（base 为空），`agentsFilesOverride` 仍可工作
- **`agentsFilesOverride`**：在默认发现后执行，保留原有结果并追加/过滤
- **虚拟文件**：path 可以是任意字符串（如 `/virtual/xxx.md`），content 会原样注入
- **与 `loadProjectContextFiles()` 的关系**：`DefaultResourceLoader` 内部调用此函数，Override 在调用后执行
- **跨项目复用**：全局 `~/.pi/agent/AGENTS.md` 对所有项目生效
- 每个 project 可以通过自己的 `AGENTS.md` / `CLAUDE.md` 覆盖或补充全局规则
- **注入格式**：所有文件包裹在同一 `<project_context>` 块中，每个文件是独立的 `<project_instructions path="...">` 标签。调试时可在系统提示词中搜索 `<project_context>` 定位所有注入内容
- **去重**：同名文件按绝对路径去重（`seenPaths`），同一物理文件不会重复注入
- **Git worktree 去重**：当 git linked worktree 嵌套在主仓库目录下时，`findShadowedContextFile()` 会自动跳过被 shadow 的重复 `AGENTS.md` / `CLAUDE.md`，避免同一文件被注入两次

## 集成踩坑：用 pi-agent 二次开发第三方 Agent 时，必须隔离宿主 CLAUDE.md

**现象**：用 pi-agent 开发一个第三方 Agent（不是把 pi-agent 自己当 coding agent 用），项目根有 `CLAUDE.md`（给 Claude Code 开发者看的项目文档）。运行时 Agent 的系统提示词里混入了这份 CLAUDE.md，而开发者写的 `.pi/SYSTEM.md` 人设、`.pi/skills/` 业务 skill 反而被淹没或行为异常。

**后果**：Agent 收到的系统提示词是"开发者文档"而非"运行时指令"，模型理解错位。更隐蔽的是——pi-agent 默认从 cwd 向上递归扫描 CLAUDE.md，父级目录（甚至磁盘根）若也有 CLAUDE.md，会被一起注入，污染源难以追踪。

**根因**：pi-agent 的 context file 兼容机制（支持 AGENTS.md / CLAUDE.md）是为"pi-agent 自己当 coding agent"的场景设计的——那个场景下 CLAUDE.md 就是给 agent 看的。但"用 pi-agent 开发另一个 Agent"时，CLAUDE.md 的语义错位了：它是宿主 Claude Code 的开发约定，不是运行时 Agent 的指令。

**对策**：二次开发场景下用 `noContextFiles: true` 关闭 context file 发现。SYSTEM.md / skills 走独立机制，不受影响：

```ts
const loader = new DefaultResourceLoader({
  cwd,
  agentDir: cwd,
  noContextFiles: true,  // ← 关键：堵住 CLAUDE.md/AGENTS.md 兼容机制
});
```

**判断标准**：
- 如果你的 pi-agent 应用是**给别人用的产品**（运行时 Agent 服务最终用户）→ 用 `noContextFiles: true`
- 如果是**自己用 pi-agent 当 coding 助手**（CLAUDE.md 本来就是给它看的）→ 保留默认

**举一反三**：同类"宿主环境与运行时 Agent 语义错位"问题也存在于 skills——pi-agent 默认会扫描全局目录 `~/.pi/agent/skills/`（宿主环境安装的 xxx-* 工具类 skill 等），把它们和项目 `.pi/skills/` 一起注入。二次开发时建议用 `skillsOverride` 白名单只保留业务 skill，详见 [09-skills.md](09-skills.md)。
