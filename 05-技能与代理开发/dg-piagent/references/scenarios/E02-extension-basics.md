# 场景：编写完整扩展 (E02)

## 什么时候用

让自定义能力挂到 Agent 会话上，**随会话生命周期自动生效**——不用每次手动调用。典型场景：

- **挂全局钩子**：会话开始时初始化资源、工具调用时记录日志、会话结束时清理
- **给 Agent 加自定义工具**：注册 LLM 可调用的工具（如 `word_count`、`fetch_weather`）
- **注册用户命令**：CLI 场景下，让用户输入 `/status` 触发扩展逻辑
- **多能力组合**：一个扩展文件内同时提供钩子 + 工具 + 命令，覆盖完整生命周期

**不适合本场景**：
- 只想拦截工具调用 → 见 [场景 E01](E01-tool-intercept.md)（更聚焦工具字段 + 拦截机制）
- 只想跟踪生命周期事件 → 见 [场景 E04](E04-lifecycle-hooks.md)（更聚焦事件时序）
- 想让扩展**持久分发**给团队 → 见 [场景 I02](I02-distribute-extension.md)（Pi Package 打包）
- 只想加载已有扩展 → 见 [场景 A06](A06-load-extensions.md)（不写新扩展）

## 范围（★ 先看这个）

本场景聚焦扩展开发的**三件套**：①事件 hook（`pi.on`）②动态工具（`pi.registerTool`）③用户命令（`pi.registerCommand`）。

扩展的其他能力（自定义消息渲染 `pi.registerMessageRenderer`、CLI 快捷键 `pi.registerShortcut`、CLI flag `pi.registerFlag`）属于低频或 CLI 专属，**不在本场景展开**——查 [07-extensions-api.md](../sdk_doc/07-extensions-api.md) 完整接口清单。

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `pi.on(eventName, handler)` | 订阅事件（hook） | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `pi.registerTool(tool)` | 动态注册 LLM 可调用的工具 | [sdk_doc/06-tools.md](../sdk_doc/06-tools.md) |
| `pi.registerCommand(name, options)` | 注册用户命令（CLI 场景下 `/name` 触发） | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `ctx.ui.setStatus(key, text)` | 在 TUI 状态栏显示信息（**mode 守卫**，见陷阱 2） | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |

> ⚠️ **关键警示（扩展独有事件）**：以下 6 个事件**只在扩展层派发**，server 层 `session.subscribe(...)` 收不到，会**静默失败**：
> - `context` — 修改发给 LLM 的消息列表
> - `tool_call` — 工具调用前拦截 / 改参
> - `tool_result` — 工具执行后改返回值
> - `before_agent_start` — 每轮开始前注入消息 / 改系统提示词
> - `input` — 变换 / 拦截用户输入
> - `model_select` — 模型切换时触发（subscribe 收不到）
>
> 证据：`model_select` 派发点见 agent-session.ts（`_emitModelSelect` → `_extensionRunner.emit`），从不进入 `AgentSessionEvent` 联合类型；扩展独有机制见 agent-session.ts 两层事件派发。详见 [04-events.md](../sdk_doc/04-events.md)。

## 扩展加载方式（★ 必读）

扩展有 **4 种加载来源**，DefaultResourceLoader 会合并所有来源：

| 来源 | 路径 / 形式 | 何时用 |
|------|-----------|--------|
| 项目级自动发现 | `<cwd>/.pi/extensions/*.ts` | 扩展属于当前项目（**项目必须受信**才加载） |
| 用户级自动发现 | `~/.pi/agent/extensions/*.ts` | 扩展属于当前用户（全局生效） |
| 显式路径 | `additionalExtensionPaths: ["路径", "npm:包名", "git+URL"]` | 扩展在其他位置 |
| 内联工厂 | `extensionFactories: [(pi) => { ... }]` | 逻辑短小，不开独立文件 |

证据：extensions/loader.ts（discoverAndLoadExtensions）、resource-loader.ts（DefaultResourceLoaderOptions）。

**完整加载流程见 [场景 A06](A06-load-extensions.md)**——本场景不重复展开。

## ExtensionFactory 签名

扩展文件的默认导出是一个工厂函数：

```ts
// 签名：ExtensionFactory = (pi: ExtensionAPI) => void | Promise<void>
// 注意：已支持 async 初始化（types.ts:1506）
export default (pi) => {
  // 在这里订阅事件 / 注册工具 / 注册命令
};
```

