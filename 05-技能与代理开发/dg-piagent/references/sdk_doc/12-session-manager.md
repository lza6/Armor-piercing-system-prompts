# 12 - 会话管理器 (SessionManager)

`SessionManager` 管理 pi-agent 的会话持久化。会话存储为 JSONL 文件（每行一个 JSON 对象），支持会话列表、恢复、分叉、导入等操作。

> **谁需要读这篇？** 当你需要直接操作会话文件（列出历史会话、手动分叉、fork 跨项目会话）时查阅。**大多数情况下不需要直接创建 SessionManager**——它由 AgentSession 内部管理，通过 `session.sessionManager` 访问即可（见 [02-agent-session.md](./02-agent-session.md) §属性速查表）。

## 静态工厂方法

所有创建 SessionManager 实例的入口都是静态方法：

### `SessionManager.create(cwd, sessionDir?, options?)`

创建新的持久化会话。

```ts
static create(cwd: string, sessionDir?: string, options?: NewSessionOptions): SessionManager
```

- `cwd`: 工作目录，存入会话头的 `cwd` 字段
- `sessionDir`: 会话文件存储目录。不传则自动使用 `~/.pi/agent/sessions/<encoded-cwd>/`
- `options.id`: 指定会话 ID（不传则自动生成 UUIDv7）
- `options.parentSession`: 设置父会话路径，用于 fork 溯源（标记本会话由哪个会话 fork 而来）

```ts
// 默认目录
const sm = SessionManager.create("/my/project");

// 自定义目录
const sm = SessionManager.create("/my/project", "/tmp/my-sessions");
```

### `SessionManager.continueRecent(cwd, sessionDir?)`

续接最近一次会话，如果没有则创建新会话。

```ts
static continueRecent(cwd: string, sessionDir?: string): SessionManager
```

按修改时间排序，自动找到最新的 `.jsonl` 文件并打开。如果目录下无任何会话文件，则等同于 `create()`。

```ts
const sm = SessionManager.continueRecent("/my/project");
```

### `SessionManager.open(path, sessionDir?, cwdOverride?)`

打开指定的会话文件。

```ts
static open(path: string, sessionDir?: string, cwdOverride?: string): SessionManager
```

- `path`: 会话文件（`.jsonl`）的完整路径
- `sessionDir`: 用于后续的 `/new` 或 `/branch` 操作。不传则取文件所在目录
- `cwdOverride`: 覆盖会话头中的 cwd。用于将别人的会话导入到自己的项目

### `SessionManager.inMemory(cwd?, options?)`

创建纯内存会话，不写入任何文件。

```ts
static inMemory(cwd?: string, options?: NewSessionOptions): SessionManager
```

- `cwd`: 工作目录，默认 `process.cwd()`
- `options.id`: 指定会话 ID（不传则自动生成 UUIDv7）
- `options.parentSession`: 设置父会话路径，用于 fork 溯源

适用于测试、瞬时场景或不想产生磁盘文件的场景：

```ts
const sm = SessionManager.inMemory();
// 也可指定 cwd 和 options
const sm = SessionManager.inMemory("/tmp/test", { id: "my-test-session" });
```

### `SessionManager.list(cwd, sessionDir?, onProgress?)`

列出某个工作目录下的所有历史会话。

```ts
static async list(cwd: string, sessionDir?: string, onProgress?: SessionListProgress): Promise<SessionInfo[]>
```

返回按 `modified` 时间倒序排列的 `SessionInfo[]`。传入自定义 `sessionDir` 时会按会话头里的 `cwd` 过滤（只返回与传入 `cwd` 匹配的会话）；不传 `sessionDir`（用默认目录）时不过滤。`continueRecent()` 同此规则。

```ts
const sessions = await SessionManager.list("/my/project");
for (const s of sessions) {
  console.log(`${s.id}: ${s.firstMessage}`);
}
```

### `SessionManager.listAll(onProgress?)` / `listAll(sessionDir?, onProgress?)`

列出**所有项目**的所有会话（遍历 `~/.pi/agent/sessions/` 下的每个子目录）。也可传入 `sessionDir` 只列出指定目录。

```ts
static async listAll(onProgress?: SessionListProgress): Promise<SessionInfo[]>
static async listAll(sessionDir?: string, onProgress?: SessionListProgress): Promise<SessionInfo[]>
```

### `SessionManager.forkFrom(sourcePath, targetCwd, sessionDir?, options?)`

从另一个项目目录的会话 fork 到当前项目。把源会话的所有 entry（除 header）复制到新文件，新会话的 `cwd` 改写为 `targetCwd`，并在 header 的 `parentSession` 字段记录源会话路径用于溯源。

```ts
static forkFrom(sourcePath: string, targetCwd: string, sessionDir?: string, options?: NewSessionOptions): SessionManager
```

