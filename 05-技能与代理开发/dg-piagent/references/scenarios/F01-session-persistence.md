# 场景：持久化会话与断点续聊 (F01)

## 什么时候用 / 不用会怎样

- **要跨进程保留对话历史**（关掉重启后接着聊、列出历史会话挑一个打开）→ 用持久化模式 `SessionManager.create / continueRecent / open`
- **只想做单元测试 / 跑一次性对话、不留文件** → 用 `SessionManager.inMemory()`，进程退出数据即丢失
- **不显式传 `sessionManager`** → `createAgentSession` 会默认走 `SessionManager.create(cwd, getDefaultSessionDir(cwd, agentDir))`，即在 `~/.pi/agent/sessions/<编码后的cwd>/` 下创建 JSONL。所以"什么也不配"也是持久化模式

不传 sessionManager 时无法控制会话文件的目录与命名，要换目录或续接历史就必须显式传。

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `SessionManager.create(cwd, sessionDir?, options?)` | 在默认目录或指定目录下新建持久化会话 | [sdk_doc/12-session-manager.md](../sdk_doc/12-session-manager.md) |
| `SessionManager.inMemory(cwd?, options?)` | 创建纯内存会话（测试 / 瞬时场景） | [sdk_doc/12-session-manager.md](../sdk_doc/12-session-manager.md) |
| `SessionManager.continueRecent(cwd, sessionDir?)` | 自动续接最近一次会话；无历史则等同于 `create` | [sdk_doc/12-session-manager.md](../sdk_doc/12-session-manager.md) |
| `SessionManager.open(path, sessionDir?, cwdOverride?)` | 打开指定 `.jsonl` 文件，可选覆盖 cwd | [sdk_doc/12-session-manager.md](../sdk_doc/12-session-manager.md) |
| `SessionManager.list(cwd, sessionDir?, onProgress?)` | 列出某个 cwd 下的所有历史会话，按 modified 倒序 | [sdk_doc/12-session-manager.md](../sdk_doc/12-session-manager.md) |
| `session.sessionId` / `session.sessionFile` | 获取会话 UUID 和 JSONL 文件路径 | [sdk_doc/02-agent-session.md](../sdk_doc/02-agent-session.md) |
| `createAgentSession(options)` | 一站式创建 session + agent + 服务 | [sdk_doc/01-create-agent-session.md](../sdk_doc/01-create-agent-session.md) |

## 实现思路

1. **持久化新建**：`SessionManager.create(cwd)` + `createAgentSession({ sessionManager })`，文件写到 `~/.pi/agent/sessions/<encoded-cwd>/`，文件名形如 `<timestamp>_<uuidv7>.jsonl`
2. **内存模式**：`SessionManager.inMemory()` 不写磁盘，`session.sessionFile` 为 `undefined`，`session.sessionManager.isPersisted()` 为 `false`；`cwd` 不传则默认 `process.cwd()`
3. **续接最近**：`SessionManager.continueRecent(cwd)` 在默认目录按修改时间找最新 `.jsonl`；找到则打开，找不到则等同于 `create(cwd)`（仍持久化、仍写默认目录）
4. **打开指定文件**：`SessionManager.open(path, sessionDir?, cwdOverride?)` 恢复任意 `.jsonl`：
   - `path` 必填，会话文件完整路径
   - `cwdOverride` **可选**，不传则取会话头里的 `cwd`，再降级到 `process.cwd()`
   - `sessionDir` **可选**，不传则从 `path` 的父目录推导（`resolve(path, "..")`）
   - 典型场景：把同事的会话文件复制过来，用 `cwdOverride` 把工作目录指到自己项目
5. **查看历史**：`SessionManager.list(cwd)` 异步返回 `SessionInfo[]`，按 `modified` 倒序（最新在前）

## 核心代码

