# 场景：官方 Subagent 扩展 (I05)

## 这是什么 / 不是什么

**是**：pi 官方仓库自带的**示例扩展** `examples/extensions/subagent/`——它注册了一个名为 `subagent` 的工具，让主 Agent 通过 tool call 调度专业化子 Agent 执行子任务。每个子 Agent 是一个**独立的 `pi` 子进程**，有完全隔离的上下文窗口。

> [!IMPORTANT]
> **Subagent 是扩展，不是 SDK 内置能力**。源码位置：[`packages/coding-agent/examples/extensions/subagent/`](https://github.com/EarendilElon/pi/tree/main/packages/coding-agent/examples/extensions/subagent)。要使用它**必须先手动安装**（symlink 到 `~/.pi/agent/extensions/`，见下方「安装步骤」）。`createAgentSession` 的 `options` 里**没有** `subagent` 相关参数。

**不是**：
- **不是 SDK 内置工具**：`createAgentSession({ tools: ["subagent"] })` **不会生效**。内置工具清单见 [06-tools.md](../sdk_doc/06-tools.md)。subagent 是扩展通过 `pi.registerTool` 注册的。
- **不是 `settings.json` 配置项**：`agentScope` / `confirmProjectAgents` **不是 settings.json 字段**（`Settings` interface 完全没有这两项，见 [`settings-manager.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/src/core/settings-manager.ts)）。它们是 **`subagent` 工具的 tool call 参数**——由 LLM 在调用 subagent 工具时传入。
- **不是同进程子 Agent**：同进程子 Agent 用 `createAgentSession` 在扩展 execute 内创建临时 session，见 [H06 模式 3b](H06-multi-agent.md)。
- **不是 RPC 多用户服务**：每个用户一个 session 的服务架构见 [sdk_doc/21-multi-agent.md](../sdk_doc/21-multi-agent.md)。

## 什么时候用 / 不用会怎样

| 触发场景 | 用什么 | 不用会怎样 |
|---------|--------|-----------|
| 主 Agent 在对话中需要专业子 Agent（侦察、规划、审查、施工） | 本场景（subagent 扩展） | 把所有职责塞一个 system prompt，模型在多任务间漂移、上下文窗口被占满 |
| 子任务**必须隔离上下文**（避免主 Agent 历史污染） | 本场景（每个子 Agent 是独立进程） | 用同进程子 Agent（H06 模式 3b）会共享进程内存但隔离 messages，仍可接受 |
| 多个独立子任务要**并行**（批量翻译、多模块侦察） | 本场景 Parallel 模式 | 顺序执行总时长 = N × 单任务时长 |
| 顺序流水线（scout → planner → worker） | 本场景 Chain 模式 + `{previous}` 占位符 | 手工串接，每步都要复制粘贴上一步输出 |
| 你需要**完全控制**子 Agent 的 cwd / 消息历史 / 扩展 | [H06 模式 1 多 Session 并行](H06-multi-agent.md)（直接 `createAgentSession`） | subagent 扩展是「黑盒」子进程，不暴露 session API |
| 简单单轮任务 | **不需要本场景**，直接用 [A01](A01-minimal-startup.md) | 引入子 Agent 抽象 = 增加复杂度无收益 |
| 同进程内做 handoff / fork | [H06 模式 2/4/5](H06-multi-agent.md) | subagent 是**新建子进程**，不共享会话树 |

## 涉及组件

> [!NOTE]
> 下表的「参数位置」明确每项的真实归属——是 **tool call 参数**（LLM 调用 subagent 工具时传入）还是 **agent 配置文件字段**（`.md` frontmatter）。这是 I05 最容易混淆的点。

| 能力 | 归属 | 详细文档 |
|------|------|---------|
| `subagent` 工具注册 | 扩展（`pi.registerTool`，非 SDK 内置） | `examples/extensions/subagent/index.ts` |
| Single/Parallel/Chain 三种模式 | 扩展内部实现（通过 tool call 参数分支） | `index.ts` |
| `agent: string`（Single 模式） | **tool call 参数** | `index.ts`；`SubagentParams.agent` |
| `task: string`（Single 模式） | **tool call 参数** | `index.ts`；`SubagentParams.task` |
| `tasks: TaskItem[]`（Parallel 模式） | **tool call 参数** | `index.ts`；`SubagentParams.tasks` |
| `chain: ChainItem[]`（Chain 模式） | **tool call 参数** | `index.ts`；`SubagentParams.chain` |
| `agentScope: "user" \| "project" \| "both"` | **tool call 参数**（默认 `"user"`） | `index.ts`；`AgentScopeSchema` |
| `confirmProjectAgents: boolean` | **tool call 参数**（默认 `true`） | `index.ts` |
| `cwd: string`（每个 task 可选） | **tool call 参数**（task 级别 cwd 覆盖） | `index.ts` |
| Agent 配置文件 `.md` frontmatter | `~/.pi/agent/agents/` 或 `.pi/agents/` 下的 markdown 文件 | `agents.ts` |
| `--mode json -p --no-session` 子进程命令 | 扩展 spawn pi CLI | `index.ts`；`args.ts` |
| `--append-system-prompt` / `--tools` / `--model` 子进程参数 | 扩展 spawn pi CLI | `index.ts`；`args.ts` |

## 安装步骤

subagent 是示例扩展，**默认未安装**。从 pi 官方仓库安装（[`examples/extensions/subagent/README.md`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/README.md)）：

```bash
# 假设 pi 仓库克隆到 ~/code/pi
cd ~/code/pi

# 1. Symlink 扩展（必须放在子目录里，入口是 index.ts）
mkdir -p ~/.pi/agent/extensions/subagent
ln -sf "$(pwd)/packages/coding-agent/examples/extensions/subagent/index.ts" \
       ~/.pi/agent/extensions/subagent/index.ts
ln -sf "$(pwd)/packages/coding-agent/examples/extensions/subagent/agents.ts" \
       ~/.pi/agent/extensions/subagent/agents.ts
# ⚠️ index.ts 通过 `import { discoverAgents } from "./agents.ts"` 依赖 agents.ts，
#    两者必须同时安装，否则扩展加载时找不到 discoverAgents 会报错。

# 2. Symlink 自带的 4 个示例 Agent
mkdir -p ~/.pi/agent/agents
for f in packages/coding-agent/examples/extensions/subagent/agents/*.md; do
  ln -sf "$(pwd)/$f" ~/.pi/agent/agents/$(basename "$f")
done

# 3. Symlink Workflow prompt 模板（可选，用于 /implement 等命令）
mkdir -p ~/.pi/agent/prompts
for f in packages/coding-agent/examples/extensions/subagent/prompts/*.md; do
  ln -sf "$(pwd)/$f" ~/.pi/agent/prompts/$(basename "$f")
done
```

> [!NOTE]
> **Windows 用户**：`ln -sf` 在原生 Windows 不工作。可以：①在 WSL 里跑上面的命令；②用 `mklink /D`（管理员权限）；③直接把整个 `subagent/` 目录复制到 `~/.pi/agent/extensions/subagent/`（升级时要手动同步）。

安装后启动 pi，工具列表里会出现 `subagent`，主 Agent 即可被 LLM 调用。

## 工作原理（端到端）

一次 subagent tool call 的完整生命周期（源码：[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）：

```
主 Agent loop 调 subagent 工具
         │
         ▼
扩展 execute(toolCallId, params, signal, onUpdate, ctx) 入口
         │
         ├─ ① discoverAgents(ctx.cwd, params.agentScope ?? "user")
         │     ↓ 读 ~/.pi/agent/agents/*.md（user scope）
         │     ↓ 读 .pi/agents/*.md（project / both scope，从 cwd 向上递归查找）
         │     ↓ parseFrontmatter 解析 name/description/tools/model
         │     ↓ agentScope="both" 时项目同名覆盖用户
         │
         ├─ ② confirmProjectAgents 检查（仅 ctx.hasUI 时弹窗）
         │
         ├─ ③ 分派到三种模式之一：
         │   • single:  runSingleAgent(agent, task, cwd?)         一次
         │   • parallel: mapWithConcurrencyLimit(tasks, 4, runSingleAgent)  并发 4
         │   • chain:   for (step in chain) { runSingleAgent(...) }   顺序
         │
         ▼
runSingleAgent 内部（每调用一次 = 启动一个子进程）：
         │
         ├─ ④ 把 agent.systemPrompt 写到临时文件（mode 0o600）
         │
         ├─ ⑤ 拼接子进程命令：
         │     pi --mode json -p --no-session
         │        --model <agent.model>           # 可选
         │        --tools <agent.tools 列表>        # 可选
         │        --append-system-prompt <临时文件>  # 非空时
         │        "Task: <task>"
         │     ↓ getPiInvocation() 决定命令（三分支）：
         │       • argv[1] 是真实脚本 → node <argv[1]> <args>（最常见，不要求 PATH 有 pi）
         │       • runtime 非通用（编译二进制如 pi.exe）→ execPath <args>
         │       • 否则 fallback 到 `pi`（要求在 PATH）
         │
         ├─ ⑥ spawn(command, args, { cwd, stdio: ["ignore","pipe","pipe"] })
         │
         ├─ ⑦ 逐行读 stdout，JSON.parse 每行：
         │     • event.type === "message_end"     → 收集 Message + usage
         │     • event.type === "tool_result_end" → 收集工具调用结果
         │   每次 emitUpdate → 主 Agent UI 看到流式进度
         │
         ├─ ⑧ signal.aborted → SIGTERM 子进程，5s 后 SIGKILL
         │
         └─ ⑨ 子进程 exitCode → 填入 SingleResult，return
```

结果聚合回主 Agent：`content: [{ type: "text", text }]` + `details: SubagentDetails`（含 `mode` / `agentScope` / `projectAgentsDir` / `results[]`）。主 Agent 看到的是子 Agent 最后一条 assistant 消息的 text 部分（[`getFinalOutput`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）。

## Agent 配置文件格式

文件位置（[`agents.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/agents.ts)）：

| 位置 | scope | 何时加载 |
|------|-------|---------|
| `~/.pi/agent/agents/*.md` | user | `agentScope` 是 `"user"` 或 `"both"` 时 |
| `<cwd>/.pi/agents/*.md`（向上递归） | project | `agentScope` 是 `"project"` 或 `"both"` 时 |

> [!IMPORTANT]
> **`agentScope="both"` 时，项目级同名 agent **覆盖**用户级**（[`agents.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/agents.ts)）。这是安全相关的设计——项目可以定制/替换用户的全局 agent。

frontmatter 字段（[`agents.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/agents.ts)）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | ✅ 是 | agent 唯一标识 |
| `description` | string | ✅ 是 | 给 LLM 看的「这个 agent 适合做什么」 |

> [!NOTE]
> **`name` 和 `description` 是联合判断**：源码 `if (!frontmatter.name || !frontmatter.description) continue`（[`agents.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/agents.ts)）——**缺任一项整个 `.md` 文件就被静默跳过**（不报错）。两个字段都不为空才进入 agent 列表。
| `tools` | string（逗号分隔） | ❌ 否 | 工具白名单，如 `read, grep, find, ls, bash`。空或省略 = 用默认工具集 |
| `model` | string | ❌ 否 | 模型 ID 或 pattern，如 `claude-haiku-4-5`。省略 = 用启动 pi 时的模型 |

正文（frontmatter 之后的 markdown）是 **system prompt**，通过 `--append-system-prompt` 传给子进程（[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）。

**最小 agent 定义**（[`examples/extensions/subagent/agents/scout.md`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/agents/scout.md)）：

```markdown
---
name: scout
description: Fast codebase recon that returns compressed context for handoff to other agents
tools: read, grep, find, ls, bash
model: claude-haiku-4-5
---

You are a scout. Quickly investigate a codebase and return structured findings...

Output format:

## Files Retrieved
List with exact line ranges:
1. `path/to/file.ts` (lines 10-50) - Description of what's here

## Key Code
Critical types, interfaces, or functions:

```typescript
interface Example {
  // actual code from the files
}
```

## Start Here
Which file to look at first and why.
```

## 三种执行模式

所有模式都通过 LLM 在主 Agent 对话中调用 `subagent` 工具触发。模式选择由 tool call 参数决定——**一次调用只能用一种模式**（[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)，`modeCount !== 1` 时返回错误并列出可用 agent）。

### Single 模式

最小调用——一个 agent 执行一个任务。

```jsonc
{
  "agent": "scout",
  "task": "Find all authentication-related code in src/",
  "cwd": "/optional/specific/cwd",     // 可选，默认用主 Agent 的 ctx.cwd
  "agentScope": "user",                 // 可选，默认 "user"
  "confirmProjectAgents": true          // 可选，默认 true（仅 hasUI 时弹窗）
}
```

参数（[`SubagentParams` index.ts](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）：

| 参数 | 类型 | 默认 | 含义 |
|------|------|------|------|
| `agent` | string | — | agent 名（对应配置文件的 `name` 字段） |
| `task` | string | — | 自包含的任务描述。**不会带主 session 历史**，必须完整 |
| `cwd` | string | `ctx.cwd` | 子进程工作目录。影响 `.pi/agents/` 查找和工具执行的相对路径 |
| `agentScope` | `"user"` / `"project"` / `"both"` | `"user"` | 加载哪些 agent 配置文件 |
| `confirmProjectAgents` | boolean | `true` | 仅 `agentScope` 含 `"project"` 且 `ctx.hasUI` 时生效 |

**失败处理**（[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）：子进程 exitCode !== 0、或 stopReason 是 `"error"` / `"aborted"` 时，返回 `isError: true`，content 是 `Agent <stopReason>: <errorMsg>`。

### Parallel 模式

多个独立任务并行执行。上限 8 个任务、4 并发（[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）。

```jsonc
{
  "tasks": [
    { "agent": "scout", "task": "Find all API route definitions" },
    { "agent": "scout", "task": "Find all database model definitions" },
    { "agent": "reviewer", "task": "Review src/auth/* for issues" }
  ],
  "agentScope": "user"
}
```

**为什么有上限**：每个子任务是独立 `pi` 子进程——8 个任务在最坏情况下会同时驻留 8 个 Node/Bun 进程，内存和 API quota 都会爆。并发 4 是平衡吞吐和资源占用的经验值。

**输出聚合**（[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）：

```
Parallel: 2/3 succeeded

### [scout] completed

<最后一条 assistant text，最多 50KB>

### [scout] completed

<...>

### [reviewer] failed (error)

<错误信息或 stderr>

---

（details 里保留每个子进程的完整 messages 和 usage）
```

> [!IMPORTANT]
> **50KB 截断是「主 Agent 可见」的上限**——`truncateParallelOutput`（[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）会把超出部分截断，但**完整结果仍保留在 tool details** 中，用户可以在 pi CLI 里按 `Ctrl+O` 展开查看。这个设计平衡了「主 Agent 上下文窗口压力」和「信息完整性」。

### Chain 模式

顺序执行多个步骤，`{previous}` 占位符引用上一步输出（[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）。

```jsonc
{
  "chain": [
    { "agent": "scout",   "task": "Find the authentication module and its tests" },
    { "agent": "planner", "task": "Based on these findings:\n{previous}\n\nSuggest a refactoring plan" },
    { "agent": "worker",  "task": "Implement step 1 of this plan:\n{previous}" }
  ]
}
```

**`{previous}` 替换规则**（[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）：每步开始前，把 task 字符串中所有 `{previous}` 替换为上一步 assistant 消息的 text 输出。**第一步没有 previous**——如果 task 里写了 `{previous}`，会被替换成空字符串。

**失败处理**（[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）：任一步骤失败（exitCode !== 0 或 stopReason 是 error/aborted）立即停止 chain，返回 `Chain stopped at step N (agent): <errorMsg>` + `isError: true`。**已完成的步骤结果仍保留在 details**。

## 子进程命令的真实参数

subagent 工具内部 spawn 的完整命令（[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）：

```bash
pi --mode json -p --no-session \
   --model <agent.model>           # 仅 frontmatter 配置了 model 时
   --tools <agent.tools 逗号拼接>   # 仅 frontmatter 配置了 tools 时
   --append-system-prompt <临时文件路径>  # 仅 systemPrompt 非空时
   "Task: <task 字符串>"
```

参数含义（[`cli/args.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/src/cli/args.ts)）：

| Flag | 含义 | 源码 |
|------|------|------|
| `--mode json` | 输出模式 = JSON（每行一个 JSON 事件）。**启动时会先输出一个 session header 行**（`sessionManager.getHeader()`，含会话元信息），随后才是事件流——扩展 `processLine` 按 `event.type` 字段过滤，无匹配 `type` 的非事件行（含 header）被静默忽略 | `args.ts` |
| `-p` / `--print` | 非交互模式：处理完 prompt 立即退出 | `args.ts` |
| `--no-session` | 不保存会话到磁盘（ephemeral） | `args.ts` |
| `--model` | 模型 ID 或 pattern（如 `sonnet:high`） | `args.ts` |
| `--tools` / `-t` | 工具白名单（逗号分隔） | `args.ts` |
| `--append-system-prompt` | 追加到默认 system prompt（可多次） | `args.ts` |

**命令解析逻辑**（[`getPiInvocation` index.ts](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）：按顺序三分支判断（命中即返回）。

| 分支 | 触发条件 | 实际启动命令 | 场景 |
|------|---------|-------------|------|
| (a) | `process.argv[1]` 是**真实存在的脚本文件**（非 bun 虚拟路径 `/$bunfs/...`） | `process.execPath <argv[1]> <args>`（即 `node main.ts <args>`） | **最常见**：`node main.ts`、`bun main.ts`、`npx tsx main.ts` 等直接跑源码 |
| (b) | runtime **不是** node/bun（`execName` 不匹配 `/^(node\|bun)(\.exe)?$/`） | `process.execPath <args>`（直接调编译二进制） | pi 已编译成单文件二进制（如 `pi.exe`、`pi` 无后缀可执行） |
| (c) | 上述都不满足（通用 runtime + 无真实脚本路径） | `pi <args>` | bun 打包模式（`argv[1]` 是虚拟路径）、部分容器场景 |

> [!IMPORTANT]
> **分支 (a) 是开发环境的常态**——用 `node main.ts`（或 `tsx`/`bun`）直接跑 pi 时，`process.argv[1]` 就是 `main.ts` 的真实路径，`fs.existsSync` 返回 true，走分支 (a) 用 `node main.ts <args>` 启动子进程。**这种情况不要求 PATH 里有 `pi`，也不会失败**。
>
> 只有走到**分支 (c)** 才需要 PATH 里有 `pi`（即 pi 被 `npm install -g` 或以编译二进制形式安装）——典型场景是 bun 打包后 `argv[1]` 为虚拟路径 `/$bunfs/root/...`、且 `process.execPath` 是 `bun`（通用 runtime），两条件同时成立时才 fallback。日常 `node main.ts` 调试不会触发此分支。

## 示例 Agent 一览

pi 自带 4 个示例 agent（[`examples/extensions/subagent/agents/`](https://github.com/EarendilElon/pi/tree/main/packages/coding-agent/examples/extensions/subagent/agents)）：

| name | 职责 | model | tools |
|------|------|-------|-------|
| `scout` | 代码库侦察——快速定位文件、抽取关键代码片段、压缩成结构化报告给下游 agent | `claude-haiku-4-5` | read, grep, find, ls, bash |
| `planner` | 实现规划——基于上游 scout 的发现，给出带优先级和依赖关系的步骤计划 | `claude-sonnet-4-5` | read, grep, find, ls |
| `reviewer` | 代码审查——检查 bug、风格、性能问题 | `claude-sonnet-4-5` | read, grep, find, ls, bash |
| `worker` | 通用施工——不限制工具，执行实际修改 | `claude-sonnet-4-5` | （不配置 → 子进程用启动 pi 的默认工具集） |

**设计模式**：scout 用便宜模型（Haiku）做高频侦察，planner/reviewer/worker 用强模型（Sonnet）做需要推理的工作。这是 subagent 模式的核心价值——**按任务类型选模型，避免用一个大模型干所有事**。

## Workflow Prompt 模板（B 档）

pi 自带 3 个 workflow prompt（[`examples/extensions/subagent/prompts/`](https://github.com/EarendilElon/pi/tree/main/packages/coding-agent/examples/extensions/subagent/prompts)），安装到 `~/.pi/agent/prompts/` 后可在 pi CLI 用 `/implement <query>` 等命令触发：

| 命令 | 流程 |
|------|------|
| `/implement <query>` | scout → planner → worker（完整实现流水线） |
| `/scout-and-plan <query>` | scout → planner（侦察 + 规划，不施工） |
| `/implement-and-review <query>` | worker → reviewer → worker（施工 + 审查 + 修复） |

模板内部就是让主 Agent 调用 `subagent` 工具的 chain 模式。你可以照着写自己的 workflow prompt，放进 `~/.pi/agent/prompts/`。

## 安全模型

> [!CAUTION]
> **项目级 agent（`.pi/agents/*.md`）是 repo 控制的 prompt**——可以指示模型读文件、执行 bash 命令、调用任何工具。**只对受信任的仓库启用** `agentScope: "project"` 或 `"both"`。

默认行为（[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）：

| `agentScope` | 加载 `~/.pi/agent/agents/` | 加载 `<cwd>/.pi/agents/` |
|--------------|---------------------------|-------------------------|
| `"user"`（默认） | ✅ | ❌ |
| `"project"` | ❌ | ✅ |
| `"both"` | ✅ | ✅（同名覆盖 user） |

**确认机制**（[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）：

- 仅当 `agentScope` 含 `"project"` **且** `confirmProjectAgents === true` **且** `ctx.hasUI === true` 时，会弹窗列出要调用的项目级 agent 名 + 来源目录，让用户确认。
- 非交互模式（自动化测试 / RPC server）`ctx.hasUI === false`，**确认机制完全失效**——直接执行。
- `confirmProjectAgents: false` 显式关闭确认（不推荐对不信任的 repo 使用）。

> [!IMPORTANT]
> **agentScope 和 confirmProjectAgents 是 tool call 参数**，意味着 **LLM 可以在调用时尝试不同的值**。如果 LLM 决定 `agentScope: "project"` 且 `confirmProjectAgents: false`，扩展在非交互模式下会**直接加载并执行项目 agent**，没有任何阻挡。在受信任的 repo 里这没问题，但在恶意 repo 里等于让模型执行任意 prompt。**不要在不信任的 repo 里启用 subagent 扩展**。

## 完整 tool call 参数清单

`SubagentParams`（[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）：

| 参数 | 类型 | 默认 | 用于模式 | 说明 |
|------|------|------|---------|------|
| `agent` | string | — | single | agent 名 |
| `task` | string | — | single | 任务描述 |
| `tasks` | `TaskItem[]` | — | parallel | 并行任务数组 |
| `chain` | `ChainItem[]` | — | chain | 顺序步骤数组 |
| `agentScope` | `"user"` / `"project"` / `"both"` | `"user"` | 全部 | 加载范围 |
| `confirmProjectAgents` | boolean | `true` | 全部 | 项目级 agent 确认（仅 hasUI） |
| `cwd` | string | `ctx.cwd` | single | 子进程工作目录 |

`TaskItem` / `ChainItem`（[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `agent` | string | ✅ | agent 名 |
| `task` | string | ✅ | 任务描述（chain 中可含 `{previous}`） |
| `cwd` | string | ❌ | 该任务的 cwd 覆盖 |

## 子进程输出与展示

每个子进程通过 stdout 输出 JSONL（每行一个事件），扩展解析后回传给主 Agent（[`index.ts`](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)）：

| 事件类型 | 含义 | 扩展处理 |
|---------|------|---------|
| `message_end` | 一条消息（user/assistant/tool）结束 | 收集到 `currentResult.messages`；assistant 消息累加 usage |
| `tool_result_end` | 工具调用结果 | 收集到 `currentResult.messages` |

**主 Agent 最终看到的**（`content[0].text`）：

| 模式 | content |
|------|---------|
| single | 子 Agent 最后一条 assistant 消息的 text 部分 |
| parallel | 汇总：`N/M succeeded` + 每个任务的 markdown 摘要（≤50KB） |
| chain | **最后一步**的 assistant text（不是每步拼接） |

**完整结果保留在 `details`**（`SubagentDetails`）：`mode` / `agentScope` / `projectAgentsDir` / `results[]`（每项含 `messages`、`usage`、`exitCode`、`stderr`、`stopReason`、`errorMessage`）。pi CLI 用户可以按 `Ctrl+O` 展开看完整输出。

## 限制与配额

| 限制 | 值 | 源码 | 说明 |
|------|------|------|------|
| Parallel 任务上限 | 8 | `MAX_PARALLEL_TASKS` ([index.ts](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)) | 超出直接报错，不执行 |
| Parallel 并发上限 | 4 | `MAX_CONCURRENCY` ([index.ts](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)) | 多出来的任务排队等待 |
| 单任务主 Agent 可见输出 | 50KB | `PER_TASK_OUTPUT_CAP` ([index.ts](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts)) | **仅 parallel 模式生效**（`truncateParallelOutput` 仅在 parallel 分支调用）。single/chain 模式主 Agent 看到完整的最后 assistant text，不截断。超出截断 + 提示完整结果在 details |
| 上下文继承 | 不继承 | 设计决定 | 子进程只接收 `task` 字符串，不带主 session 历史 |
| 持久化 | `--no-session` | 子进程参数 | 子 Agent 不写磁盘 |
| Agent 发现时机 | 每次调用 | ([agents.ts](https://github.com/EarendilElon/pi/blob/main/packages/coding-agent/examples/extensions/subagent/agents.ts)) | 允许运行中编辑 agent 配置文件，下次调用生效 |

## 横向：与 H06 多 Agent 的边界

| 维度 | 本场景（I05 subagent 扩展） | [H06 模式 3b](H06-multi-agent.md)（扩展内 createAgentSession） |
|------|----------------------------|---------------------------------------------------------|
| 子 Agent 进程 | 独立子进程（`spawn pi`） | 同进程（`createAgentSession`） |
| 上下文隔离 | 完全隔离（独立进程内存） | 隔离 messages 但共享 agentDir（auth/models） |
| 配置方式 | `.md` 配置文件（声明式） | 代码（命令式） |
| 工具选择 | frontmatter `tools` 字段 | `createAgentSession({ tools: [...] })` |
| 模型选择 | frontmatter `model` 字段 | `createAgentSession({ model })` |
| 流式输出 | JSONL stdout 解析 | `session.subscribe` 直接回调 |
| 资源开销 | 高（每子任务一个进程） | 低（共享进程） |
| 错误处理 | exitCode + stderr | try/catch + dispose |
| 扩展性 | 加 `.md` 文件即可 | 改扩展代码 |
| 适合 | 声明式、强隔离、并行/链式流水线 | 需要精细控制子 Agent 行为 |

**H06 模式 3a**（[`H06-multi-agent.md`](H06-multi-agent.md)）是本场景的简介版——H06 只讲它是什么 + 指向 I05，**本场景是 subagent 扩展的权威完整文档**。两处不重复维护：H06 改架构性结论，I05 改具体配置和工具细节。

## 常见误期待与陷阱

1. **「subagent 是 SDK 内置工具」** → **错**。它是 `examples/extensions/subagent/` 下的**示例扩展**，需要手动 symlink 安装。`createAgentSession({ tools: ["subagent"] })` 不会生效。
2. **「在 settings.json 配 `agentScope` / `confirmProjectAgents`」** → **错**。`Settings` interface（`settings-manager.ts`）**没有这两个字段**。它们是 `subagent` 工具的 **tool call 参数**，由 LLM 在调用工具时传入。
3. **「装了 subagent 扩展就能用」** → **看情况**。还要在 `~/.pi/agent/agents/` 放至少一个 `.md` 配置文件。否则工具调用会返回 `Unknown agent: "xxx". Available agents: none.`
4. **「agent 配置文件必须 4 个字段都写」** → **错**。`name` / `description` 必填，`tools` / `model` 可省略（用默认）。但**正文不能为空**——`systemPrompt.trim()` 为空时不传 `--append-system-prompt`，子 Agent 没有专属指令。
5. **「项目 agent 在 `.pi/agents/` 下就会加载」** → **错**。默认 `agentScope="user"`，**完全不读项目目录**。必须显式传 `agentScope: "project"` 或 `"both"` 才会加载。
6. **「`agentScope="both"` 会合并 user + project 所有 agent」** → **对，但同名覆盖**——项目同名 agent 会**完全替换**用户 agent，不是合并字段。
7. **「`confirmProjectAgents` 总会弹窗」** → **错**。三个条件**同时满足**才弹窗：①`agentScope` 含 `"project"`；②`confirmProjectAgents === true`；③`ctx.hasUI === true`。**非交互模式（自动化 / RPC）`hasUI === false`，确认完全失效**。
8. **「LLM 不能绕过 `confirmProjectAgents`」** → **错**。LLM 可以在 tool call 里传 `confirmProjectAgents: false`——扩展会直接执行项目 agent。**这是已知的安全特性**，依赖「不要在不信任的 repo 启用 subagent」。
9. **「parallel 模式的 8/4 上限是软限制」** → **错**。8 是硬上限——`tasks.length > 8` 直接返回错误（`index.ts`）。4 是并发上限，多出来的任务排队。
10. **「50KB 截断丢数据」** → **错**。截断只影响「主 Agent 可见的 content」；**完整结果保留在 tool `details`** 中，CLI 用户按 `Ctrl+O` 可看。
11. **「chain 模式失败会自动重试」** → **错**。任一步骤失败立即停止整个 chain，已完成的步骤保留在 details。要重试得 LLM 重新发起 tool call。
12. **「chain 第一步可以用 `{previous}`」** → **技术上可以但无意义**——第一步的 `{previous}` 被替换成空字符串。
13. **「subagent 共享主 Agent 的 cwd」** → **看参数**。默认 `cwd = ctx.cwd`（主 Agent 的 cwd）；但 single/task/chain 都支持 `cwd` 字段覆盖。
14. **「subagent 是唯一的多 Agent 方案」** → **错**。还有 [H06 模式 1 多 Session 并行](H06-multi-agent.md)（直接 `createAgentSession` + `Promise.all`）、[H06 模式 3b 同进程子 Agent](H06-multi-agent.md)（扩展内 `createAgentSession`）。subagent 扩展适合「声明式配置 + 独立进程隔离」的场景。
15. **「subagent 用 `pi` 全局命令」** → **看运行方式，但大多数情况不用**。`getPiInvocation` 三分支：①`argv[1]` 是真实脚本（`node main.ts` 最常见）→ `node main.ts <args>`，**不要求 PATH 有 pi**；②runtime 非通用（编译二进制如 `pi.exe`）→ 直接调二进制；③通用 runtime + 虚拟脚本（bun 打包）→ 才 fallback 到 `pi`（要求 PATH）。日常 `node main.ts` 开发走分支 ①，不会因 PATH 缺 pi 而失败。

## 变体与延伸

| 变体 | 怎么改 | 参考 |
|------|-------|------|
| 同进程子 Agent（不要独立进程开销） | 在扩展 `pi.registerTool` 的 execute 里直接 `createAgentSession` | [H06 模式 3b](H06-multi-agent.md) |
| 多 Agent 完整协作（handoff / fork / runtime 切换） | 用 `AgentSessionRuntime` + `pi.registerCommand` | [H06](H06-multi-agent.md) |
| 多 Agent 完整架构文档（5 种模式 + 通信方式） | 官方文档 | [sdk_doc/21-multi-agent.md](../sdk_doc/21-multi-agent.md) |
| 自定义工具定义（`defineTool`） | 内置工具白名单 / 工具参数 schema | [sdk_doc/06-tools.md](../sdk_doc/06-tools.md) |
| 扩展注册机制（`pi.registerTool`） | ExtensionAPI 完整字段 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| 项目级资源信任机制 | `defaultProjectTrust` settings | [B02](B02-settings.md) |
