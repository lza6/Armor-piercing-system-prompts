# 场景：限制工具执行目录 (D06)

## 什么时候用 / 不用会怎样

**该用本场景**：

- **多租户 / 共享环境**：多个用户的 Agent 跑在同一台机器上，每个用户只能读写自己目录
- **生产服务集成**：把 Agent 嵌入 Web/Server 应用，限制它只能在 `/data/sandbox/<userId>/` 这种业务隔离目录内操作
- **审计合规**：除拦截破坏性命令（见 [D04](D04-confirm-destructive.md)）外，再做一道路径白名单校验，防止模型被 prompt 注入诱导读 `/etc/passwd` 或写 `~/.ssh/`
- **CI / 沙箱执行**：Agent 跑在不信任的代码上，必须限制只能改仓库目录，不能动 `node_modules/` / `.git/`

**不用会怎样**：

- Agent 任意读 `process.cwd()` 之外的文件——模型一旦理解了主机结构，可以读 SSH key、token、其他项目源码
- Agent 任意写——`write` / `edit` 能覆盖任意有权限的文件，包括 `~/.bashrc`、`/etc/profile.d/*.sh` 这类持久化攻击载体
- bash 工具更危险——`cat /etc/passwd` / `curl ... | sh` / `rm -rf ~` 完全无防护

**不适合本场景**：

- 拦截破坏性命令（`rm -rf` / `git push --force`）→ 见 [D04 工具调用安全闸门](D04-confirm-destructive.md)
- 修改工具参数（如改写路径）而非阻断 → 见 [E01 拦截与修改工具调用](E01-tool-intercept.md)
- 修改工具返回结果（如脱敏文件内容） → 见 [D05 自定义工具输出渲染](D05-tool-result-render.md)
- 流式处理工具执行过程 → 见 [E06 流式处理工具输出](E06-streaming-transform.md)

---

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `pi.on("tool_call", handler)` | 拦截工具调用事件，handler 返回 `{ block: true, reason }` 阻断 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `event.input`（`ToolCallEvent.input`） | 工具参数对象，按 toolName 分强类型；路径字段见下方表格 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `ctx.cwd`（`ExtensionContext.cwd`） | 当前 session 的工作目录，所有内置工具用它解析相对路径 | - |
| `return { block: true, reason }` | 阻断执行，reason 会回到模型上下文 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |

