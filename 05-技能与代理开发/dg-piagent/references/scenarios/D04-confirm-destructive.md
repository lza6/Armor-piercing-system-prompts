# 场景：工具调用安全闸门 (D04)

## 什么时候用 / 不用会怎样

**该用本场景**：

- **生产环境防误操作**：Agent 自动跑 `rm -rf` / `git push --force` / 写 `/etc/` 这类破坏性命令前要拦下
- **多租户 / 共享环境**：不同用户的 prompt 跑在同一台机器，需要确保路径不越权
- **审计合规**：所有工具调用前后要留日志，满足安全合规要求
- **自定义工具的输入校验**：业务工具（如 `execute_sql`）的 schema 校验之外再加一道防线

**不用会怎样**：

- Agent 自由执行模型生成的 bash/write/edit——一旦 prompt 表述模糊或模型误判，可能造成不可逆破坏（递归删、覆盖系统文件、推送敏感数据）
- 自定义工具收到非法参数时直接执行，业务边界完全依赖 schema

**不适合本场景**：

- 需要路径白名单（更结构化的越权防护）→ 见 [D06 限制工具执行目录](D06-protected-paths.md)
- 只修改工具参数（如加默认值）而非阻断 → 见 [E01 拦截与修改工具调用](E01-tool-intercept.md)
- 仅记录日志不做拦截 → 见 [E06 流式处理工具输出](E06-streaming-transform.md)

---

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `pi.on("tool_call")` | 拦截工具调用事件，可在执行前修改参数或阻止 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `return { block: true, reason }` | 阻止工具执行，返回原因给 Agent | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `event.input` mutation | 直接修改 `event.input` 中的字段，改变工具收到的参数 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `ctx.ui.confirm(title, message, opts?)` | 弹出确认对话框（**TUI 和 RPC 模式可用**；print/json 模式见下方陷阱） | 终端 UI API（TUI/RPC 可用，本 skill 未收录，查 SDK 源码） |

