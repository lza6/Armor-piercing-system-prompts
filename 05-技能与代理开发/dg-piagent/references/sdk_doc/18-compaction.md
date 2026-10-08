# Compaction -- 上下文压缩与摘要

## 为什么需要 Compaction

大语言模型都有上下文窗口上限（如 128K、200K tokens）。当会话持续进行，历史消息不断累积，迟早会撞到这个天花板。**没有 compaction，会话会在 token 超限时报错或直接被截断，前面的工作成果全部丢失。**

Compaction 就是解决这个问题的：在上下文快满的时候，自动把早期消息"压缩"成一份结构化摘要，用几百个 token 的摘要替代几万 token 的原始历史，为新消息腾出空间。压缩后模型依然知道"之前做了什么、做到哪了、接下来该干什么"，不会丢失上下文脉络。

## 概述

Compaction 是 pi-agent 的**上下文窗口管理机制**。当会话历史积累到接近模型上下文窗口上限时，系统会自动（或手动）对早期消息进行摘要压缩，生成结构化摘要替换原始历史。

核心价值：
- **突破上下文窗口限制**：理论上支持无限长度的会话，不会因 token 超限而中断
- **保留关键信息**：摘要格式包含 Goal、Progress、Key Decisions、Next Steps 等结构化字段
- **文件追踪**：自动记录压缩区间内的文件读写操作，摘要中附带 `<read-files>` 和 `<modified-files>` 标签
- **增量更新**：支持对已有摘要做增量更新（追加新消息），而不是每次从头摘要全部历史
- **分支感知**：找到合适的"切割点"，避免在对话中途（tool call 序列中）截断
- **扩展事件**：通过 `session_before_compact` / `session_compact` 事件允许扩展定制压缩行为

源码位置：

| 内容 | 路径 | npm 包 |
|------|------|--------|
| 压缩主逻辑、准备、摘要生成 | `packages/coding-agent/src/core/compaction/compaction.ts` | `@earendil-works/pi-coding-agent` |
| 序列化、文件操作追踪 | `packages/coding-agent/src/core/compaction/utils.ts` | 同上 |
| 扩展事件类型定义 | `packages/coding-agent/src/core/extensions/types.ts` | 同上 |
| 会话 entry 类型（CompactionEntry 等） | `packages/coding-agent/src/core/session-manager.ts` | 同上 |

> **注意**：`pi-agent-core`（`packages/agent/`）中有一套独立的 compaction 实现，API 签名不同（使用 `Models` 对象、`Result<>` 包装等）。本文档以 `pi-coding-agent` 为准——这是 `pi.on()` 扩展事件和 `@earendil-works/pi-coding-agent` 导入所使用的版本。

> **版本兼容性**：本文档基于源码最新版（`repo/packages/coding-agent/src/`）。较旧的已发布 npm 版本可能缺少部分新 API（如 `generateSummaryWithUsage`、`retry`/`callbacks` 参数、`CompactionResult.usage` 等可能在旧版本的 `.d.ts` 中不存在，具体差异未独立验证）。如需使用这些功能，请从源码构建或升级到较新版本。

---

## 两层 API 速览（★ 必读，避免层混用）

pi-coding-agent 的 compaction 能力暴露在**两个不同的 API 层**，签名、触发方式、事件都不一样。本文档的所有示例都会标注它属于哪一层——**请勿在同一个变量上混用两层 API**（例如扩展层的 `context` 上没有 `.entries`，subscribe 层的 `AgentSession` 上没有 `compact(options: CompactOptions)` 签名）。

