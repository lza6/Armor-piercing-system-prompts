# 06 - 工具系统 (Tools)

`@earendil-works/pi-coding-agent` 导出完整工具系统，支持使用内置工具（read / bash / edit / write / grep / find / ls）和注册自定义工具。

**什么时候需要自定义工具**：内置工具覆盖的是通用 coding 操作（读写文件、跑命令）。当你要让 agent 调用你的业务能力——查数据库、调内部服务、读特定索引、操作私有 API——时，就需要 `customTools`。不用的话 agent 只能通过 bash 绕路（如 `curl` 内部接口、写临时脚本），既慢又容易出错，还把敏感凭据暴露进 shell 历史；而一个封装好的 customTool 可以在 `execute` 内部安全地取凭据、做参数校验、返回结构化结果。

## 内置工具一览

| 工具名 | 功能 | 所属类别 |
|--------|------|----------|
| `read` | 读取文件内容 | coding / readonly |
| `bash` | 执行 shell 命令 | coding |
| `edit` | 精确字符串替换编辑 | coding |
| `write` | 写入/覆盖文件 | coding |
| `grep` | 基于 ripgrep 的内容搜索 | readonly |
| `find` | 文件 glob 匹配搜索 | readonly |
| `ls` | 列出目录内容 | readonly |

## 工厂函数

这些是 SDK 导出的工具工厂函数，可接受自定义 cwd 和选项：

```ts
import {
  createReadTool, createBashTool, createEditTool, createWriteTool,
  createGrepTool, createFindTool, createLsTool,
  createCodingTools,      // [read, bash, edit, write]
  createReadOnlyTools,    // [read, grep, find, ls]
} from "@earendil-works/pi-coding-agent";
```

所有单工具工厂共享模式：`createXxxTool(cwd: string, options?: XxxToolOptions)`。

除了单工具工厂和已列出的 `createCodingTools` / `createReadOnlyTools`，SDK 还导出以下批量工厂（`core/tools/index.ts`）：

| 函数 | 返回类型 | 说明 |
|------|---------|------|
| `createCodingToolDefinitions(cwd, options?)` | `ToolDefinition[]` | coding 类定义数组（read, bash, edit, write），返回原始定义而非 AgentTool 实例 |
| `createReadOnlyToolDefinitions(cwd, options?)` | `ToolDefinition[]` | readonly 类定义数组（read, grep, find, ls） |
| `createAllToolDefinitions(cwd, options?)` | `Record<ToolName, ToolDefinition>` | 全部 7 个工具的原始定义，以 Record 形式返回 |
| `createAllTools(cwd, options?)` | `Record<ToolName, Tool>` | 全部 7 个工具的 AgentTool 实例，以 Record 形式返回 |

`createXxxToolDefinitions` 系列适用于需要拿到 `ToolDefinition` 对象（如传入 `customTools` 或进一步包装）的场景；`createXxxTools` 系列返回的是已实例化的 `AgentTool` 对象。

## 在会话中注册自定义工具