> ⚠️ **关键集成坑 1：`tool_call` 是 6 个扩展独有事件之一**（另五个是 `context` / `tool_result` / `before_agent_start` / `input` / `model_select`）。在 server 层用 `session.subscribe("tool_call", ...)` 会**静默失败**——外部事件流根本不派发这个 type。想做服务端路径限制**必须走扩展**。详见 [04-events.md 坑 4](../sdk_doc/04-events.md#坑-46-个扩展独有事件sessionsubscribe-静默收不到-最大集成坑)。

> ⚠️ **关键集成坑 2：`ctx.ui.confirm` 是 CLI 专属**。如果你想在越权时弹框让用户确认（而非硬阻断），`ctx.ui.confirm` 在 `createAgentSession` 默认 `mode="print"` 下 `hasUI=false`，调用会 no-op（返回 false）——见 `runner.ts` 的 `noOpUIContext`。Web 场景请用硬阻断或自定义 RPC 通道。详见 [D04 与 setStatus 配合](D04-confirm-destructive.md#与-setstatus-配合cli-专属)。

---

## 默认行为（★ 必读）

**工具启用规则**（与 [A04](A04-tool-whitelist.md) / [D01](D01-custom-tool.md) / [D02](D02-dynamic-tools.md) / [D04](D04-confirm-destructive.md) / [D05](D05-tool-result-render.md) 一致）：

| `createAgentSession` 配置 | 内置工具 | 扩展工具 | customTools |
|--------------------------|---------|---------|-------------|
| 不传 `tools`（默认） | 启用 `["read", "bash", "edit", "write"]` | 自动启用 | 自动启用 |
| `tools: ["read", "bash"]` | 仅 read + bash | **必须列入才启用** | **必须列入才启用** |
| `noTools: "all"` | 全禁用 | 全禁用 | 全禁用 |
| `noTools: "builtin"` | 全禁用 | 自动启用 | 自动启用 |
| `excludeTools: ["bash"]` | bash 禁用，其他启用 | 自动启用（除非在 exclude 中） | 自动启用（除非在 exclude 中） |

**关键事实**：

1. `find` / `grep` / `ls` **不是默认启用**——它们是内置工具但不默认激活（默认 `initialActiveToolNames` 只含 `read/bash/edit/write`）。需要在 `tools: [...]` 中显式列入才会暴露给模型（`sdk.ts` 的 `defaultActiveToolNames`）
2. 一旦工具被启用，所有注册的 `tool_call` handler 都会拦截它——不能只拦 read 不拦 write（除非 handler 内部按 `event.toolName` 分发）
3. `tool_call` handler 在工具**真正执行前**触发，阻断后工具不会运行；handler 在**模型流式输出 tool call 后**触发，所以阻断会消耗一次模型 turn（阻断 reason 进入对话，模型会看到"路径越权"并自行调整）

**证据**：`sdk.ts`（`defaultActiveToolNames` 只含 read/bash/edit/write）、`agent-session.ts`（`_refreshToolRegistry` 中的 `isAllowedTool` 过滤）。

---

## 内置工具字段对照表（★ 必读）

**这是 D06 的核心表**——拦截 handler 要从 `event.input` 取路径字段，字段名错了就完全失效（D04 横向提示的 P0 高发区）。

| 工具名 | 路径字段 | 是否可选 | 含义 | 证据 |
|--------|---------|---------|------|------|
| `read` | `path` | 必填 | 要读的文件路径（相对或绝对） | `read.ts` |
| `write` | `path` | 必填 | 要写的文件路径（相对或绝对） | `write.ts` |
| `edit` | `path` | 必填 | 要编辑的文件路径（相对或绝对） | `edit.ts` |
| `bash` | `command` | 必填 | 要执行的 shell 命令（**不是路径**，需要 shell parser 才能提取里面的路径） | `bash.ts` |
| `find` | `path` | 可选 | 搜索根目录（默认当前目录） | `find.ts` |
| `grep` | `path` | 可选 | 搜索目录或文件（默认当前目录） | `grep.ts` |
| `ls` | `path` | 可选 | 要列出的目录（默认当前目录） | `ls.ts` |

**非路径字段，不要按路径拦截**：

| 工具名 | 字段 | 为什么不能按路径拦 |
|--------|------|------------------|
| `find` | `pattern` | 是 glob 模式（如 `*.ts`），不是路径——拦了模型连"找所有 ts 文件"都做不了 |
| `grep` | `pattern` | 是搜索内容（正则或字符串），不是路径 |
| `grep` | `glob` | 是文件类型过滤（如 `*.ts`），不是路径 |

> ★ **历史误区**：旧版本 skill 文档曾把字段写成 `file_path`、内置工具写成 `glob`。**真实 schema 是 `path`**，渲染层兼容 `file_path` 仅为向后兼容旧 prompt（`read.ts`），但**模型实际生成的参数严格按 schema 走**——拦截 handler 里读 `file_path` 会拿到 `undefined`，白名单形同虚设。同样，**内置工具叫 `find` 不叫 `glob`**。

---

## ToolCallEvent 字段

每个 `tool_call` handler 收到的 `event` 对象结构（`types.ts`）：

| 字段 | 类型 | 说明 |
|------|------|------|
| `type` | `"tool_call"` | 事件类型标识 |
| `toolCallId` | `string` | 唯一调用 ID，可用来关联 tool_result |
| `toolName` | `"bash" \| "read" \| "edit" \| "write" \| "grep" \| "find" \| "ls" \| string` | 工具名；自定义工具是任意 string |
| `input` | 强类型工具 input | 按 toolName 窄化（如 `ReadToolInput` 含 `path/offset/limit`） |

**handler 签名与返回值**：

```ts
type Handler = (event: ToolCallEvent, ctx: ExtensionContext) =>
  Promise<ToolCallEventResult | void> | ToolCallEventResult | void;
```

`ToolCallEventResult`（`types.ts`）：

| 字段 | 类型 | 作用 |
|------|------|------|
| `block` | `boolean` | true 时阻断工具执行 |
| `reason` | `string` | 阻断原因，会进模型上下文 |

> ⚠️ **改参数走 mutation，不走返回值**。`ToolCallEventResult` **只有 block + reason**——要修改路径必须直接改 `event.input.path = ...`（`types.ts` 注释明确说明）。要修改参数（而非阻断）请走 [E01](E01-tool-intercept.md)，不要在 D06 里做。

---

## emitToolCall 执行机制（★ 多 handler 时必读）

`ExtensionRunner.emitToolCall` 的执行顺序（`runner.ts`）：

1. **按扩展注册顺序**遍历所有扩展
2. 在每个扩展内**按 handler 注册顺序**串行调用
3. **遇到 `block: true` 立即短路返回**——后面的 handler 不再执行
4. **handler 抛异常会冒泡**——`emitToolCall` **没有 try/catch**（对比 `emitUserBash` 有 try/catch + `emitError`）。如果你的 handler 可能抛异常（如读不存在的配置文件），**自己包 try/catch**，否则会中断整个 agent loop

**与 tool_result 的关键区别**：

| 维度 | tool_call | tool_result |
|------|-----------|-------------|
| 阻断 | `block: true` 短路 | 没有 block 字段，无法阻断（工具已执行完） |
| 改参数 | mutate `event.input` 有效 | 改 `event.input` 无效（已执行） |
| 改结果 | N/A | mutate `event.content` / `event.isError` 或返回 `{ content, details, isError }` |
| 异常处理 | **未捕获** | 有 try/catch + emitError（异常不冒泡） |
| 典型用途 | 路径白名单、危险命令阻断、参数改写 | 脱敏、截断、格式标准化、错误友好化 |

详见 [D05 handler 执行机制](D05-tool-result-render.md#handler-执行机制)。

---

## 实现思路

1. 在扩展工厂中接收允许的根目录列表（**闭包捕获**，不要找不存在的 `pi.getSettings()`）
2. 用 `pi.on("tool_call", handler)` 拦截所有工具调用
3. 按 `event.toolName` 查「路径字段表」取出路径值
4. 用 `path.resolve(ctx.cwd, rawPath)` 解析为绝对路径（**不要用 `process.cwd()`**）
5. 用 `normalized.startsWith(root + path.sep) || normalized === root` 判断是否在白名单内
6. 不在白名单内则 `return { block: true, reason }`

**关键安全要点**：

- **始终用绝对路径比较**，避免 `../` 绕过
- **解析后必须 `path.normalize()`**——`/data/safe/../etc/passwd` 解析后是 `/etc/passwd`，但如果不 normalize 就 `startsWith("/data/safe")` 会误判通过
- **比较时加 `path.sep` 后缀**——`/data/safe-secret`.startsWith(`/data/safe`) 会误判通过，必须 `startsWith(root + path.sep) || === root`
- **bash 工具的 `command` 字段不能按字符串匹配路径**——shell 命令有无穷变体（`r""m`、`rm -r -f`、`$(...)`、变量替换），正则极易绕过。bash 路径限制的正解是**用沙箱（chroot / container / 自定义 BashOperations.exec 拦截 spawn）**，见 `bash.ts` 的 `BashOperations` 注入点

---

## 核心代码

### 方案一：硬编码白名单（最简版）

```ts
import path from "node:path";
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";

// 工具名 → 路径字段名（来自内置工具 schema，详见上方字段对照表）
const PATH_FIELDS: Record<string, string> = {
  read: "path",
  write: "path",
  edit: "path",
  find: "path",
  grep: "path",
  ls: "path",
  // bash 不在这个表里——command 不是路径，需要 shell parser，见下方变体 D
};

export default (pi: ExtensionAPI) => {
  pi.on("tool_call", (event, ctx) => {
    const field = PATH_FIELDS[event.toolName];
    if (!field) return; // 非路径工具（含 bash、自定义工具），放行

    const raw = (event.input as Record<string, unknown>)[field];
    if (typeof raw !== "string") return; // 字段缺失或类型不对，放行让 schema 校验报错

    // 解析为绝对路径——必须用 ctx.cwd，不能用 process.cwd()
    // 内置工具内部用 resolveToCwd（含 ~/@ 展开），这里简化用 path.resolve
    const resolved = path.normalize(path.resolve(ctx.cwd, raw));

    // 白名单判断：必须加 path.sep 后缀，避免 safe-secret 绕过
    const ALLOWED_ROOTS = [ctx.cwd]; // 简化示例，实际应从扩展工厂参数注入
    const allowed = ALLOWED_ROOTS.some(
      (root) => resolved === root || resolved.startsWith(root + path.sep)
    );

    if (!allowed) {
      return {
        block: true,
        reason: `路径越权：${raw}（解析为 ${resolved}）不在允许目录 ${ALLOWED_ROOTS.join(", ")} 内`,
      };
    }
  });
};
```

### 方案二：可配置白名单（生产推荐）

通过扩展工厂参数注入配置（`pi.getSettings()` **不存在**——见陷阱 5）：

```ts
import path from "node:path";
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";

export interface ProtectedPathsOptions {
  /** 允许的根目录（绝对路径或相对 process.cwd()） */
  allowedRoots: string[];
  /** 可选：自定义工具的路径字段映射（如 execute_sql 的 db_path） */
  customToolPathFields?: Record<string, string>;
}

// 包成工厂函数——配置在 createAgentSession 调用时注入
export function createProtectedPathsExtension(options: ProtectedPathsOptions) {
  const ALLOWED_ROOTS = options.allowedRoots.map((p) => path.resolve(p));

  const PATH_FIELDS: Record<string, string> = {
    read: "path",
    write: "path",
    edit: "path",
    find: "path",
    grep: "path",
    ls: "path",
    ...(options.customToolPathFields ?? {}),
  };

  return (pi: ExtensionAPI) => {
    pi.on("tool_call", (event, ctx) => {
      const field = PATH_FIELDS[event.toolName];
      if (!field) return;

      const raw = (event.input as Record<string, unknown>)[field];
      if (typeof raw !== "string") return;

      // ⚠️ 注意：内置工具用 resolveToCwd 处理 ~ 和 @ 前缀
      // 这里只处理常规路径，~ 开头的路径会被 path.resolve 当成文件名
      const resolved = path.normalize(path.resolve(ctx.cwd, raw));

      const allowed = ALLOWED_ROOTS.some(
        (root) => resolved === root || resolved.startsWith(root + path.sep)
      );

      if (!allowed) {
        return {
          block: true,
          reason: `路径越权：${raw} 不在允许目录内`,
        };
      }
    });
  };
}

// 使用
import { createAgentSession } from "@earendil-works/pi-coding-agent";

const session = createAgentSession({
  cwd: "/data/sandbox/user123",
  model,
  extensions: [
    createProtectedPathsExtension({
      allowedRoots: ["/data/sandbox/user123", "/tmp/agent-cache"],
    }),
  ],
});
```

---

## 变体与延伸

### 变体 A：与 destructive 命令拦截配合（A 档）

路径白名单 + 危险命令拦截同时使用——见 [D04 工具调用安全闸门](D04-confirm-destructive.md)。D04 管"命令危险性"，D06 管"路径越权"，正交互补。

### 变体 B：find/grep 限制搜索范围（A 档）

`find` 和 `grep` 的 `path` 字段是搜索根目录。即使 `path` 字段为空（默认当前目录），模型也可能传 `/etc` / `/root` 探测系统。同样拦截：

```ts
// 在 PATH_FIELDS 里加上 find / grep / ls 后，handler 自动生效
// 如果想更严格——禁止搜索任何非白名单目录（即使只是读权限）：
// 上方核心代码已经覆盖，因为 find/grep/ls 的 path 字段也在表里
```

### 变体 C：自定义工具的路径校验（A 档）

业务工具（如 `execute_sql`）也可能接收路径参数。通过 `customToolPathFields` 注入字段映射：

```ts
createProtectedPathsExtension({
  allowedRoots: [projectRoot],
  customToolPathFields: {
    execute_sql: "db_path",
    upload_file: "file_path", // 注意：自定义工具的 schema 自定义，可以叫 file_path
    // ⚠️ 内置 read/write/edit 的字段必须是 path，不要在这里覆盖
  },
});
```

### 变体 D：bash 命令拦截（C 档，慎用）

bash 的 `command` 字段**不是路径**——shell 命令有无数变体绕过字符串匹配（`r""m -rf`、`rm -r -f`、`$(rm -rf)`、变量替换、base64 解码执行等）。**正则拦截是 D04 场景的简化方案**，正确做法：

1. **shell parser**：用 [shell-quote](https://www.npmjs.com/package/shell-quote) 解析 AST，提取所有路径 token 再校验
2. **沙箱执行**：自定义 `BashOperations.exec`，在 spawn 前包装命令（如 `bwrap --ro-bind / / --bind $ALLOWED_ROOT $ALLOWED_ROOT --unshare-all bash -c "$cmd"`）
3. **完全禁用 bash**：`excludeTools: ["bash"]`，改用 read/write/edit 等结构化工具，再用 D06 限制路径

bash 字符串拦截的正则示例见 [D04](D04-confirm-destructive.md#核心代码)（仅作简单示例，不要在生产依赖）。

### 变体 E：日志记录所有路径访问（A 档）

在 `tool_result` 事件里审计实际访问的路径（包含结果是否成功），形成完整访问日志：

```ts
pi.on("tool_result", (event) => {
  if (event.toolName === "read" || event.toolName === "write" || event.toolName === "edit") {
    console.log(`[audit] ${event.toolName} ${event.input.path} → isError=${event.isError}`);
  }
});
```

详见 [D05](D05-tool-result-render.md) 和 [E06](E06-streaming-transform.md)。

### 变体 F：动态白名单（A 档）

根据当前 prompt 上下文或用户身份动态调整白名单。`ctx.sessionManager` 提供只读访问，可用 `session_start` 事件初始化白名单：

```ts
let currentRoots: string[] = [];

pi.on("session_start", (event, ctx) => {
  // SessionStartEvent 只有 type/reason/previousSessionFile，无 sessionId
  // 用 ctx.sessionManager.getSessionId() 获取 session ID
  const sessionId = ctx.sessionManager.getSessionId();
  currentRoots = computeRootsForUser(sessionId);
});

pi.on("tool_call", (event, ctx) => {
  // 用 currentRoots 做校验
});
```

---

## 陷阱（★ 必读）

1. **`process.cwd()` ≠ `ctx.cwd`**：SDK 集成场景下，扩展进程的 `process.cwd()` 可能是 SDK 启动目录，而 session 的 `cwd` 是 `createAgentSession({ cwd })` 配置的目录。两者**通常不同**。必须用 `ctx.cwd`。证据：`types.ts`

2. **`startsWith` 不加 `path.sep` 后缀**：`"/data/safe-secret".startsWith("/data/safe")` 是 `true`——会误判通过。必须 `resolved.startsWith(root + path.sep) || resolved === root`。Windows 下 sep 是 `\`，跨平台用 `path.sep` 不要硬编码 `/`

3. **不 `path.normalize` 解析后的路径**：`path.resolve("/data/safe", "../etc/passwd")` 返回 `/data/etc/passwd`，本身已 normalize；但如果原始路径里混入 `..`，**必须再 normalize 一次**保险。`path.resolve` 实际会 normalize，但显式调用更清晰

4. **`file_path` 是幻觉字段名**：内置 read/write/edit 的 schema 字段是 `path`，不是 `file_path`。`file_path` 只是渲染层的向后兼容代码（`read.ts`）。模型按 schema 生成参数，handler 读 `file_path` 会拿到 `undefined`——白名单形同虚设

5. **`pi.getSettings()` 是幻觉 API**：`ExtensionAPI` 接口**没有** `getSettings` / `settings` / `config` 方法（`types.ts`）。配置只能通过扩展工厂参数闭包注入，或读 `process.env` / `ctx.sessionManager`

6. **内置工具叫 `find` 不叫 `glob`**：D04 横向提示的 P0 高发区。`find` 用 glob pattern 作为搜索条件（`pattern` 字段），但工具名是 `find`

7. **`pattern` 字段不是路径**：`find.pattern` 是 glob 模式（如 `*.ts`），`grep.pattern` 是搜索内容。按 pattern 拦截会阻止合法查询

8. **`tool_call` 是扩展独有事件**：server 层 `session.subscribe("tool_call")` 静默失败。想做服务端拦截必须走扩展（`types.ts`）

9. **bash `command` 字段不能按字符串拦**：shell 有无数变体绕过正则。bash 命令限制的正解是沙箱或 shell parser，见变体 D

10. **handler 抛异常会中断 agent loop**：`emitToolCall` **没有 try/catch**（`runner.ts`），对比 `emitUserBash` 有。如果你的 handler 可能抛异常（如读配置失败），自己包 try/catch

11. **`ctx.ui.confirm` 在 Web 场景是 no-op**：默认 `mode="print"` 时 `hasUI=false`，所有 UI 方法都返回安全默认值（confirm 返回 false）。不能依赖它做交互式确认。详见 [D04 与 setStatus 配合](D04-confirm-destructive.md#与-setstatus-配合cli-专属)

12. **多 handler 串行 + block 短路**：如果同时注册了 D04 危险命令拦截和 D06 路径拦截，按扩展注册顺序执行。前一个 `block: true` 会让后面的 handler 不跑。注意 handler 顺序

---

## 横向联动

- [D04 工具调用安全闸门](D04-confirm-destructive.md) — 危险命令拦截，与 D06 互补
- [D05 自定义工具输出渲染](D05-tool-result-render.md) — 修改工具返回结果（脱敏 / 截断）
- [E01 拦截与修改工具调用](E01-tool-intercept.md) — 修改而非阻断工具参数
- [E06 流式处理工具输出](E06-streaming-transform.md) — 记录执行日志
- [A04 工具白名单](A04-tool-whitelist.md) — `tools` / `excludeTools` 配置项
- [A06 自定义工具](A06-custom-tool.md) — 默认启用规则
- [sdk_doc/04-events.md](../sdk_doc/04-events.md) — `tool_call` 扩展独有事件详解
- [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) — 扩展 API 完整参考
- [sdk_doc/06-agent-session.md](../sdk_doc/06-agent-session.md) — `createAgentSession` 配置项
