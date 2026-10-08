# 场景：通过扩展注册与覆盖工具 (D02)

> ⭐ **先读**：本场景只讲「**通过扩展系统**（`pi.registerTool`）注册工具」的 API 用法。如果要写一个全新的自定义工具定义，先看 [D01-custom-tool.md](D01-custom-tool.md) 的 `defineTool` 详解和 `execute()` 五参数签名；本场景默认你已掌握 D01 的核心数据模型（`ToolDefinition` / `AgentToolResult` / `ExtensionContext` / `onUpdate`）。

## 什么时候用 / 不用会怎样

**适合**：
1. **工具需要随扩展生命周期管理**——按需启用/禁用、复用跨会话、与其他扩展能力（事件 hook、命令）组合
2. **覆盖或包装内置工具**——给 `read` 加路径白名单、给 `bash` 加命令审计、给 `write` 加权限校验
3. **运行时按条件注册**——例如扩展启动时检测到 `.pi/skills/` 存在才注册 `skill_search` 工具；或检测到 OAuth token 才注册需要授权的工具

**不适合**：
- 工具定义只在一个 SDK 调用点用 → 直接 `createAgentSession({ customTools: [...] })`，更简单（见 [D01](D01-custom-tool.md)）
- 需要在工具执行前做拦截/修改/阻止 → 用 `pi.on("tool_call")` 事件（见 [E01-tool-intercept.md](E01-tool-intercept.md)），不要用覆盖
- 想完全替换某个内置工具的语义（如把 `read` 改成"读数据库"）→ **强烈不建议**，模型仍按内置语义来调用，会造成行为混乱；用新工具名（如 `db_lookup`）

**不用会怎样**：每次 SDK 集成都要在 `createAgentSession` 调用点重复传 `customTools`；无法复用；无法包装内置工具；无法做条件注册。

---

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `pi.registerTool(tool)` | 在扩展内注册工具（含覆盖同名内置工具） | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `defineTool` | 包装工具定义（**仅传给 customTools 数组时需要**；`pi.registerTool` 不必包） | [sdk_doc/06-tools.md](../sdk_doc/06-tools.md) |
| `ExtensionAPI`（默认导出函数的 `pi` 参数类型） | 扩展入口 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `createReadToolDefinition` 等工厂 | 拿到内置工具的 ToolDefinition 用于包装 | [sdk_doc/06-tools.md §内置工具的安全边界](../sdk_doc/06-tools.md) |

> **对称工厂清单**：除了 `createReadToolDefinition`，还有 `createBashToolDefinition` / `createEditToolDefinition` / `createWriteToolDefinition`（都在 `tools/index.ts` 导出）。包装 `bash`/`edit`/`write` 时用法与变体 A 完全对称。

> **ToolDefinition 新增字段（v0.82.0+）**：`constrainedSampling` 用于请求 provider 侧结构化输出约束。类型是 `false | ConstrainedSamplingConfig`——显式置 `false` 可禁用该约束（等价于 `undefined`）。详见 [sdk_doc/06 §ToolDefinition](../sdk_doc/06-tools.md)。

> ⚠️ **import 来源**：TypeBox 从 `typebox` 包导入（CHANGELOG v0.69.0 已把第一方代码迁移到 `typebox` 1.x）。
> ```ts
> import { Type } from "typebox";                              // ✅ 推荐（新代码必须用）
> import { Type } from "@sinclair/typebox";                    // ⚠️ 不推荐；根 import 仍向后兼容（loader 把 `@sinclair/typebox` 别名到 `typebox`），但新代码请用 `typebox`
> ```
> loader 的别名表同时覆盖根路径、`/compile`、`/value` 子路径（`loader.ts` 的 `VIRTUAL_MODULES` 与 `getAliases`）。所以根 import 不会报错，但官方迁移方向明确是 `typebox`——为避免未来移除别名后踩坑，新代码一律用 `typebox`。

---

## 工具启用规则

`createAgentSession` 的工具启用规则（`sdk.ts` 的 `allowedToolNames` / `initialActiveToolNames` 推导 + `agent-session.ts` 的 `_refreshToolRegistry` 过滤执行）：

| 配置 | 启用的工具 |
|------|-----------|
| 不传 `tools` 且不传 `noTools` | `["read", "bash", "edit", "write"]` + **所有 customTools + 所有扩展工具** |
| `tools: [...]` 显式提供 | **只启用列表中的**（customTools 和扩展工具必须显式列入才生效） |
| `noTools: "all"` | 全部禁用（连 customTools 和扩展工具也禁用） |
| `noTools: "builtin"` | 禁用 4 个默认内置（`read/bash/edit/write`），**保留** customTools 和扩展工具 |
| `excludeTools: ["bash"]` | 在 `tools` 之后应用，按名排除单个（同样作用于扩展工具和 customTools） |