## ExtensionContext（事件 handler 的 ctx）

事件 handler 接收 `(event, ctx)`，其中 `ctx: ExtensionContext` 提供运行时上下文。**关键字段**：

| 字段 | 类型 | 含义 |
|------|------|------|
| `ctx.ui` | `ExtensionUIContext` | UI 方法（setStatus / notify / select / confirm / input 等） |
| `ctx.mode` | `"tui" \| "rpc" \| "json" \| "print"` | 当前运行模式，**默认 "print"**（见陷阱 2） |
| `ctx.hasUI` | `boolean` | 是否有可交互 UI（tui / rpc 为 true） |
| `ctx.cwd` | `string` | 当前会话工作目录（**不是 `process.cwd()`**） |
| `ctx.sessionManager` | `ReadonlySessionManager` | 会话管理器（只读） |
| `ctx.modelRegistry` | `ModelRegistry` | 模型注册表（含 `complete()` / `getApiKeyAndHeaders()`，types.ts） |
| `ctx.model` | `Model<any> \| undefined` | 当前模型 |
| `ctx.scopedModels` | `readonly ScopedModel[]` | 已解析的模型列表（每项含 model + thinkingLevel?，types.ts） |
| `ctx.thinkingLevel` | `ThinkingLevel \| undefined` | 当前思考等级（off/minimal/low/medium/high/xhigh/max） |
| `ctx.signal` | `AbortSignal \| undefined` | 当前流式操作的 abort signal |
| `ctx.abort()` | `() => void` | 中断当前 agent 操作 |
| `ctx.shutdown()` | `() => void` | 优雅关闭 pi 并退出 |
| `ctx.isIdle()` | `() => boolean` | Agent 是否空闲 |
| `ctx.isProjectTrusted()` | `() => boolean` | 当前项目是否受信 |
| `ctx.hasPendingMessages()` | `() => boolean` | 是否有待处理消息 |
| `ctx.getContextUsage()` | `() => ContextUsage \| undefined` | 获取上下文使用情况 |
| `ctx.compact()` | `(options?) => void` | 触发上下文压缩 |
| `ctx.getSystemPrompt()` | `() => string` | 获取当前系统提示词 |

> ⚠️ **`ExtensionContext` 没有 `session` 字段、没有 `waitForIdle` 方法**。证据：extensions/types.ts。
>
> `waitForIdle` 只在 `ExtensionCommandContext`（命令 handler 的 ctx）上——见下方「注册命令」节。

## 核心代码

```ts
// my-extension.ts
import { defineTool } from "@earendil-works/pi-coding-agent";
import { Type } from "typebox";

export default (pi) => {
  // --- 生命周期 hook ---

  pi.on("agent_start", (event, ctx) => {
    // event: { type: "agent_start" }，几乎不带数据
    console.log("[MyExt] Agent 启动");
    // ⚠️ setStatus 在 mode="print" 下是 no-op，生产代码建议加守卫
    if (ctx.hasUI) {
      ctx.ui.setStatus("my-ext", "MyExt 已就绪");
    }
  });

  pi.on("tool_call", (event, ctx) => {
    // event.toolName: "bash" | "read" | "edit" | "write" | "grep" | "find" | "ls" | string
    // event.input: 工具参数（可 mutate 改参数，见 E01）
    // event.toolCallId: string
    console.log(`[MyExt] 工具调用: ${event.toolName}`);
    // 如需阻断：return { block: true, reason: "..." }
    // ⚠️ handler 抛异常会等同 block（见陷阱 3）
  });

  pi.on("tool_result", (event, ctx) => {
    // event.isError: boolean（是否错误）
    // event.content: (TextContent | ImageContent)[]（返回给 LLM 的内容）
    // event.details: 工具特定的结构化数据
    if (event.isError) {
      // 美化错误信息
      // 返回值字段都是可选：content? / details? / isError? / usage?
      return {
        content: [{ type: "text", text: "(操作失败，请尝试其他方法)" }],
        isError: false,
      };
    }
  });

  pi.on("agent_end", (event, ctx) => {
    // event: { type: "agent_end", ... }（不可作唯一结束信号，见 04-events 坑 1）
    if (ctx.hasUI) {
      ctx.ui.setStatus("my-ext", undefined); // 清除状态栏
    }
    console.log("[MyExt] Agent 结束");
  });

  // --- 注册自定义工具 ---

  pi.registerTool(
    defineTool({
      name: "word_count",
      label: "Word Count",
      description: "Count words in text",
      parameters: Type.Object({
        text: Type.String({ description: "Text to count" }),
      }),
      // execute 签名：(toolCallId, params, signal, onUpdate, ctx) => Promise<AgentToolResult>
      // 注意：AgentToolResult 没有 isError 字段（与 tool_result handler 不同）
      async execute(toolCallId, params, signal, onUpdate, ctx) {
        const count = params.text.split(/\s+/).length;
        return {
          content: [{ type: "text", text: `词数: ${count}` }],
          details: { count },
        };
      },
    }),
  );

  // --- 注册自定义命令 ---
  // TUI 下用户输入 /status 触发；Web / Server 集成下
  // 调用 session.prompt("/status args") 同样会触发（见陷阱 4）

  pi.registerCommand("status", {
    description: "显示扩展状态",
    // handler 签名：(args: string, ctx: ExtensionCommandContext) => Promise<void>
    // 注意 args 是字符串（用户输入的参数），不是对象
    // ctx 是 ExtensionCommandContext（继承 ExtensionContext + waitForIdle 等会话控制方法）
    handler: async (args, ctx) => {
      await ctx.waitForIdle(); // ⚠️ 只能在 command handler 里调，事件 handler 没有
      ctx.ui.notify("MyExt 正在运行中");
    },
  });
};
```

