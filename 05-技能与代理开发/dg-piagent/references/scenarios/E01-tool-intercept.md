# 场景：拦截与修改工具调用 (E01)

## 什么时候用 / 不用会怎样

**该用本场景**：

- **静默改写参数**：模型生成的工具参数不完美，你想加默认值（`write` 强制 utf-8 编码）、补全路径（相对路径补 `cwd`）、注入保护标志（`rm` 加 `-i`），且**不想让 Agent 知道发生了修改**
- **硬阻断非法调用**：违反业务规则的调用（如访问未授权资源、调用未上线工具），直接拒绝并告诉 Agent 原因
- **多租户 / 生产集成**：服务端集成 Agent 时，所有工具调用要过一道业务校验，拦截器是唯一的关卡（订阅式 `session.subscribe` 在 server 层**收不到** `tool_call`，见下方集成坑）
- **审计 / 合规**：拦截器同时承担"准入 + 日志"两重职责，比 `tool_result` 事后审计更早介入

**不用会怎样**：

- 模型自由发挥工具参数——`rm` 不加保护标志、`write` 用系统默认编码、相对路径在不同 cwd 下解析出意外文件
- 业务规则只在 prompt 里写"不要做 X"——prompt 注入或模型理解偏差时直接绕过
- 想事后审计只能用 `tool_result`（已经执行了），破坏已经发生

**不适合本场景**：

- 完全的路径白名单（结构化的越权防护）→ 见 [D06 限制工具执行目录](D06-protected-paths.md)
- 危险命令模式匹配 + 用户确认弹框（`rm -rf` / `git push --force` 这类）→ 见 [D04 工具调用安全闸门](D04-confirm-destructive.md)
- 修改工具**返回结果**（如脱敏文件内容）→ 见 [D05 自定义工具输出渲染](D05-tool-result-render.md) / [E06 流式处理工具输出](E06-streaming-transform.md)
- **完全替换 bash 执行逻辑**（如自己实现一个沙箱 runner）→ 不是 `tool_call` 的能力，要走 `user_bash` 事件返回 `BashOperations`（types.ts）

---

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `pi.on("tool_call", handler)` | 在工具执行前拦截调用，可阻断或改参 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `return { block: true, reason }` | 阻断工具执行，reason 会回到模型上下文 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `event.input` 直接 mutation | 改字段值（不重新校验 schema），工具收到改后的参数 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |

