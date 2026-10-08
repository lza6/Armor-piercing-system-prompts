# 场景：流式处理工具输出 (E06)

## 什么时候用 / 不用会怎样

**该用本场景**：

- **跨工具统一脱敏**：所有工具返回内容里的密钥（`sk-***`、`Bearer ***`）、Token、内网 IP，在送回 LLM **之前**统一替换——比在每个工具内部各自处理更集中
- **错误友好化**：工具失败时原始堆栈/英文报文直接回到 LLM，模型可能基于错误信息继续无效重试。先在 `tool_result` 阶段把错误改写成可操作的提示（如「未找到 ID=xxx 的用户，请用 lookup_user 工具核查」）
- **结果增强 / 截断**：为特定工具的返回添加摘要、行数统计、结构化标记；或超长输出在送 LLM 前截断到 N 行（减少上下文消耗）
- **流式进度展示（CLI 专属，且仅 bash 工具真正触发）**：在 TUI 状态栏显示 `bash: 正在执行 npm install...`，让用户看见工具正在工作而非卡死
- **审计 / 合规**：所有工具结果统一过一道日志，记录 toolName / args / isError / 处理后 content 摘要

**不用会怎样**：

- 工具原始输出直接回到 LLM——API Key、Token 之类敏感信息进入对话历史，可能在后续被模型复述或日志泄露
- 错误信息晦涩，模型基于错误内容继续无效重试，浪费 token
- 大文件内容（read 整个 10MB 日志）直接占满 context window

**不适合本场景**：

- **完全替换工具执行逻辑**（如自己实现沙箱 runner）→ 不是 `tool_result` 能做的，要在工具定义层重写 `execute`，见 [D01 自定义工具](D01-custom-tool.md)
- **TUI 终端美化（自定义渲染组件）** → `tool_result` 只改**送给 LLM 的内容**；要让**终端用户**看到表格/高亮/折叠，用 `defineTool({ renderResult })`（仅 interactive 模式生效），见 [D05 自定义工具输出渲染](D05-tool-result-render.md)
- **拦截工具调用 / 改参数 / 阻断执行** → 那是 `tool_call` 事件的职责（执行前），见 [E01 拦截与修改工具调用](E01-tool-intercept.md)
- **工具调用前弹框确认（`rm -rf` 类）** → 见 [D04 工具调用安全闸门](D04-confirm-destructive.md)
- **修改最终消息 / 用户看到的内容** → `tool_result` 改的是「工具结果送 LLM」的内容，不直接控制 UI 渲染；要改最终消息用 `message_end` 事件（见 [04-events](../sdk_doc/04-events.md)）

---

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `pi.on("tool_execution_update", handler)` | 监听工具执行中的流式部分输出（**仅监听，handler 返回值被忽略**） | 本文档 §ToolExecutionUpdateEvent |
| `pi.on("tool_result", handler)` | 工具执行完毕后、结果送 LLM 前的后处理钩子 | 本文档 §ToolResultEvent |
| `pi.on("tool_execution_start" / "tool_execution_end", handler)` | 工具开始 / 结束的生命周期标记（仅监听） | 本文档 §三个事件的时序 |
| `return { content, details, isError, usage }` | **唯一**能修改工具结果的方式（mutation 无效，见陷阱 1） | 本文档 §ToolResultEventResult |