| 维度 | subscribe 层（SDK 集成者） | 扩展层（Extension 开发者） |
|------|---------------------------|---------------------------|
| 代表对象 | `AgentSession`（`agent-session.ts`） | `ExtensionContext`（`extensions/types.ts`，即 `pi.on` 回调里的 `ctx`） |
| 触发压缩 | `agentSession.compact(customInstructions?: string): Promise<CompactionResult>`（await，返回结果） | `ctx.compact(options?: CompactOptions): void`（fire-and-forget，不 await；结果通过 `onComplete`/`onError` 回调） |
| 监听事件 | `agentSession.on(...)` 监听 `AgentSessionEvent`：`compaction_start` / `compaction_end` / `summarization_retry_*` | `pi.on("session_before_compact", ...)` / `pi.on("session_compact", ...)`（可拦截/替换摘要） |
| 读取会话 | `agentSession.sessionManager.getBranch()` / `.getEntries()` / `agentSession.model` | `ctx.sessionManager`（`ReadonlySessionManager`）/ `ctx.model` |
| 典型读者 | 自己跑 agent loop、做 UI 反馈、想主动触发压缩的集成者 | 写扩展、想拦截或替换摘要行为的开发者 |

**关键区别**：

- **subscribe 层事件 ≠ 扩展层事件**。如果你用 `agentSession.on("session_compact", ...)` 不会触发——`session_compact` 是扩展层事件（`pi.on`）。subscribe 层对应的是 `compaction_end`。
- **`CompactOptions` 只存在于扩展层**。subscribe 层的 `AgentSession.compact()` 只接收一个可选的 `customInstructions: string`，不接收 `{ customInstructions, onComplete, onError }` 对象。
- **扩展层的 `compact()` 是 fire-and-forget**：返回 `void`，不返回 `Promise`，结果只能通过 `onComplete`/`onError` 回调拿到。

---

## 架构概览

```
会话消息 → 判断是否需要压缩 → 找到切割点
           ↓
    prepareCompaction() → compact() → CompactionResult
           ↓
    摘要写入会话树（CompactionEntry）
           ↓
    后续上下文 = 摘要 + 保留消息
```

### 自动压缩触发流程

压缩在 agent loop 中自动触发，流程如下：

1. 每轮对话结束后，agent loop 检查上下文 token 量
2. `shouldCompact()` 返回 `true` 时 → 调用 `prepareCompaction()` 生成准备数据 → 调用 `compact()` 生成摘要
3. 摘要作为 `CompactionEntry` 写入会话树
4. agent 状态原地更新：`this.agent.state.messages = sessionContext.messages`。会话本身并不"重新加载"——`buildContextEntries` 在内存里重算路径，找到最新 `CompactionEntry`，只保留 `[compaction, ...firstKeptEntryId 之后的消息]`，跳过被摘要的旧 entry。因此后续请求的上下文 = `CompactionEntry.summary` + `firstKeptEntryId` 之后的消息

三种触发原因（通过 `reason` 字段暴露给扩展）：

| `reason` 值 | 含义 |
|-------------|------|
| `"threshold"` | 上下文 token 触及 `shouldCompact` 阈值，自动触发 |
| `"overflow"` | 上下文 token 已超过模型窗口上限，紧急压缩。**是否重试被中断的 turn 取决于 `stopReason`**：仅当 `assistantMessage.stopReason !== "stop"`（即 length 截断或真正溢出）时 `willRetry = true` 并重试；若 `stopReason === "stop"`（回答已完成但仍超窗）则 `willRetry = false`，只压缩不重试 |
| `"manual"` | 用户执行 `/compact` 命令手动触发 |

---

## API 签名与参数表

### 判断与估算

```ts
// 是否应该触发压缩
export function shouldCompact(
  contextTokens: number,
  contextWindow: number,
  settings: CompactionSettings
): boolean

// 从 Usage 中计算上下文 token 总数
export function calculateContextTokens(usage: Usage): number

// 估算消息列表的上下文 token 使用情况（精确版）
export function estimateContextTokens(messages: AgentMessage[]): ContextUsageEstimate

// 估算单条消息的 token 数（粗略版，char/4 启发式）
export function estimateTokens(message: AgentMessage): number

// 获取最后一条有效 assistant 消息的 Usage
export function getLastAssistantUsage(entries: SessionEntry[]): Usage | undefined
```

#### `ContextUsageEstimate`

