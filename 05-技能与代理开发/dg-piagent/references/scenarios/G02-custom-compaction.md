# 场景：自定义压缩策略 (G02)

## 目标

通过 `session_before_compact` / `session_compact` 扩展事件介入压缩流程——**替换**默认摘要为自定义实现、**取消**压缩、或在压缩完成后做副作用（通知、日志、追踪）。

## 什么时候用

- **默认摘要不够聚焦**：内置摘要模板是通用化的（Goal / Progress / Decisions / Next Steps），如果你要突出业务维度（API 变更、错误历程、SQL 订单流），需要自己生成摘要
- **要换摘要模型**：默认用对话同款模型生成摘要；你想用更便宜的 Haiku 或自研摘要器
- **要可观测性**：压缩完成后写日志、发 SSE、更新监控指标
- **要取消压缩**：某些 reason（如 `manual`）下你不想让压缩发生

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `pi.on("session_before_compact")` | 压缩前拦截，可替换摘要或取消 | [sdk_doc/04-events.md](../sdk_doc/04-events.md)、[sdk_doc/18-compaction.md](../sdk_doc/18-compaction.md) |
| `pi.on("session_compact")` | 压缩完成后通知（只读） | 同上 |
| `SessionBeforeCompactResult` | 扩展返回值：`{ cancel?, compaction? }`（**注意：没有 `customInstructions` 字段**） | [sdk_doc/18-compaction.md](../sdk_doc/18-compaction.md) |
| `ctx.compact(options?)` | 主动触发压缩（不等待结果） | ExtensionContext 方法，详见 [sdk_doc/04-events.md](../sdk_doc/04-events.md) |

