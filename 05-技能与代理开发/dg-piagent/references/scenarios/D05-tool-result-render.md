# 场景：自定义工具输出渲染 (D05)

## 什么时候用 / 不用会怎样

**适合**：

- **错误信息友好化**：工具执行失败时原始堆栈/英文报文直接返给 LLM，模型可能基于错误信息继续无效重试。先在 `tool_result` 阶段把错误改写成可操作的提示（如「未找到 ID=xxx 的用户，请用 lookup_user 工具核查」）
- **敏感信息脱敏**：bash / read 工具的输出可能包含 API key、token、密码——必须在送给 LLM 前替换或遮蔽
- **输出长度控制**：read 大文件、grep 命中过多时截断到合理长度，避免污染上下文窗口
- **格式标准化**：所有工具结果统一转 Markdown / 纯文本 / JSON，让下游消费者（UI、日志、LLM 推理）处理逻辑统一
- **TUI 终端美化（CLI 专属）**：自定义工具在交互式终端里用 `renderCall` / `renderResult` 渲染表格、高亮、折叠组件

**不用会怎样**：

- 工具输出的原始字节流直接进 LLM 上下文——长输出挤占 token、错误堆栈混淆模型判断、敏感数据外泄
- 自定义工具在 TUI 里只有默认的纯文本展示，无法做表格 / 进度条 / 折叠树等可视化

**不适合本场景**：