| 字段 | 类型 | 说明 |
|------|------|------|
| `tokens` | `number` | 估算的总上下文 token 数 |
| `usageTokens` | `number` | 最近一次 assistant usage 报告的 token 数 |
| `trailingTokens` | `number` | 最近一次 usage 之后新增消息的估算 token 数 |
| `lastUsageIndex` | `number \| null` | 提供了 usage 的那条消息在消息列表中的索引，无则为 null |

#### `CompactionSettings`

| 字段 | 类型 | 说明 | 默认值 |
|------|------|------|--------|
| `enabled` | `boolean` | 是否启用自动压缩 | `true` |
| `reserveTokens` | `number` | 为摘要 prompt 和模型输出保留的 token | `16384` |
| `keepRecentTokens` | `number` | 压缩后保留的最近上下文 token 近似值（按 `estimateTokens` 的 char/4 启发式估算，非精确值） | `20000` |

> **`reserveTokens` 的双重用途**：`reserveTokens`（默认 16384）同时决定两件事——
> (1) `shouldCompact` 的触发阈值（`contextWindow - reserveTokens`，见「关键细节 1」）；
> (2) 摘要 LLM 调用的 `maxTokens` 上限：`Math.min(Math.floor(0.8 * reserveTokens), model.maxTokens)`（compaction.ts:637-640）。也就是说，`reserveTokens` 越大，触发压缩越早，同时摘要可输出的 token 越多。

### 切点查找

```ts
// 找到当前轮次的起始位置
export function findTurnStartIndex(
  entries: SessionEntry[],
  entryIndex: number,
  startIndex: number
): number

// 找到压缩切割点
export function findCutPoint(
  entries: SessionEntry[],
  startIndex: number,
  endIndex: number,
  keepRecentTokens: number
): CutPointResult
```

#### `CutPointResult`

| 字段 | 类型 | 说明 |
|------|------|------|
| `firstKeptEntryIndex` | `number` | 压缩后保留的第一条 entry 索引 |
| `turnStartIndex` | `number` | 如果切割点在一个轮次中间，指向该轮次的起始位置；否则 -1 |
| `isSplitTurn` | `boolean` | 切割点是否分裂了一个进行中的轮次 |

### 准备与执行

```ts
// 准备压缩：分析会话树，生成 CompactionPreparation
// 返回 undefined 表示不需要压缩（如最后一条 entry 已是 compaction）
// ⚠️ prepareCompaction 是内部函数，不在主入口 @earendil-works/pi-coding-agent 导出
// 用户级手动压缩应通过 session.compact(options?: CompactOptions) 触发
export function prepareCompaction(
  pathEntries: SessionEntry[],
  settings: CompactionSettings
): CompactionPreparation | undefined

// 执行压缩：调用 LLM 生成摘要，出错时 throw
export async function compact(
  preparation: CompactionPreparation,
  model: Model<any>,
  apiKey: string | undefined,
  headers?: Record<string, string>,
  customInstructions?: string,
  signal?: AbortSignal,
  thinkingLevel?: ThinkingLevel,
  streamFn?: StreamFn,
  env?: Record<string, string>,
  retry?: RetryPolicy,            // 瞬态 provider 故障重试策略
  callbacks?: RetryCallbacks,     // 重试生命周期回调
): Promise<CompactionResult>

// 生成摘要文本（不含压缩框架逻辑），出错时 throw
export async function generateSummary(
  currentMessages: AgentMessage[],
  model: Model<any>,
  reserveTokens: number,
  apiKey: string | undefined,
  headers?: Record<string, string>,
  signal?: AbortSignal,
  customInstructions?: string,
  previousSummary?: string,
  thinkingLevel?: ThinkingLevel,
  streamFn?: StreamFn,
  env?: Record<string, string>,
  retry?: RetryPolicy,
  callbacks?: RetryCallbacks,
): Promise<string>

// 返回 { text, usage } 的薄包装（generateSummary 是它的 text 字段快捷方式）
export async function generateSummaryWithUsage(
  /* 同 generateSummary 参数 */
): Promise<{ text: string; usage: Usage }>

// 序列化会话消息为纯文本（供摘要 prompt 使用）
export function serializeConversation(messages: Message[]): string
```

