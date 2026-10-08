# 场景：工具白名单 (A04)

## 目标

限制 Agent 可用的工具集，例如只允许只读工具，禁用写入类工具。

> **默认行为**：不传 `tools` 时，pi-agent 默认启用 4 个内置工具——`read`、`bash`、`edit`、`write`（外加扩展注册的工具；`customTools` 需显式传入）。如果 Agent 面向外部用户或需要安全边界，必须显式设置白名单。

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `createAgentSession` 的 `tools` 选项 | 传入工具名数组，**仅启用指定工具**（覆盖内置 + 扩展 + 自定义全部来源） | [sdk_doc/06-tools.md](../sdk_doc/06-tools.md) |
| `createAgentSession` 的 `excludeTools` 选项 | 黑名单模式：禁用指定工具（在 `tools` 之后生效） | [sdk_doc/06-tools.md](../sdk_doc/06-tools.md) |
| `createAgentSession` 的 `noTools` 选项 | 快速禁用全部或仅内置工具（`"all"` / `"builtin"`） | [sdk_doc/06-tools.md](../sdk_doc/06-tools.md) |
| 内置工具名（共 7 个） | `read`、`bash`、`edit`、`write`、`grep`、`find`、`ls` | [sdk_doc/06-tools.md](../sdk_doc/06-tools.md) |

## 实现思路

1. 调用 `createAgentSession({ tools: [...] })` 传入白名单——Agent 只会看到白名单中的工具
2. 如果只想禁用个别工具而非全部重列，用 `excludeTools: ["bash"]` 更简洁
3. 如果完全不想要工具，用 `noTools: "all"` 比 `tools: []` 更语义化

**重要**：`tools` 白名单对**所有工具来源**一视同仁——包括内置工具、扩展注册的工具、`customTools`。只传 `["read"]` 意味着即便扩展注册了工具或通过 `customTools` 传入了自定义工具，它们也不会被激活。

**安全考量**：在生产环境或对外暴露的 Agent 中，建议始终设置 `tools` 白名单，避免 Agent 执行 `write`、`bash` 等危险操作。

## 核心代码

```ts
import { createAgentSession } from "@earendil-works/pi-coding-agent";

// 只读模式：仅允许读取类工具（find 是 glob 文件搜索）
const { session } = await createAgentSession({
  tools: ["read", "grep", "find", "ls"],
});

// 精确控制：允许读取 + 编辑，但禁止执行 shell
const { session: s2 } = await createAgentSession({
  tools: ["read", "write", "edit", "grep", "find", "ls"],
});

// 黑名单模式：允许全部但禁用 bash（比白名单更简洁）
const { session: s3 } = await createAgentSession({
  excludeTools: ["bash"],
});

// 完全禁用内置工具，但保留扩展/自定义工具
const { session: s4 } = await createAgentSession({
  noTools: "builtin",
});

// 纯对话模式：完全无工具
const { session: s5 } = await createAgentSession({
  noTools: "all",
});
```

## 运行时切换

```ts
// 对话中途收紧权限——下次 turn 生效
// 注意：传入的工具名必须在工具注册表中存在，未知名称会被静默忽略
session.setActiveToolsByName(["read", "grep"]);
```

## 变体与延伸
- 完整工具列表及参数说明 → 见 [sdk_doc/06-tools.md](../sdk_doc/06-tools.md)
- 自定义工具（如计算器、日期查询） → 见 [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md)
- 工具拦截与修改 → 见 [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md)