```ts
import { createAgentSession, SessionManager } from "@earendil-works/pi-coding-agent";

// 1. 内存模式（测试，不写磁盘）
const { session: memSession } = await createAgentSession({
  sessionManager: SessionManager.inMemory(), // cwd 默认 process.cwd()
});
// memSession.sessionFile === undefined

// 2. 新建持久化会话（写到 ~/.pi/agent/sessions/<encoded-cwd>/ 下）
const { session } = await createAgentSession({
  sessionManager: SessionManager.create(process.cwd()),
});
console.log(`会话文件: ${session.sessionFile}`, `ID: ${session.sessionId}`);

// 3. 继续最近一次会话（无历史则自动新建，行为等同 create）
const { session: recent } = await createAgentSession({
  sessionManager: SessionManager.continueRecent(process.cwd()),
});

// 4. 打开指定 JSONL 文件（可传 cwdOverride 切到不同工作目录）
const { session: opened } = await createAgentSession({
  sessionManager: SessionManager.open("/path/to/session.jsonl"),
  // 想把工作目录从原始 cwd 换到本地目录时：
  // sessionManager: SessionManager.open("/path/to/session.jsonl", undefined, "/my/local/cwd"),
});

// 5. 列出所有历史会话
const sessions = await SessionManager.list(process.cwd());
for (const s of sessions) {
  console.log(s.id, s.firstMessage, s.modified.toISOString());
}

// 无论哪种模式，最终都要释放（取消 retry/compaction/branchSummary/bash/agent，invalidate extension runner，移除监听器，清理 session resources）
try {
  await session.prompt("hello");
} finally {
  session.dispose();
}
```

## SessionInfo 字段含义

`SessionManager.list()` 和 `listAll()` 返回数组的元素结构（`session-manager.ts` SessionInfo 接口）：

| 字段 | 类型 | 含义 |
|------|------|------|
| `path` | `string` | 会话文件完整路径 |
| `id` | `string` | 会话 UUID（与首行 `session.id` 一致） |
| `cwd` | `string` | 工作目录（来自会话头） |
| `name?` | `string` | 通过 `appendSessionInfo` 设置的显示名 |
| `parentSessionPath?` | `string` | 若是 fork 来的，指源会话路径 |
| `created` | `Date` | 会话创建时间 |
| `modified` | `Date` | 最后修改时间 |
| `messageCount` | `number` | 消息条目数量 |
| `firstMessage` | `string` | 第一条 user 消息的文本（无消息时为 `"(no messages)"`） |
| `allMessagesText` | `string` | 仅 user + assistant 消息文本拼接的纯文本（不含 tool_result，可用于搜索） |

## 默认会话目录的编码规则

不传 `sessionDir` 时，目录由 `getDefaultSessionDir(cwd)` 推导：

```
~/.pi/agent/sessions/--<encoded-cwd>--/
                       ^^^^^^^^^^^^^^
                       把绝对路径里的特殊字符替换后拼出的目录名
```

例如 Windows 下 `D:\my\project` 会被编码成 `--D--my-project--`（`D` + `-`(from `:`) + `-`(from `\`) + `my` + `-`(from `\`) + `project`，首尾再包 `--`）。每个 cwd 独占一个子目录，所以不同项目互不污染；想跨项目统一管理就用 `SessionManager.listAll()`。

## 常见误期待与陷阱

- **加载历史会话 ≠ 重新执行工具**：`open()` / `continueRecent()` 只把消息条目读进上下文，工具调用结果以 `tool_result` 形式还原，不会再次触发任何 tool handler。若要重跑，需手动重发消息。
- **inMemory 也算 session**：内部只是 `persist=false`，仍可正常 `prompt / subscribe / steer`；只是关闭后数据丢失，且 `session.sessionFile === undefined`。
- **open 不传 cwdOverride 时 cwd 来自文件**：因此把会话文件复制到另一台机器后，默认 cwd 仍是原作者写的路径——务必用 `cwdOverride` 显式覆盖。
- **`createAgentSession` 自身有默认行为**：不传 `sessionManager` 时会自动调 `SessionManager.create(cwd, getDefaultSessionDir(cwd, agentDir))`，即默认就写盘。想做内存测试必须显式传 `SessionManager.inMemory()`。
- **`continueRecent` 不带 `sessionDir` 时不会跨 cwd 收集**：它只在该 cwd 对应的默认目录里找；要跨所有项目找用 `listAll()`。
- **不要手动改 `.jsonl`**：append-only 树结构靠 `id`/`parentId` 维护，手工编辑容易破坏 leaf 指针。要做分叉/重写请用 `branch()` / `branchWithSummary()` / `resetLeaf()`（见 [12-session-manager.md](../sdk_doc/12-session-manager.md) §分叉类）。

## 变体与延伸

- 运行时动态切换会话 → 见 [场景 F02](F02-session-runtime.md)
- 会话进行中中止当前操作 → 见 [场景 F04](F04-abort-session.md)
- 会话历史消息的读取与操控（steer 注入）→ 见 [场景 F05](F05-steer-session.md)
- 自定义 SessionManager 的 cwd 陷阱 → 见 [场景 A05](A05-custom-cwd.md) ★ 涉及自定义 SessionManager 时务必先读