> **关于错误处理**：`compact()` 和 `generateSummary()` 出错时直接 throw，不返回 `Result<>` 包装。调用时用 try/catch 处理异常。
>
> **弹性压缩（Resilient compaction）**：`compact()` / `generateSummary()` / `generateSummaryWithUsage()` 都支持 `retry` + `callbacks` 参数，瞬态 provider 故障（500/502/503/overloaded/rate limit/timeout 等）会按 `settings.retry` 自动重试。
>
> ⚠️ **主入口导出**：`compact` / `generateSummary` / `generateSummaryWithUsage` / `DEFAULT_COMPACTION_SETTINGS` 等都从 `@earendil-works/pi-coding-agent` 主入口导出。但 `prepareCompaction` / `CompactionPreparation` / `estimateContextTokens` / `ContextUsageEstimate` / `CompactionDetails` / `completeSummarization` 等**不在主入口**——需要从子路径导入或用 `session.compact()` 替代。

#### `CompactionPreparation`

| 字段 | 类型 | 说明 |
|------|------|------|
| `firstKeptEntryId` | `string` | 保留的第一条 entry 的 UUID |
| `messagesToSummarize` | `AgentMessage[]` | 将被压缩进摘要的消息 |
| `turnPrefixMessages` | `AgentMessage[]` | 如果切割点分裂了一个轮次，这里是轮次前缀的消息 |
| `isSplitTurn` | `boolean` | 是否分裂了轮次 |
| `tokensBefore` | `number` | 压缩前的估算 token 数 |
| `previousSummary` | `string \| undefined` | 上一次压缩的摘要（用于增量更新） |
| `fileOps` | `FileOperations` | 被压缩区间内的文件操作记录 |
| `settings` | `CompactionSettings` | 使用的压缩设置 |

#### `CompactionResult`

| 字段 | 类型 | 说明 |
|------|------|------|
| `summary` | `string` | 替代被压缩历史的摘要文本 |
| `firstKeptEntryId` | `string` | 保留的第一条 entry UUID |
| `tokensBefore` | `number` | 压缩前估算的 token 数 |
| `estimatedTokensAfter?` | `number` | 压缩后估算的 token 数（可选） |
| `usage?` | `Usage` | 压缩 LLM 调用的 token 使用量（持久化到 `CompactionEntry`，纳入会话总计） |
| `details?` | `T` | 可选，实现细节（默认填充 `CompactionDetails`: `{ readFiles, modifiedFiles }`） |

#### `CompactionEntry`（写入会话树的记录）

压缩完成后，`CompactionResult` 会被包装成 `CompactionEntry` 写入会话树：

| 字段 | 类型 | 说明 |
|------|------|------|
| `type` | `"compaction"` | entry 类型标识 |
| `id` | `string` | entry UUID |
| `parentId` | `string` | 父节点 UUID |
| `timestamp` | `string` | ISO 8601 创建时间戳（`new Date().toISOString()`） |
| `summary` | `string` | 压缩摘要文本 |
| `firstKeptEntryId` | `string` | 保留的第一条 entry UUID |
| `tokensBefore` | `number` | 压缩前 token 数 |
| `fromHook?` | `boolean` | 是否由扩展生成（`true`）而非内置逻辑 |
| `usage?` | `Usage` | 压缩 LLM 调用的 token 使用量（纳入会话总计） |
| `details?` | `T` | 可选，实现细节 |

### Branch Summary（分支摘要）

分支摘要是当用户从一个会话分支跳转到另一个分支时，对离开的分支做的摘要。与 compaction 共用底层摘要引擎。

