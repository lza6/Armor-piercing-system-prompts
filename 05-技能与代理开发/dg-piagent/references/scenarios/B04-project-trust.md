# 场景：自定义项目信任策略 (B04)

## 目标
拦截 pi-agent 的「项目信任」决策点，根据企业策略自动信任白名单目录、给 CI/CD 跳过信任提示、或为不信任目录提供自定义警告。

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `pi.on("project_trust")` | 拦截信任决策 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `ctx.isProjectTrusted()` | 在其他 handler 中读取信任状态 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |
| `ctx.mode` | 区分 tui/rpc/json/print 模式 | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |

## 背景

pi-agent 在打开一个项目时，会询问用户是否信任该目录（信任后才允许写文件、跑 bash 等）。这是默认行为，但在以下场景需要自定义：

> **触发前提**：`project_trust` 事件**仅在项目含信任相关资源时才触发**——即 `.pi/` 下存在 settings.json / extensions / skills / prompts / themes / SYSTEM.md / APPEND_SYSTEM.md，或 cwd 及祖先目录存在 `.agents/skills/`。不含这些资源的项目会被**自动信任**（`resolveProjectTrusted` 直接返回 `true`），扩展的 `project_trust` handler 根本不会被调用。
>
> ⚠️ **用户全局 `~/.agents/skills` 被特意排除**：源码把 `~/.agents/skills`（即 `$HOME/.agents/skills`）视为可信的用户级资源，即使 cwd 是 `$HOME` 也不会触发信任。只有项目本地或祖先目录里的 `.agents/skills` 才算"需要信任的项目资源"。所以把扩展装到 `~/.agents/skills` 不会触发信任弹窗——这是有意为之的安全设计。

- **企业环境**：白名单内的目录（如 `~/work/*`）自动信任，不弹窗
- **CI/CD（rpc/json/print 模式）**：完全跳过信任检查，无人值守
- **扩展前置检查**：扩展自己的危险操作前再次确认信任状态

## 实现思路

1. 在扩展中订阅 `project_trust` 事件
2. handler 接收 `ProjectTrustEvent`（含 cwd），返回 `ProjectTrustEventResult`（含 decision + remember）
3. handler 是 **特殊签名**（非通用 `ExtensionHandler`）：返回 `Promise<ProjectTrustEventResult> | ProjectTrustEventResult`
4. 在其他 handler 中可用 `ctx.isProjectTrusted()` 读取当前决策

## 核心代码

```ts
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";

export default function (pi: ExtensionAPI) {
  // 企业白名单目录
  const TRUSTED_PREFIXES = [
    "/home/user/work/",
    "D:\\work\\",
  ];

  pi.on("project_trust", (event, ctx) => {
    // 模式 1：CI/CD 无人值守 — rpc/json/print 模式自动信任
    // ⚠️ CI 场景更推荐直接用 `pi --approve`（-a）命令行参数：它走 trustOverride，
    //    在决策链第①步直接命中，根本不进扩展、不写磁盘。这里的扩展内判断适合
    //    无法改启动参数、或需要按 cwd/策略细控的场景。
    if (ctx.mode !== "tui") {
      return { trusted: "yes", remember: true };
    }

    // 模式 2：白名单目录自动信任，不弹 UI
    if (TRUSTED_PREFIXES.some(p => event.cwd.startsWith(p))) {
      return { trusted: "yes", remember: true };
    }

    // 模式 3：黑名单目录自动拒绝
    if (event.cwd.includes("/tmp/untrusted/")) {
      return { trusted: "no", remember: false };
    }

    // 模式 4：交给默认 UI（返回 undecided）
    return { trusted: "undecided" };
  });

  // 在其他 hook 中读信任状态
  pi.on("tool_call", (event, ctx) => {
    // 这里只判断 toolName、不读 event.input，直接用字面量比较即可。
    // 若需要读取 event.input.command 等（要类型窄化），请改用官方守卫：
    //   if (isToolCallEventType("bash", event)) { event.input.command ... }
    // 因为 CustomToolCallEvent.toolName 是 string，直接 === "bash" 在 TS 层面不会自动窄化。
    if (event.toolName === "bash" && !ctx.isProjectTrusted()) {
      // 不信任项目里的 bash 调用直接阻断
      return { block: true, reason: "Bash 在不信任项目中已被扩展拦截" };
    }
  });
}
```

## ProjectTrustEvent / Result 字段