```ts
// 把 A 项目的会话 fork 到 B 项目，继续在 B 项目里对话
const sm = SessionManager.forkFrom(
  "/path/to/projectA/.pi/agent/sessions/.../source.jsonl",  // 源会话
  "/path/to/projectB",                                       // 目标项目 cwd
);
// 新会话：cwd = /path/to/projectB，parentSession 指向源文件，内容继承源会话全部分支
```

## 实例方法

SessionManager 是 **append-only 树结构**——每条 entry 都有 `id` 和 `parentId`，"leaf 指针"标记当前位置。追加创建当前 leaf 的子节点；分叉移动 leaf 到历史节点，原分支不变。

### 追加类（append*）

每个 `append*` 调用都会创建当前 leaf 的子节点、推进 leaf，返回 entry id。

```ts
appendMessage(message: Message | CustomMessage | BashExecutionMessage): string
appendThinkingLevelChange(thinkingLevel: string): string
appendModelChange(provider: string, modelId: string): string
appendCompaction<T = unknown>(summary: string, firstKeptEntryId: string, tokensBefore: number, details?: T, fromHook?: boolean, usage?: Usage): string  // summary=压缩摘要；firstKeptEntryId=压缩后保留的第一条 entry 的 id（之前的被摘要替代）；tokensBefore=压缩前的 token 数（展示压缩幅度）
appendCustomEntry(customType: string, data?: unknown): string                    // 扩展状态持久化（不进 LLM 上下文）
appendCustomMessageEntry<T = unknown>(customType: string, content: string | (TextContent | ImageContent)[], display: boolean, details?: T): string  // 注入 LLM 上下文
appendSessionInfo(name: string): string                                          // 设置会话显示名
appendLabelChange(targetId: string, label: string | undefined): string           // 给 entry 打/清标签
```

> ⚠️ `CompactionSummaryMessage` 和 `BranchSummaryMessage` 不能通过 `appendMessage` 写——必须用 `appendCompaction()` / `branchWithSummary()`，便于系统统一索引。

### 读取类（get*）

```ts
getCwd(): string                                  // 工作目录
getSessionDir(): string                           // 会话文件目录
getSessionId(): string                            // 会话 UUID
getSessionFile(): string | undefined              // 会话文件路径（inMemory 时为 undefined）
isPersisted(): boolean                            // 是否落盘
usesDefaultSessionDir(): boolean                  // 是否使用默认目录
getLeafId(): string | null                        // 当前 leaf entry id
getLeafEntry(): SessionEntry | undefined          // 当前 leaf 完整 entry
getEntry(id: string): SessionEntry | undefined    // 任意 entry
getChildren(parentId: string): SessionEntry[]     // 直接子节点
getLabel(id: string): string | undefined          // entry 的标签
getBranch(fromId?: string): SessionEntry[]        // 从 entry 走到 root 的所有节点（含所有类型）
buildSessionContext(): SessionContext              // 发给 LLM 的解析后上下文（处理 compaction/branch_summary）
buildContextEntries(): SessionEntry[]               // 返回处理了 compaction 的 entry 列表（不含 SessionContext 封装，用于遍历/检查）
getHeader(): SessionHeader | null                 // 会话头
getEntries(): SessionEntry[]                      // 所有 entry（浅拷贝；append-only 不可改）
getTree(): SessionTreeNode[]                      // 树形结构（含 label/labelTimestamp）
getSessionName(): string | undefined              // 从 session_info entry 解析出的显示名
```

`buildSessionContext()` 是核心方法——沿着 root→leaf 路径遍历，处理 compaction 和 branch_summary，返回发给 LLM 的最终上下文：

```ts
interface SessionContext {
  messages: AgentMessage[];                              // LLM 可见的消息列表（含 compaction summary）
  thinkingLevel: string;                                 // 当前思考模式（"off" | "minimal" | "low" | "medium" | "high" | "xhigh" | "max"，完整定义见 agent-core 的 ThinkingLevel 类型；xhigh/max 仅部分模型支持）
  model: { provider: string; modelId: string } | null;   // 当前模型配置
}
```

> **`buildContextEntries()` vs `buildSessionContext()`**：前者返回 `SessionEntry[]`（原始 entry 列表，可看 `type`/`message` 等字段，适合**遍历/检查条目**）；后者返回 `SessionContext`（含 messages + thinkingLevel + model 的完整封装，messages 数组里 compaction/branch_summary 已被处理成 user 消息，适合**发给 LLM**）。
>
> **扩展中的类型**：`ctx.sessionManager` 的类型是 `ReadonlySessionManager`（`Pick<SessionManager, …>` 子集，共 14 个只读方法：`getCwd` / `getSessionDir` / `getSessionId` / `getSessionFile` / `getLeafId` / `getLeafEntry` / `getEntry` / `getLabel` / `getBranch` / `buildContextEntries` / `getHeader` / `getEntries` / `getTree` / `getSessionName`）。**不包含**：`getChildren`、`usesDefaultSessionDir`、`isPersisted`、所有 `append*` / `branch*`、`buildSessionContext`、`setSessionFile`、`newSession`。在扩展里调这些会类型报错——扩展中如需完整 SessionManager 能力需通过其他途径获取。完整清单见源码 `ReadonlySessionManager in session-manager.ts:190-206`。