```ts
// ⚠️ 返回类型：直接返回 BranchSummaryResult，不再 Result<> 包装
// 错误通过 BranchSummaryResult 的 error? / aborted? 字段传递
export async function generateBranchSummary(
  entries: SessionEntry[],
  options: GenerateBranchSummaryOptions
): Promise<BranchSummaryResult>

export function prepareBranchEntries(
  entries: SessionEntry[],
  tokenBudget?: number
): BranchPreparation

// ⚠️ 第一个参数类型：ReadonlySessionManager（不是 Session）
export async function collectEntriesForBranchSummary(
  session: ReadonlySessionManager,
  oldLeafId: string | null,
  targetId: string
): Promise<CollectEntriesResult>
```

**`BranchSummaryResult` 字段**（不再有 `BranchSummaryError` 类型）：

```ts
interface BranchSummaryResult {
  summary?: string;       // 摘要文本（aborted/error 时 undefined）
  usage?: Usage;          // LLM 调用 token 使用量
  readFiles?: string[];   // 摘要过程中读到的文件
  modifiedFiles?: string[]; // 摘要过程中改动的文件
  aborted?: boolean;      // 是否被中断
  error?: string;         // 错误信息（正常完成时 undefined）
}
```

> **`BranchSummaryDetails`**：分支摘要写入 `BranchSummaryEntry.details` 的文件追踪结构，与 `CompactionDetails` 同构：`{ readFiles: string[]; modifiedFiles: string[] }`（branch-summarization.ts:44-47）。

---

## 摘要格式

Compaction 生成的摘要采用结构化格式：

```
## Goal
[用户想要达成什么目标]

## Constraints & Preferences
- [用户提到的约束、偏好]

## Progress
### Done
- [x] [已完成的任务]

### In Progress
- [ ] [进行中的工作]

### Blocked
- [阻碍进展的问题]

## Key Decisions
- **[决策]**：[简要理由]

## Next Steps
1. [下一步计划]

## Critical Context
- [继续工作所需的关键信息]

<read-files>
path/to/file1.ts
</read-files>

<modified-files>
path/to/file2.ts
</modified-files>
```

---

## 使用示例

> 下面把示例拆成两条主线：**SDK 集成者**（subscribe 层，用 `AgentSession`）和**扩展开发者**（扩展层，用 `pi.on` + `ExtensionContext`）。每个示例都标注了所属层——不要把两层的 API 混在同一个变量上。

### 主线 A：SDK 集成者（subscribe 层 / `AgentSession`）

适用场景：你自己跑 agent loop，想监听压缩状态、主动触发压缩、做 UI 反馈。

#### A1. 检查是否需要压缩

```ts
import {
  shouldCompact,
  calculateContextTokens,
  getLastAssistantUsage,
  DEFAULT_COMPACTION_SETTINGS,
} from "@earendil-works/pi-coding-agent"

// agentSession 是 AgentSession 实例（subscribe 层）
// 注意：AgentSession 没有 .entries 属性，entry 通过 sessionManager.getBranch() 读取
const entries = agentSession.sessionManager.getBranch()
const lastUsage = getLastAssistantUsage(entries)
const contextTokens = lastUsage ? calculateContextTokens(lastUsage) : 0

const contextWindow = agentSession.model?.contextWindow ?? 128000

if (shouldCompact(contextTokens, contextWindow, DEFAULT_COMPACTION_SETTINGS)) {
  console.log("需要压缩！")
  // 见 A2：触发压缩流程
}
```

#### A2. 手动触发压缩并监听事件

```ts
// subscribe 层：监听 AgentSessionEvent（注意是 compaction_end，不是 session_compact）
agentSession.on((event) => {
  if (event.type === "compaction_start") {
    console.log("压缩开始，原因:", event.reason) // "manual" | "threshold" | "overflow"
  }
  if (event.type === "compaction_end") {
    console.log("压缩结束，是否被中断:", event.aborted)
    if (event.result) {
      console.log("摘要 token 前后:", event.result.tokensBefore, "→", event.result.estimatedTokensAfter)
    }
    if (event.errorMessage) console.error("压缩错误:", event.errorMessage)
  }
})

// subscribe 层：AgentSession.compact() 签名是 compact(customInstructions?: string): Promise<CompactionResult>
// 注意：只接收一个可选字符串，不接收 { customInstructions, onComplete, onError } 对象（那是扩展层）
try {
  const result = await agentSession.compact("Focus on API changes")
  console.log("压缩摘要:", result.summary)
  console.log("保留起始 entry:", result.firstKeptEntryId)
} catch (err) {
  console.error("压缩失败:", err)
}
```

