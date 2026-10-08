# 场景：拦截与变换用户输入 (E05)

## 什么时候用 / 不用会怎样

**该用本场景**：

- **前置命令解析（非 `/` 前缀）**：解析自定义前缀命令（如 `!help`、`!status`、`!clear`），在 Agent 看到前完成处理
- **自动注入项目上下文**：在每次用户输入末尾追加运行时信息（如工作目录、git 分支、时间戳），省得 prompt 里写死
- **输入规范化**：trim 空白、限制最大长度、统一全半角、替换敏感词
- **多语言预处理**：把用户中文输入先翻译成英文再喂给英文模型
- **审计 / 日志**：所有用户输入落库前先过一道记录

**不用会怎样**：

- 想统一加项目上下文只能写进 systemPrompt——但 systemPrompt 是静态的，拿不到「这一轮用户具体输入了什么」
- 想做命令分发只能用 `registerCommand`——但那只能处理 `/` 开头的命令，且每个命令一个 handler，无法统一拦截
- 想做敏感词过滤只能依赖模型自觉——prompt 注入或模型理解偏差时直接绕过

**不适合本场景**：

- 注册 `/` 开头的命令 → 见 [场景 E02 扩展基础](E02-extension-basics.md)（`registerCommand`）。**注意 `/` 前缀的输入根本到不了 input 事件**——见下方陷阱 1（4 级命令解析优先级）
- 修改系统提示词（每轮动态注入规则） → 见 [场景 E04 模式一](E04-lifecycle-hooks.md#模式一用-before_agent_start-改系统提示词)（`before_agent_start` 钩子）
- 改工具调用参数 / 阻断工具执行 → 见 [场景 E01](E01-tool-intercept.md)（`tool_call` 钩子）
- 改 assistant 的最终回复 → 见 [场景 E04 模式二](E04-lifecycle-hooks.md#模式二用-message_end-替换最终消息)（`message_end` 钩子）
- 流式渲染逐 token 的 assistant 输出 → 见 [场景 E06](E06-streaming-transform.md)（`message_update`）

---

## 范围（★ 先看这个）

本场景聚焦 **`input` 事件**——用户提交文本（或图片）后、Agent 真正处理前的拦截点。唯一能力是**改文本 / 改图片 / 短路**，不涉及系统提示词、工具调用、消息流。

不在本场景展开：

- `/command` 命令注册机制 → [E02](E02-extension-basics.md)
- 系统提示词动态修改 → [E04 模式一](E04-lifecycle-hooks.md#模式一用-before_agent_start-改系统提示词)
- 工具拦截 → [E01](E01-tool-intercept.md)

---

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `pi.on("input", handler)` | 拦截用户输入，可变换文本 / 短路 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `return { action: "continue" }` | 显式放行（等价于不 return） | 本文档 |
| `return { action: "transform", text, images? }` | 替换文本 / 图片，Agent 看到的是替换后的内容 | 本文档 |
| `return { action: "handled" }` | 标记已处理，**Agent 完全收不到这条输入** | 本文档 |

> ⚠️ **关键集成坑 1：`input` 是扩展独有事件**（与 `context` / `tool_call` / `tool_result` / `before_agent_start` / `model_select` / `user_bash` / `resources_discover` / `session_*` 等并列——这类事件仅在 ExtensionAPI 的 `on()` 上可注册，不在 `AgentSessionEvent` / `AgentEvent` 联合类型里）。在 server 层用 `session.subscribe("input", ...)` 会**静默失败**——`AgentSessionEvent` 联合类型（agent-session.ts）根本没有 `input` type，handler 注册了但永远不被调用。详见 [04-events.md 坑 4](../sdk_doc/04-events.md#坑-46-个扩展独有事件sessionsubscribe-静默收不到-最大集成坑)。

> ⚠️ **关键集成坑 2：`ctx.ui.notify` 在 SDK 场景是 no-op**。`ExtensionMode` 默认是 `"print"`，此时 `ctx.ui` 是 `noOpUIContext`——`notify` / `setStatus` 不报错但不显示，`confirm` 返回 false（runner.ts）。生产代码必须加 `ctx.hasUI` 守卫，详见 [E04 陷阱 1](E04-lifecycle-hooks.md#陷阱-1ctxui-默认是-no-opmode-守卫)。

---

## 陷阱 1（★ 最大坑）：4 级命令解析优先级——`/` 开头根本到不了 input 事件

这是 input 事件最容易踩的坑。用户提交一段文本后，SDK 的处理顺序（agent-session.ts）：

```
用户输入 text
    ↓
① 扩展命令解析（text.startsWith("/")）
    ↓ 若 /xxx 匹配到 registerCommand 注册的命令 → 执行后 return，input 事件不触发
② emitInput 事件（本场景）
    ↓ handler 可以变换 / 短路
③ Skill 命令展开（/skill:name args）
    ↓
④ Prompt template 展开（/template args）
    ↓
⑤ 原样作为 user message 喂给 LLM
```

**关键事实**：

- 第 ① 步只在 `text.startsWith("/")` 时尝试（agent-session.ts）。匹配规则是 `_tryExecuteExtensionCommand`——从 `/` 后取命令名，查 `registerCommand` 注册表，命中就执行后 return，**input 事件根本不会被调用**
- 第 ② 步（input 事件）**在 skill / template 展开之前**——意味着 handler 看到的是用户原始输入，未经任何展开
- 第 ③④ 步用 `expandedText`（input 事件可能变换后的文本）做展开

**实践建议**：

- 想用 input 事件做命令解析，**命令前缀不能用 `/`**（会被 ① 截胡）。可以用 `!`、`#`、`@` 等
- 想注册 `/command`，走 [E02 `registerCommand`](E02-extension-basics.md)。**两者可以并存**：`registerCommand` 管 `/` 前缀（命中即执行，不触发 input 事件），input 事件管 `!` / `#` / `@` 等其他前缀——互不干扰。**真正的"互斥"只针对同一个 `/` 前缀**：`/cmd` 被 `_tryExecuteExtensionCommand` 截胡后不会进 input 事件（证据：agent-session.ts 的 `if (expandPromptTemplates && text.startsWith("/"))` 守卫只对 `/` 开头生效）
- 想拦截 `/skill:` 或 `/template`，input 事件**做不到**——这两个展开发生在 input 之后。需要走 `before_agent_start` 拿 `prompt` 字段（已经是展开后的）

证据：agent-session.ts、_tryExecuteExtensionCommand。

> ⚠️ **例外**：`expandPromptTemplates=false` 时，`/` 开头的文本**不尝试命令匹配**（agent-session.ts 的条件守卫 `if (expandPromptTemplates && text.startsWith("/"))`）。此时 `/xxx` 会原样进入 input 事件。`expandPromptTemplates` 默认为 `true`，但可以通过 `session.prompt(message, { expandPromptTemplates: false })` 显式关闭。

---

## InputEvent payload 字段

handler 第一个参数 `event`（types.ts）：

| 字段 | 类型 | 说明 |
|------|------|------|
| `type` | `"input"` | 事件类型标识 |
| `text` | `string` | 用户输入的文本（未经任何展开，是原始文本） |
| `images` | `ImageContent[] \| undefined` | 用户附带的图片（如截图、粘贴的图） |
| `source` | `"interactive" \| "rpc" \| "extension"` | 输入来源——`interactive` 是用户在 TUI 直接输入；`rpc` 是 SDK / RPC 通道调用 `session.prompt()`；`extension` 是其他扩展通过 `pi.sendUserMessage()` 发送的。不指定时默认为 `"interactive"`（`options?.source ?? "interactive"`） |
| `streamingBehavior` | `"steer" \| "followUp" \| undefined` | 仅在 agent 正在流式处理时（isStreaming=true）有值，指定新消息如何排队。idle 状态为 `undefined` |

> **`source` 的用途**：可以根据来源差异化处理（如 rpc 来源强制做敏感词过滤、interactive 来源放宽规则）。
>
> **`streamingBehavior` 的用途**：知道用户想 steer（插入当前 turn）还是 followUp（下一轮处理）后，可以决定是否做异步重活（steer 场景 handler 不能阻塞太久）。
>
> **官方标杆示例**（`examples/extensions/input-transform-streaming.ts`）：用户提到 "diff/changes/modified" 时注入 `git diff --stat` 作为上下文，但 steer 期间跳过 exec——因为 steer 是用户在模型说话中途打岔纠正，**延迟比上下文更重要**。核心模式：
>
> ```ts
> pi.on("input", async (event) => {
>   // steer 期间跳过耗时的 git exec，让纠正尽快送达模型
>   if (event.streamingBehavior === "steer") return { action: "continue" };
>   if (!/\b(changes?|diff|modified)\b/i.test(event.text)) return { action: "continue" };
>   const { stdout, code } = await pi.exec("git", ["diff", "--stat"]);
>   if (code !== 0 || !stdout.trim()) return { action: "continue" };
>   return { action: "transform", text: `${event.text}\n\nCurrent uncommitted changes:\n\`\`\`\n${stdout.trim()}\n\`\`\`` };
> });
> ```
>
> 这是 `streamingBehavior` 最有代表性的用法——**idle 时做重 transform，steer 时降级放行**。

## InputEventResult：handler 返回值（★ 核心）

`InputEventResult` 是**联合类型**，有 3 个合法 action（types.ts）：

| 返回值 | 行为 | 典型用途 |
|--------|------|---------|
| `{ action: "continue" }` | 显式放行——文本不变，Agent 看到原始输入 | 等价于不 return，但更显式 |
| `{ action: "transform", text: string, images?: ImageContent[] }` | 替换文本（必填）和 / 或图片（可选）。Agent 看到替换后的内容 | 注入项目上下文、翻译、规范化 |
| `{ action: "handled" }` | 短路——Agent **完全收不到**这条输入，prompt 不发，turn 不启动 | 自定义命令分发（`!help` 类）、敏感词静默拦截 |
| 不 return（undefined） | 等价于 `"continue"`，放行 | 简单的只读 hook（如日志） |

> **只想改图片、文本不动？`text` 仍是必填**。`transform` 的 `text` 字段不可省略——"只改图片"时要把 `event.text` 原样回传，否则类型不通过：
>
> ```ts
> // ❌ 错：省略 text，类型校验失败（TS 报错 / 运行时丢文本）
> pi.on("input", (event, ctx) => {
>   return { action: "transform", images: event.images?.filter(notUnsafe) };
> });
>
> // ✅ 对：只过滤图片，文本原样回传
> pi.on("input", (event, ctx) => {
>   if (!event.images?.length) return;  // 没图片，放行
>   const safe = event.images.filter((img) => !isUnsafeImage(img));
>   return { action: "transform", text: event.text, images: safe };
> });
> ```
>
> 证据：`InputEventResult` 的 transform 分支是 `{ action: "transform"; text: string; images?: ImageContent[] }`——`text` 必填，`images` 可选（types.ts）。

> ⚠️ **关键陷阱 2：mutation 不生效**。
>
> 看源码 `emitInput`（runner.ts）：handler 调用前会**重建 event 对象**，传入 `currentText` 的副本。handler 内部修改 `event.text = "xxx"` **不会更新 runner 持有的 `currentText`**。如果不返回 `{ action: "transform", text }`，下一轮 handler 和最终发给 Agent 的还是原始文本。
>
> ```ts
> // ❌ 错：mutation 无效
> pi.on("input", (event, ctx) => {
>   event.text = event.text.trim();  // ← 改了也没用
> });
>
> // ✅ 对：必须 return transform
> pi.on("input", (event, ctx) => {
>   return { action: "transform", text: event.text.trim() };
> });
> ```
>
> 这与 `tool_call` 的 mutation 模式（[E01](E01-tool-intercept.md#二级静默修改mutation)）**完全不同**——`tool_call` 的 `event.input` 是同一引用，mutation 跨 handler 透明；`input` 事件的 `event` 每次循环重建，mutation 无效。

## ExtensionContext（handler 的 ctx）

handler 第二个参数 `ctx: ExtensionContext`——**与所有其他事件 handler 共享同一类型**。完整字段表见 [E04 ExtensionContext](E04-lifecycle-hooks.md#extensioncontext事件-handler-的-ctx) 或 [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md#extensioncontext)。

**最常用字段**：

| 字段 | 用法 |
|------|------|
| `ctx.cwd` | **不要用 `process.cwd()`**——`ctx.cwd` 才是 session 配置的工作目录（[E01 陷阱 1](E01-tool-intercept.md#陷阱-1processcwd-vs-ctxcwd)） |
| `ctx.hasUI` / `ctx.mode` | 决定能否调 `ctx.ui.*`（SDK 集成默认 `print` 模式 no-op，[E04 陷阱 1](E04-lifecycle-hooks.md#陷阱-1ctxui-默认是-no-opmode-守卫)） |
| `ctx.sessionManager` | 只读会话管理（拿历史消息、当前 session 信息） |

---

## emitInput 执行机制（★ 多 handler 必读）

`emitInput`（runner.ts）：

> ⚠️ **hasHandlers 守卫**：agent-session.ts 在调用 `emitInput` 前会检查 `hasHandlers("input")`——如果没有扩展注册了 `input` handler，`emitInput` **根本不会被调用**，零开销跳过。

```ts
async emitInput(text, images, source, streamingBehavior): Promise<InputEventResult> {
  const ctx = this.createContext();
  let currentText = text;
  let currentImages = images;

  for (const ext of this.extensions) {
    for (const handler of ext.handlers.get("input") ?? []) {
      try {
        const event: InputEvent = {
          type: "input",
          text: currentText,           // ← 注意：每次重建 event，传 currentText 的当前值
          images: currentImages,
          source,
          streamingBehavior,
        };
        const result = await handler(event, ctx);
        if (result?.action === "handled") return result;        // ← handled 立即短路
        if (result?.action === "transform") {
          currentText = result.text;                              // ← transform 更新 currentText
          currentImages = result.images ?? currentImages;
        }
        // continue / undefined → 不改 currentText，进下一个 handler
      } catch (err) {
        this.emitError({ extensionPath: ext.path, event: "input", error: ... });
        // ⚠️ 异常被 try/catch，不影响 agent 和后续 handler
      }
    }
  }
  return currentText !== text || currentImages !== images
    ? { action: "transform", text: currentText, images: currentImages }
    : { action: "continue" };
}
```

**五个关键特性**：

1. **串行执行**：handler 按 `extensionFactories` 数组顺序 + 每个扩展内 handler 注册顺序依次 `await`
2. **handled 短路**：任一 handler 返回 `{ action: "handled" }` 立即 `return`，后续 handler 不再调用（与 `tool_call` 的 block 短路类似）
3. **transform 链式覆盖**：前一个 handler 的 `transform` 更新 `currentText`，下一个 handler 看到的是**已变换的文本**（链式加工）
4. **mutation 无效**：event 每次循环重建，改 `event.text` 不影响 `currentText`——**必须 return transform**
5. **handler 异常被捕获**：try/catch + emitError，**不影响 agent 和后续 handler**——与生命周期钩子一致（[E04 陷阱 2](E04-lifecycle-hooks.md#陷阱-2生命周期钩子异常处理与-tool_call-不同)），与 `tool_call` 不同（[E01 陷阱 9](E01-tool-intercept.md#陷阱-9handler-抛异常会中断-agent-loop)）

**链式 transform 示例**：扩展 A 在文本末尾追加 `[A]`，扩展 B 追加 `[B]`。最终 Agent 看到的是 `原始文本[A][B]`——两个扩展都跑了，且 B 看到 A 的修改。

---

## 核心代码

三种递进模式：**只读 hook（日志）→ 变换（追加上下文）→ 短路（命令分发）**。

```ts
export default (pi) => {
  // ① 只读：记录所有用户输入（不返回 = 等价 continue）
  pi.on("input", (event, ctx) => {
    console.log(`[audit] source=${event.source} text=${event.text.slice(0, 80)}`);
    // 不 return，原始输入透明传递
  });

  // ② 变换：追加项目上下文 + 规范化
  pi.on("input", (event, ctx) => {
    const trimmed = event.text.trim();
    if (trimmed.length === 0) {
      // 空输入放行（让模型自己应对）
      return { action: "continue" };
    }
    // ⚠️ 用 ctx.cwd 不是 process.cwd()
    const tagged = `${trimmed}\n\n[运行时上下文: cwd=${ctx.cwd}, time=${new Date().toISOString()}]`;
    return { action: "transform", text: tagged };
  });

  // ③ 短路：自定义命令分发（前缀不能用 /，会被扩展命令截胡——见陷阱 1）
  pi.on("input", (event, ctx) => {
    const text = event.text;
    if (text.startsWith("!help")) {
      // ⚠️ ctx.ui.notify 在 SDK 场景（mode=print）是 no-op——加 hasUI 守卫
      if (ctx.hasUI) {
        ctx.ui.notify("可用命令: !help, !status, !clear", "info");
      } else {
        console.log("[my-ext] 可用命令: !help, !status, !clear");
      }
      return { action: "handled" };  // ← Agent 收不到这条输入
    }
    if (text.startsWith("!status")) {
      console.log("[my-ext] 系统正常运行中");
      return { action: "handled" };
    }
    // 其他输入：不 return，进下一个 handler
  });
};
```

> **代码块之间有呼吸**：上方展示了三类模式。下面分别展开两类最易踩坑的（命令分发 + 多 handler 链式 transform）。

---

## 模式一：自定义命令分发（`!` 前缀）

**场景**：在 Agent 之外提供本地命令（`!help` / `!status` / `!clear`），不消耗 LLM 调用。

**为什么不用 `registerCommand`**：

- `registerCommand` 只能注册 `/command`，前缀固定
- `registerCommand` 命中后**直接执行**，不会触发 input 事件（见陷阱 1 的 4 级优先级）
- 想用 `!` 等非 `/` 前缀，只能走 input 事件

```ts
const commands: Record<string, (args: string, ctx: ExtensionContext) => string> = {
  help: () => "可用命令: !help, !status, !clear <n>",
  status: (_args, ctx) => `cwd=${ctx.cwd}, idle=${ctx.isIdle()}`,
  clear: () => "（已清除，请在客户端实现真实清理逻辑）",
};

pi.on("input", (event, ctx) => {
  const match = event.text.match(/^!(\w+)(\s.*)?$/);
  if (!match) return;  // 不是命令，放行

  const [, name, argsPart] = match;
  const handler = commands[name];
  if (!handler) {
    if (ctx.hasUI) ctx.ui.notify(`未知命令: !${name}`, "warning");
    return { action: "handled" };  // 未知命令也短路，避免 !xxx 当普通文本喂给 LLM
  }

  const output = handler(argsPart?.trim() ?? "", ctx);
  if (ctx.hasUI) {
    ctx.ui.notify(output, "info");
  } else {
    console.log(`[my-ext] !${name}: ${output}`);
  }
  return { action: "handled" };
});
```

**关键点**：

- 前缀用 `!` 不用 `/`（避开扩展命令截胡）
- `ctx.hasUI` 守卫——SDK 场景回退到 `console.log`
- 未知命令也短路，避免 `!xxx` 当普通文本喂给 LLM 造成幻觉

---

## 模式二：多 handler 链式 transform（追加多层上下文）

**场景**：扩展 A 注入运行时上下文（cwd / 时间），扩展 B 做敏感词过滤，扩展 C 翻译。三者独立维护、可组合。

```ts
// extension A: 注入运行时上下文
pi.on("input", (event, ctx) => {
  return {
    action: "transform",
    text: `${event.text}\n\n[cwd=${ctx.cwd}, t=${Date.now()}]`,
  };
});

// extension B: 敏感词替换（看到 A 加过的文本）
const SENSITIVE = ["password", "token", "secret"];
pi.on("input", (event, ctx) => {
  let text = event.text;
  for (const word of SENSITIVE) {
    text = text.replaceAll(new RegExp(word, "gi"), "***");
  }
  // 注意：即便没替换也要返回 transform（保持链路一致），否则后续 handler 看到的还是原始文本
  return { action: "transform", text };
});

// extension C: 中文 → 英文翻译（异步）
pi.on("input", async (event, ctx) => {
  if (!/[\u4e00-\u9fa5]/.test(event.text)) return;  // 无中文，放行
  const translated = await translateText(event.text, "zh", "en");  // translateText 是用户自定义的翻译函数（非 SDK API）
  return { action: "transform", text: translated };
});
```

**链式顺序的重要性**：

- 顺序由 `createAgentSession({ extensions: [A, B, C] })` 数组顺序决定
- A → B → C：B 看到 A 的输出，C 看到 B 的输出
- 如果把 C（翻译）放最前面，B 的敏感词正则就匹配不到中文了

**handler 异常**：单个 handler 抛异常会被 try/catch（runner.ts），记录到扩展错误流，**不影响后续 handler 和 agent**。

---

## 关键陷阱（5 条）

### 陷阱 1：`/` 前缀命令被扩展命令截胡

见上方 [陷阱 1（★ 最大坑）](#陷阱-1最大坑4-级命令解析优先级开头根本到不了-input-事件)。简而言之：想用 input 事件做命令分发，前缀不能用 `/`。

### 陷阱 2：mutation 无效（与 tool_call 不同）

`emitInput` 每次循环重建 event 对象，传 `currentText` 的**值副本**。handler 内部改 `event.text` 不影响 `currentText`——**必须 return transform**。

对比 `tool_call`：`event.input` 是同一引用，mutation 跨 handler 透明（[E01](E01-tool-intercept.md#二级静默修改mutation)）。两者机制完全不同，不要混淆。

证据：runner.ts（每次循环新建 `event: InputEvent = {...}`）。

### 陷阱 3：`ctx.ui.*` 在 SDK 场景是 no-op

`ExtensionMode` 默认 `"print"`（SDK 集成场景），`ctx.ui` 是 `noOpUIContext`——`notify` / `setStatus` 不报错但不显示。生产代码必须加 `ctx.hasUI` 守卫。

证据：runner.ts、runner.ts（默认 mode）。详见 [E04 陷阱 1](E04-lifecycle-hooks.md#陷阱-1ctxui-默认是-no-opmode-守卫)。

### 陷阱 4：`process.cwd()` ≠ `ctx.cwd`

SDK 集成场景下扩展进程的 `process.cwd()` 不等于 session 的 `cwd`。`createAgentSession({ cwd })` 是必填配置，handler 内取工作目录必须用 `ctx.cwd`（types.ts）。

详见 [E01 陷阱 1](E01-tool-intercept.md#陷阱-1processcwd-vs-ctxcwd)。

### 陷阱 5：`input` 是扩展独有事件（subscribe 静默失败）

server 层 `session.subscribe("input", ...)` **永远不会被触发**，且不报错不警告。`AgentSessionEvent` 联合类型（agent-session.ts）根本没有 `input` type。

想在 server 层变换用户输入只能在调用 `session.prompt()` 前自己改文本——扩展层的 input 事件**到不了** server 层。

证据：types.ts（`on(event: "input", ...)` 只在 ExtensionAPI 上）、agent-session.ts（AgentSessionEvent 无 input）。

---

## 变体与延伸

- **配合 `session.steer()` 在流式过程中注入消息** → **直接调 `session.steer(text)` / `session.followUp(text)` 不触发 input 事件**（这两个方法走 `_expandSkillCommand` + `_queueSteer`/`_queueFollowUp`，不经 `emitInput`）。**但流式期间用 `session.prompt(text, { streamingBehavior: "steer" })` 提交的消息仍会触发 input 事件**——`prompt` 里的 `emitInput` 调用不区分 streaming 状态（只是 `streamingBehavior` 参数在 idle 时传 `undefined`）。即"steer 的消息走不走 input"取决于入口方法，不是取决于"是不是 steer"（[F05](F05-steer-session.md)）
- **基于 session 状态的动态规则** → `session_start` 初始化闭包变量，input handler 里使用（详见 [E04 变体 F](E01-tool-intercept.md#变体-f基于-session-状态的动态规则)）
- **改系统提示词（每轮动态注入规则）** → 用 `before_agent_start`（[E04 模式一](E04-lifecycle-hooks.md#模式一用-before_agent_start-改系统提示词)），不是 input 事件
- **流式渲染逐 token 的 assistant 输出** → [E06](E06-streaming-transform.md)（`message_update`）
- **过滤图片输入** → input 事件可以 `transform` 时替换 `images` 字段（如剥离不安全图片）
- **审计所有用户输入** → 只读 hook（不 return）+ 落库（注意 source 区分 interactive / rpc / extension）

---

## 横向联动

- [E01 工具拦截](E01-tool-intercept.md)：`tool_call` 钩子（mutation 模式 vs input 的 return transform 模式——注意两者机制不同）
- [E02 扩展基础](E02-extension-basics.md)：`registerCommand`（管 `/` 前缀，命中即执行不触发 input 事件；与 input 事件处理的 `!command` 等其他前缀可并存）
- [E04 生命周期钩子](E04-lifecycle-hooks.md)：`before_agent_start` / `message_end`（改 systemPrompt / 改最终消息）
- [E06 流式变换](E06-streaming-transform.md)：`message_update` 流式逐 token 处理
- [C02 prompt 模板](C02-prompt-templates.md)：`/template` 展开机制（在 input 事件之后才展开）
- [F05 会话 steer](F05-steer-session.md)：直接调 `steer()`/`followUp()` 不触发 input 事件，但流式期间用 `prompt(text, { streamingBehavior: "steer" })` 仍触发
- [sdk_doc/04-events.md](../sdk_doc/04-events.md)：事件系统（含扩展独有事件警示）
- [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md)：ExtensionAPI 完整接口