> ⚠️ **关键集成坑**：`tool_call` 是**扩展独有事件**之一（典型代表：`context` / `tool_call` / `tool_result` / `before_agent_start` / `input` / `model_select` 等——非穷举，完整清单见 [04-events.md](../sdk_doc/04-events.md)；凡是 `session.subscribe` 收不到的事件都属于此类）。在 server 层用 `session.subscribe` 监听 `tool_call` 会**静默失败**——handler 被调用但 type 分支永不命中。想做服务端拦截**必须走扩展**。详见 [04-events.md 坑 4](../sdk_doc/04-events.md#坑-46-个扩展独有事件sessionsubscribe-静默收不到-最大集成坑)。

---

## 默认行为（★ 必读）

**工具启用规则**（与 [A04](A04-tool-whitelist.md) / [D01](D01-custom-tool.md) / [D02](D02-dynamic-tools.md) 一致）：

| `createAgentSession` 配置 | 内置工具 | 扩展工具 | customTools |
|--------------------------|---------|---------|-------------|
| 不传 `tools`（默认） | 启用 `["read", "bash", "edit", "write"]` | 自动启用 | 自动启用 |
| `tools: ["read", "bash"]` | 仅 read + bash | **必须列入才启用** | **必须列入才启用** |
| `noTools: "all"` | 全禁用 | 全禁用 | 全禁用 |
| `noTools: "builtin"` | 全禁用 | 自动启用 | 自动启用 |
| `excludeTools: ["bash"]` | bash 禁用，其他启用 | 自动启用（除非在 exclude 中） | 自动启用（除非在 exclude 中） |

**关键事实**：

- `tool_call` 拦截器对**所有已启用的工具**生效——无论内置、扩展注册、还是 customTools。不需要在 `tools` 数组里额外登记拦截器
- 拦截器只对**实际会被调用的工具**触发；被 `tools` / `excludeTools` / `noTools` 禁用的工具不会触发 `tool_call`
- 证据：`agent-session.ts` 的 `_refreshToolRegistry` 对 customTools 和 extensionTools 走同一过滤；`runner.ts` 的 `emitToolCall` 在工具实际执行前派发

---

## 核心机制：`tool_call` 事件与返回值

### ToolCallEvent 字段

所有工具的 `tool_call` 事件结构都是 `{ type, toolCallId, toolName, input }`，但 `input` 的字段因工具而异：

| 工具 | `event.toolName` | `event.input` 字段 | 证据 |
|------|-----------------|-------------------|------|
| bash | `"bash"` | `{ command: string, timeout?: number }` | `bash.ts` `bashSchema`（`timeout` 单位：**秒**） |
| read | `"read"` | `{ path: string, offset?: number, limit?: number }` | `read.ts` `readSchema` |
| write | `"write"` | `{ path: string, content: string }` | `write.ts` `writeSchema` |
| edit | `"edit"` | `{ path: string, edits: Array<{ oldText, newText }> }` | `edit.ts` `editSchema` |
| grep | `"grep"` | `{ pattern: string, path?: string, ... }` | grep.ts |
| find | `"find"` | `{ pattern: string, path?: string, ... }` | find.ts |
| ls | `"ls"` | `{ path?: string, ... }` | ls.ts |
| 自定义工具 | `string` | `Record<string, unknown>` | `types.ts` `CustomToolCallEvent` |

> ⚠️ **字段名注意**：内置文件工具（read/write/edit）的路径字段统一是 **`path`**，**不是 `file_path`**。早期文档和示例中常见的 `file_path` 是 edit.ts 渲染层的兼容别名（`edit.ts` `RenderableEditArgs`），不在 schema 中，**不能用作拦截判断**。

> ⚠️ **timeout 单位易混**：bash 工具的 `event.input.timeout` 单位是**秒**（`bash.ts` schema 注释 "Timeout in seconds"），而 `ctx.ui.confirm` 的 `opts.timeout` 单位是**毫秒**。两者出现在同一份扩展代码里时容易写反。

> 💡 **TS 类型窄化**：`ToolCallEvent` 是 discriminated union，`event.input` 类型随 `toolName` 收窄。直接写 `if (event.toolName === "bash") { event.input.command }` 在 TS 中需要断言——官方提供类型守卫 `isToolCallEventType("bash", event)`（`types.ts`）自动窄化，自定义工具则需显式泛型 `isToolCallEventType<"my_tool", MyToolInput>("my_tool", event)`。

### handler 返回值语义

```ts
pi.on("tool_call", (event, ctx) => {
  // 1. 阻断：返回 { block: true, reason }，工具不会执行，reason 返给 Agent
  if (dangerous) {
    return { block: true, reason: "危险命令已阻止" };
  }

  // 2. 修改参数：直接修改 event.input 字段（不替换整个对象）
  event.input.command = modifiedCmd;

  // 3. 条件放行：return 不返回 block 字段（或 return undefined）
  // 后续 handler 会被继续调用
});
```

**返回类型**：`ToolCallEventResult = { block?: boolean, reason?: string }`，**只有这两个字段**——没有 confirm / mutate / allow 等其他字段（证据：`types.ts` `ToolCallEventResult`）。

**第三条阻断路径——抛异常**：除了显式 `return { block: true }`，handler 内 `throw` 也会阻断工具。`agent-session.ts` 的 `beforeToolCall` 钩子用 `try/catch` 包裹了 `runner.emitToolCall(...)`（`agent-session.ts`），捕获异常后**重抛**为 `Extension failed, blocking execution: ...`，工具因此不执行——相当于隐式 block，但 reason 是错误信息而非自定义文案。即阻断工具执行有两种显式方式：(1) `return { block: true, reason }`（推荐，reason 可控）；(2) handler 内 `throw`（reason 固定为错误信息，适合校验失败时直接报错）。

**多 handler 短路机制**（`runner.ts` `emitToolCall`）：

1. 多个扩展注册 `tool_call` handler 时，按 `extensionFactories` 数组顺序串行执行
2. 一旦某个 handler 返回 `{ block: true }`，**立即短路返回**——不再调用后续扩展的 handler
3. 返回 `undefined` / `{}` / `{ block: false }` 等不阻断的值时，继续给后面的 handler 看
4. `event.input` mutation **跨 handler 透明**：后面的 handler 看到的是前面修改过的 input

---

## 核心代码

三级递进策略：**硬阻断（block）→ 静默修改（mutation）→ 用户确认（confirm，仅 CLI）**。

```ts
export default (pi) => {
  // 危险命令正则库（按需扩展）
  const DANGEROUS_PATTERNS = [
    { pattern: /rm\s+-rf\s+\//, reason: "禁止递归删除根目录" },
    { pattern: /git\s+push\s+--force/, reason: "禁止 force push" },
    { pattern: /sudo\s+/, reason: "禁止 sudo 提权" },
    { pattern: /chmod\s+777/, reason: "禁止 777 权限" },
  ];

  pi.on("tool_call", (event, ctx) => {
    // --- bash 工具：命令模式匹配 ---
    if (event.toolName === "bash") {
      const cmd = event.input.command || "";

      // 一级：硬阻断——绝对危险，直接拒绝
      for (const { pattern, reason } of DANGEROUS_PATTERNS) {
        if (pattern.test(cmd)) {
          return { block: true, reason: `${reason}（命令：${cmd}）` };
        }
      }

      // 二级：静默修改——为裸 rm 加 -i 确认标志
      // 注意：只有在 cmd 包含独立 rm 且没有 -i flag 时才加
      if (/\brm\b/.test(cmd) && !/(^|\s)-[a-z]*i/.test(cmd)) {
        event.input.command = cmd.replace(/\brm\b/, "rm -i");
      }
    }

    // --- write 工具：路径校验 ---
    if (event.toolName === "write") {
      const target = event.input.path;  // 注意：字段名是 path 不是 file_path
      if (target?.startsWith("/etc/") || target?.startsWith("/usr/")) {
        return { block: true, reason: `不允许写入系统目录：${target}` };
      }
    }

    // --- edit 工具：路径 + 内容校验 ---
    if (event.toolName === "edit") {
      const target = event.input.path;
      if (target?.startsWith("/etc/")) {
        return { block: true, reason: `不允许编辑系统目录：${target}` };
      }
    }
  });
};
```

> **代码块之间有呼吸**：上方示例展示了硬阻断 + 静默修改两类。第三级"用户确认"是 CLI 专属能力，下方单独说明。

---

## 三级策略详解

### 一级：硬阻断（block）

最简单也最安全——返回 `{ block: true, reason }`，工具不执行，Agent 收到 reason 作为工具错误结果。

**适用场景**：绝对禁止的操作（rm -rf /、git push --force、写系统目录、未授权路径）。

**Agent 行为**：Agent 看到 reason 后**可能重试**（如换路径、换命令）。如果想彻底阻止某类意图，需要把 reason 写清楚（如"请改用 --dry-run 模式"引导模型走正确路径）。

### 二级：静默修改（mutation）

直接修改 `event.input` 的字段值，工具收到的就是修改后的参数。Agent **不知道发生了修改**。

**适用场景**：

- 为危险命令加保护标志（`rm` → `rm -i`）
- 强制使用安全默认值（`write` 加 `encoding: "utf-8"`）
- 路径规范化（相对路径补全为绝对路径）

**注意事项**：

- **直接修改字段值，不要替换整个 `event.input` 对象**——后续 handler 看到的引用会丢失
- mutation **跨 handler 透明**：如果扩展 A 把 `command` 改成 `rm -i xxx`，扩展 B 看到的就是改后的值
- **不重新做 schema 校验**（`types.ts` `ToolCallEvent` 注释明确：No re-validation is performed after mutation）

### 三级：用户确认（confirm）—— ⚠️ TUI/RPC 可用，print/json 模式不可用

在 handler 内调用 `ctx.ui.confirm(title, message, opts?)` 弹出确认对话框，等待用户按键。

**真实签名**：

```ts
ctx.ui.confirm(
  title: string,    // 对话框标题（如"确认执行危险命令"）
  message: string,  // 提示内容（如"即将执行：rm -rf build/，是否继续？"）
  opts?: { signal?: AbortSignal; timeout?: number }  // 可选：signal 编程式取消 / timeout 超时自动关闭（单位：毫秒）
): Promise<boolean>
```

**关键陷阱：Web/SSE 场景下永远返回 false**

`runner.ts` 的 `noOpUIContext` 定义了：

```ts
const noOpUIContext: ExtensionUIContext = {
  confirm: async () => false,  // ← print/json 模式下永远 false（无 uiContext 时使用）；RPC 模式提供真实 UI 不走此分支
  select: async () => undefined,
  input: async () => undefined,
  notify: () => {},
  // ...
};
```

`ctx.hasUI === false` 时（即 **print / json 模式**，SDK 集成的默认场景。**注意 RPC 模式有 UI**——通过 RPC 消息与客户端交互），调 `ctx.ui.confirm` 直接返回 `false`——意味着**所有需要确认的操作都会被拒绝**，没有任何提示。

**判断方式**：

```ts
pi.on("tool_call", async (event, ctx) => {
  if (event.toolName === "bash" && event.input.command?.includes("rm ")) {
    // 仅在有 UI 的 CLI 场景下弹确认
    if (ctx.hasUI) {
      const ok = await ctx.ui.confirm(
        "确认执行危险命令",
        `即将执行：${event.input.command}`
      );
      if (!ok) {
        return { block: true, reason: "用户已取消执行" };
      }
    } else {
      // Web 场景：默认阻断，或自实现 WS 通信让前端确认
      return { block: true, reason: "Web 场景需自实现确认流程" };
    }
  }
});
```

**Web 场景的替代方案**：

1. **服务端拦截 + WS 推送 + 前端弹窗**：扩展层在 `tool_call` 收到后，通过自维护的 WS 连接把 pending 操作推到前端，前端弹 confirm UI，用户点击后回传结果，扩展层再根据结果 return block 或放行
2. **预审模式**：在 prompt 进入 agent 前用 `input` 事件做规则匹配，匹配到的操作直接拒绝（不需要等工具调用阶段）
3. **审计 + 后置告警**：放弃实时确认，在 `tool_call` / `tool_result` 全程记录日志，用 `agent_settled`（v0.80.4+，比 `agent_end` 更可靠——所有 retry/compaction/queue 处理完才触发）写完成日志，异常操作用 `ctx.abort()` 中止后续

---

## handler 执行顺序与 mutation 透明性

多个扩展都有 `tool_call` handler 时形成责任链：

```
extensionFactories: [extA, extB, extC]
                         ↓
            tool_call event 派发顺序：
            extA.handler → extB.handler → extC.handler → 工具实际执行
```

**关键规则**（`runner.ts` `emitToolCall`）：

1. **串行执行**：handler 是 `await` 的，前一个完成才执行下一个
2. **block 立即短路**：任何 handler 返回 `{ block: true }` 立即返回，不再调用后续 handler
3. **mutation 累积**：每个 handler 看到的 `event.input` 是前面所有 handler 修改过的版本
4. **不重新校验**：所有 handler 跑完后，工具收到的 `input` 可能已经面目全非——**不会做 schema 重新校验**

**实践建议**：如果想做"先记录原始参数 → 再做修改 → 再做阻断判断"的链式处理，必须控制 `extensionFactories` 顺序。

---

## 变体与延伸

### 变体 A：危险命令模式库（推荐）

把危险模式抽成可配置的数组，方便维护：

```ts
const DANGEROUS_PATTERNS: Array<{ pattern: RegExp; reason: string; level: "block" | "warn" }> = [
  { pattern: /rm\s+-rf\s+\//, reason: "禁止递归删除根目录", level: "block" },
  { pattern: /git\s+push\s+--force/, reason: "禁止 force push", level: "block" },
  { pattern: /curl\s+.*\|\s*sh/, reason: "禁止管道执行远程脚本", level: "block" },
  { pattern: /:\(\)\s*\{\s*:\|:&\s*\};:/, reason: "fork bomb", level: "block" },
  // 可从外部配置文件加载
];

pi.on("tool_call", (event, ctx) => {
  if (event.toolName !== "bash") return;
  const cmd = event.input.command || "";
  for (const rule of DANGEROUS_PATTERNS) {
    if (rule.pattern.test(cmd)) {
      if (rule.level === "block") {
        return { block: true, reason: `${rule.reason}（命令：${cmd}）` };
      }
      ctx.ui.notify(`⚠️ ${rule.reason}`, "warning");
    }
  }
});
```

> ⚠️ **notify 可见性**：`ctx.ui.notify` 仅在 **TUI / RPC 模式**下实际显示（runner.ts 的 noOpUIContext 把 `notify` 实现为空函数）。print / json 模式下这条警告**不会显示**——Web 集成场景需自实现告警通道（如 WS 推送 / 日志写入），不能依赖 notify 提示用户。

### 变体 B：edit 工具的 oldText 内容审计

阻止 edit 把敏感字符串（如 API key、密码）写入文件：

```ts
const SENSITIVE_PATTERNS = [
  /sk-[a-zA-Z0-9]{20,}/,    // OpenAI API key
  /AKIA[A-Z0-9]{16}/,        // AWS access key
  /password\s*=\s*["'][^"']+["']/i,
];

pi.on("tool_call", (event, ctx) => {
  if (event.toolName !== "edit") return;
  const { path, edits } = event.input;
  for (const edit of edits || []) {
    if (SENSITIVE_PATTERNS.some(p => p.test(edit.newText || ""))) {
      return { block: true, reason: `检测到敏感信息写入：${path}` };
    }
  }
});
```

### 变体 C：自定义工具的输入二次校验

schema 校验是结构性的（类型对、字段齐），二次校验是业务性的（值在合法范围内）：

```ts
pi.on("tool_call", (event, ctx) => {
  if (event.toolName === "execute_sql") {
    const sql = event.input.sql || "";
    // 阻止所有非查询语句
    if (/^\s*(DROP|DELETE|TRUNCATE|ALTER|CREATE|INSERT|UPDATE)\b/i.test(sql)) {
      return {
        block: true,
        reason: "execute_sql 仅允许 SELECT 语句。如需写操作请用 execute_mutation 工具",
      };
    }
  }
});
```

### 变体 D：动态启用/禁用拦截器

> ⚠️ **适用前提**：此变体依赖 `pi.registerCommand` 注册的命令入口。命令只能在 **CLI（TUI）/ RPC 模式**下由用户触发（`interactive-mode.ts` 把扩展命令暴露为斜杠命令，`rpc-mode.ts` 经 RPC 请求派发）；**print / json 模式没有命令入口**，纯 SDK 集成场景需用 flag / env / 外部配置文件替代命令切换逻辑。

通过运行时标志位控制拦截是否生效（无需卸载扩展）：

```ts
let interceptEnabled = true;
pi.registerCommand("toggle-intercept", {
  description: "切换工具拦截开关",
  handler: async (_args, ctx) => {
    interceptEnabled = !interceptEnabled;
    ctx.ui?.notify(`工具拦截：${interceptEnabled ? "ON" : "OFF"}`, "info");
  },
});

pi.on("tool_call", (event, ctx) => {
  if (!interceptEnabled) return;  // 直接放行
  // ... 拦截逻辑
});
```

### 变体 E：审计日志（不阻断）

只记录工具调用，不做任何拦截——可用于合规审计：

```ts
pi.on("tool_call", (event, ctx) => {
  // 不 return block，工具正常执行
  console.log(JSON.stringify({
    type: "tool_call_audit",
    timestamp: new Date().toISOString(),
    toolName: event.toolName,
    toolCallId: event.toolCallId,
    input: event.input,
    cwd: ctx.cwd,
  }));
});
```

> 💡 **推荐用 `agent_settled` 做全程审计**：如果你需要「本次 agent 完全跑完（所有 retry/compaction/queue 处理完）后写一条完成日志」的语义，用 `agent_settled`（两层都派发）替代 `agent_end`。`agent_end` 在 retry/compaction 等中间状态也会触发，不适合做"本次会话已结束"的完成信号。

### 变体 F：与工具结果处理联动

`tool_call` 拦截 + `tool_result` 后处理组合，覆盖完整的"前置检查 + 后置审计"链路。完整示例见 [E02 编写完整扩展](E02-extension-basics.md)。

---

## 陷阱

### 陷阱 1：`file_path` 字段名幻觉（★ P0 高发）

内置文件工具（read/write/edit）的路径字段**统一是 `path`**，不是 `file_path`。`event.input.file_path` 永远是 `undefined`，保护逻辑直接失效：

```ts
// ❌ 错误：永远不会触发 block
if (event.input.file_path?.startsWith("/etc/")) { ... }

// ✅ 正确
if (event.input.path?.startsWith("/etc/")) { ... }
```

**核查源码**：`write.ts` `writeSchema`、`read.ts` `readSchema`、`edit.ts` `editSchema`。

### 陷阱 2：`ctx.ui.confirm` 单参数用法（★ P0 高发）

`confirm` 的真实签名是 **`(title, message, opts?)`** 两个字符串参数，不是 `(message)`：

```ts
// ❌ 错误：title 错位为整条消息，message 为 undefined
const ok = await ctx.ui.confirm("是否执行此命令？");

// ✅ 正确
const ok = await ctx.ui.confirm(
  "确认执行危险命令",
  `即将执行：${cmd}`
);
```

### 陷阱 3：`ctx.ui.confirm` 在 Web 场景下永远返回 false

`runner.ts` 的 `noOpUIContext` 让 `ctx.hasUI === false` 时所有 UI 方法都返回 undefined / false。**非 TUI 非 RPC 的 SDK 集成场景（print/json 模式）下默认 `ctx.hasUI === false`**——调 confirm 等于直接拒绝所有操作。RPC 模式有 UI（通过 RPC 消息与客户端交互）。

**对策**：用 `ctx.hasUI` 判断 + Web 替代方案（见上方"三级策略 → 三级"）。

### 陷阱 4：`tool_call` 是扩展独有事件

`tool_call` 是**扩展独有事件**之一（典型代表：`context` / `tool_call` / `tool_result` / `before_agent_start` / `input` / `model_select` 等——非穷举，完整清单见 [04-events.md](../sdk_doc/04-events.md)）。**server 层 `session.subscribe` 收不到**——handler 被调用但 type 分支永不命中，无任何报错。

想做服务端拦截必须走扩展机制（`extensionFactories`）。详见 [04-events.md 坑 4](../sdk_doc/04-events.md#坑-46-个扩展独有事件sessionsubscribe-静默收不到-最大集成坑)。

### 陷阱 5：mutation 顺序依赖

多扩展有 `tool_call` handler 时按 `extensionFactories` 数组顺序串行，`event.input` mutation 跨 handler 累积。如果想做"先记录原始值 → 再修改 → 再判断"，必须显式控制 `extensionFactories` 顺序。

### 陷阱 6：mutation 不重新校验 schema

工具实际执行前**不会重新跑 schema 校验**（`types.ts` `ToolCallEvent` 注释明确）。mutation 后的 input 可能违反 schema（如把 `command: string` 改成 `command: number`），工具内部解析会出错。

**对策**：mutation 时保持字段类型与原始 schema 一致。

### 陷阱 7：`return { block: false }` 不会阻断但会覆盖 reason

关键行为（`runner.ts` 的 `emitToolCall` 对每个 handler 独立调用）：

1. **handler 间不传递返回值**——后续 handler 看不到前一个 handler 的 `reason`
2. **`block: false` 的 reason 不传给工具**——工具正常执行，reason 被丢弃
3. **只有 `block: true` 的 reason 才传给 Agent**——`agent-loop.ts` 只在 `block === true` 时使用 reason

### 陷阱 8：async handler 必须 await

如果 handler 内有异步操作（如 `ctx.ui.confirm` 返回 Promise），必须 `async (event, ctx) => { ... await ... }`，不能写成 `(event, ctx) => { ctx.ui.confirm(...).then(...) }`——后者 handler 会立即返回 undefined，confirm 还没等到结果工具已经执行完了。

**底层原因**：`runner.ts` 的 `emitToolCall` 不像 `emitToolResult` / `emitUserBash` 那样带 try/catch 包裹——它对每个 handler 直接 `await handler(event, ctx)`，任何异常（包括未 await 的 Promise rejection）会**直接冒泡**到 `beforeToolCall` 钩子被捕获并重抛为 "Extension failed, blocking execution"。所以"忘 await"不只是逻辑错（放行了本该拦截的操作），还可能以意外抛错的形式悄悄阻断执行。

---

## 横向联动

- [E01 拦截与修改工具调用](E01-tool-intercept.md) — `tool_call` 的通用拦截场景（不限于危险命令）
- [D06 限制工具执行目录](D06-protected-paths.md) — 路径白名单（结构化越权防护）
- [D05 自定义工具输出渲染](D05-tool-result-render.md) — 在 `tool_result` 阶段修改展示
- [E02 编写完整扩展](E02-extension-basics.md) — `tool_call` + `tool_result` + 生命周期的完整组合
- [E06 流式处理工具输出](E06-streaming-transform.md) — `tool_execution_update` 流式 hook
- [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) — `pi.on` / `ExtensionContext` / `ExtensionUIContext` 完整接口
- [sdk_doc/04-events.md](../sdk_doc/04-events.md) — 扩展独有事件清单（`tool_call` 是其中之一）
- [A06 默认行为](A06-load-extensions.md) — `createAgentSession` 工具启用规则