## ExtensionCommandContext（命令 handler 的 ctx）

命令 handler 接收的 `ctx` 是 `ExtensionCommandContext`，**继承 `ExtensionContext`** 并扩展了会话控制方法：

| 字段 / 方法 | 含义 |
|-----------|------|
| `ctx.waitForIdle()` | 等待 agent 完成流式输出 |
| `ctx.newSession(options?)` | 开启新会话（可指定 parentSession / setup） |
| `ctx.fork(entryId, options?)` | 从特定 entry 分叉新会话 |
| `ctx.navigateTree(targetId, options?)` | 导航到会话树中的其他节点 |
| `ctx.switchSession(sessionPath, options?)` | 切换到另一个会话文件 |
| `ctx.reload()` | 重新加载当前会话 |
| `ctx.getSystemPromptOptions()` | 获取当前系统提示词构建选项 |

证据：extensions/types.ts。

**注意**：事件 handler（如 `pi.on("agent_start", ...)`）的 ctx 是 `ExtensionContext`，**没有这些方法**。在事件 handler 里调 `ctx.waitForIdle()` 会运行时报错。

> ⚠️ **会话切换后旧 ctx 失效**：调用 `ctx.newSession()` / `fork()` / `switchSession()` / `reload()` 后，**旧 ctx 与此前捕获的 pi 实例全部失效**（runner 被标记为 stale，后续调用 `assertActive()` 抛错）。`newSession` / `fork` / `switchSession` 的后续逻辑必须放进 `withSession` 回调、用回调里传入的新 ctx；`reload` 后也不要再用 await 之前捕获的旧 ctx。证据：runner.ts `invalidate` / `assertActive`。

## 陷阱与坑

### 坑 1：`tool_call` 是扩展独有事件，subscribe 静默失败

server 层 `session.subscribe("tool_call", ...)` 不会触发。要在扩展里订阅 `tool_call` 才能收到。6 个扩展独有事件：`context` / `tool_call` / `tool_result` / `before_agent_start` / `input` / `model_select`。**扩展独有 vs 双层 vs subscribe 独有的权威派发分类见 [04-events.md 坑 4](../sdk_doc/04-events.md)**，本场景只给概要。

### 坑 2：`ctx.ui.*` 默认是 no-op（mode 守卫）

`ExtensionMode` 默认是 `"print"`（如 SDK 集成 / `--print` 模式），此时 `ctx.ui` 是 `noOpUIContext`——所有 UI 方法都是空操作。证据：runner.ts、runner.ts。

**生产代码必须加守卫**：

```ts
pi.on("agent_start", (event, ctx) => {
  if (ctx.hasUI) {           // 或 ctx.mode === "tui"
    ctx.ui.setStatus("my-ext", "已就绪");
  }
});
```