**关键事实**（`agent-session.ts`）：`_refreshToolRegistry` 内的过滤逻辑是：
```ts
const isAllowedTool = (name: string): boolean =>
  (!allowedToolNames || allowedToolNames.has(name)) && !excludedToolNames?.has(name);

const allCustomTools = [...registeredTools, ...this._customTools.map(...)]
  .filter((tool) => isAllowedTool(tool.definition.name));  // ← 扩展工具和 customTools 走同一过滤
```

> ⚠️ **覆盖优先级**：`allCustomTools` 数组中 `customTools` 排在扩展工具之后，`definitionRegistry.set` 后者覆盖前者——所以同名时 `customTools` 覆盖扩展工具。

所以：
- 不传 `tools`（也不传 `noTools: "all"`）→ `_allowedToolNames = undefined`（白名单不收紧）。此时扩展工具 **自动启用**，靠的不是 `allowedToolNames`，而是构造期 `_buildRuntime({ includeAllExtensionTools: true })` 传入的 `includeAllExtensionTools` 标志——`_refreshToolRegistry` 在 `allowedToolNames` 为空时走该分支把所有扩展工具加入 active 集合（`agent-session.ts`）
- `noTools: "builtin"` → `initialActiveToolNames = []`（清空内置默认工具，sdk.ts）但 `includeAllExtensionTools` 仍为 `true`，所以 customTools 和扩展工具保留
- 传 `tools: ["read", "bash"]` 但不含 `"my_ext_tool"` → `my_ext_tool` **被禁用**（白名单收紧，`isAllowedTool` 返回 false；即便扩展里 `pi.registerTool` 已注册）

> ⚠️ **常见踩坑**：以为「扩展里 registerTool 了就一定生效」。实际仍受 `createAgentSession({ tools })` 白名单约束。**两种正确做法**：要么 `tools` 省掉，要么 `tools` 显式包含扩展工具名。

---

## 核心机制：registerTool 的覆盖语义

`pi.registerTool(tool)` 在扩展加载时把 `tool.name → tool` 写入 `extension.tools` Map（`loader.ts`）。session 初始化时，`_refreshToolRegistry` 把所有扩展工具和 customTools 合并进 `definitionRegistry`（`agent-session.ts`）：

```ts
// 简化逻辑
for (const tool of allCustomTools) {
  definitionRegistry.set(tool.definition.name, {  // ← Map.set 同名覆盖
    definition: tool.definition,
    sourceInfo: tool.sourceInfo,
  });
}
```

**Map.set 同名覆盖**意味着：扩展注册的工具会**覆盖同名的内置工具**或先注册的其他扩展工具。这是"包装内置工具"模式的实现基础。

> ⚠️ **覆盖同时作用于两层**：`_refreshToolRegistry` 不仅把同名覆盖写进 `_toolDefinitions`（prompt/definition 层，决定模型看到的工具列表），也写进 `_toolRegistry`（运行时调用层，决定模型调用时实际执行的实现）。具体是先用 `wrappedBuiltInTools` 建 `_toolRegistry`，再用 `wrappedExtensionTools`（含 customTools，顺序同 `allCustomTools`：扩展工具在前、customTools 在后）`.set()` 覆盖（`agent-session.ts`）。所以同名替换会**真正改变运行时行为**，不只是 prompt 层。优先级一致：**customTools 胜出 > 扩展工具 > 内置工具**。

**注册时机约束**（`loader.ts` 注释）：
> `registerTool() is valid during extension load; refresh is only needed post-bind.`

也就是说，`pi.registerTool` **设计上就是在扩展默认导出函数体内调用**。完整时序链条是：

1. **扩展加载期**：`refreshTools` 是 no-op stub（`loader.ts`），所以加载期间调 `registerTool` 只是把工具写进 `extension.tools` Map，不会立即刷新
2. **session 绑定**：`ExtensionRunner.bindActions` 把真实 `refreshTools` 实现注入 runtime（`runner.ts`）
3. **首次刷新**：真正触发首次工具注册表刷新的是 `AgentSession` 构造期 `_buildRuntime → _refreshToolRegistry`（`agent-session.ts`），不是 runner 绑定动作本身——这一步把所有扩展工具和 customTools 合并进 definitionRegistry 和 toolRegistry