### 主线 B：扩展开发者（扩展层 / `ExtensionContext` / `pi.on`）

适用场景：你写扩展，想拦截压缩、替换摘要内容、或用扩展 API 触发压缩。

#### B1. 用扩展 API 触发压缩（fire-and-forget）

```ts
// 扩展层：ExtensionContext.compact(options?: CompactOptions): void
// 不返回 Promise，结果只能通过 onComplete/onError 回调拿
export default function (pi) {
  pi.command("my-compact", (ctx) => {
    ctx.compact({
      customInstructions: "Focus on API changes",
      onComplete: (result) => {
        console.log("压缩摘要:", result.summary)
        console.log("压缩前 token 数:", result.tokensBefore)
        console.log("保留起始 entry:", result.firstKeptEntryId)
      },
      onError: (err) => console.error("压缩失败:", err),
    })
  })
}
```

#### B2. 拦截 / 替换压缩事件（扩展层事件）

```ts
// 扩展层事件（pi.on），不是 AgentSessionEvent。subscribe 层用 agentSession.on('compaction_end') 不等效。
pi.on("session_before_compact", async (event) => {
  console.log("即将压缩，当前 token 数:", event.preparation.tokensBefore)
  console.log("触发原因:", event.reason)         // "manual" | "threshold" | "overflow"
  console.log("失败会重试?", event.willRetry)     // overflow 且 stopReason !== "stop" 时为 true
  // 返回 { cancel: true } 可取消压缩
  // 返回 { compaction: CompactionResult } 可替换内置摘要为自定义实现
})

pi.on("session_compact", (event) => {
  console.log("压缩完成，摘要长度:", event.compactionEntry.summary.length)
  console.log("触发原因:", event.reason)
  console.log("失败会重试?", event.willRetry)
  console.log("来自扩展?", event.fromExtension)   // boolean
})
```

### 高级用法：直接调用 `compact()`（★ 非稳定 API，慎用）

> ⚠️ **警告**：此方式依赖内部子路径 `@earendil-works/pi-coding-agent/core/compaction/compaction`（`prepareCompaction` 不在主入口导出），**不属于稳定公开 API**，升级时可能被破坏。一般 SDK 集成者请用 A2 的 `agentSession.compact()`，扩展开发者请用 B1 的 `ctx.compact()`。仅在你需要完全控制摘要 LLM 调用（自定义 model/apiKey/streamFn）时才考虑此方式。

```ts
import { compact, DEFAULT_COMPACTION_SETTINGS } from "@earendil-works/pi-coding-agent"
// ⚠️ 内部子路径导入，非稳定 API
import { prepareCompaction } from "@earendil-works/pi-coding-agent/core/compaction/compaction"

const settings = { ...DEFAULT_COMPACTION_SETTINGS, keepRecentTokens: 30000 }

// entries 通常来自 agentSession.sessionManager.getBranch()
const preparation = prepareCompaction(entries, settings)
if (preparation) {
  try {
    const result = await compact(
      preparation,
      model,                        // Model<any>
      process.env.API_KEY,          // apiKey
      undefined,                     // headers（可选）
      "Focus on API changes",        // customInstructions（可选）
    )

    console.log("压缩摘要:", result.summary)
    console.log("压缩前 token 数:", result.tokensBefore)
  } catch (err) {
    console.error("压缩失败:", err)
  }
}
```

### 扩展层事件类型参考

#### SessionBeforeCompactEvent 字段