> ⚠️ **typebox 需手动安装**：下方示例中的 `Type`（用于描述工具参数 schema）来自 [`typebox`](https://www.npmjs.com/package/typebox) 包。它是 `pi-coding-agent` 的**直接运行时依赖**（`dependencies` 中，版本 1.3.7），但 npm/pnpm 不会把它提升（hoist）到你项目顶层 `node_modules`，直接 `import { Type } from "typebox"` 会报 `Cannot find package 'typebox'`。需在自己的项目里显式安装（版本与 SDK 内置一致即可，如 `^1.3.x`）：
>
> ```bash
> npm install typebox
> ```

有两种方式注册自定义工具：

### 方式一：通过 `createAgentSession()` 的 `customTools` 参数

```ts
import { createAgentSession, defineTool } from "@earendil-works/pi-coding-agent";
import { Type } from "typebox";

const myTool = defineTool({
  name: "my_tool",
  label: "My Tool",
  description: "Does something useful",
  parameters: Type.Object({
    input: Type.String({ description: "The input string" }),
  }),
  async execute(toolCallId, params, signal, onUpdate, ctx) {
    return {
      content: [{ type: "text", text: `Processed: ${params.input}` }],
      details: { result: params.input.toUpperCase() },
    };
  },
});

const { session } = await createAgentSession({
  tools: ["read", "bash", "my_tool"],  // 显式声明允许的工具（含自定义）
  customTools: [myTool],
});
```

### 方式二：在 Extension 内通过 `pi.registerTool()` 注册

```ts
const myExtension: ExtensionFactory = (pi) => {
  pi.registerTool(defineTool({
    name: "custom_search",
    label: "Custom Search",
    description: "Search custom index",
    parameters: Type.Object({ query: Type.String() }),
    async execute(toolCallId, params, signal, onUpdate, ctx) {
      // ...
    },
  }));
};
```

### 两种方式怎么选

| 维度 | 方式一 `customTools` | 方式二 `pi.registerTool` |
|------|---------------------|-------------------------|
| 归属层级 | SDK 级（`createAgentSession` 参数） | 扩展级（走 extension runner） |
| 生命周期 | 随 session 创建固定 | 随扩展 reload 重建 |
| `sourceInfo` | `<sdk:工具名>` | 扩展路径 |
| 分发复用 | 适合一次性嵌入 | 适合随扩展分发给多个项目 |
| 受 `tools`/`excludeTools`/`noTools` 过滤 | 是 | 是（二者一致） |

**决策提示**：纯 SDK 嵌入、工具只服务当前会话 → 用 `customTools`；要让工具随扩展分发出可复用的能力包、参与扩展热重载 → 用 `registerTool`。

### customTools 与 `tools` 白名单的两种用法（★ 易踩坑）

`customTools` 是否要进 `tools` 白名单，取决于你是否同时传了 `tools`：

**(a) 不传 `tools`（默认全开）——customTools 自动激活**

```ts
// 不传 tools：默认 read/bash/edit/write 全部 active，
// customTools 通过 includeAllExtensionTools 自动加入 active 列表，无需列白名单
const { session } = await createAgentSession({
  customTools: [myTool],
  // 没有 tools 字段
});
```

**(b) 传 `tools`（收紧工具集）——customTool 名必须进白名单**

```ts
// 传 tools：只有白名单内的工具激活；
// 此时 customTool 名必须显式列入，否则被 isAllowedTool 滤掉
const { session } = await createAgentSession({
  tools: ["read", "my_tool"],   // 想保留 bash/edit/write 也要一并列出
  customTools: [myTool],
});
```

> ⚠️ 常见误用：只想"加一个自定义工具"却画蛇添足地传了 `tools: ["my_tool"]`，结果把 read/bash/edit/write 全滤掉了。**纯加工具场景用 (a)，不要传 `tools`**。

## ToolDefinition 接口详解

```ts
interface ToolDefinition<TParams extends TSchema, TDetails = unknown, TState = any> {
  name: string;                    // 工具名，LLM 调用时使用
  label: string;                   // UI 中显示的人类可读标签
  description: string;             // 给 LLM 看的工具描述

  promptSnippet?: string;          // 出现在系统提示词 "可用工具" 区块的一行摘要
  promptGuidelines?: string[];     // 追加到系统提示词 "Guidelines" 区块的提示项

  parameters: TParams;             // TypeBox schema，定义工具参数
  constrainedSampling?: false | ConstrainedSamplingConfig;  // 工具约束采样（见下文详解）
  renderShell?: "default" | "self"; // 控制外壳渲染方式

  prepareArguments?: (args: unknown) => Static<TParams>;
  executionMode?: ToolExecutionMode; // "sequential" | "parallel"

  execute(
    toolCallId: string,
    params: Static<TParams>,
    signal: AbortSignal | undefined,     // 用于响应取消信号
    onUpdate: AgentToolUpdateCallback<TDetails> | undefined, // 流式更新回调
    ctx: ExtensionContext,              // 扩展上下文
  ): Promise<AgentToolResult<TDetails>>;

  renderCall?: (args: Static<TParams>, theme: Theme, context: ToolRenderContext<TState, Static<TParams>>) => Component;
  renderResult?: (result: AgentToolResult<TDetails>, options: ToolRenderResultOptions, theme: Theme, context: ToolRenderContext<TState, Static<TParams>>) => Component;
}
```

> ℹ️ **`renderCall` / `renderResult` 是 interactive 模式（TUI）专属**：它们返回 `Component`（终端 UI 组件），只在 TUI 渲染工具调用/结果时被调用。**SDK 嵌入场景（RPC/print/json 模式）不渲染 Component，这两个字段可完全省略**。SDK 用户只需实现 `name` / `label` / `description` / `parameters` / `execute` 即可。

### `defineTool()` 包装函数

```ts
export function defineTool<TParams extends TSchema, TDetails = unknown, TState = any>(
  tool: ToolDefinition<TParams, TDetails, TState>,
): ToolDefinition<TParams, TDetails, TState> & AnyToolDefinition;
```

**作用**：保持 `parameters` 的 TypeBox 类型推断不被 `customTools` 数组类型收窄为 `unknown`。当把工具定义赋值给变量或在数组中传递时，强烈推荐使用。

## AgentToolResult 结构

`execute()` 必须返回该类型：

```ts
interface AgentToolResult<TDetails> {
  content: (TextContent | ImageContent)[];  // 返回给 LLM 的内容
  details: TDetails;                         // 扩展专用元数据（不进 LLM 上下文）
  usage?: Usage;                             // 工具自身的 token 使用量（工具内部调 LLM 时回传）。注意：它**纳入 cost/token 展示总计**（归入 "Tools/summaries" 桶），但**不参与 context 窗口核算**——context 只算 assistant 消息的 provider usage，tool usage 不会喂回去膨胀上下文
  addedToolNames?: string[];                 // transcript-level 动态工具注册（动态工具发现的替代路径）
  terminate?: boolean;                       // 设为 true 可在当前工具批次完成后提前终止 agent loop
}
```

### constrainedSampling（工具约束采样）

**解决什么问题**：LLM 调用工具时，参数可能是任意自由文本——但你希望某些字段严格符合结构（如完整 JSON、正则约束、Lark 语法）。`constrainedSampling` 字段让 provider 在采样阶段就强制约束，提升工具参数生成质量（尤其结构化数据 / JSON 输出场景）。

**两种约束类型**（`ConstrainedSamplingConfig`，见 `packages/ai/src/types.ts`）：

```ts
type ConstrainedSamplingConfig =
  | { type: "json_schema"; strict: "prefer" | "require" }   // JSON Schema 约束（"require" = 必须支持，"prefer" = 优先但允许降级）
  | { type: "grammar"; variants: Partial<Record<GrammarFormat, string>> }  // 语法约束（GrammarFormat = "openai_lark" | "openai_regex"）
```

**用法示例**：

```ts
const strictJsonTool = defineTool({
  name: "structured_query",
  // ...
  parameters: Type.Object({ /* ... */ }),
  // 强制 LLM 按工具的 TypeBox schema 严格生成参数
  constrainedSampling: { type: "json_schema", strict: "require" },
  async execute(...) { /* ... */ },
});

const grammarTool = defineTool({
  name: "regex_match",
  // ...
  // 用 OpenAI Lark 语法 + 备用 regex 两种格式约束
  constrainedSampling: {
    type: "grammar",
    variants: {
      openai_lark: 'start: "hello" " " NAME',
      openai_regex: '^hello [A-Z]+$',
    },
  },
  async execute(...) { /* ... */ },
});

const unconstrainedTool = defineTool({
  name: "free_form",
  // ...
  constrainedSampling: false,  // 显式禁用约束采样（等价于 undefined）
  async execute(...) { /* ... */ },
});
```

**注意**：provider 能力元数据会自动检测是否支持，不支持的 provider 会忽略约束并按普通采样生成（`strict: "require"` 的工具在不支持 provider 上可能降级）。

- `content` 中的文字会参与后续的 LLM 对话上下文
- `details` 是工具返回的结构化数据，用于 UI 渲染或扩展内部逻辑
- `terminate` 仅当同一批次所有工具都返回 `terminate: true` 时生效，agent 不再继续下一轮 LLM 调用

## execute() 各参数说明

| 参数 | 类型 | 说明 |
|------|------|------|
| `toolCallId` | `string` | 本次工具调用的唯一 ID |
| `params` | `Static<TParams>` | 经 TypeBox schema 校验后的参数 |
| `signal` | `AbortSignal \| undefined` | 取消信号，用于支持中断长时间运行的工具 |
| `onUpdate` | `AgentToolUpdateCallback \| undefined` | 流式更新回调，可在执行中途推送进度 |
| `ctx` | `ExtensionContext` | 工具运行时的会话上下文，常用：`ctx.cwd`（项目根）、`ctx.model`（当前模型）、`ctx.modelRegistry`、`ctx.mode`、`ctx.sessionManager`、`ctx.ui`（UI 交互，仅 interactive/RPC 模式可用，可用 `ctx.hasUI` 判断）、`ctx.signal`（abort 信号，用于响应中断）、`ctx.abort()`（自行中断）、`ctx.compact()`（触发压缩）、`ctx.getSystemPrompt()`、`ctx.scopedModels` 等。⚠️ 注意：`getFlag()` **不在** `ctx` 上——它在扩展 factory 的 `pi`（`ExtensionAPI`）对象上，即 `pi.getFlag("x")`，在 `execute()` 的 `ctx` 里取不到 |

## 关键注意事项

1. **工具名称唯一性（静默 first-wins）**：当多个扩展注册同名工具时，**先注册的扩展胜出，后者被静默忽略**——没有任何 diagnostic 提示。diagnostic 体系（`ResourceDiagnostic`）只覆盖 shortcut 和 command，不覆盖 tool。因此想用"晚加载的扩展覆盖前者"不可靠；需要覆盖时改用 `customTools`（数组中后写的工具在 registry 中后注册，配合白名单可精确控制），或用 `pi.setActiveTools()` 动态切换。
2. **`tools` 白名单**：`createAgentSession({ tools: [...] })` 中的工具名必须包含自定义工具名，否则不会启用
3. **`noTools` / `excludeTools`**：`noTools: "builtin"` **默认不激活**内置工具（read/bash/edit/write 仍在 registry，未被删除——后续可通过 `setActiveTools` 重新激活），但扩展工具不受影响；`noTools: "all"` 则连扩展工具的默认激活也一并跳过（`initialActiveToolNames` 为空）。`excludeTools: ["bash"]` 按名排除单个工具。配合 `tools` 白名单灵活控制工具集
4. **`prepareArguments`**：在 schema 校验之前运行，用于兼容性转换或参数清洗
5. **`signal` 参数**：实现工具时务必检查 `signal.aborted`，在长时间操作中及时终止
6. **`renderShell: "self"`**：告诉 UI 框架不要包裹默认的外壳，由工具自己渲染边框
7. **`withFileMutationQueue`**：SDK 导出的高阶函数（`core/tools/index.ts`），用于给文件操作加串行队列。同一文件的写操作排队执行，不同文件的仍可并行。内置 `edit` / `write` 工具已内置使用；自定义文件操作工具如有并发写入冲突，可手动包裹 `withFileMutationQueue(filePath, async () => { ... })`。签名：`withFileMutationQueue<T>(filePath: string, fn: () => Promise<T>): Promise<T>`

> 其他内置工具（bash / edit / write / grep / find / ls）各自的 Options 接口（`BashToolOptions` / `EditToolOptions` 等）详见源码：
> `core/tools/{bash,edit,write,grep,find,ls}.ts`

## 内置工具的安全边界（实测）

### 内置 read tool 不支持路径白名单

**现象**：用 `createReadTool(cwd)` 创建的内置 read tool，`cwd` 只决定**相对路径的解析基目录**，不限制可读范围。模型理论上可以读项目任意文件。

**后果**：处理含 `.env`（API Key）、源码、密钥等敏感文件的项目时，存在提示词注入风险——恶意用户输入可能诱导模型 `read .env`，把密钥泄露到对话上下文。

**`ReadToolOptions` 实际字段**（确认于 pi-coding-agent 类型定义）：

```ts
interface ReadToolOptions {
  autoResizeImages?: boolean;   // 图片自动缩放（默认 true，最大 2000x2000）
  operations?: ReadOperations;  // 自定义文件读取实现（用于 SSH 等远程场景）
}
```

**没有** `allowedPaths` / `sandbox` / `rootDir` 之类的路径限制字段。其他内置工具（bash / edit / write）的 options 同样不含路径白名单——内置工具的定位是"通用 coding 工具"，安全边界需使用者自行加。

> 💡 **bash 工具的命令拦截点**：虽然没有路径白名单，`BashToolOptions` 提供了 `spawnHook?: BashSpawnHook`（签名 `(context: BashSpawnContext) => BashSpawnContext`），在命令真正 spawn 之前被调用，可改写或拒绝命令——这是 bash 工具内置的安全/审计挂钩点。详见 `core/tools/bash.ts` 的 `BashSpawnHook` 类型。

**对策**：用 `createReadToolDefinition()` 拿到内置定义，包装一层路径校验。保留内置的 offset / limit / 截断 / 图片处理能力，只在执行前做路径检查：

```ts
import {
  createReadToolDefinition,
  defineTool,
} from "@earendil-works/pi-coding-agent";
import * as path from "node:path";

const ALLOWED_DIR = path.resolve(process.cwd(), ".pi", "skills");
const baseRead = createReadToolDefinition(process.cwd());

export const readSkillTool = defineTool({
  ...baseRead,
  name: "read",
  description:
    baseRead.description +
    "。安全限制：仅允许读取 .pi/skills/ 目录内文件。",
  async execute(toolCallId, params, signal, onUpdate, ctx) {
    const requested = path.resolve(process.cwd(), params.path);
    // 必须严格位于 ALLOWED_DIR 内（防 ../ 逃逸、防 .pi\skills-evil\ 前缀欺骗）
    const allowed =
      requested === ALLOWED_DIR ||
      requested.startsWith(ALLOWED_DIR + path.sep);
    if (!allowed) {
      return {
        content: [
          {
            type: "text" as const,
            text: `权限拒绝：read 工具只能读取 ${ALLOWED_DIR} 内的文件。`,
          },
        ],
      };
    }
    return baseRead.execute(toolCallId, params, signal, onUpdate, ctx);
  },
});
```

**典型应用场景**：当 read tool 主要用于"让模型按需读 skill 正文"时（见 [09-skills.md](09-skills.md) 的「Skill 正文不自动注入」踩坑节），把范围限制在 `.pi/skills/` 既满足 pi-agent 渐进式披露的设计意图，又彻底杜绝敏感文件泄露。