```ts
interface ProjectTrustEvent {
  type: "project_trust";
  cwd: string;                    // 待决策的项目目录
}

interface ProjectTrustContext {
  cwd: string;
  mode: "tui" | "rpc" | "json" | "print";
  hasUI: boolean;
  ui: Pick<ExtensionUIContext, "select" | "confirm" | "input" | "notify">;
}
// 上面那行 Pick<...> 的意思是：ctx.ui 只有 select / confirm / input / notify 这 4 个方法，
// 比 ExtensionUIContext 少了 setStatus / setWidget / setFooter 等（见 §4 UI 限制）。

type ProjectTrustEventDecision = "yes" | "no" | "undecided";

interface ProjectTrustEventResult {
  trusted: ProjectTrustEventDecision;
  remember?: boolean;             // 是否持久化决策（避免下次再问）
}
```

**返回值语义**：
- `"yes"` + `remember: true` — 信任并持久化（写入 `~/.pi/agent/trust.json`），下次同目录不再问
- `"yes"` + `remember: false` — 仅本次会话信任，不持久化
- `"no"` — 标记为不信任，pi 不会加载该项目的 `.pi/` 资源（项目级 settings/extensions/skills/prompts/themes/SYSTEM.md/APPEND_SYSTEM.md 全部跳过，相当于用全局配置跑；会话本身照常继续，不等于"中止会话"）
- `"undecided"` — 本扩展不做决定，继续询问后续扩展；**只有所有扩展都返回 `"undecided"`** 时，才回落到默认决策链（trustStore → `defaultProjectTrust` → UI）

## 关键细节

### 1. handler 签名特殊

不同于其他 `ExtensionHandler<Event, Result>`，`project_trust` 的 handler 类型是 `ProjectTrustHandler`：
```ts
type ProjectTrustHandler = (
  event: ProjectTrustEvent,
  ctx: ProjectTrustContext
) => Promise<ProjectTrustEventResult> | ProjectTrustEventResult;
```
注意 `ctx` 是 **`ProjectTrustContext`**（不是 `ExtensionContext`）——只有 `cwd / mode / hasUI / ui` 四个字段，没有 `sessionManager` 等。

### 2. 与 `defaultProjectTrust` setting 的关系

settings 中可设 `defaultProjectTrust: "always" | "never" | "ask"`（默认 `"ask"`）。这是 pi 内置的兜底逻辑——如果没有任何扩展订阅 `project_trust`（或所有扩展都返回 `"undecided"`），就用这个 setting。

扩展订阅后的返回值优先级高于 setting。

**完整决策链顺序**（`resolveProjectTrusted` 依次检查，命中即返回）：

1. `trustOverride`——外部强制覆盖（如 CI 的 `pi --approve` / `-a` 参数），设了直接返回，**跳过扩展和 setting**
2. `hasTrustRequiringProjectResources(cwd)`——无信任相关资源则自动信任（见顶部“触发前提”）
3. 扩展 `project_trust` handler——first-match-wins（见 §3）
4. `ProjectTrustStore`——`~/.pi/agent/trust.json` 中的持久化决策，按祖先路径前缀匹配（信任了父目录则子目录也生效）
5. `defaultProjectTrust` setting——`"always"` 信任 / `"never"` 拒绝 / `"ask"` 继续下一步
6. UI 提示（仅 `hasUI` 时），让用户选择；无 UI 则默认拒绝

### 3. 多个扩展订阅时的合并（first-match-wins）

如果多个扩展都订阅了 `project_trust`，pi 按**首个裁决机制**（first-match-wins）逐一调用：**第一个返回 `"yes"` 或 `"no"` 的扩展立即决定结果**，后续扩展不再被调用。只有当所有扩展都返回 `"undecided"` 时，才回落到默认决策链（trustStore → `defaultProjectTrust` → UI）。

> ⚠️ **多扩展协作时不能假设所有 handler 都会被调用。** 例如：扩展 A 是白名单（对 `~/work/*` 返回 `"yes"`），扩展 B 是黑名单（对 `/tmp/evil/` 返回 `"no"`）。如果 A 先注册且返回 `"yes"`，B 根本不会被调用——黑名单静默失效。正确做法是把多个策略放进**同一个 handler 内部按优先级裁决**，而不是拆到多个扩展里。

### 4. UI 限制

`ProjectTrustContext.ui` 只暴露 4 个方法（`select / confirm / input / notify`），不能调用 `setStatus / setWidget / setFooter` 等——信任决策发生在会话完全启动之前。

## 变体与延伸

- 想看完整 ExtensionContext 字段表 → 见 [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) 的「ExtensionContext」节
- 想拦截工具调用 → 见 [场景 E01](E01-tool-intercept.md)
- 想了解 RPC/CI 模式 → `ctx.mode` 取值为 `"tui" | "rpc" | "json" | "print"`（见上文 `ProjectTrustContext`）。CI/无人值守场景既可在扩展里判断 `ctx.mode !== "tui"` 自动信任，更推荐直接用 `pi --approve`（`-a`）命令行参数，它在决策链第①步（`trustOverride`）即命中，完全跳过扩展和 UI