> **派发层提醒**：`session_before_compact` / `session_compact` 是**扩展层 `pi.on` 事件**，外部宿主用 `session.subscribe` **收不到**。subscribe 层只能收到 `compaction_start` / `compaction_end`（不带 preparation 详情）。详见 [04-events.md 坑 4](../sdk_doc/04-events.md#坑-46-个扩展独有事件sessionsubscribe-静默收不到-最大集成坑)。

## ⚠️ 最大陷阱：扩展返回 `customInstructions` 会被静默忽略

**事实**：`SessionBeforeCompactResult` 只有 `cancel?: boolean` 和 `compaction?: CompactionResult` 两个字段（源码：`extensions/types.ts`）。**没有 `customInstructions` 字段**。

- **manual 路径**：`session.compact(instructions)` 把入参 `instructions` 透传给 `compact()`；扩展返回的 `result.customInstructions` **没有任何代码读取**。
- **auto 路径**（自动压缩）：传给 `compact()` 的 `customInstructions` **被硬编码为 `undefined`**，无论扩展返回什么都不会被使用。

**结论**：想"给摘要模型注入额外指令"，扩展层 **没有官方路径**。真正能做的只有两条：
1. **完全替换摘要**：`return { compaction: CompactionResult }`，自己生成 summary 字符串
2. **用户侧传入**：调用 `session.compact("Focus on API changes")` 或在扩展中用 `ctx.compact({ customInstructions: "..." })` 把指令传给 manual 压缩（扩展无法干预 auto 压缩的指令）

## 实现思路

### 模式 A：完全替换摘要（推荐用于"自定义聚焦"）

扩展自己调摘要模型（或调外部服务），构造 `CompactionResult` 返回。SDK 会跳过内置摘要逻辑，直接把你的结果写入会话树。

```ts
import {
  convertToLlm,
  serializeConversation,
  type ExtensionFactory,
} from "@earendil-works/pi-coding-agent";

const myCompactionExt: ExtensionFactory = (pi) => {
  pi.on("session_before_compact", async (event, ctx) => {
    const { preparation, reason, willRetry } = event;

    // event.preparation.messagesToSummarize 是 AgentMessage[]
    // convertToLlm 转 LLM Message[]，serializeConversation 转纯文本
    const convoText = serializeConversation(
      convertToLlm(preparation.messagesToSummarize),
    );

    // 用你自己的模型 / prompt 生成摘要
    const summary = await mySummarizer.summarize(convoText, {
      focusOn: "api-changes",
      previousSummary: preparation.previousSummary, // 增量摘要
    });

    // 必须返回完整的 CompactionResult
    return {
      compaction: {
        summary,                                      // 你的摘要文本
        firstKeptEntryId: preparation.firstKeptEntryId, // 通常沿用 preparation 的
        tokensBefore: preparation.tokensBefore,
        details: { /* 自定义数据，可选 */ },
        // usage?: Usage,  // 可选：LLM 调用统计，如自定义摘要模型有 usage 可填
      },
    };
  });
};

export default myCompactionExt;
```

**关键细节**：
- `firstKeptEntryId` **必须**用 `preparation.firstKeptEntryId`——这是 SDK 切点查找的结论，自己随便填会导致会话树结构错乱
- `tokensBefore` 建议沿用 `preparation.tokensBefore`，便于监控
- `details` 是可选的自定义数据（如 `{ readFiles, modifiedFiles }`），写入 `CompactionEntry.details`

### 模式 B：取消压缩

```ts
pi.on("session_before_compact", (event, ctx) => {
  if (event.reason === "manual") {
    // 用户手动 /compact 时拒绝；但 overflow 必须放行（否则上下文会爆）
    return { cancel: true };
  }
});
```

⚠️ **不要无脑 cancel overflow**——overflow 是上下文已经超窗口的紧急压缩，取消会让下一轮 LLM 调用直接失败。

### 模式 C：压缩完成后做副作用

```ts
pi.on("session_compact", (event, ctx) => {
  const { compactionEntry, fromExtension, reason, willRetry } = event;

  // 通知用户（TUI/RPC 模式可用，JSON/print 模式下 ctx.hasUI 为 false）
  if (ctx.hasUI) {
    ctx.ui.notify(
      `上下文已压缩（${reason}），保留了从 ${compactionEntry.firstKeptEntryId.slice(0, 8)} 起的消息`,
      "info",
    );
  }

  // 持久化到外部日志（如 trace 表）
  // 注意：在 TUI 模式下慎用 console.log——会破坏渲染
  if (ctx.mode === "rpc" || ctx.mode === "json") {
    console.log("[Compaction]", {
      summaryLen: compactionEntry.summary.length,
      tokensBefore: compactionEntry.tokensBefore,
      fromExtension,
      willRetry,
    });
  }
});
```

**字段速查**（源码 `extensions/types.ts`）：

| 字段 | 类型 | 含义 |
|------|------|------|
| `compactionEntry` | `CompactionEntry` | 已写入会话树的压缩记录（含 `summary` / `firstKeptEntryId` / `tokensBefore` / `fromHook` / `details`） |
| `fromExtension` | `boolean` | 是否由扩展生成（即模式 A 触发）vs 内置逻辑 |
| `reason` | `"manual" \| "threshold" \| "overflow"` | 触发原因 |
| `willRetry` | `boolean` | overflow 时为 `true`——压缩后会重试被中断的 turn |

## 触发阈值与 CompactionSettings

压缩的触发公式（源码 `compaction.ts`）：

```
shouldCompact = contextTokens > contextWindow - settings.reserveTokens
```

即"上下文已用 token 数 > 模型窗口 - 预留值"时触发。**`reserveTokens` 不是"阈值"本身**，而是"给摘要 prompt 和模型输出预留的空间"——反推出阈值。

**默认值**（`compaction.ts`）：

| 字段 | 类型 | 说明 | 默认值 |
|------|------|------|--------|
| `enabled` | `boolean` | 是否启用自动压缩 | `true` |
| `reserveTokens` | `number` | 为摘要 prompt 和模型输出保留的 token（反推触发阈值） | `16384` |
| `keepRecentTokens` | `number` | 压缩后保留的最近上下文 token 近似值 | `20000` |

> ⚠️ **CompactionSettings 只有这 3 个字段**，没有 `threshold` / `triggerRatio` 等。调整触发时机只能改 `reserveTokens`，调整保留量改 `keepRecentTokens`。修改方式见 [场景 B02](B02-settings.md)。

## event 字段速查

### `SessionBeforeCompactEvent`（源码 `extensions/types.ts`）

| 字段 | 类型 | 含义 |
|------|------|------|
| `preparation` | `CompactionPreparation` | 切点分析结果（含 `firstKeptEntryId` / `messagesToSummarize` / `turnPrefixMessages` / `isSplitTurn` / `tokensBefore` / `previousSummary` / `fileOps` / `settings`） |
| `branchEntries` | `SessionEntry[]` | 当前分支的所有 entry（用于自定义分析） |
| `customInstructions?` | `string` | **入参**——manual 路径下是 `session.compact(x)` 传入的 x，auto 路径下恒为 `undefined`。**不是返回字段** |
| `reason` | `"manual" \| "threshold" \| "overflow"` | 触发原因 |
| `willRetry` | `boolean` | overflow 时为 `true`（压缩后会重试被中断的 turn） |
| `signal` | `AbortSignal` | 可传递给自定义 LLM 调用，支持取消 |

### `SessionBeforeCompactResult`（源码 `extensions/types.ts`）

| 字段 | 类型 | 含义 |
|------|------|------|
| `cancel?` | `boolean` | `true` 取消本次压缩 |
| `compaction?` | `CompactionResult` | 提供则跳过内置摘要，直接使用此结果 |

## ctx.compact(options) — 主动触发

`ExtensionContext` 提供 `compact(options?)` 方法（源码 `extensions/types.ts`），可在任意事件 handler 中触发压缩（不等待完成）：

```ts
pi.on("turn_end", (event, ctx) => {
  const usage = ctx.getContextUsage();
  if (usage && usage.tokens != null && usage.tokens > usage.contextWindow * 0.85) {
    ctx.compact(); // 软触发，不等结果
  }
});
```

**注意**：`ctx.compact()` 是 fire-and-forget（源码返回 `void`），想拿结果可订阅 `session_compact` 事件，或使用 `CompactOptions.onComplete` 回调。`ctx.compact()` 触发的压缩 reason 为 `"manual"`。

## 变体与延伸

- 自动总结每轮历史 → 见 [场景 G03](G03-auto-summarize.md)
- 压缩算法与切割点详解 → 见 [sdk_doc/18-compaction.md](../sdk_doc/18-compaction.md)
- SettingsManager 调整压缩参数 → 见 [场景 B02](B02-settings.md)
- 分支切换时的摘要（branch summary）→ 见 [sdk_doc/18-compaction.md](../sdk_doc/18-compaction.md) § Branch Summary