```ts
interface SessionBeforeCompactEvent {
  type: "session_before_compact";
  preparation: CompactionPreparation;          // 切点/上下文/保留消息等准备信息
  branchEntries: SessionEntry[];               // 当前分支的所有 entry
  customInstructions?: string;                 // 用户或之前扩展提供的额外指令（可修改后传回）
  reason: "manual" | "threshold" | "overflow"; // 触发原因
  willRetry: boolean;                          // overflow 且 stopReason !== "stop" 时为 true
  signal: AbortSignal;                         // 可传递给 LLM 调用
}
```

#### SessionCompactEvent 字段

```ts
interface SessionCompactEvent {
  type: "session_compact";
  compactionEntry: CompactionEntry;            // 已写入会话树的压缩记录
  fromExtension: boolean;                      // 是否由扩展生成（vs pi 内置逻辑）
  reason: "manual" | "threshold" | "overflow";
  willRetry: boolean;
}
```

### subscribe 层事件参考（`AgentSessionEvent`）

> 这些事件用 `agentSession.on(listener)` 监听，与上面的扩展层事件**不同**。做 UI 反馈（进度条、错误提示）通常用这一层。

| 事件类型 | 关键字段 | 说明 |
|---------|---------|------|
| `compaction_start` | `reason` | 压缩开始（manual/threshold/overflow） |
| `compaction_end` | `reason`, `result?`, `aborted`, `willRetry`, `errorMessage?` | 压缩结束；`result` 为 `CompactionResult` 或 `undefined`（失败/取消时） |
| `summarization_retry_scheduled` | `attempt`, `maxAttempts`, `delayMs`, `errorMessage` | 瞬态 provider 故障，准备重试摘要 LLM 调用 |
| `summarization_retry_attempt_start` | `source: "compaction" \| "branchSummary"`, `reason?` | 重试开始 |
| `summarization_retry_finished` | — | 重试链结束 |

#### 扩展自定义摘要（完整示例）

```ts
import { convertToLlm, serializeConversation } from "@earendil-works/pi-coding-agent"

pi.on("session_before_compact", async (event) => {
  const { preparation } = event

  // 将 AgentMessage[] 转为 LLM Message[]，再序列化为纯文本
  const conversationText = serializeConversation(
    convertToLlm(preparation.messagesToSummarize)
  )
  // 输出格式：
  // [User]: 消息内容
  // [Assistant thinking]: 推理内容
  // [Assistant]: 回复内容
  // [Assistant tool calls]: read(path="...")
  // [Tool result]: 工具输出

  // 用你自己的模型生成摘要
  const summary = await myCustomModel.summarize(conversationText)

  // ⚠️ 注意：这里返回的 summary 不会被 pi 自动附加 <read-files>/<modified-files> 标签
  // （只有内置 compact() 分支才附加）。需要文件追踪的话，扩展得自己拼接，见「关键细节 7」
  return {
    compaction: {
      summary,
      firstKeptEntryId: preparation.firstKeptEntryId,
      tokensBefore: preparation.tokensBefore,
      details: { /* 自定义数据 */ },
    }
  }
})
```

---

## 关键细节与陷阱

### 1. `shouldCompact` 的计算逻辑

公式：`contextTokens > contextWindow - reserveTokens`。即当上下文 token 数超过 (模型窗口 - 保留 token) 时触发压缩。`reserveTokens` 默认 16384，意味着在 128K 窗口的模型上，约 112K token 时触发。

### 2. 切割点选择确保不破坏工具调用链

`findCutPoint` 只在"安全的位置"切割：user、assistant、bashExecution、custom、branchSummary、compactionSummary 消息处。不会在 toolResult 中间切割，也不会在 compaction entry 上切割。

### 3. 轮次分裂处理

如果切割点恰好在一个轮次中间（例如 assistant 还在执行工具调用），`isSplitTurn` 会为 `true`。此时 `compact()` 会分别生成：历史摘要 + 轮次前缀摘要，两者合并后仍是用户可读的上下文。