> ⚠️ **关键集成坑 1：`tool_call` 是扩展独有事件之一**——与拦截/上下文直接相关的扩展独有事件共 6 个：`context` / `tool_call` / `tool_result` / `before_agent_start` / `input` / `model_select`（另有 `before_provider_*` / `user_bash` 等扩展独有事件，见 [04-events.md](../sdk_doc/04-events.md) 完整分类）。在 server 层用 `session.subscribe("tool_call", ...)` 会**静默失败**——外部事件流（`AgentSessionEvent`，types.ts）根本不派发这个 type，handler 注册了但永远不被调用。想做服务端工具拦截**必须走扩展**。详见 [04-events.md 坑 4](../sdk_doc/04-events.md#坑-46-个扩展独有事件sessionsubscribe-静默收不到-最大集成坑)。

> ⚠️ **关键集成坑 2：`ctx.ui.confirm` 是 CLI 专属**。如果你想在拦截时弹框让用户选择（而非硬阻断），`ctx.ui.confirm` 在 `createAgentSession` 默认 `mode="print"` 下 `hasUI=false`，调用会 no-op 返回 `false`——见 runner.ts 的 `noOpUIContext`。Web/SDK 场景请用硬阻断或自定义 RPC 通道。详见 [D04 三级策略](D04-confirm-destructive.md#三级策略详解)。

---

## 默认行为（★ 必读）

**工具启用规则**（与 [A04](A04-tool-whitelist.md) / [D01](D01-custom-tool.md) / [D02](D02-dynamic-tools.md) / [D04](D04-confirm-destructive.md) / [D05](D05-tool-result-render.md) / [D06](D06-protected-paths.md) 一致）：

| `createAgentSession` 配置 | 内置工具 | 扩展工具 | customTools |
|--------------------------|---------|---------|-------------|
| 不传 `tools`（默认） | 启用 `["read", "bash", "edit", "write"]` | 自动启用 | 自动启用 |
| `tools: ["read", "bash"]` | 仅 read + bash | **必须列入才启用** | **必须列入才启用** |
| `noTools: "all"` | 全禁用 | 全禁用 | 全禁用 |
| `noTools: "builtin"` | 全禁用 | 自动启用 | 自动启用 |
| `excludeTools: ["bash"]` | bash 禁用，其他启用 | 自动启用（除非在 exclude 中） | 自动启用（除非在 exclude 中） |

**关键事实**：

1. `find` / `grep` / `ls` **不是默认启用**——它们是扩展工具，必须显式 `tools: [...]` 列入才会暴露给模型（agent-session.ts）
2. 一旦工具被启用，所有注册的 `tool_call` handler 都会拦截它——**不能"只拦 read 不拦 write"**（除非 handler 内部按 `event.toolName` 分发）
3. `tool_call` handler 在工具**真正执行前**触发，阻断后工具不会运行；handler 在**模型流式输出 tool call 后**触发，所以阻断会消耗一次模型 turn（阻断 reason 进入对话，模型会看到"该操作被禁止"并自行调整）

**证据**：agent-session.ts（`isAllowedTool` 过滤）、agent-session.ts（`beforeToolCall` 钩子注入点，emitToolCall 在此调用）。

---

## 内置工具字段对照表（★ 必读）

拦截 handler 要从 `event.input` 取字段，**字段名错了就完全失效**——这是 D04/D06 横向审计反复发现的 P0 高发区。

| 工具名 | `event.toolName` | `event.input` 关键字段 | 含义 | 证据 |
|--------|------------------|----------------------|------|------|
| bash | `"bash"` | `command: string`（必填）/ `timeout?: number` | shell 命令字符串（**不是路径**） | bash.ts |
| read | `"read"` | `path: string`（必填）/ `offset?` / `limit?` | 要读的文件路径 | read.ts |
| write | `"write"` | `path: string`（必填）/ `content: string`（必填） | 要写的文件路径 + 内容 | write.ts |
| edit | `"edit"` | `path: string`（必填）/ `edits: Array<{ oldText, newText }>` | 要编辑的文件 + 替换列表 | edit.ts |
| grep | `"grep"` | `pattern: string`（必填，搜索内容）/ `path?` / `glob?` / `ignoreCase?` / `literal?` / `context?` / `limit?` | pattern 是**搜索内容**不是路径 | grep.ts |
| find | `"find"` | `pattern: string`（必填，glob 模式）/ `path?` / `limit?` | pattern 是**glob 模式**（如 `*.ts`）不是路径 | find.ts |
| ls | `"ls"` | `path?: string` / `limit?` | 要列出的目录 | ls.ts |
| 自定义工具 | `string` | `Record<string, unknown>` | 由 `registerTool` 的 schema 决定 | types.ts |

> ⚠️ **write 没有 `encoding` 字段**。早期 skill 文档曾建议 `event.input.encoding = "utf-8"`——查 write.ts 真实 schema 只有 `path` + `content`，内部固定用 utf-8 写入（write.ts `fsWriteFile(path, content, "utf-8")`）。mutation 设置 `encoding` 字段**没有任何效果**，会被工具忽略。如果想改编码只能完全重写工具。

> ⚠️ **read 字段是 `path` 不是 `file_path`**。`file_path` 是 edit.ts 渲染层的兼容别名（edit.ts `RenderableEditArgs`），不在 schema 中。模型按 schema 严格生成 `path`，拦截 handler 读 `event.input.file_path` 会拿到 `undefined`。

---

## ToolCallEvent 字段

每个 `tool_call` handler 收到的 `event` 对象（types.ts）：

| 字段 | 类型 | 说明 |
|------|------|------|
| `type` | `"tool_call"` | 事件类型标识 |
| `toolCallId` | `string` | 唯一调用 ID，可用来关联 `tool_result` |
| `toolName` | `"bash" \| "read" \| "edit" \| "write" \| "grep" \| "find" \| "ls" \| string` | 工具名；自定义工具是任意 string |
| `input` | 强类型工具 input | 按 toolName 窄化（如 `BashToolInput` 含 `command/timeout`） |

`ExtensionContext`（handler 第二个参数 `ctx`）关键字段（types.ts）：

| 字段 | 类型 | 说明 |
|------|------|------|
| `ctx.cwd` | `string` | 当前 session 的工作目录，所有内置工具用它解析相对路径 |
| `ctx.ui` | `ExtensionUIContext` | CLI 专属 UI 接口（`hasUI=false` 时为 no-op） |
| `ctx.hasUI` | `boolean` | 是否有可交互 UI（TUI/RPC 模式 true，print/json 模式 false） |
| `ctx.mode` | `"tui" \| "rpc" \| "json" \| "print"` | 当前运行模式 |
| `ctx.sessionManager` | `ReadonlySessionManager` | 只读会话管理 |
| `ctx.abort()` | - | 中止当前 agent 操作 |

> 注意：`ExtensionContext` **没有 `session` 字段**（D01 已确认）。想操作 session 要走 `ctx.sessionManager`。

---

## handler 签名与返回值

```ts
pi.on("tool_call", (event: ToolCallEvent, ctx: ExtensionContext) => {
  // 1. 阻断：返回 { block: true, reason }，工具不会执行，reason 返给 Agent
  if (dangerous) {
    return { block: true, reason: "操作不允许：..." };
  }

  // 2. 修改参数：直接 mutate event.input 的字段（不要替换整个对象）
  event.input.command = modifiedCmd;

  // 3. 条件放行：return 不返回 block 字段（或 return undefined）
  // 后续 handler 会被继续调用
});
```

**返回类型**：`ToolCallEventResult = { block?: boolean, reason?: string }`——**只有这两个字段**（证据：types.ts）。

| 返回值 | 行为 |
|--------|------|
| `{ block: true, reason }` | 工具不执行，reason 进入对话作为 tool error |
| `{ block: true }`（不带 reason） | 工具不执行，Agent 收到默认错误 |
| `undefined` / `{}` / `{ block: false }` | 继续给后续 handler |
| 返回其他字段（如 `{ allow: true }` / `{ mutate: ... }`） | **无效**，被忽略 |

**改参数走 mutation，不走返回值**。源码注释（types.ts）明确："To modify arguments, mutate `event.input` in place instead."

> ⚠️ 与 `tool_result` 的返回值机制不同：`tool_result` 走返回值 merge（返回 `{ content, details, isError, usage }` 替换字段，含 token 用量修改）。两者不要混淆。

---

## emitToolCall 执行机制（★ 多 handler 必读）

`emitToolCall` 在 runner.ts 实现，是 E01 拦截器能否正确工作的核心：

```ts
async emitToolCall(event: ToolCallEvent): Promise<ToolCallEventResult | undefined> {
  const ctx = this.createContext();
  let result: ToolCallEventResult | undefined;

  for (const ext of this.extensions) {
    const handlers = ext.handlers.get("tool_call");
    if (!handlers || handlers.length === 0) continue;

    for (const handler of handlers) {
      const handlerResult = await handler(event, ctx);  // 串行 await

      if (handlerResult) {
        result = handlerResult as ToolCallEventResult;
        if (result.block) {
          return result;  // ← 遇 block 立即短路
        }
      }
    }
  }
  return result;
}
```

**四个关键特性**：

1. **串行执行**：handler 按 `extensionFactories` 数组顺序 + 每个扩展内 handler 注册顺序依次 `await`
2. **block 短路**：任一 handler 返回 `{ block: true }` 立即 `return`，后续 handler 不再调用（与 `tool_result` 不同，后者没有短路）
3. **mutation 跨 handler 透明**：前面 handler 改了 `event.input.command`，后面的 handler 看到的就是改后的值（因为传的是同一引用）
4. **handler 内部没有 try/catch**：runner 自己不捕获 handler 异常——但外层 agent-session.ts 的 `beforeToolCall` 钩子用 try/catch 包裹了 `emitToolCall` 调用，异常会冒泡为"Extension failed, blocking execution"，**等同于 block**

**与 `emitToolResult` 的关键区别**：

| 特性 | `emitToolCall`（拦截器） | `emitToolResult`（结果改写） |
|------|------------------------|---------------------------|
| 触发时机 | 工具执行**前** | 工具执行**后** |
| 能否阻断 | ✅ 返回 `{ block: true }` | ❌ 不能 block（已经执行完了） |
| 改内容方式 | mutate `event.input`（不返回） | 返回 `{ content?, details?, isError? }` merge |
| 多 handler 短路 | ✅ block 短路 | ❌ 不短路，所有 handler 都跑完，结果累积 merge |
| handler 异常 | runner 不捕获，外层捕获→等同 block | runner 有 try/catch（runner.ts） |
| 典型用途 | 准入控制 / 改参数 / 路径校验 | 脱敏 / 加水印 / 错误重试 |

证据：runner.ts（emitToolCall）、runner.ts（emitToolResult，串行 + merge 累积 + try/catch + emitError）。

> ⚠️ **别把 `tool_call` 和 `tool_execution_*` 混了**：两者都"关于工具"但分属不同层、不同能力。
>
> | 事件 | 层 | 时机 | 能否阻断/改参 | 哪层收得到 |
> |---|---|---|---|---|
> | `tool_call`（本场景主角） | 扩展独有 | 工具执行**前** | ✅ block / mutate `event.input` | 仅 `pi.on("tool_call")`，subscribe 收不到 |
> | `tool_execution_start` / `update` / `end` | agent-core 生命周期 | 执行**过程中** / **后** | ❌ 只读观测（拿 `toolCallId` / `toolName` / `result`） | `pi.on` 和 `session.subscribe` **都**收得到 |
>
> 即：想拦截/改参只能用 `tool_call`；只想观测执行耗时/结果，用 `tool_execution_*` 更简单且 subscribe 层也能用。证据：`AgentEvent` 含 `tool_execution_*`（agent-core types.ts），`ExtensionEvent` 额外含 `ToolCallEvent`（coding-agent types.ts）。

---

## 核心代码

三级递进策略：**硬阻断（block）→ 静默修改（mutation）→ 多工具分发**。

```ts
export default (pi) => {
  pi.on("tool_call", (event, ctx) => {
    // --- 1. bash：危险命令模式匹配 + 保护标志注入 ---
    if (event.toolName === "bash") {
      const cmd = event.input.command || "";

      // 硬阻断：绝对禁止的操作
      if (/\brm\s+-rf\s+\//.test(cmd)) {
        return { block: true, reason: "禁止递归删除根目录" };
      }
      if (/\bgit\s+push\s+--force\b/.test(cmd)) {
        return { block: true, reason: "禁止 force push（如需覆盖远程，请走 rebase 流程）" };
      }

      // 静默修改：为裸 rm 加 -i 确认标志（模型不知情）
      // 注意：只有在 cmd 包含独立 rm 且没有 -i flag 时才加
      if (/\brm\b/.test(cmd) && !/(^|\s)-[a-z]*i/.test(cmd)) {
        event.input.command = cmd.replace(/\brm\b/, "rm -i");
      }
    }

    // --- 2. write：路径越权防护（注意字段是 path 不是 file_path） ---
    if (event.toolName === "write") {
      const target = event.input.path;  // ← 字段必须是 path
      if (target?.startsWith("/etc/") || target?.startsWith("/usr/")) {
        return { block: true, reason: `不允许写入系统目录：${target}` };
      }
      // ⚠️ write 没有 encoding 字段，下面这行是无效的（早期文档误导）
      // event.input.encoding = event.input.encoding || "utf-8";
    }

    // --- 3. read：相对路径越权防护 ---
    if (event.toolName === "read") {
      const target = event.input.path;  // ← 字段必须是 path
      if (target?.includes("..")) {
        return { block: true, reason: "禁止路径遍历（..）" };
      }
    }

    // --- 4. 自定义工具：业务规则校验 ---
    if (event.toolName === "execute_sql") {
      const sql = (event.input.sql as string) || "";
      if (/DROP\s+TABLE/i.test(sql)) {
        return { block: true, reason: "禁止 DROP TABLE（生产数据库保护）" };
      }
    }

    // 其他工具：不返回 block，自动放行给后续 handler
  });
};
```

> **代码块之间有呼吸**：上方示例展示了硬阻断 + 静默修改 + 多工具分发三类模式。下面分别展开。

---

## 三种模式详解

### 一级：硬阻断（block）

最简单也最安全——返回 `{ block: true, reason }`，工具不执行，Agent 收到 reason 作为 tool error。

**适用场景**：绝对禁止的操作（写系统目录、未授权路径、危险 SQL、调用未上线工具）。

**Agent 行为**：Agent 看到 reason 后**可能重试**（如换路径、换命令）。如果想彻底阻止某类意图，需要把 reason 写清楚（如"请改用 --dry-run 模式"引导模型走正确路径）。

**消耗一次 turn**：block 后模型会基于 reason 决定下一步——可能放弃、可能换方法、可能误判后再次尝试。频繁 block 会拉长对话。

### 二级：静默修改（mutation）

直接修改 `event.input` 的字段值，工具收到的是修改后的参数。Agent **不知道发生了修改**。

**适用场景**：

- 为危险命令加保护标志（`rm` → `rm -i`）
- 强制使用安全默认值（write 固定 utf-8——⚠️ 但 write 没有 encoding 字段，这条作废）
- 路径规范化（相对路径补全为绝对路径——但要注意与内置工具的 `resolveToCwd` 一致，见下方陷阱 1）
- 给自定义工具补默认参数（如 `execute_sql` 加 `readonly: true`）

**注意事项**：

- **直接修改字段值，不要替换整个 `event.input` 对象**——后续 handler 看到的引用会丢失
- mutation **跨 handler 透明**：扩展 A 改了 `command`，扩展 B 看到的就是改后的值
- **不重新做 schema 校验**（types.ts 注释明确："No re-validation is performed after mutation"）——你设置的非法字段不会被拒绝，但工具执行时可能报错

### 三级：多工具按 `toolName` 分发

一个 handler 处理所有工具时，用 `event.toolName === "xxx"` 分支。如果想结构化，可以抽 helper：

```ts
const handlers: Record<string, (input: any, ctx: ExtensionContext) => ToolCallEventResult | void> = {
  bash: (input, _ctx) => {
    if (/\brm\s+-rf\s+\//.test(input.command)) {
      return { block: true, reason: "禁止 rm -rf /" };
    }
  },
  write: (input, _ctx) => {
    if (input.path?.startsWith("/etc/")) {
      return { block: true, reason: "禁止写系统目录" };
    }
  },
  // 自定义工具也走同一套
  execute_sql: (input, _ctx) => {
    if (/DROP\s+TABLE/i.test(input.sql)) {
      return { block: true, reason: "禁止 DROP TABLE" };
    }
  },
};

export default (pi) => {
  pi.on("tool_call", (event, ctx) => {
    const handler = handlers[event.toolName];
    return handler?.(event.input, ctx);
  });
};
```

> 注意：**不能"只拦某些工具"**——`pi.on("tool_call")` 注册一次就对所有启用工具生效。想做选择性拦截只能在 handler 内部按 `event.toolName` 分发（如上）。

> **TS 类型提示**：`event.toolName === "bash"` 可以工作但不会自动窄化 `event.input` 的类型。SDK 提供了类型守卫 `isToolCallEventType("bash", event)`（types.ts），调用后 TS 会把 `event` 收窄为 `BashToolCallEvent`，`event.input` 自动获得 `command` / `timeout` 字段的智能提示。签名：`isToolCallEventType(toolName: string, event: ToolCallEvent): event is XxxToolCallEvent`，内置工具名（`"bash"` / `"read"` / `"edit"` / `"write"` / `"grep"` / `"find"` / `"ls"`）有重载，自定义工具用泛型：`isToolCallEventType<"my_tool", MyInput>("my_tool", event)`。

---

## 关键陷阱（12 条）

### 陷阱 1：`process.cwd()` vs `ctx.cwd`

SDK 集成场景下扩展进程的 `process.cwd()` 不等于 session 的 `cwd`。`createAgentSession({ cwd })` 是必填配置，拦截器内取工作目录必须用 `ctx.cwd`（types.ts）。

更隐蔽的坑：内置工具内部用 `resolveToCwd`（path-utils.ts），里面调 `normalizePath({ expandTilde: true, stripAtPrefix: true })`——会展开 `~` 到 `os.homedir()`（paths.ts），剥离 `@` 前缀。拦截器里裸 `path.resolve(ctx.cwd, value)` **不展开 `~`**——意味着模型传 `~/.ssh/id_rsa` 在拦截器看来是 `<cwd>/.ssh/id_rsa`（在白名单内），但实际工具执行时会被解析为 home dir（越权）。修复需要复用 `resolveToCwd` 或手动展开 `~`。详见 [D06 陷阱 1](D06-protected-paths.md)。

### 陷阱 2：`startsWith` 不加 `path.sep`

```ts
// ❌ 错误：/usr-other-project 会被拦截
if (target.startsWith("/usr")) { ... }

// ✅ 正确
if (target.startsWith("/usr/")) { ... }
```

### 陷阱 3：mutation 后不 normalize 路径

模型可能传 `/etc/./passwd`、`/etc/../etc/passwd`、`//etc/passwd` 等等价路径。裸字符串匹配会被绕过。需要先 `path.normalize` / `path.resolve` 再比对。

### 陷阱 4：`event.input.encoding = "utf-8"` 是无效的

write 工具 schema 没有 encoding 字段（write.ts），内部固定用 utf-8。mutation 设置 encoding 会被忽略。早期 skill 文档的示例曾这样写——是幻觉。

### 陷阱 5：`event.input.file_path` 不存在

read/write/edit 的字段是 **`path`**，不是 `file_path`。`file_path` 是 edit.ts 渲染层兼容别名（edit.ts），不在 schema。读 `event.input.file_path` 拿 `undefined`。D04/D06 横向提示的 P0 高发区。

### 陷阱 6：bash 不能用字符串正则拦

`rm -rf /` 的变体有无数：`r""m`、`rm -r -f /`、`$(rm -rf /)`、变量替换 `${cmd}`、base64 解码执行 `echo "cm0gLXJmIC8=" | base64 -d | sh` 等。字符串正则极易绕过。

**正解**：

- 用 `shell-quote` 等 parser 拆 token
- 用沙箱 / 容器 / BashOperations.exec 注入（bash.ts）
- 完全禁用 bash 工具（`excludeTools: ["bash"]`）
- 见 [D04 bash 命令拦截正解](D04-confirm-destructive.md)

### 陷阱 7：`tool_call` 是扩展独有事件（再强调）

server 层 `session.subscribe("tool_call", ...)` **静默失败**。server 层想做工具拦截**必须走扩展**，没有第二条路。与拦截/上下文直接相关的 6 个扩展独有事件：`context` / `tool_call` / `tool_result` / `before_agent_start` / `input` / `model_select`（subscribe 层的 `AgentSessionEvent` 收不到这几个 type；其他扩展独有事件见 [04-events.md](../sdk_doc/04-events.md) 完整分类）。详见 [04-events.md 坑 4](../sdk_doc/04-events.md#坑-46-个扩展独有事件sessionsubscribe-静默收不到-最大集成坑)。

### 陷阱 8：多 handler block 短路

如果扩展 A 和扩展 B 都注册了 `tool_call` handler：

- A 返回 `{ block: true }` → B 不会被调用
- A 返回 `undefined` → B 继续跑，看到 A 的 mutation
- A 抛异常 → 外层捕获（等同 block），B 不会被调用

设计多个拦截器时要注意顺序：硬规则放前面，软修改放后面。

### 陷阱 9：handler 抛异常会中断 agent loop

`emitToolCall` 内部没有 try/catch（runner.ts），异常冒泡到外层 `beforeToolCall`（agent-session.ts）后被捕获并转成 "Extension failed, blocking execution" 错误——**等同于 block**，但 reason 不可控。

最佳实践：handler 内部自己 try/catch，把异常转成明确的 `{ block: true, reason: "..." }`。

### 陷阱 10：替换 bash 执行逻辑不是 tool_call 的能力

想完全替换 bash 的执行方式（如自己实现一个 Docker 沙箱 runner），应该用 `user_bash` 事件返回 `BashOperations`（types.ts），不是在 `tool_call` 里拦。`tool_call` 只能在执行前后做拦截/改参，**不能替换执行本身**。

### 陷阱 11：`ctx.ui.confirm` 在 Web/SDK 场景 no-op

`hasUI=false`（print/json 模式，SDK 集成默认）下 `ctx.ui.confirm` 直接返回 false（runner.ts）。想在拦截时让用户确认要走 RPC 自定义通道或硬阻断。详见 [D04 ctx.ui.confirm](D04-confirm-destructive.md#三级策略详解)。

### 陷阱 12：mutation 不重新校验 schema

你设置任何字段都会被保留，**但 mutation 发生在 schema 校验之"后"**——执行顺序是 `prepareToolCallArguments` → `validateToolArguments` → `beforeToolCall`（即 emitToolCall，agent-loop.ts），所以 schema 不会替你拦截坏值。例如把 `event.input.timeout` 改成字符串 `"30"`，schema 校验阶段已经跑完、不会报错，错误要到**工具真正执行时**才暴露（运行时类型不匹配抛异常，外层转成 tool error 消耗一次 turn）。改之前务必先查工具的 schema 定义，自己保证类型正确。

---

## 变体与延伸

### 变体 A：与 D04 配合做"双重闸门"

D04 管**命令危险性**（`rm -rf` / `git push --force` / `sudo`），E01 管**业务规则**（写未授权路径 / 调用未上线工具 / 危险 SQL）。两个 handler 串行注册：

```ts
// extension A: D04 风格的危险命令匹配
pi.on("tool_call", (event, ctx) => {
  if (event.toolName === "bash" && /\brm\s+-rf\b/.test(event.input.command)) {
    return { block: true, reason: "危险命令已阻止" };
  }
});

// extension B: E01 风格的业务规则
pi.on("tool_call", (event, ctx) => {
  if (event.toolName === "execute_sql" && /DROP/i.test(event.input.sql)) {
    return { block: true, reason: "禁止 DROP" };
  }
});
```

按 [陷阱 8](#陷阱-8多-handler-block-短路) 的执行顺序，A 阻断时 B 不跑。详见 [D04](D04-confirm-destructive.md)。

### 变体 B：与 D06 配合做"路径白名单"

D06 是 E01 在路径维度的强化版——D06 提供结构化的白名单工厂（`createProtectedPathsExtension(options)`），E01 提供单点改写。两者可以叠加：D06 管路径越权、E01 管参数默认值。详见 [D06](D06-protected-paths.md)。

### 变体 C：与 tool_result 配合做"准入 + 审计"

`tool_call` 在执行前拦截（准入），`tool_result` 在执行后改写（审计）。两个事件配合可以覆盖完整链路：

```ts
pi.on("tool_call", (event, ctx) => {
  // 准入：拒绝非法调用
  if (isForbidden(event)) {
    return { block: true, reason: "禁止" };
  }
  // 记准入日志
  logToolCall(event);
});

pi.on("tool_result", (event, ctx) => {
  // 审计：记录实际执行结果
  logToolResult(event);
  // 脱敏：把返回内容里的 token 替换掉
  return { content: sanitizeContent(event.content) };
});
```

注意：`tool_result` 也是扩展独有事件，subscribe 层收不到。详见 [D05](D05-tool-result-render.md) / [E06](E06-streaming-transform.md)。

### 变体 D：自定义工具的字段校验

`customTools` 注册的工具也会触发 `tool_call`（agent-session.ts）。可以在 schema 校验之外再加一道业务校验：

```ts
pi.on("tool_call", (event, ctx) => {
  if (event.toolName === "send_email") {
    const to = event.input.to as string;
    if (!isAllowedDomain(to)) {
      return { block: true, reason: `禁止发往域 ${domain}（白名单外）` };
    }
  }
});
```

详见 [D01 自定义工具](D01-custom-tool.md)。

### 变体 E：find/grep 的搜索范围限制

find/grep 的 `path` 字段也要校验（否则模型可能传 `/etc` / `/root` 探测系统目录）。注意 `pattern` 字段不是路径——拦了模型连"找所有 ts 文件"都做不了。详见 [D06 变体 B](D06-protected-paths.md)。

### 变体 F：基于 session 状态的动态规则

通过 `session_start` 事件初始化规则，在 `tool_call` 里使用闭包变量：

```ts
// ⚠️ userId 等业务身份信息必须从扩展闭包外注入，不能从事件取
// （SessionStartEvent 只有 type/reason/previousSessionFile，不携带用户身份）
const userId = getCurrentUserId();  // 从扩展模块作用域 / 外部配置 / 包一层工厂函数传入

export default (pi) => {
  let rules: Rule[] = [];

  pi.on("session_start", (event, ctx) => {
    // 根据 session 启动 reason 决定是否加载规则
    // （event.reason: "startup" | "reload" | "new" | "resume" | "fork"）
    rules = loadRulesForUser(userId);
  });

  pi.on("tool_call", (event, ctx) => {
    for (const rule of rules) {
      const result = rule.check(event);
      if (result?.block) return result;
    }
  });
};
```

> ⚠️ **常见幻觉**：早期版本曾写 `loadRulesForUser(event.metadata?.userId)`——但 `SessionStartEvent` **没有 `metadata` 字段**（types.ts：仅 `type` / `reason` / `previousSessionFile?`），`event.metadata` 永远是 `undefined`，规则会被静默加载为空。多租户的用户身份、租户 ID 等**业务上下文必须通过扩展工厂闭包外注入**（如从模块作用域变量、配置文件、或包一层工厂函数传入），`ExtensionContext`（types.ts）也只有 `sessionManager` / `cwd` / `ui` / `hasUI` / `mode` / `abort()`，同样不含用户身份。

详见 [E04 生命周期钩子](E04-lifecycle-hooks.md)。

---

## 横向联动

- [A04 工具白名单](A04-tool-whitelist.md)：`tools` / `excludeTools` / `noTools` 配置层过滤
- [A06 加载扩展](A06-load-extensions.md)：扩展加载机制（默认行为的源头）
- [D01 自定义工具](D01-custom-tool.md)：customTools 也会被 `tool_call` 拦截
- [D04 工具调用安全闸门](D04-confirm-destructive.md)：危险命令匹配 + 用户确认弹框
- [D05 自定义工具输出渲染](D05-tool-result-render.md)：执行后改返回结果
- [D06 限制工具执行目录](D06-protected-paths.md)：结构化路径白名单
- [E02 扩展基础](E02-extension-basics.md)：扩展骨架
- [E04 生命周期钩子](E04-lifecycle-hooks.md)：`session_start` 等钩子
- [E06 流式处理工具输出](E06-streaming-transform.md)：流式改写工具过程
- [sdk_doc/04-events.md](../sdk_doc/04-events.md)：事件系统（含扩展独有事件警示）
- [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md)：ExtensionAPI 完整接口