### 坑 3：`tool_call` handler 抛异常等同 block，但 reason 不可控

`emitToolCall` **内部没有 try/catch**（与 `emitToolResult` / `emitAgentStart` 等不同），handler 抛异常会冒泡到外层 `beforeToolCall` 钩子。外层把异常转成 "Extension failed, blocking execution" 错误——**等同 block，但 reason 不可控**。

证据：runner.ts（无 try/catch）、agent-session.ts（外层 try/catch）。

**建议**：在 `tool_call` handler 内部包自己的 try/catch，返回可控的 `{ block: true, reason: "..." }`：

```ts
pi.on("tool_call", (event, ctx) => {
  try {
    // 业务逻辑
  } catch (err) {
    return { block: true, reason: `MyExt 校验失败: ${err}` };
  }
});
```

> ⚠️ 对比：`emitToolResult` 有 try/catch，handler 异常被捕获并 `emitError`，**不影响 agent**。详见 [场景 E01](E01-tool-intercept.md) 的「emitToolCall 执行机制」专节。

### 坑 4：`registerCommand` 的触发入口是"任何 `/` 开头的 prompt"，不是 TUI 专属

`pi.registerCommand` 注册的命令在 TUI 下由用户输入 `/命令名` 触发；在 **Web / Server 集成下，调用 `session.prompt("/命令名 参数")` 同样会触发**——`prompt()` 内部识别 `/` 前缀并路由到 `_tryExecuteExtensionCommand`，与运行模式无关。RPC 模式还通过 `get_commands` 把扩展命令清单暴露给客户端。

**误判后果**：若按"命令仅 TUI 可用"假设设计 Web 集成，会误以为不能用命令，实际可行。反过来——如果命令里有副作用（写文件、发请求），在 Web/Server 下被 `session.prompt("/cmd")` 误触也要防范。

证据：`prompt()` 内 `/` 路由 agent-session.ts、`_tryExecuteExtensionCommand` agent-session.ts、RPC `get_commands` 暴露命令清单 rpc-mode.ts。

### 坑 5：`ctx.waitForIdle()` 只在 command handler 里可用

事件 handler（如 `agent_start` / `tool_call`）的 ctx 是 `ExtensionContext`，**没有** `waitForIdle` / `newSession` / `fork` 等会话控制方法。这些方法只在 `ExtensionCommandContext`（命令 handler 的 ctx）上。

### 坑 6：扩展文件路径错误会被静默忽略

如果扩展路径不存在 / 文件没 `export default` 一个函数，加载器会记录 error，但**不会中断会话**。排查时检查 stderr 是否有 "Extension does not export a valid factory function" 等信息。证据：loader.ts。

## 扩展层其他能力：registerProvider

除了本场景聚焦的 hook / tool / command 三件套，扩展还能**注册自定义 LLM 供应商**。两种重载（types.ts）：

```ts
pi.registerProvider(provider: Provider);            // 传 Provider 对象
pi.registerProvider(name: string, config: ProviderConfig); // 传名字 + 配置
```

用途：让扩展接入非内置的 LLM 供应商。证据：extensions/types.ts。

## 变体与延伸

- 工具拦截细节（字段对照、阻断机制、改参方式）→ [场景 E01](E01-tool-intercept.md)
- 生命周期事件时序与完整清单 → [场景 E04](E04-lifecycle-hooks.md)
- 加载已有扩展（路径 / npm / git）→ [场景 A06](A06-load-extensions.md)
- 危险命令二次确认 → [场景 D04](D04-confirm-destructive.md)（命令层闸门）
- 工具结果美化渲染 → [场景 D05](D05-tool-result-render.md)
- 路径白名单（限制工具访问目录）→ [场景 D06](D06-protected-paths.md)
- 输入变换 → [场景 E05](E05-input-transform.md)
- 流式输出变换 → [场景 E06](E06-streaming-transform.md)

## 横向联动

- [sdk_doc/04-events.md](../sdk_doc/04-events.md) — 完整事件清单 + 扩展独有 vs 双层派发分类
- [sdk_doc/06-tools.md](../sdk_doc/06-tools.md) — 内置工具 + `defineTool` + customTools 参数
- [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) — ExtensionAPI 完整接口
- [sdk_doc/08-resource-loader.md](../sdk_doc/08-resource-loader.md) — DefaultResourceLoader 选项