**不建议**在事件 handler 里调用 `pi.registerTool`：每次调用都会立即触发 `runtime.refreshTools()` 刷新工具注册表（而非等到下一轮 turn），可能导致非预期的工具列表变动，增加排查复杂度。

---

## 核心代码：注册新工具

```ts
// extensions/my-tools-extension.ts
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";
import { Type } from "typebox";

// factory 支持 async：默认导出可以是 (pi) => void 或 async (pi) => Promise<void>
// 需要先读远程配置 / 异步初始化再注册工具时用 async（loader 会 await factory(api)）
export default (pi: ExtensionAPI) => {
  // ✅ registerTool 本身就有泛型推断，不需要 defineTool 包裹
  pi.registerTool({
    name: "dynamic_query",
    label: "Dynamic Query",
    description: "Query external data at runtime. Use when the user asks about ...",
    parameters: Type.Object({
      query: Type.String({ description: "The query string" }),
    }),
    async execute(toolCallId, params, signal, onUpdate, ctx) {
      // 长任务建议检查 signal
      if (signal?.aborted) throw new Error("Aborted before start");
      const result = `Result for: ${params.query}`;
      return {
        content: [{ type: "text", text: result }],
        details: { source: "ext", length: result.length },
      };
    },
  });
};
```

**代码讲解**：
- `pi: ExtensionAPI` 是扩展默认导出函数的参数类型（建议显式标注）。**`pi` 是扩展的唯一入口句柄**——所有扩展能力（注册工具 / 注册命令 / 监听事件 / 切换工具集）都挂在这一个对象上，不要在 factory 返回后另外构造"扩展上下文"
- `pi.registerTool(tool)` 直接接收 ToolDefinition 对象，**不需要 `defineTool` 包裹**（D01 用 defineTool 是为了保 TypeBox 泛型推断不被数组类型收窄；registerTool 本身就有正确泛型推断，`types.ts`）
- `execute` 五参数签名与 D01 完全一致（`toolCallId` / `params` / `signal` / `onUpdate` / `ctx`），详见 [D01 §execute() 函数签名](D01-custom-tool.md)
- `description` 决定模型何时选择调用——写清楚「做什么」+「适合什么场景」

### 扩展如何被加载

扩展文件放对位置才会被发现：

| 位置 | 用途 |
|------|------|
| `<cwd>/.pi/extensions/*.ts` | 项目级扩展，自动发现 |
| `~/.pi/agent/extensions/*.ts` | 用户全局扩展，自动发现 |
| `settings.json` 的 `extensions: [...]` 数组 | 显式指定路径 |
| `new DefaultResourceLoader({ additionalExtensionPaths: [...] })` | SDK 集成时传入 |
| `new DefaultResourceLoader({ extensionFactories: [(pi) => {...}] })` | 内联工厂函数 |

SDK 集成示例（含 try/finally + dispose，**官方示例的标准模式**）：

```ts
import {
  createAgentSession,
  DefaultResourceLoader,
  SessionManager,
  getAgentDir,                            // ← 返回 ~/.pi/agent（受 PI_AGENT_DIR 环境变量影响）
} from "@earendil-works/pi-coding-agent";
import { Type } from "typebox";

// ⚠️ cwd / agentDir 是必填字段（DefaultResourceLoaderOptions 非 ?），漏传在严格模式下编译失败。
//    官方标准模式见 createAgentSession 的 JSDoc 示例（sdk.ts）。
const resourceLoader = new DefaultResourceLoader({
  cwd: process.cwd(),                     // ← 必填：项目根目录
  agentDir: getAgentDir(),                // ← 必填：用户级 agent 配置目录（~/.pi/agent）
  extensionFactories: [
    (pi) => {
      pi.registerTool({
        name: "dynamic_query",
        label: "Dynamic Query",
        description: "Query external data at runtime",
        parameters: Type.Object({ query: Type.String() }),
        async execute(toolCallId, params, signal, onUpdate, ctx) {
          return {
            content: [{ type: "text", text: `Result: ${params.query}` }],
            details: {},
          };
        },
      });
    },
  ],
});
// ⚠️ 传入自定义 resourceLoader 时，createAgentSession 不会再自动 reload
//    （sdk.ts 只在未传 loader 的分支里自动 reload），所以这里必须手动调一次。
await resourceLoader.reload();

const { session } = await createAgentSession({
  resourceLoader,
  sessionManager: SessionManager.inMemory(),
});

try {
  await session.prompt("Please query 'hello' using dynamic_query.");
} finally {
  session.dispose();
}
```

---

## 变体

### 变体 A：包装内置工具（推荐模式）