> ⚠️ **关键集成坑 1：`tool_result` 是 6 个扩展独有事件之一**（另五个是 `context` / `tool_call` / `before_agent_start` / `input` / `model_select`）。在 server 层用 `session.subscribe("tool_result", ...)` 会**静默失败**——外部事件流根本不派发这个 type。想做服务端工具结果后处理**必须走扩展**。详见 [04-events.md 坑 4](../sdk_doc/04-events.md#坑-46-个扩展独有事件sessionsubscribe-静默收不到-最大集成坑)。

> ⚠️ **关键集成坑 2：`tool_execution_*` 事件的 handler 返回值被忽略**。这 3 个事件用通用 `emit` 派发（runner.ts 的通用 `emit`），不处理 handler return——尝试 `return { suppress: true }` / `return { cancel: true }` 都**完全无效**。要"不显示某个工具的流式"只能在 handler 内部按 `event.toolName` 分支 `return`（什么都不做）。

---

## 默认行为

**工具启用规则**（与 [A04](A04-tool-whitelist.md) / [A06](A06-xxx.md) / [D01](D01-custom-tool.md) / [D04](D04-confirm-destructive.md) / [D05](D05-tool-result-render.md) / [E01](E01-tool-intercept.md) 一致）：

| `createAgentSession` 配置 | 内置工具 | 扩展工具 | customTools |
|--------------------------|---------|---------|-------------|
| 不传 `tools`（默认） | 启用 `["read", "bash", "edit", "write"]` | 自动启用 | 自动启用 |
| `tools: ["read", "bash"]` | 仅 read + bash | **必须列入才启用** | **必须列入才启用** |
| `noTools: "all"` | 全禁用 | 全禁用 | 全禁用 |
| `noTools: "builtin"` | 全禁用 | 自动启用 | 自动启用 |
| `excludeTools: ["bash"]` | bash 禁用，其他启用 | 自动启用（除非在 exclude 中） | 自动启用（除非在 exclude 中） |

> 📌 **此表为跨文档复用**：工具启用规则的**权威落点是 [A04 工具白名单](A04-tool-whitelist.md)**，本表仅为方便阅读而内联。尤其 `noTools: "builtin"` 行的「扩展/customTools 自动启用」语义依赖 SDK 层 `includeAllExtensionTools` 分支是否命中（sdk.ts 的 `createAgentSession` 未显式传该参数），实际激活行为以 A04 为准。本场景只关心「被启用的工具才会触发 `tool_result`」这一条。

**关键事实**：

1. `tool_result` 钩子对**所有已启用的工具**生效——无论内置、扩展注册、还是 customTools
2. 没启用（被 `tools` / `excludeTools` / `noTools` 过滤掉）的工具**不会触发** `tool_result`
3. `tool_execution_update` 只有工具内部主动调用了 `onUpdate()` 回调才会触发——**不是所有工具都会发**（详见下方「内置工具 onUpdate 支持表」）

**证据**：agent-session.ts 的默认工具初始化。

---

## 三个事件的时序

工具调用的完整生命周期是 **5 个事件串行触发**（agent-loop.ts 的 `executeToolCallsSequential` / `executeToolCallsParallel` + `prepareToolCall` + `executePreparedToolCall`）：

```
LLM 生成 tool call
      ↓
┌──────────────────────────────────────────────┐
│ tool_execution_start ← agent-core，仅监听    │  先 emit（agent-loop.ts:445/500）
│      ↓ （紧接着 prepareToolCall）            │
│ tool_call            ← 扩展层，可阻断/改参   │  见 E01（在 tool_execution_start 之后、真正 execute 之前）
│      ↓ （如果没被阻断，真正 execute）        │
│ tool_execution_update ← 仅监听（多次，可选）│  ← 本场景核心
│      ↓                                       │
│ tool_result          ← 扩展层，可改返回内容  │  ← 本场景核心
│      ↓ （在 finalizeExecutedToolCall 内触发）│
│ tool_execution_end   ← agent-core，仅监听    │  最后 emit（在 finalize 返回后）
│      ↓                                       │
│ 内容送 LLM 作为 toolResult message           │
└──────────────────────────────────────────────┘
```

> ⚠️ **时序关键（易踩坑）**：`tool_execution_start` 在 `tool_call`（扩展层 `beforeToolCall` 钩子）**之前**触发。源码里 agent-loop 先 `await emit({type:"tool_execution_start"})`（`executeToolCallsSequential` 第 445 行 / `executeToolCallsParallel` 第 500 行），再 `await prepareToolCall()`（第 452/507 行）——而 `prepareToolCall` 内部才调 `config.beforeToolCall`（第 619-628 行），即 `tool_call` 事件。**所以「tool_execution_start 已发」不代表「工具一定被执行」**——`tool_call` handler 仍可在其后阻断（`return { block: true }`）。要拦截/改参请用 `tool_call`（见 E01），不要依赖 `tool_execution_start` 的先后顺序。

**关键边界**：

- `tool_execution_*` 三个事件由 **agent-core 层**（`packages/agent/`）发射，agent-session 转发到扩展层（agent-session.ts 的 `_emitExtensionEvent` 转发逻辑）
- 这三个事件**两层都派发**（扩展层 + session.subscribe 都能收），不是扩展独有
- 但扩展层 handler 的**返回值会被忽略**（用通用 `emit`）
- `tool_result` 是**扩展独有**（只在扩展层派发），且 handler 返回值**生效**（必须用专用 `emitToolResult`）
- ⚠️ **tool_result 虽然是"执行后处理"，但在 tool_execution_end 之前触发**——`afterToolCall`（内含 `emitToolResult`）在 `finalizeExecutedToolCall` 内部调用，而 `emitToolExecutionEnd` 在 `finalizeExecutedToolCall` 返回后才调用

**`tool_execution_end` 的 payload（勿与 `tool_result` 混淆）**：types.ts 的 `ToolExecutionEndEvent` 只有 5 个字段——`type` / `toolCallId` / `toolName` / `result: any` / `isError: boolean`。注意它的 `result` 是**完整的 `AgentToolResult` 对象**（含 content/details/usage/terminate 等），与 `tool_result` 事件的 `content` 数组结构不同。`tool_execution_end` 是**仅监听**事件，handler 返回值被忽略——想改返回内容只能在 `tool_result` handler 里 `return { content, ... }`。

---

## ToolExecutionUpdateEvent payload

`pi.on("tool_execution_update", handler)` 收到的 event 结构（types.ts 的 `ToolExecutionUpdateEvent`）：

| 字段 | 类型 | 说明 |
|------|------|------|
| `type` | `"tool_execution_update"` | 事件类型字面量 |
| `toolCallId` | `string` | 唯一调用 ID，可关联 `tool_execution_start` / `tool_result` |
| `toolName` | `string` | 工具名（`"bash"` / `"read"` / 自定义工具名等） |
| `args` | `any` | 工具收到的参数（**只读**，改了也没用——工具已经在执行中） |
| `partialResult` | `any`（类型定义层面） | **工具内部 `onUpdate()` 回调传入的部分结果**，结构是 `{ content: (TextContent \| ImageContent)[], details: any }`——**没有 `progress` 字段**，要传进度可放 `details`。运行时结构由 `AgentToolUpdateCallback` 约束 |

**partialResult 的来源**（agent-loop.ts 的 `executePreparedToolCall`）：

```ts
// agent-core 在调用 tool.execute 时注入 onUpdate 回调
const result = await prepared.tool.execute(
  prepared.toolCall.id,
  prepared.args,
  signal,
  (partialResult) => {              // ← 这就是 onUpdate
    emit({
      type: "tool_execution_update",
      toolCallId: prepared.toolCall.id,
      toolName: prepared.toolCall.name,
      args: prepared.toolCall.arguments,
      partialResult,                 // ← 工具传入什么，event.partialResult 就是什么
    });
  },
);
```

> ⚠️ **实现细节（影响「实时性」预期）**：上面代码块是简化版。实际 `executePreparedToolCall`（agent-loop.ts:671-695）**并非同步立即 emit**——onUpdate 回调把每次 `emit(...)` 调用 push 到一个 `updateEvents` 数组（第 681-691 行），等工具 `execute` 返回后才 `await Promise.all(updateEvents)`（第 695 行）批量 flush。**意味着**：工具执行过程中多次调 `onUpdate` 产生的多个 `tool_execution_update` 事件，可能在工具结束后才**批量到达** handler，而非每次调用都实时触发。对 SDK 集成者而言：不要假设「onUpdate 一调、handler 立刻能收到」——尤其对长时工具（如 bash 跑 `npm install`），handler 看到的事件可能是滞后的快照序列。

**onUpdate 回调签名**（agent/types.ts 的 `AgentToolUpdateCallback`）：

```ts
type AgentToolUpdateCallback<T = any> = (partialResult: AgentToolResult<T>) => void;
```

工具作者负责决定**何时调**、**调多少次**、**partialResult 里放什么**。例如 bash 工具每 100ms 节流一次输出快照（bash.ts 的 `emitOutputUpdate`）：

```ts
onUpdate({
  content: [{ type: "text", text: snapshot.content || "" }],
  details: {
    truncation: snapshot.truncation.truncated ? snapshot.truncation : undefined,
    fullOutputPath: snapshot.fullOutputPath,
  },
});
```

### 内置工具 onUpdate 支持表

| 工具 | 是否支持 onUpdate | 说明 |
|------|-----------------|------|
| bash | ✅ 支持 | 节流 100ms 发送当前 stdout+stderr 快照，含 truncation 信息 |
| read | ❌ 不支持 | 参数签名有 `_onUpdate?` 但未调用（read.ts） |
| edit | ❌ 不支持 | 同上，参数命名为 `_onUpdate`（前缀 `_` 表明未用） |
| write | ❌ 不支持 | 同上 |
| grep / find / ls | ❌ 不支持 | 参数命名为 `_onUpdate` |
| 自定义工具 | ✅ 如果作者实现 | 在 `defineTool` 的 `execute` 中调用 `onUpdate?.(...)`——见 [D01 onUpdate 回调](D01-custom-tool.md#onupdate-流式更新回调) |

> **结论**：内置工具中**只有 bash 真正触发 `tool_execution_update`**。其他内置工具的 `tool_execution_update` 事件**永远不会触发**（参数未调用）。想在自定义工具里支持流式，必须自己在 `execute` 里调 `onUpdate?.(...)`——详见 [D01 §onUpdate 流式更新回调](D01-custom-tool.md#onupdate-流式更新回调)。

> ⚠️ **bash 首次空 partialResult**：bash 工具在命令开始执行时会先发一次 `onUpdate({ content: [], details: undefined })`（bash.ts）——handler 首次收到的 partialResult 可能是**空内容数组**，需要做空值判断。

---

## ToolResultEvent payload

`pi.on("tool_result", handler)` 收到的 event 结构（types.ts 的 `ToolResultEventBase`）：

| 字段 | 类型 | 说明 |
|------|------|------|
| `type` | `"tool_result"` | 事件类型字面量 |
| `toolCallId` | `string` | 唯一调用 ID |
| `toolName` | `"bash" \| "read" \| "edit" \| "write" \| "grep" \| "find" \| "ls" \| string` | 工具名；自定义工具是任意 string |
| `input` | `Record<string, unknown>` | 工具收到的原始参数（**已执行完，mutation 无效**——见 E01 的 input 字段表） |
| `content` | `(TextContent \| ImageContent)[]` | 工具返回的内容数组（`TextContent = { type: "text", text: string }`） |
| `isError` | `boolean` | 是否作为错误返回（throw 出来也是 `isError=true`） |
| `details` | 因工具而异 | 结构化元数据，**不送 LLM**——仅供 UI 或扩展内部使用 |
| `usage` | `Usage \| undefined` | 工具执行用量统计（token 数等），**不送 LLM** |

**按工具分 details 类型**（与 [D05](D05-tool-result-render.md#按工具分-details-类型) 一致）：

| 工具 | `event.toolName` | `event.details` 类型 |
|------|-----------------|---------------------|
| bash | `"bash"` | `BashToolDetails \| undefined` |
| read | `"read"` | `ReadToolDetails \| undefined` |
| edit | `"edit"` | `EditToolDetails \| undefined` |
| write | `"write"` | `undefined`（write 无 details） |
| grep / find / ls | `"grep"` 等 | 各自 `*ToolDetails \| undefined` |
| 自定义工具 | `string` | `unknown` |

---

## ToolResultEventResult：handler 返回值

**handler 签名**（types.ts 的 `ExtensionHandler`）：

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

**返回值结构**（types.ts 的 `ToolResultEventResult`）：

```ts
interface ToolResultEventResult {
  content?: (TextContent | ImageContent)[];  // 替换 content（整体替换，不是追加）
  details?: unknown;                          // 替换 details
  isError?: boolean;                          // 覆盖 isError 标记
  usage?: Usage;                              // 覆盖工具用量统计（token 数等）
}
```

**关键**：**4 个字段，全是 optional**。没有 `block` / `skip` / `suppress` / `display` / `meta` 等其他字段——任何尝试 return 这些字段的代码都**完全无效**（会被忽略）。

**与 `tool_call` 的关键区别**：

| 维度 | `tool_call`（执行前，见 E01） | `tool_result`（执行后，本场景） |
|------|----------------------|------------------------|
| 能否阻断 | ✅ `return { block: true, reason }` | ❌ **不能阻断**——工具已执行完毕 |
| 能否改 input | ✅ mutate `event.input`（**E01 特有**） | ❌ mutation 无效（已执行完） |
| 能否改 content | ❌ 工具还没执行，没有 content | ✅ **必须 `return { content: [...] }`**（mutation 无效，见陷阱 1） |
| 能否改 isError | ❌ 同上 | ✅ `return { isError: false }` 覆盖 |
| 典型用途 | 阻断危险操作 / 修改参数 | 脱敏 / 截断 / 错误友好化 |

---

## ExtensionContext（事件 handler 的 ctx）

每个事件 handler 接收 `(event, ctx)`，其中 `ctx: ExtensionContext` 提供运行时上下文。**关键字段**（types.ts 的 `ExtensionContext`，跨链 [E04 ExtensionContext](E04-lifecycle-hooks.md#extensioncontext事件-handler-的-ctx)）：

| 字段 | 类型 | 说明 |
|------|------|------|
| `ctx.cwd` | `string` | 当前 session 的工作目录 |
| `ctx.ui` | `ExtensionUIContext` | UI 方法（setStatus / notify / select / confirm / input 等） |
| `ctx.hasUI` | `boolean` | 是否有可交互 UI（tui / rpc 为 true，**print / json 为 false**） |
| `ctx.mode` | `"tui" \| "rpc" \| "json" \| "print"` | 当前运行模式，**默认 "print"** |
| `ctx.sessionManager` | `ReadonlySessionManager` | 只读会话管理 |
| `ctx.model` | `Model<any> \| undefined` | 当前模型 |
| `ctx.abort()` | - | 中止当前 agent 操作 |

> ⚠️ `ExtensionContext` **没有 `session` 字段**、**没有 `waitForIdle` 方法**（[E04 陷阱 5](E04-lifecycle-hooks.md#陷阱 5)）。`waitForIdle` 只在 `ExtensionCommandContext`（命令 handler 的 ctx）上。

> ⚠️ **`ctx.ui.setStatus(key, text)`** 签名是**双参数**（types.ts 的 `ExtensionUIContext.setStatus`）：
> - `key: string` — 状态栏位置标识（如 `"streaming"` / `"lifecycle"`）
> - `text: string | undefined` — 显示文本，`undefined` 清除
>
> **不是**单参数（早期 d.ts 审计第 20/21 项有误），但**必须加 `hasUI` 守卫**（见陷阱 3）。

---

## emitToolResult 执行机制（★ 修正 mutation 误解的核心证据）

runner.ts 的 `emitToolResult` 的完整实现要点：

```
extensionFactories: [extA, extB, extC]
                         ↓
            tool_result event 派发顺序：
            extA.handler → extB.handler → extC.handler → 内容送 LLM
```

1. **浅拷贝 event**（`const currentEvent = { ...event }`）——顶层字段独立，但 `content` / `details` 等**嵌套对象与原 event 共享引用**
2. **串行执行 + 检测返回值**——只有 handler **返回** `content` / `details` / `isError` / `usage` 字段时，才合并到 `currentEvent` 并设 `modified=true`
3. **没短路**：所有 handler 都会跑完（不像 `tool_call` 的 `block` 立即短路）
4. **必须 modified 才返回**（`if (!modified) return undefined`）
5. **try/catch + emitError**：handler 抛异常不影响后续 handler 执行

> ⚠️ **与 `emitToolCall` 的异常处理差异**：`emitToolResult` 对每个 handler 都包了 try/catch（异常被吞掉、调 `emitError`、不再抛）；但 `emitToolCall`（runner.ts）**没有 try/catch**——`tool_call` handler 抛异常会直接中断整个 emit 链，异常向上冒泡。写链式 `tool_call` handler 时务必自行 `try/catch`（这正好是 E01 的职责范围，本场景 `tool_result` 已有保护）。

**调用方的消费方式**（agent-session.ts 的 `afterToolCall` 回调）：

```ts
const hookResult = await runner.emitToolResult({ ... });
if (!hookResult) return undefined;          // ← 没有 return 就完全不变
return {
  content: hookResult.content,              // ← 只看返回值
  details: hookResult.details,              // ⚠️ hookResult 存在但无 details → undefined
  isError: hookResult.isError ?? isError,
  usage: hookResult.usage,                  // ⚠️ 同理
};
```

> ⚠️ **details 回退陷阱**：agent-session 层（上方代码）直接用 `hookResult?.details`——如果 handler 只 return 了 `content` 而**省略** `details`，返回的 `details` 字段是 `undefined`。agent-loop 层有 `afterResult.details ?? result.details` 兜底（不会丢失原始 details），但**最佳实践是显式 return 需要保留的字段**：`return { content: newContent, details: event.details }`。

**核心结论**：agent-session 完全基于 **`hookResult` 返回值**替换内容。**mutation 写法（`event.content[0].text = "..."`）不会被检测，最终 `return undefined`，原内容不变**。

---

## 核心代码

### 模式一：流式进度监听（仅 TUI / RPC 模式有效）

适用：CLI 终端里给用户看「工具正在工作」的视觉反馈。**Web/Server 集成场景请跳过本节**——`tool_execution_update` handler 拿到的 partialResult 你只能自记日志，没法回流给 LLM（内容已经在 LLM 调下一个工具前就送出去了）。

```ts
import type { ExtensionFactory } from "@earendil-works/pi-coding-agent";

const streamingProgressExtension: ExtensionFactory = (pi) => {
  pi.on("tool_execution_update", (event, ctx) => {
    // ⚠️ handler 返回值被忽略（通用 emit），不能 return { suppress: true }
    // 想跳过某工具的展示，只能内部 return（什么都不做）

    // 想拿到 partialResult 里的文本，从 event.partialResult.content 取
    const text =
      event.partialResult?.content
        ?.filter((c): c is { type: "text"; text: string } => c.type === "text")
        .map((c) => c.text)
        .join("\n") ?? "";

    if (event.toolName === "bash") {
      // bash 流式输出通常太长，只在 debug 模式才显示
      if (!process.env.DEBUG) return;
    }

    // ⚠️ ctx.ui.* 在 mode="print"（默认）下是 no-op
    // SDK / Web 集成必须加 hasUI 守卫
    if (ctx.hasUI) {
      const truncated = text.length > 60 ? text.slice(0, 57) + "..." : text;
      // setStatus 双参数：(key, text) — key 是状态栏位置标识
      ctx.ui.setStatus("streaming", `${event.toolName}: ${truncated || "处理中..."}`);
    } else {
      // Web/Server 场景：自记日志或推到 SSE
      console.log(`[streaming] ${event.toolName}: ${text.length} chars`);
    }
  });

  // tool_execution_end 时清除状态栏（避免残留）
  pi.on("tool_execution_end", (_event, ctx) => {
    if (ctx.hasUI) {
      ctx.ui.setStatus("streaming", undefined);
    }
  });
};

export default streamingProgressExtension;
```

**关键修正点**（vs 旧版示例）：

1. ❌ 旧版 `event.update?.progress` → ✅ `event.partialResult.content`
2. ❌ 旧版 `return { suppress: true }` → ✅ 内部 `return`（什么都不做）
3. ❌ 旧版 `ctx.ui.setStatus(...)` 无守卫 → ✅ `if (ctx.hasUI)` 守卫 + `else` 回退
4. ❌ 旧版单参数（早期 d.ts 审计误导） → ✅ 双参数 `(key, text)`

### 模式二：脱敏与错误友好化（推荐 SDK 集成用）

适用：跨工具统一规则（脱敏、截断、错误友好化）。**不依赖 UI，Web/Server 都能用**。

```ts
import type { ExtensionFactory, TextContent } from "@earendil-works/pi-coding-agent";

const SENSITIVE_PATTERNS = [
  /sk-[a-zA-Z0-9]{20,}/g,                 // OpenAI API key
  /AKIA[A-Z0-9]{16}/g,                    // AWS access key
  /gh[pousr]_[A-Za-z0-9]{36}/g,           // GitHub token
  /Bearer\s+[^\s]+/g,                     // Bearer token
];

const toolResultTransformExtension: ExtensionFactory = (pi) => {
  pi.on("tool_result", (event, _ctx) => {
    // ★ 必须 return 才生效（mutation 无效，见陷阱 1）
    // 步骤 1：找第一个 text content block
    const textBlock = event.content.find(
      (c): c is TextContent => c.type === "text",
    );
    if (!textBlock) return;

    let text = textBlock.text;
    let modified = false;

    // 步骤 2：错误友好化（isError=true 时改写为可操作的提示）
    if (event.isError) {
      const userFriendly = mapErrorToHint(event.toolName, text);
      if (userFriendly) {
        // ⚠️ 要构造**新的 content 数组**整体返回，不能只改 textBlock.text
        return {
          content: [
            { type: "text", text: userFriendly.hint },
          ],
          isError: false,                              // 改成非错误，让模型当成正常结果处理
          details: { originalError: text, ...event.details },  // 原错误保留到 details（不送 LLM）
        };
      }
    }

    // 步骤 3：敏感信息脱敏
    for (const pattern of SENSITIVE_PATTERNS) {
      if (pattern.test(text)) {
        pattern.lastIndex = 0;
        text = text.replace(pattern, "***REDACTED***");
        modified = true;
      }
    }

    // 步骤 4：为 read 工具添加行数统计
    if (event.toolName === "read") {
      const lineCount = text.split("\n").length;
      text = `[文件共 ${lineCount} 行]\n\n${text}`;
      modified = true;
    }

    // 步骤 5：只有修改了才返回新 content（不修改的 handler 不 return，避免无谓替换）
    if (modified) {
      // ⚠️ 必须**重新构造数组**，不能改原数组元素后 return 原 event.content
      // （虽然浅拷贝下两者引用相同，但语义上「构造新数组」更清晰，且避免陷阱 1 的误解）
      return {
        content: [{ type: "text", text }],
      };
    }
  });
};

function mapErrorToHint(toolName: string, errorText: string): { hint: string } | undefined {
  // 简单示例：按工具名 + 错误模式给出可操作提示
  if (toolName === "read" && errorText.includes("ENOENT")) {
    return { hint: "文件不存在，请用 ls 工具确认路径，或检查相对路径基准（cwd）" };
  }
  if (toolName === "bash" && errorText.includes("command not found")) {
    return { hint: "命令不存在，请用 which/whereis 确认是否安装" };
  }
  return undefined;
}

export default toolResultTransformExtension;
```

**关键修正点**（vs 旧版示例）：

1. ❌ 旧版 `event.content?.find(...).text = ...` mutation → ✅ 收集变换后 `return { content: [...] }`
2. ❌ 旧版错误处理直接 return 替换 content 但保留 isError=true → ✅ 同时覆盖 `isError: false` + 原错误存到 `details`（不送 LLM）
3. ✅ 添加 `if (modified)` 守卫避免无修改也触发替换

### 模式三：多 handler 链式 transform

适用：多个扩展各自负责一类变换（脱敏 / 截断 / 增强），按注册顺序串行累加修改。

```ts
// 扩展 A：脱敏
const redactExtension: ExtensionFactory = (pi) => {
  pi.on("tool_result", (event, ctx) => {
    const textBlock = event.content.find(c => c.type === "text") as TextContent | undefined;
    if (!textBlock) return;

    const redacted = textBlock.text.replace(/sk-[a-zA-Z0-9]+/g, "***");
    // 重新构造数组返回
    return {
      content: event.content.map(c =>
        c.type === "text" ? { ...c, text: redacted } : c
      ),
    };
  });
};

// 扩展 B：截断到 1000 字符
const truncateExtension: ExtensionFactory = (pi) => {
  pi.on("tool_result", (event, ctx) => {
    // 注意：这里看到的 event.content 是 extA 处理后的版本（runner merge 累积）
    const textBlock = event.content.find(c => c.type === "text") as TextContent | undefined;
    if (!textBlock || textBlock.text.length <= 1000) return;

    return {
      content: event.content.map(c =>
        c.type === "text" ? { ...c, text: c.text.slice(0, 1000) + "\n[...truncated]" } : c
      ),
    };
  });
};

// 注册顺序决定执行顺序
const config = {
  extensions: [redactExtension, truncateExtension],  // 先脱敏，再截断
};
```

**关键**：

- `emitToolResult` 按扩展数组顺序串行执行，每个 handler 看到的是前一个 handler **返回值合并后**的 `currentEvent`（runner.ts）
- mutation 不被检测——每个 handler 都必须 `return { content: [...] }` 才能让下游看到变化
- handler 抛异常被 try/catch + emitError 吞掉，不影响后续 handler

---

## 陷阱

### 陷阱 1（★ 最大坑）：mutation 无效，必须 return

**错误写法**（旧版 E06 示例就是这种）：

```ts
pi.on("tool_result", (event, ctx) => {
  const textContent = event.content?.find(c => c.type === "text");
  if (textContent) {
    textContent.text = textContent.text.replace(/sk-\w+/g, "***");  // ❌ 直接 mutate
  }
  // 没有 return
});
```

**为什么无效**：runner.ts 的 `emitToolResult` 只在 handler **返回** `content` / `details` / `isError` / `usage` 字段时设置 `modified=true`。mutation 不被检测，最终 `return undefined`，agent-session 拿到 undefined 后**完全不变**（agent-session.ts 的 `afterToolCall` 回调）。

**正确写法**：

```ts
pi.on("tool_result", (event, ctx) => {
  const textBlock = event.content.find(c => c.type === "text") as TextContent | undefined;
  if (!textBlock) return;

  const newText = textBlock.text.replace(/sk-\w+/g, "***");
  // ✅ 重新构造数组 + return
  return {
    content: event.content.map(c =>
      c === textBlock ? { ...c, text: newText } : c
    ),
  };
});
```

> ⚠️ **这与 `tool_call` 的 mutation 模式完全相反**（[E01 陷阱 2](E01-tool-intercept.md#陷阱 2)）：`tool_call` 的 `event.input` 是同一引用，跨 handler 透明 mutation 有效；但 `tool_result` 的 mutation **无效**。原因：emitToolCall 直接传 event 不拷贝（runner.ts 的 `emitToolCall`），emitToolResult 浅拷贝 + 只看返回值。

### 陷阱 2：`tool_execution_update` 没有 `progress` 字段

**错误写法**：

```ts
pi.on("tool_execution_update", (event, ctx) => {
  const progress = event.update?.progress;  // ❌ event 没有 update 字段，也没有 progress
  ctx.ui.setStatus("progress", `${progress}%`);
});
```

**为什么无效**：types.ts 的 `ToolExecutionUpdateEvent` 字段是 `partialResult`，结构由工具作者决定（一般是 `AgentToolResult` 形态 `{ content, details }`）——**没有标准化的 progress 字段**。

**正确写法**：从 `partialResult.content` 取文本；要进度百分比，需要工具作者自己把进度放进 `details`（自定义约定）。

### 陷阱 3：`ctx.ui.*` 默认是 no-op（mode 守卫）

**错误写法**：

```ts
pi.on("tool_execution_update", (event, ctx) => {
  ctx.ui.setStatus("streaming", `${event.toolName}...`);  // ❌ Web/Server 场景 no-op，没反馈
});
```

**为什么无效**：`createAgentSession` 默认 `mode="print"`，此时 `ctx.ui = noOpUIContext`（runner.ts 的 `noOpUIContext`），所有 `ctx.ui.*` 方法都是空实现。`setStatus` / `notify` / `confirm` 等**全部静默失败**。

**正确写法**：加 `hasUI` 守卫 + `else` 分支回退（参考模式一）：

```ts
if (ctx.hasUI) {
  ctx.ui.setStatus("streaming", text);
} else {
  console.log(`[streaming] ${event.toolName}: ${text}`);
}
```

> 与 [E01 陷阱 11](E01-tool-intercept.md#陷阱 11) / [E04 陷阱 1](E04-lifecycle-hooks.md#陷阱 1) / [E05 陷阱 3](E05-input-transform.md#陷阱 3) 完全一致——所有用到 `ctx.ui.*` 的地方都要加守卫。

### 陷阱 4：`tool_execution_update` handler 返回值被忽略

**错误写法**：

```ts
pi.on("tool_execution_update", (event, ctx) => {
  if (event.toolName === "bash") {
    return { suppress: true };  // ❌ 完全无效
  }
});
```

**为什么无效**：runner.ts 的通用 `emit` 对**非 session_before 事件**完全不读 handler 返回值。`suppress` / `cancel` / `skip` 等字段都是幻觉。

**正确写法**：handler 内部按 toolName 分支，不显示的工具直接 `return`（什么都不做）：

```ts
pi.on("tool_execution_update", (event, ctx) => {
  if (event.toolName === "bash") return;  // 不处理 bash 的流式
  // 处理其他工具...
});
```

### 陷阱 5：内置工具中只有 bash 真正发 `tool_execution_update`

**错误假设**：以为所有内置工具都会触发 `tool_execution_update`。

**实际情况**：见上方「内置工具 onUpdate 支持表」——**只有 bash 真正调用 onUpdate**，read / edit / write / grep / find / ls 的 `_onUpdate` 参数**从未被调用**（参数命名前缀 `_` 表明未用）。

**结论**：如果你的 `tool_execution_update` handler 想捕获 read 大文件的流式进度，**根本收不到事件**——要么改用 `tool_result` 事后处理，要么自定义工具包装 read 并在 execute 中调 onUpdate（见 [D01](D01-custom-tool.md#onupdate-流式更新回调)）。

### 陷阱 6（★ 集成坑）：`tool_result` 是 6 个扩展独有事件之一

`tool_result` **不在 `session.subscribe` 的事件流中**——在 server 层用 `session.subscribe("tool_result", ...)` 会**静默失败**（注册了但永远不被调用）。

**6 个扩展独有事件**：
1. `context` — 修改发给 LLM 的消息列表
2. `tool_call` — 工具执行前拦截（[E01](E01-tool-intercept.md)）
3. `tool_result` — 工具结果后处理（本场景）
4. `before_agent_start` — 改系统提示词
5. `input` — 变换用户输入（[E05](E05-input-transform.md)）
6. `model_select` — 模型选择

**证据**：agent-session.ts 的 `AgentSessionEvent` 联合类型 不含上述 6 个 type。

想做服务端工具结果后处理**必须走扩展**（在 `createAgentSession({ extensions: [...] })` 中注册 handler）。

### 陷阱 7：`tool_execution_*` 不是扩展独有（与 tool_result 相反）

`tool_execution_start` / `tool_execution_update` / `tool_execution_end` **两层都派发**：

- 扩展层：`pi.on("tool_execution_update", ...)` ✅ 能收到
- 外部层：`session.subscribe(event => { if (event.type === "tool_execution_update") ... })` ✅ 也能收到

但两层的 **handler 返回值都被忽略**（都用通用 `emit`）。

**证据**：agent-session.ts 的 `_emitExtensionEvent` 转发逻辑 中 agent-session 接收 agent-core 发的 tool_execution_* 事件后转发到扩展层；同时 `_emit(event)` 也派发到 session.subscribe 监听器。

### 陷阱 8：handler 异常被吞，不影响后续

`emitToolResult` 对每个 handler 用 try/catch 包裹（runner.ts 的 `emitToolResult`），抛异常时调 `emitError`（不是再抛）。**意味着**：

- 一个扩展的 bug **不会中断**整个 agent loop
- 但你会看到 `extension_error` 事件（可订阅监听）
- 异常 handler 的返回值**丢失**（return 的 content 不生效）

调试时如果发现「handler 跑了但内容没变」，检查 console / extension_error 事件——可能 handler 抛了异常被静默吞掉。

---

## 横向联动

- [E01 拦截与修改工具调用](E01-tool-intercept.md) — `tool_call` 事件（执行前），mutation 有效（与 `tool_result` 完全相反）
- [E02 扩展基础](E02-extension-basics.md) — ExtensionFactory / ExtensionMode / ExtensionCommandContext
- [E04 生命周期钩子](E04-lifecycle-hooks.md) — ExtensionContext 字段表、ctx.ui 守卫
- [E05 用户输入变换](E05-input-transform.md) — `input` 事件（6 个扩展独有事件之一），mutation 也无效
- [D01 自定义工具](D01-custom-tool.md) — `onUpdate` 回调签名、如何在自定义工具中实现流式
- [D05 自定义工具输出渲染](D05-tool-result-render.md) — `defineTool({ renderResult })` 仅 TUI 生效，与 `tool_result` 钩子互补
- [04-events 事件系统](../sdk_doc/04-events.md) — 6 个扩展独有事件清单、派发对比
- [07-extensions-api 扩展 API](../sdk_doc/07-extensions-api.md) — `pi.on` 注册 API 总览