合并产物格式（`---` 分隔，compaction.ts:881）：

```
[历史摘要正文]

---

**Turn Context (split turn):**

[轮次前缀摘要正文]
```

自定义 `details` 时要注意：这个 `---` 分隔符会出现在最终 `CompactionEntry.summary` 字符串里，不是单独的字段。

### 4. 增量摘要

如果之前有过压缩（存在 `CompactionEntry`），`prepareCompaction` 会从**上一个 `CompactionEntry.summary`**（不是 `details`）提取 `previousSummary`（compaction.ts:726-730）。这允许后续压缩只对"新增消息"生成摘要并合并到已有摘要中，而不是每次都重新摘要全部历史。

### 5. 摘要模型可以不同于对话模型

`compact()` 接受独立的 `model` 参数，这意味着可以用更小/更便宜的模型做摘要（如 Haiku），而对话使用 Sonnet/Opus。

### 6. toolResult 内容会被截断

`serializeConversation` 中 `toolResult` 的输出被截断到 2000 字符（`TOOL_RESULT_MAX_CHARS = 2000`）。截断方式是**保留开头 + 追加标记**：`${text.slice(0, 2000)}\n\n[... N more characters truncated]`（`truncateForSummary`，utils.ts:95-99）。这是为了避免过大的工具输出塞爆摘要 prompt，也意味着你不需要自己再处理截断——传入完整内容即可。

### 7. 文件操作追踪与 `computeFileLists` 去重语义

无论压缩还是分支摘要，都会提取压缩区间内 assistant 消息中的 `read`、`write`、`edit` 工具调用参数，在摘要末尾以结构化标签附加。这确保后续会话仍知道哪些文件被操作过。

`computeFileLists`（utils.ts:62-67）的去重语义：

- `readFiles` = **只读未改**的文件（`read` 集合减去 `edited ∪ written`，再排序）
- `modifiedFiles` = `edited ∪ written` 并集，去重排序

也就是说，一个文件既被 `read` 又被 `edit`，只会出现在 `modifiedFiles` 里，不会同时出现在两个列表。扩展自定义 `details` 时按这个语义填，避免重复。

> ⚠️ **扩展自定义摘要的关键陷阱**：扩展通过 `session_before_compact` 返回 `{ compaction: CompactionResult }` 时，你返回的 `summary` **不会**再被 `formatFileOperations` 处理——自动附加 `<read-files>`/`<modified-files>` 标签的逻辑只在内置 `compact()` 分支里（compaction.ts:905-906）。agent-session.ts 的 `compact()`/`_runAutoCompaction()` 拿到 `extensionCompaction.summary` 后是原样写入 `CompactionEntry`（agent-session.ts:1836-1842）。所以如果你希望摘要里带文件标签，**扩展得自己拼接**。参考模式：

```ts
pi.on("session_before_compact", async (event) => {
  const { preparation, branchEntries } = event
  // 用 preparation.fileOps 算出文件清单（FileOperations 已提取好）
  const { readFiles, modifiedFiles } = computeFileLists(preparation.fileOps)
  const summary = await myCustomModel.summarize(/* ... */)
  // 自己拼标签
  const tags = formatFileOperations(readFiles, modifiedFiles)
  return {
    compaction: {
      summary: summary + tags,          // ← 必须自己拼
      firstKeptEntryId: preparation.firstKeptEntryId,
      tokensBefore: preparation.tokensBefore,
      details: { readFiles, modifiedFiles },
    },
  }
})
```

（`computeFileLists` / `formatFileOperations` 从子路径 `@earendil-works/pi-coding-agent/core/compaction/utils` 导入，非主入口。）

### 8. 错误处理方式

`compact()` 和 `generateSummary()` 在出错时直接 throw Error（不是返回 `Result<>`）。摘要生成失败的原因包括：
- `stopReason === "error"` → throw `"Summarization failed: ..."`
- `firstKeptEntryId` 缺失 → throw `"First kept entry has no UUID..."`

调用时用 try/catch 捕获。