- 在工具执行**之前**拦截或修改参数 → 见 [E01 拦截与修改工具调用](E01-tool-intercept.md)（用 `tool_call` 事件）
- 工具执行**过程中的流式更新**（如批量任务的进度）→ 在 `execute()` 内调 `onUpdate?.()` 回调，详见 [D01 变体 C](D01-custom-tool.md#变体-c长任务--signal--onupdate)
- 全局消息流的 token 级转换 → 见 [E06 流式处理工具输出](E06-streaming-transform.md)（`message_update` / `tool_execution_update`）
- 危险命令阻断（执行前阻止）→ 见 [D04 工具调用安全闸门](D04-confirm-destructive.md)

---

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `pi.on("tool_result")` 扩展事件 | 工具执行**完成后**、结果送 LLM **之前**的统一后处理钩子 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `defineTool({ renderCall, renderResult })` | 在工具定义时声明 TUI 渲染器（**仅 interactive 模式生效**） | [sdk_doc/06-tools.md](../sdk_doc/06-tools.md) |
| `ctx.ui.setStatus(key, text)` | 渲染时设置 TUI 状态栏（**CLI 专属，hasUI=false 时静默失败**） | 终端 UI API（CLI 专属，本 skill 未收录，查 SDK 源码） |

> ⚠️ **关键集成坑 #1**：`tool_result` 是 **6 个扩展独有事件**之一（另五个是 `context` / `tool_call` / `before_agent_start` / `input` / `model_select`）。在 server 层用 `session.subscribe("tool_result", ...)` 会**静默失败**——handler 注册了但 type 分支永不命中。想做服务端结果后处理**必须走扩展机制**（`extensionFactories` + `pi.on`）。详见 [04-events.md 坑 4](../sdk_doc/04-events.md#坑-46-个扩展独有事件sessionsubscribe-静默收不到-最大集成坑)。

> ⚠️ **关键集成坑 #2**：`renderCall` / `renderResult` / `ctx.ui.setStatus` 等 TUI 渲染能力**仅在 interactive（tui）模式下被实际渲染**。SDK 集成（`createAgentSession`）、`print` / `json` 模式下 `ctx.hasUI === false`，所有 UI 方法是 no-op，渲染器返回的 Component 不会被显示。**Web/Server 集成只能用 `tool_result` 事件做"修改 LLM 看到的内容"，不能用渲染器做"修改用户看到的界面"**——后者要自己在前端实现。

> ⚠️ **import 来源**：TypeBox 从 `typebox` 包导入（**不是** `@sinclair/typebox`，CHANGELOG v1.9 已统一）。
> ```ts
> import { Type } from "typebox";                              // ✅ 正确
> import { Type } from "@sinclair/typebox";                    // ❌ 旧包名，已废弃
> ```

---

## 默认行为（★ 必读）

**工具启用规则**（与 [A04](A04-tool-whitelist.md) / [D01](D01-custom-tool.md) / [D02](D02-dynamic-tools.md) / [D04](D04-confirm-destructive.md) 一致）：

| `createAgentSession` 配置 | 内置工具 | 扩展工具 | customTools |
|--------------------------|---------|---------|-------------|
| 不传 `tools`（默认） | 启用 `["read", "bash", "edit", "write"]` | 自动启用 | 自动启用 |
| `tools: ["read", "bash"]` | 仅 read + bash | **必须列入才启用** | **必须列入才启用** |
| `noTools: "all"` | 全禁用 | 全禁用 | 全禁用 |
| `noTools: "builtin"` | 全禁用 | 自动启用 | 自动启用 |
| `excludeTools: ["bash"]` | bash 禁用，其他启用 | 自动启用（除非在 exclude 中） | 自动启用（除非在 exclude 中） |

**关键事实**：

- `tool_result` 钩子对**所有已启用的工具**生效——无论内置、扩展注册、还是 customTools
- 没启用（被 `tools` / `excludeTools` / `noTools` 过滤掉）的工具**不会触发** `tool_result`
- 证据：`agent-session.ts` `isAllowedTool` 对 customTools 和 extensionTools 走同一过滤；`runner.ts` `emitToolResult` 在工具实际执行后派发

---

## 核心机制：`tool_result` 事件

### ToolResultEvent 字段（`types.ts`）

所有工具的 `tool_result` 事件结构都是 `{ type, toolCallId, toolName, input, content, isError, details, usage }`：

| 字段 | 类型 | 说明 |
|------|------|------|
| `type` | `"tool_result"` | 事件类型字面量 |
| `toolCallId` | `string` | 本次工具调用的唯一 ID |
| `toolName` | `string` | 工具名（按工具分 narrow 类型，详见下表） |
| `input` | `Record<string, unknown>` | 工具收到的原始参数（**与 `tool_call` 事件同引用**，但 `tool_result` 阶段做 mutation 不再生效——已执行完） |
| `content` | `(TextContent \| ImageContent)[]` | **将送 LLM 的内容**——修改这里改变模型看到的结果 |
| `isError` | `boolean` | 是否作为错误返回（throw 出来也是 isError=true） |
| `details` | 因工具而异 | 结构化元数据，**不送 LLM**——仅供 UI 或扩展内部使用 |
| `usage` | `Usage \| undefined` | 工具执行自身的用量统计（token 等）。**可读**（如按 token 计费、统计），要改用量走 `ToolResultEventResult.usage` 返回 |

**按工具分 details 类型**：

| 工具 | `event.toolName` | `event.details` 类型 |
|------|-----------------|---------------------|
| bash | `"bash"` | `BashToolDetails \| undefined` |
| read | `"read"` | `ReadToolDetails \| undefined` |
| edit | `"edit"` | `EditToolDetails \| undefined` |
| write | `"write"` | `undefined`（write 无 details） |
| grep / find / ls | `"grep"` 等 | 各自 `*ToolDetails \| undefined` |
| 自定义工具 | `string` | `unknown` |

### handler 签名（`types.ts`）

```ts
type ExtensionHandler<E, R = undefined> = (
  event: E,
  ctx: ExtensionContext,
) => Promise<R | void> | R | void;

pi.on("tool_result", (event, ctx) => {
  // event: ToolResultEvent（按 toolName narrow）
  // ctx: ExtensionContext（hasUI / ui / cwd / mode / sessionManager 等）
  // 返回：ToolResultEventResult | void
});
```

### handler 返回值（`types.ts`）

```ts
interface ToolResultEventResult {
  content?: (TextContent | ImageContent)[];  // 替换 content（整体替换，不是追加）
  details?: unknown;                          // 替换 details
  isError?: boolean;                          // 覆盖 isError 标记
  usage?: Usage;                              // 覆盖工具用量统计（token 数等）
}
```

**关键**：**只有这 4 个字段，全是 optional**。没有 `block` / `skip` / `display` / `meta` 等其他字段（任何尝试 return 这些字段的代码都是基于幻觉）。

**与 `tool_call` 的关键区别**：

| 维度 | `tool_call`（执行前） | `tool_result`（执行后） |
|------|----------------------|------------------------|
| 能否阻断 | ✅ `return { block: true, reason }` | ❌ **不能阻断**——工具已执行完毕 |
| 能否改 input | ✅ mutate `event.input` | ❌ 改了也没用（已执行完） |
| 能否改 content | ❌ 工具还没执行，没有 content | ✅ 替换 `event.content` 或 return `content` |
| 能否改 isError | ❌ 同上 | ✅ 覆盖 `event.isError` |
| 典型用途 | 阻断危险操作 / 修改参数 | 脱敏 / 截断 / 错误友好化 |

### handler 执行机制（`runner.ts`）

```
extensionFactories: [extA, extB, extC]
                         ↓
            tool_result event 派发顺序：
            extA.handler → extB.handler → extC.handler → 内容送 LLM
```

1. **串行执行 + merge 累积**：handler 返回的 `content` / `details` / `isError` 字段被合并到 `currentEvent`，下一个 handler 看到的是合并后的版本
2. **没有短路**：所有 handler 都会跑完（不像 `tool_call` 的 `block` 立即短路）
3. **⚠️ 只有 return 风格有效**（★ P0 事实）：
   - **runner 只看 handler 的返回值**（`runner.ts`）：`const handlerResult = await handler(currentEvent, ctx); if (!handlerResult) continue;`——没返回值就 `continue`，`modified` 永远不变 true
   - **runner 最终返回**：`modified=false` 时直接 `return undefined`（`runner.ts`），调用方拿不到 hookResult 就走 fallback 用原始 result（`agent-session.ts`）
   - **正确写法（return 风格）**：`return { content: [...] }` → runner merge 到 currentEvent，最终返回给调用方替换原始 result
4. **mutate 风格：表面"看起来生效"，实则依赖隐晦副作用，禁止使用**：
   - **emitToolResult 浅拷贝 event**（`runner.ts` `const currentEvent = { ...event }`）——`currentEvent.content` 与原 `event.content` **共享同一个数组引用**
   - **mutation 的两种写法，行为截然不同**：
     - `event.content[0].text = "..."`（**改数组元素**）→ 因为浅拷贝共享引用，原 `result.content` 数组也被改了。runner 返回 undefined → 调用方 fallback 用原始 result → **恰好读到被 mutate 过的内容**。**这不是 runner "重新合并"，是调用方 fallback 路径的副作用泄漏**
     - `event.content = [...]`（**替换整个数组字段**）→ 只改了 currentEvent 的字段，`modified` 仍是 false，runner 返回 undefined，调用方 fallback 用原始 result 的**旧数组** → **完全无效，mutate 被静默丢弃**
   - **结论**：mutate 风格的"有效性"取决于调用方恰好在 hookResult=undefined 时 fallback 到被 mutate 过的引用——这是实现细节耦合，不是合约。**未来 runner 若改成深拷贝或调用方若不再 fallback，所有 mutate 写法立刻失效**
5. **与 `emitToolCall` 的对比**（mutation 在 tool_call 有效，在 tool_result 是地雷）：
   - `emitToolCall`（`runner.ts`）**直接传 event 给 handler，没有浅拷贝**——handler 改 `event.input.xxx` 立刻反映到调用方对象
   - `emitToolResult` 多了 `{ ...event }` 浅拷贝 + `modified` 标志位 + 返回值合并三件套，**只信任返回值**
6. **推荐做法**：**永远用 return 风格**，把 handler 当纯函数。mutate 风格不仅容易踩"替换数组失效"的坑，还会让代码读者误以为有合约保障

---

## 核心代码：方案一 `tool_result` 钩子（推荐 SDK 集成用）

适用：跨工具统一规则（脱敏、截断、错误友好化）。**不依赖 UI，Web/Server 都能用**。

```ts
import type { ExtensionFactory } from "@earendil-works/pi-coding-agent";

const SENSITIVE_PATTERNS = [
  /sk-[a-zA-Z0-9]{20,}/g,                 // OpenAI API key
  /AKIA[A-Z0-9]{16}/g,                    // AWS access key
  /gh[pousr]_[A-Za-z0-9]{36}/g,           // GitHub token
  /password\s*=\s*["'][^"']+["']/gi,      // 密码赋值
];

const extension: ExtensionFactory = (pi) => {
  pi.on("tool_result", (event, ctx) => {
    // ⚠️ 全程用 return 风格——mutation 在 tool_result 不可靠（详见下文陷阱 8）
    // runner 浅拷贝 event 后只看 handler 返回值，mutation 改数组元素虽然因
    // 浅拷贝共享引用"看起来生效"，但替换数组字段的 mutation 会静默失效

    // --- 1. 脱敏 + 2. 截断：基于原 content 构造新数组 ---
    const newContent = event.content.map((block) => {
      if (block.type !== "text") return block;
      let text = block.text;
      // 脱敏
      for (const pattern of SENSITIVE_PATTERNS) {
        text = text.replace(pattern, "[REDACTED]");
      }
      // 截断
      if (text.length > 2000) {
        text =
          text.slice(0, 2000) +
          `\n\n[已截断，原长度: ${text.length} 字符，使用 offset 翻页或 grep 精确查找]`;
      }
      return { ...block, text };
    });

    // --- 3. 错误友好化：把 isError=true 的内容改写为可操作提示 ---
    if (event.isError) {
      const original = newContent[0]?.type === "text" ? newContent[0].text : "";
      return {
        content: [{
          type: "text" as const,
          text: `工具 ${event.toolName} 执行失败。原始错误：${original.slice(0, 200)}。请尝试换参数或改用其他工具。`,
        }],
        // isError 保持 true，不覆盖
      };
    }

    // 脱敏/截断后的内容
    return { content: newContent };
  });
};

export default extension;
```

**代码讲解**：

- **全程 return 风格**：handler 永远返回一个 `{ content: ... }` 对象，由 runner 合并。**不要 mutate `event.content[0].text = ...`**——tool_result 的 mutation 不可靠（详见陷阱 8），替换数组字段的 mutate 会静默失效
- **`.map()` 构造新数组**：脱敏/截断时用 `event.content.map(b => ({...b, text: ...}))` 生成新数组再 return，比 mutate 更安全也更适合链式组合
- **`type: "text" as const`**：返回 content 数组时 TypeScript 需要 const 断言才能 narrow `TextContent` 联合类型
- **`isError: true` 保留**：让 Agent 知道这是失败，引导它换策略；改成 `false` 会让模型误以为成功，详见陷阱 6
- **return 风格的语义**：想修改就 `return { content: ... }`，由 runner 合并；**不想改就 `return undefined`（或不写 return）**——runner 会 `continue`，`modified` 保持 false，最终返回 undefined 让调用方走 fallback 用原始 result。这是"我这次不改"的标准写法，不是反模式

---

## 核心代码：方案二 `renderCall` / `renderResult`（仅 TUI 美化）

适用：自定义工具在交互式终端（CLI）里美化展示。**Web/Server 集成完全不适用**——渲染器返回的 Component 不会被显示。

### 真实签名（`types.ts`）

```ts
interface ToolDefinition<TParams, TDetails, TState> {
  // ...其他字段

  /** 工具调用阶段的渲染（参数流式生成时也调用） */
  renderCall?: (
    args: Static<TParams>,                              // 已校验的参数
    theme: Theme,                                        // 当前主题
    context: ToolRenderContext<TState, Static<TParams>>, // 渲染上下文
  ) => Component;

  /** 工具结果阶段的渲染 */
  renderResult?: (
    result: AgentToolResult<TDetails>,                   // 工具返回值
    options: ToolRenderResultOptions,                    // { expanded: boolean; isPartial: boolean }
    theme: Theme,
    context: ToolRenderContext<TState, Static<TParams>>,
  ) => Component;
}
```

### ToolRenderContext 字段（`types.ts`）

| 字段 | 类型 | 含义 |
|------|------|------|
| `args` | `TArgs` | 当前工具调用的参数（与 `renderCall` 的 args 同源，可在 `renderResult` 里回看） |
| `toolCallId` | `string` | 本次工具执行的唯一 ID（跨 call/result 渲染稳定） |
| `invalidate` | `() => void` | 触发本行重绘（用于定时刷新，如 bash 的耗时显示） |
| `lastComponent` | `Component \| undefined` | 上次返回的组件（**增量更新关键**——复用而非重建） |
| `state` | `TState` | 渲染器的私有状态（如 startedAt / interval），由 `tool-execution.ts` 初始化 |
| `cwd` | `string` | 工作目录 |
| `executionStarted` | `boolean` | 是否已开始执行（参数完成后到执行前为 false） |
| `argsComplete` | `boolean` | 参数是否已完整（流式生成时可能 partial） |
| `isPartial` | `boolean` | 结果是否为流式部分（onUpdate 触发的渲染） |
| `expanded` | `boolean` | 结果视图是否展开 |
| `showImages` | `boolean` | TUI 是否允许显示内联图片 |
| `isError` | `boolean` | 结果是否为错误 |

> ⚠️ **没有 `context.tui` 字段**——历史文档可能写错。所有渲染操作通过 `new Text(...)` / `new Box(...)` 等 TUI 组件构造，配合 `context.lastComponent` 做增量更新。

### 完整示例（参照 bash.ts 的真实写法）

```ts
import { defineTool } from "@earendil-works/pi-coding-agent";
import { Type } from "typebox";
import { Text } from "@earendil-works/pi-tui";  // TUI 组件库

interface QueryTableState {
  startedAt?: number;
  endedAt?: number;
}

const tableTool = defineTool({
  name: "query_table",
  label: "Query Table",
  description: "Query database and return tabular data",
  parameters: Type.Object({
    sql: Type.String({ description: "SQL query" }),
  }),
  async execute(toolCallId, params, signal, onUpdate, ctx) {
    const rows = [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }];
    return {
      content: [{ type: "text", text: JSON.stringify(rows) }],
      details: { rowCount: rows.length },
    };
  },

  // 渲染调用阶段：显示 SQL 查询语句
  renderCall(args, _theme, context) {
    // 增量更新模式：复用 lastComponent 避免每帧重建
    const text = (context.lastComponent as Text | undefined) ?? new Text("", 0, 0);
    text.setText(`查询: ${args.sql}`);
    return text;
  },

  // 渲染结果阶段：显示返回行数 + 耗时
  renderResult(result, options, _theme, context) {
    const state = context.state as QueryTableState;

    // 记录开始/结束时间
    if (context.executionStarted && state.startedAt === undefined) {
      state.startedAt = Date.now();
    }
    if (!options.isPartial || context.isError) {
      state.endedAt ??= Date.now();
    }

    const text = (context.lastComponent as Text | undefined) ?? new Text("", 0, 0);
    const rowCount = (result.details as { rowCount?: number } | undefined)?.rowCount ?? 0;
    const elapsed = state.endedAt && state.startedAt
      ? `${((state.endedAt - state.startedAt) / 1000).toFixed(1)}s`
      : "...";
    text.setText(`返回 ${rowCount} 行 · 耗时 ${elapsed}`);
    return text;
  },
});
```

**代码讲解**：

- **`new Text("", 0, 0)`**：构造空白文本组件，参数是 `(text, paddingX, paddingY)`——第二三个参数是**左右/上下内边距**（默认各 1），不是坐标。`new Text("", 0, 0)` 即无内边距；bash.ts / read.ts 等内置工具传 `0, 0` 是为了让输出紧贴边框；想要默认 1 格内边距直接 `new Text(text)` 省略后两参即可
- **`context.lastComponent as Text | undefined`**：复用上次返回的组件做增量更新——避免每帧重建（renderCall 在参数流式生成时可能每 token 调一次）
- **`context.state`**：跨 call/result 渲染共享的私有状态，由 tool-execution.ts 初始化为 `{}`，渲染器按需扩展
- **`options.isPartial`**：true 表示是 onUpdate 触发的流式渲染（结果还没最终确定），false 表示最终结果

---

## 变体

### 变体 A：图片内容替换为占位符

read 工具读图片文件会返回 `ImageContent`，但 Web/Server 场景下 LLM 可能不支持图片输入：

```ts
pi.on("tool_result", (event, ctx) => {
  // ⚠️ 不能 mutate `event.content = ...`——替换数组字段的 mutation 在 tool_result 静默失效
  // 必须用 return 风格（详见陷阱 8）
  return {
    content: event.content.map((block) => {
      if (block.type === "image") {
        return {
          type: "text" as const,
          text: `[图片内容，${block.mimeType}，大小未披露]`,
        };
      }
      return block;
    }),
  };
});
```

**何时用**：目标模型不支持 vision（如 deepseek-coder、code 系列），或想节省 token。

### 变体 B：bash 命令输出的 ANSI 颜色码清理

```ts
pi.on("tool_result", (event, ctx) => {
  if (event.toolName !== "bash") return;
  // ⚠️ 不能 mutate `block.text = ...`——改数组元素的 mutation 虽因浅拷贝"看似生效"
  // 但依赖调用方 fallback 路径的副作用，不是合约。用 return 风格（详见陷阱 8）
  return {
    content: event.content.map((block) => {
      if (block.type !== "text") return block;
      return {
        ...block,
        text: block.text.replace(/\u001b\[[0-9;]*[a-zA-Z]/g, ""),
      };
    }),
  };
});
```

**何时用**：bash 输出含 `ls --color` / `grep --color` 等颜色码，LLM 看到转义字符会困惑。

### 变体 C：read 工具大文件自动分页提示

> ⚠️ **字段结构务必核对**：read 工具的 `event.details` 真实类型是 `ReadToolDetails = { truncation?: TruncationResult }`——**截断标志在 `details.truncation?.truncated`，不是顶层 `details.truncated`**；`TruncationResult` **没有 `nextOffset` 字段**（字段是 `content / truncated / truncatedBy / totalLines / totalBytes / outputLines / outputBytes / lastLinePartial / firstLineExceedsLimit / maxLines / maxBytes`）。read 工具的翻页参数叫 `offset`（schema 字段，1-indexed 行号），但 details 里**不主动给出"下一个 offset"**——需要开发者用「当前 offset + 已输出行数」自行计算。若写成 `details.truncated` / `details.nextOffset`，type cast 能绕过编译，但运行时永远进不去分支，**静默失效**。

```ts
pi.on("tool_result", (event, ctx) => {
  if (event.toolName !== "read") return;
  const details = event.details as
    | { truncation?: { truncated?: boolean; outputLines?: number; totalLines?: number } }
    | undefined;
  const trunc = details?.truncation;
  if (!trunc?.truncated) return; // 只在确实被截断时处理

  // 计算下一页 offset：当前起点（默认第 1 行）+ 本次已输出行数
  // read.ts 内部用 nextOffset = endLineDisplay + 1，这里复刻同款算法
  const startOffset = (event.input.offset as number | undefined) ?? 1;
  const nextOffset = startOffset + (trunc.outputLines ?? 0);
  const total = trunc.totalLines ?? "?";

  return {
    content: [
      ...(event.content as any[]),
      {
        type: "text" as const,
        text: `\n[提示：文件共 ${total} 行，本次已截断。下次调用 read 时传 offset=${nextOffset} 继续读取下一页]`,
      },
    ],
  };
});
```

**何时用**：希望 LLM 自动用 offset 翻页读取大文件（否则它可能以为内容就这么短）。

> 参考：read 工具自身在 content 末尾已经会追加 `[Showing lines X-Y of Z. Use offset=N to continue.]`（read.ts:295-310），所以这个 handler 主要用于「额外加一段中文提示」或「定制提示措辞」的场景。如果不打算改写，让 read 自己提示即可。

### 变体 D：自定义工具的结构化 details 转换

```ts
pi.on("tool_result", (event, ctx) => {
  if (event.toolName !== "search_code") return;
  // details 里有结构化的命中列表，content 里是 LLM 友好的摘要
  // 这里在 details 上补一个聚合字段给前端 UI 用
  const details = event.details as { hits?: Array<{ file: string; line: number }> } | undefined;
  if (details?.hits) {
    return {
      details: {
        ...details,
        filesAffected: [...new Set(details.hits.map((h) => h.file))],
        totalHits: details.hits.length,
      },
    };
  }
});
```

**何时用**：details 是程序看的（不送 LLM），但前端 UI 想从 details 里取聚合数据展示。

### 变体 E：TUI 自定义渲染器（仅 CLI）

用 `Box` / `Text` 组合做表格、进度条等复杂终端界面。**仅在 interactive 模式生效**，参考 `packages/coding-agent/src/core/tools/bash.ts` 的 `BashResultRenderComponent` 实现复杂动画效果。

**何时用**：CLI 工具的终端美化，非 SDK 集成场景。

### 变体 F：错误改写 + 静默化（陷阱 6 警示）

```ts
pi.on("tool_result", (event, ctx) => {
  if (!event.isError) return;
  // ⚠️ 危险：把 isError 从 true 改成 false 会让 Agent 误以为成功
  // 仅在极特殊场景（如调试时屏蔽已知无害错误）使用
  return {
    content: [{ type: "text" as const, text: "(操作已完成)" }],
    isError: false,
  };
});
```

**何时用**：**几乎从不用**。除非你完全确定错误对 Agent 后续推理无害，否则保留 `isError: true` 让模型自适应。

---

## 与 `setStatus` 配合（CLI 专属，★ 必读陷阱）

`ctx.ui.setStatus` 的真实签名是 `setStatus(key: string, text: string | undefined)` —— **两个参数**（`types.ts`）：

```ts
// ❌ 错误：单参数，text 错位为 key
ctx.ui.setStatus("正在处理...");

// ✅ 正确：key + text 两参数
ctx.ui.setStatus("query_table", "正在执行 SQL...");
// 清除：
ctx.ui.setStatus("query_table", undefined);
```

**`hasUI=false` 时静默失败**（`runner.ts`、`agent-session.ts`）：

```ts
const noOpUIContext: ExtensionUIContext = {
  setStatus: () => {},   // ← SDK/print/json 模式下永远是 no-op
  notify: () => {},
  // ...
};
```

`createAgentSession` 默认不绑 `uiContext` → runner 走 `noOpUIContext` → `hasUI === false`，调 setStatus 没任何效果（也不报错）。

> 注：`hasUI()` 的真实判定是 `this.uiContext !== noOpUIContext`（runner.ts:442-444），**与 `mode` 无直接关系**——是"没提供 uiContext"导致 hasUI 为 false，而不是 `mode="print"` 本身。`createAgentSession` 默认不绑 uiContext 所以结果对（hasUI=false）；但 RPC 模式（绑了 uiContext）下 hasUI 为 true。判断时直接用 `ctx.hasUI`，不要假设 `mode` 决定一切。

```ts
pi.on("tool_result", (event, ctx) => {
  if (ctx.hasUI) {
    ctx.ui.setStatus("query_table", `处理 ${event.toolName} 结果...`);
  }
  // ...处理逻辑
  if (ctx.hasUI) {
    ctx.ui.setStatus("query_table", undefined);  // 清除
  }
});
```

**Web/Server 场景的替代方案**：自实现 SSE / WS 推送把状态变化发给前端，前端用自己的 UI 显示。

---

## 陷阱

### 陷阱 1：`tool_result` 是扩展独有事件（★ 最大集成坑）

`tool_result` 是 6 个扩展独有事件之一（另五个是 `context` / `tool_call` / `before_agent_start` / `input` / `model_select`）。在 server 层用 `session.subscribe("tool_result", ...)` **静默失败**——handler 注册了但 type 分支永不命中，无任何报错。想做服务端结果后处理必须走扩展机制。详见 [04-events.md 坑 4](../sdk_doc/04-events.md#坑-46-个扩展独有事件sessionsubscribe-静默收不到-最大集成坑)。

### 陷阱 2：`context.tui` 字段幻觉（★ P0 高发）

`ToolRenderContext` **没有 `tui` 字段**。真实字段见上方字段表。渲染操作通过 `new Text(...)` / `new Box(...)` 配合 `context.lastComponent` 完成。

```ts
// ❌ 错误：context.tui 不存在，运行时 TypeError
renderCall: (args, theme, context) => context.tui.text("...");

// ✅ 正确
renderCall: (args, theme, context) => {
  const text = (context.lastComponent as Text | undefined) ?? new Text("", 0, 0);
  text.setText("...");
  return text;
};
```

### 陷阱 3：`renderCall` / `renderResult` 在非 TUI 模式下不渲染

SDK 集成（`createAgentSession`）默认 `mode = "print"`，渲染器返回的 Component 不会被显示。**渲染器是 CLI 专属能力**，Web/Server 集成只能用 `tool_result` 事件修改"LLM 看到的内容"，不能用渲染器修改"用户看到的界面"——后者要自己在前端实现。

### 陷阱 4：`setStatus` 单参数用法

`setStatus(key, text)` 是两个参数（`types.ts`）。单参数会让 text 错位为 key，实际状态栏不显示内容。

### 陷阱 5：`setStatus` 在 `hasUI=false` 时静默失败

`createAgentSession` 默认不绑 `uiContext` → runner 用 `noOpUIContext` → `ctx.hasUI === false`。调 `ctx.ui.setStatus` 是 no-op，**没有任何报错**。必须用 `ctx.hasUI` 判断或 try/catch 兜底。（`hasUI` 看的是 `uiContext !== noOpUIContext`，不看 `mode`——见上文 §与 setStatus 配合 的说明）

### 陷阱 6：把 `isError: true` 改成 `false` 让 Agent 误以为成功

```ts
// ⚠️ 危险
if (event.isError) {
  return { content: [...], isError: false };
}
```

这会让 Agent 把失败当成功继续推理，可能产生幻觉链路（如「文件已保存」后继续做依赖该文件的操作）。**只在极特殊场景用**（如屏蔽已知无害错误），且必须在 content 里说清楚实际情况。

### 陷阱 7：修改 `event.input` 在 `tool_result` 阶段无效

`tool_result` 事件虽然带 `input` 字段（与 `tool_call` 同引用），但工具**已经执行完毕**——改 input 不会改变已发生的结果。要改参数必须在 `tool_call` 阶段（见 [E01](E01-tool-intercept.md)）。

### 陷阱 8：handler 用 mutate 风格（★ P0 事实）

**tool_result 的 mutate 风格不是"风格选择"，是地雷**。详见上文「核心机制 §6」：

```ts
// ⚠️ 危险：mutate 改数组元素看似生效，实则依赖浅拷贝 + 调用方 fallback 的副作用
pi.on("tool_result", (event, ctx) => {
  event.content[0].text = "...";   // runner 看不到这次修改
  // 没有 return → runner modified=false → 返回 undefined → 调用方 fallback 用 result.content
  // 因为浅拷贝共享引用，恰好读到 mutate 过的数组——不是合约，是实现泄漏
});

// ⚠️ 致命：mutate 替换整个数组字段完全无效
pi.on("tool_result", (event, ctx) => {
  event.content = [{ type: "text", text: "全部替换" }];  // 静默丢弃！
  // modified 仍为 false，调用方 fallback 用原始 result 的旧 content
});
```

**对策**：**永远用 return 风格**，不依赖任何 mutation：
```ts
// ✅ 正确
pi.on("tool_result", (event, ctx) => {
  return {
    content: [{ type: "text" as const, text: "全部替换" }],
  };
});
```

**与 `tool_call` 的关键区别**（容易混淆点）：`tool_call` 阶段的 mutation 是有效的（emitToolCall 不做浅拷贝，直接传 event 给 handler）。但到了 `tool_result` 阶段，runner 多了浅拷贝 + modified 标志位 + 返回值合并三件套，**只信任返回值**。详见上文「核心机制 §5 与 emitToolCall 的对比」。

### 陷阱 9：返回 `display` / `meta` 等幻觉字段

`ToolResultEventResult` **只有 `content` / `details` / `isError` / `usage`** 四字段。任何尝试 return `display` / `meta` / `block` / `skip` 等字段的代码都是基于幻觉（历史文档可能写错过）。核查源码：`types.ts`。

### 陷阱 10：handler 内 throw 会被吞掉

handler 内 throw 不会终止 agent loop——错误会被 runner.ts 的 try/catch 捕获，然后通过 `this.emitError(...)` 派发给扩展错误流（errorListeners），上层宿主通常会把 listener 收到的消息显示为警告/diagnostic。但工具结果照原样送 LLM（`runner.ts`：catch 后 `modified` 保持 false，调用方 fallback 用原始 `result.content`）。要主动让 Agent 看到错误，必须 `return { isError: true, content: [...] }`。

### 陷阱 11：`renderResultStream` 字段不存在

早期文档可能提到 `renderResultStream` 用于流式渲染——**当前 source 中 `ToolDefinition` 只有 `renderCall` 和 `renderResult`**，没有 `renderResultStream`。流式更新通过 `execute()` 内调 `onUpdate` + `renderResult` 的 `options.isPartial` 配合实现。

### 陷阱 12：handler 返回对象但漏 `usage` 会丢失原始用量（★ 隐性坑）

调用方 `agent-session.ts` 对 `content` / `isError` 都有 fallback（`hookResult?.content ?? result.content`、`hookResult?.isError ?? isError`），**唯独 `usage` 没有**：源码是 `usage: hookResult?.usage`（无 `?? result.usage`）。

后果：如果你的 handler `return { content: newContent }`（只想改内容、没动用量），最终送给 Agent 的 `usage` 会是 `undefined`——**原始 `result.usage` 丢失**。对统计/计费场景是隐性数据丢失。

```ts
// ❌ 漏 usage：原始用量丢失
pi.on("tool_result", (event) => {
  return { content: redactedContent };
});

// ✅ 改内容的同时保留用量
pi.on("tool_result", (event) => {
  return { content: redactedContent, usage: event.usage };
});
```

**对策**：handler 一旦 return 了对象，就把 `usage: event.usage` 显式带上。若完全 `return undefined`（不改），则无此问题——调用方会用原始 result。

---

## 横向联动

- [D01 开发自定义工具](D01-custom-tool.md) — 工具定义的完整 API（execute / parameters / onUpdate）
- [D02 动态注册工具](D02-dynamic-tools.md) — 运行时注册 / 覆盖工具
- [D04 工具调用安全闸门](D04-confirm-destructive.md) — `tool_call` 事件 + 阻断机制（D05 是 `tool_result` 后处理，互补关系）
- [D06 限制工具执行目录](D06-protected-paths.md) — 路径白名单
- [E01 拦截与修改工具调用](E01-tool-intercept.md) — `tool_call` 通用拦截（修改参数）
- [E02 编写完整扩展](E02-extension-basics.md) — `tool_call` + `tool_result` + 生命周期的完整组合
- [E06 流式处理工具输出](E06-streaming-transform.md) — `tool_execution_update` 流式 hook
- [A06 默认行为](A06-load-extensions.md) — `createAgentSession` 工具启用规则
- [sdk_doc/06-tools.md](../sdk_doc/06-tools.md) — 工具系统完整 API 参考
- [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) — `pi.on` / `ExtensionContext` 完整接口
- [sdk_doc/04-events.md](../sdk_doc/04-events.md) — 扩展独有事件清单（`tool_result` 是其中之一）