不是「替换实现」，而是「拿原定义 → 加自定义校验 → 调原 execute」。保留原工具的 offset/limit/截断/图片处理等能力，只加一层壳。

```ts
import {
  createReadToolDefinition,
  type ExtensionAPI,
} from "@earendil-works/pi-coding-agent";
import { Type } from "typebox";
import * as path from "node:path";

const ALLOWED_DIR = path.resolve(process.cwd(), ".pi", "skills");

export default (pi: ExtensionAPI) => {
  const baseRead = createReadToolDefinition(process.cwd());

  pi.registerTool({
    ...baseRead,                            // 保留所有原字段（parameters / promptSnippet 等）
    name: "read",                           // 同名覆盖内置 read
    description:
      baseRead.description +
      "。安全限制：仅允许读取 .pi/skills/ 目录内文件。",
    async execute(toolCallId, params, signal, onUpdate, ctx) {
      const requested = path.resolve(process.cwd(), params.path);
      const allowed =
        requested === ALLOWED_DIR ||
        requested.startsWith(ALLOWED_DIR + path.sep);
      if (!allowed) {
        return {
          content: [{
            type: "text" as const,
            text: `权限拒绝：read 工具只能读取 ${ALLOWED_DIR} 内的文件。`,
          }],
          details: { denied: true },
        };
      }
      // 委派给原 execute，保留内置行为
      return baseRead.execute(toolCallId, params, signal, onUpdate, ctx);
    },
  });
};
```

**何时用**：限制内置工具能力（路径白名单 / 命令审计 / 权限校验） / 改 cwd / 加日志。**强烈推荐此模式**，而非完全替换 execute 逻辑。

### 变体 B：纯审计覆盖（保留原行为）

只在执行前后插入逻辑，不改变原 execute 行为：

```ts
import { createReadToolDefinition, type ExtensionAPI } from "@earendil-works/pi-coding-agent";

export default (pi: ExtensionAPI) => {
  const baseRead = createReadToolDefinition(process.cwd());

  pi.registerTool({
    ...baseRead,
    name: "read",
    async execute(toolCallId, params, signal, onUpdate, ctx) {
      console.log(`[AUDIT] read: ${params.path}`);
      const start = Date.now();
      const result = await baseRead.execute(toolCallId, params, signal, onUpdate, ctx);
      console.log(`[AUDIT] read done in ${Date.now() - start}ms`);
      return result;
    },
  });
};
```

**何时用**：合规审计 / 性能监控 / 调试。

### 变体 C：条件注册

扩展启动时根据环境决定是否注册：

```ts
import fs from "node:fs";
import path from "node:path";
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";
import { Type } from "typebox";

export default (pi: ExtensionAPI) => {
  const skillsDir = path.join(process.cwd(), ".pi", "skills");
  if (!fs.existsSync(skillsDir)) return;  // 没装 skills 就不注册

  pi.registerTool({
    name: "skill_search",
    label: "Skill Search",
    description: "Search available skills by keyword",
    parameters: Type.Object({ keyword: Type.String() }),
    async execute(toolCallId, params, signal, onUpdate, ctx) {
      // ...
    },
  });
};
```

**何时用**：能力按需启用（OAuth 未授权就不注册需要授权的工具 / 没装某个目录就不提供相关工具）。

### 变体 D：完全替换（不推荐）

```ts
pi.registerTool({
  name: "read",  // 同名但完全换实现
  label: "Read (DB)",
  description: "Read from database instead of filesystem",
  parameters: Type.Object({ table: Type.String() }),
  async execute(toolCallId, params, signal, onUpdate, ctx) {
    // 完全不调 baseRead，模型按"读文件"语义来调却得到"读数据库"行为
    // ...
  },
});
```

**强烈不建议**：模型仍按"read = 读文件"的语义来调用，行为与预期不符会造成混乱。**用新工具名**（如 `db_lookup`）更清晰。

### 变体 E：工具 + 事件 hook + 命令组合

详见 [E02-extension-basics.md](E02-extension-basics.md) 完整示例。组合 `pi.registerTool` + `pi.on("tool_call")` + `pi.registerCommand` 可覆盖完整 Agent 生命周期。

---

## 陷阱