### 分叉类（branch*）

```ts
branch(branchFromId: string): void                            // leaf 移到指定 entry，下次 append 创建新分支
resetLeaf(): void                                              // leaf 重置为 null，下次 append 创建新 root（用于重编辑首条消息）
branchWithSummary(branchFromId: string | null, summary: string, details?: unknown, fromHook?: boolean, usage?: Usage): string  // 分叉并写入被丢弃路径的摘要
createBranchedSession(leafId: string): string | undefined     // 创建只含 root→leaf 路径的新会话文件，返回路径
setSessionFile(sessionFile: string): void                     // 切换会话文件（resume/branch 用）
newSession(options?: NewSessionOptions): string | undefined   // 在同目录创建新会话
```

**典型操作：分叉到历史消息并改写**

```ts
// 1. 找到想分叉到的 entry（比如第 3 条用户消息）
const entries = sm.getEntries();
const targetEntry = entries.find(e =>
  e.type === "message" && e.message.role === "user"
  // 按条件筛选目标消息
);

// 2. 分叉（leaf 移到该 entry）
if (targetEntry) {
  sm.branch(targetEntry.id);

  // 3. 下次 appendMessage 会从该 entry 创建新分支
  //    原分支不受影响，保留在文件中
  sm.appendMessage({ role: "user", content: "换个方向..." });
}
```

## SessionInfo 结构

`list()` 和 `listAll()` 返回的数组元素：

```ts
interface SessionInfo {
  path: string;            // 会话文件路径
  id: string;              // 会话 UUID
  cwd: string;             // 工作目录
  name?: string;           // 用户自定义显示名（通过 session_info entry 设置）
  parentSessionPath?: string; // fork 来源路径
  created: Date;           // 创建时间
  modified: Date;          // 最后修改时间
  messageCount: number;    // 消息数量
  firstMessage: string;    // 第一条用户消息文本
  allMessagesText: string; // 所有消息拼接文本
}
```

## 会话文件格式

会话文件是 JSONL 格式，第一行是会话头（SessionHeader），后续每行是一个会话条目（SessionEntry）：

**首行 — SessionHeader:**
```json
{"type":"session","version":3,"id":"01jr...","timestamp":"2026-06-06T...","cwd":"/my/project"}
```

字段：`type`（固定 `"session"`）、`version`（当前 3，旧文件可缺省）、`id`、`timestamp`、`cwd`，以及可选的 `parentSession`（fork 来源路径，仅 `forkFrom` 产生时存在）。

**后续 — SessionEntry 类型:**
- `"message"` — LLM/user 消息
- `"compaction"` — 上下文压缩记录
- `"branch_summary"` — 分支摘要
- `"thinking_level_change"` — 思考模式变更
- `"model_change"` — 模型切换
- `"custom"` — 扩展自定义数据持久化
- `"custom_message"` — 扩展注入 LLM 上下文
- `"label"` — 用户标注/书签
- `"session_info"` — 会话元数据（如显示名）

## 关键注意事项

1. **inMemory 会话不持久化**：关闭后数据丢失，仅适用于测试
2. **cwd 存储在会话头**中：`open()` 默认使用其中 cwd；传入 `cwdOverride` 可覆盖
3. **会话版本自动迁移**：打开旧版会话文件时自动执行 v1->v2->v3 迁移
4. **sessionDir**：不传时自动推导到 `~/.pi/agent/sessions/<编码后的cwd>/`，每个项目独立目录
5. **会话条目有树结构**：每个 entry 的 `id`/`parentId` 构成树，支持分叉和时间线导航
6. **文件落盘延迟（重要陷阱）**：`create()` 并不会立即写盘——会话文件在**第一条 assistant 消息**到达后才真正写入磁盘（在此之前只生成文件名，entries 暂存内存）。这是 `_persist` 的 `hasAssistant` 守卫行为（见 `SessionManager._persist in session-manager.ts:1015-1042`）。因此测试中若在 `create()` 后立刻去 inspect 文件，会发现文件不存在或为空。需要立即落盘时用 `inMemory()` + 手动序列化，或等首条 assistant 后再读文件。
7. **环境变量覆盖**：`PI_CODING_AGENT_DIR` 可覆盖 agent 根目录（默认 `~/.pi/agent`），从而改变整个 sessions 根目录位置；`PI_CODING_AGENT_SESSION_DIR` 覆盖会话目录。CI / 容器 / 多实例场景常用（见 `getAgentDir in config.ts:515-521`）。