1. **以为「扩展里 registerTool 了就一定生效」** → 实际受 `createAgentSession({ tools })` 白名单约束（A06 横向，`agent-session.ts`）。要么 `tools` 省掉，要么显式包含扩展工具名。
2. **`pi.registerTool(defineTool({...}))`** → 多余的 `defineTool` 包裹。`registerTool` 本身有泛型推断（`types.ts`），直接传 ToolDefinition 对象即可。`defineTool` 只在传 `customTools: [...]` 数组时需要（防止字面量类型被数组收窄）。
3. **完全替换内置工具**（变体 D） → 模型按原语义调用却得到新行为，造成混乱。要包装用变体 A 模式（`...baseRead` + 自定义 execute 内调 `baseRead.execute`）；要替换用新工具名。
4. **在事件 handler 里调 `pi.registerTool`** → 虽然 runtime 允许（只要 session 未失效），但会让工具列表变化难以追踪（每次调用都立即触发 `refreshTools()`）。**registerTool 设计用途是扩展加载阶段**（`loader.ts`），运行时需要动态切换工具集用 `pi.setActiveTools`（`ExtensionAPI` 上的方法，事件处理器的 `ctx` 上没有此方法）。对比：
   ```ts
   // ❌ 错误：在事件 handler 里 registerTool，工具列表非预期变动
   pi.on("tool_call", (event) => {
     if (event.toolName === "bash") {
       pi.registerTool({ name: "audit_bash", /* ... */ });  // 立即 refresh，难追踪
     }
   });

   // ✅ 正确：加载阶段 register 全集，运行时用 setActiveTools 切换可见集
   export default (pi: ExtensionAPI) => {
     pi.registerTool({ name: "audit_bash", /* ... */ });    // 加载阶段注册
     pi.registerTool({ name: "plain_bash",  /* ... */ });
     pi.on("tool_call", (event) => {
       if (event.toolName === "bash") {
         pi.setActiveTools(["read", "edit", "audit_bash"]);  // 全量替换 active 集
       }
     });
   };
   ```
5. **`ctx.session` 不存在** → ExtensionContext 没有 `session` 字段，只有 `sessionManager`（只读）。要操作会话走扩展事件。
6. **`onUpdate` 参数形态错误** → 必须传 `AgentToolResult` 形态（`{ content: [...], details: {...} }`），**没有 `progress` 字段**。
7. **`import { Type } from "@sinclair/typebox"`** → 不推荐（CHANGELOG v0.69.0 迁移到 `typebox` 1.x）。根 import 仍被 loader 别名兼容，不会报错，但新代码必须用 `typebox`（无 scope 前缀）。
8. **扩展文件放错位置** → 不在 `.pi/extensions/` / `~/.pi/agent/extensions/` / `settings.json` 的 `extensions` 数组 / `additionalExtensionPaths` / `extensionFactories` 任一处，扩展不会被加载（5 种发现方式见上方「扩展如何被加载」表格；`settings.json` 由 `packageManager.resolve()` 解析）。
9. **没 try/finally + dispose** → 会话文件残留，长期运行时累积垃圾。所有 SDK 集成示例都应有 finally 块。
10. **想"卸载"工具** → registerTool 没有对应的 `unregisterTool` API。要临时禁用某工具用 `pi.setActiveTools([...])` 显式列出允许集合——**注意 `setActiveTools` 是全量替换整个 active 集合，不是增量移除**（内部 `setActiveToolsByName` 用 `[...new Set(nextActiveToolNames)]` 重建集合，`agent-session.ts`），每次调用必须传完整允许清单，不能"在当前集合上减一个"。要彻底卸载只能在命令处理器（`ExtensionCommandContext`）中调用 `ctx.reload()` 重新加载扩展，或通过 `/reload` 命令触发（交互模式 CLI 专属）。事件处理器的 `ctx`（`ExtensionContext`）上没有 `reload()` 方法。

---

## 横向联动

- **D01**（自定义工具）：本场景的铺垫——`defineTool` / `execute` / `ExtensionContext` / `AgentToolResult` 的完整详解都在 D01
- **E01**（工具拦截）：拦截用 `pi.on("tool_call")`，覆盖用 `pi.registerTool`——**两者不冲突**，可组合（拦截做条件阻断，覆盖做行为定制）
- **E02**（扩展基础）：完整扩展示例（生命周期 hook + 工具 + 命令）
- **A04**（系统提示词）：`promptSnippet` / `promptGuidelines` 是改系统提示词的轻量方式
- **A06**（默认行为）：扩展工具也受 `_allowedToolNames` 过滤；不传 `tools` 时靠构造期 `includeAllExtensionTools` 标志自动加入 active 集
- **sdk_doc/06**（工具系统）：完整 ToolDefinition 接口参考 + 内置工具安全边界
- **sdk_doc/07**（扩展 API）：`ExtensionAPI` / `ExtensionContext` 完整接口
- **sdk_doc/11**（集成踩坑）：SDK 集成模式 + try/finally 模式
