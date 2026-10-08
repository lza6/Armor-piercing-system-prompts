# 场景：企业接口接入评估（H07）

## 这是什么 / 不是什么

**是**：用户给出企业内部 LLM 接口文档（粘贴文本 / 文件 / Postman 截图），你判断能不能接入 pi-agent，输出**两样东西**：① 评估总结（自由发挥，不套模板）② models.json 配置初稿。

**不是**：
- **不是代测**：AI 一般访问不到企业内网，测试必须由用户执行。你只负责解读用户贴回来的响应/报错。
- **不是最终判定**：接口文档常不完整、甚至不准确（**文档说支持 ≠ 真的支持**）。你的结论是初稿，必须实测确认。

## 什么时候用

用户给了企业接口文档 / 内网模型地址，问「能不能接」「帮我配一下」「和 OpenAI 兼容吗」。与 [H02](H02-custom-provider.md) 的关系：H07 负责「判断 + 出 models.json 初稿」；要在扩展里用代码注册，再去 H02 把配置翻译成 `pi.registerProvider`。

## 核对清单（判定依据，逐条对照文档）

**硬性要求（任一不满足 → 接不了或上转换层）**：

1. **接口形态**：`POST {baseUrl}/chat/completions` + OpenAI 风格 `messages` 数组（`baseUrl` 只填到 `/v1` 级，别带 `/chat/completions`）；
2. **SSE 流式**：Pi Agent 永远发 `stream: true`，**没有非流式模式**；
3. **鉴权**：标准 `Authorization: Bearer <key>`（Key 解析成功就自动带上）；其它鉴权格式用 `headers` 字段自定义。

**判定清单**（逐条对照文档，不满足看右列）：

| 核对项 | 不满足时 |
|--------|---------|
| 端点 `/chat/completions` 结尾 | 路径对不上 → 接不了，或转换层 |
| 支持流式 | 不支持 → 接不了 |
| `messages` 是 `{role, content}`，role 认 `system`/`user`/`assistant`/`tool` | 只不认 `tool` → 转换层改角色；整体不符 → 接不了 |
| chunk 含 `choices[0].delta.content` / `delta.tool_calls` | Anthropic 格式（`content_block_delta`）→ 换 `api: "anthropic-messages"`；其它 → 接不了 |
| 鉴权是 Bearer | 其它方式 → `headers` 字段 |
| 认 `max_completion_tokens` 还是 `max_tokens` | 只认旧版 → `maxTokensField: "max_tokens"` |
| 支持 `tools` 字段 / 工具定义认 `strict` | 不认 tools → 清空工具集（代价大，见附录）；不认 strict → `supportsStrictMode: false` |
| 流式 `usage` 是否真返回 | 空/null → `supportsUsageInStreaming: false` |
| 推理模型思考参数 | 不认 `reasoning_effort` → `supportsReasoningEffort: false`；`thinkingFormat` 自建网关一般走默认 `"openai"`（完整取值见 `OpenAICompletionsCompat.thinkingFormat`，`packages/ai/src/types.ts`，共 11 个值） |

**compat 速查**（写进 models.json 的模型定义，字段都可选，**多数情况先别填**——先跑通，报什么错再对号入座）：

- 老版参数：`maxTokensField` / `supportsUsageInStreaming` / `supportsStore`
- 消息/角色/工具：`supportsDeveloperRole`（只认 system）/ `supportsStrictMode` / `requiresAssistantAfterToolResult` / `requiresToolResultName`
- 思考参数：`thinkingFormat` / `supportsReasoningEffort`
- 完整字段见 SDK 类型 `OpenAICompletionsCompat`（`packages/ai/src/types.ts`），其余（缓存类、厂商专用类）企业内网基本用不到

**兜底判断**：

- compat 救不了（role 只认三种、特殊路径、消息结构整体不符）→ 建议**转换层**：本地反向代理改写请求（改角色、改头、拼路径、删字段），几十行代码；
- 模型本身**能力缺失**（不支持流式、不支持工具）→ 转换层变不出来，只能取舍（见附录）；
- 不是 OpenAI 格式 → 换 `api` 协议名（`anthropic-messages` / `google-generative-ai` 等），每种协议各有一套自己的 compat（各协议的 compat 字段见 `packages/ai/src/types.ts` 中对应的 `*Compat` 接口，如 `AnthropicMessagesCompat`）。

## 输出

1. **总结**（自由发挥）：能不能接 / 文档里哪些点**存疑**——文档没写的一律标存疑，提醒实测确认 / 有什么坑（鉴权格式、字段名差异、工具调用风险）。
2. **models.json 配置初稿**（占位符让用户替换）：

```json
{
  "providers": {
    "enterprise": {
      "baseUrl": "http://内网地址/v1",
      "api": "openai-completions",
      "apiKey": "你的_appKey",
      "models": [
        {
          "id": "internal-model",
          "name": "内网模型",
          "compat": {
            "maxTokensField": "max_tokens",
            "supportsUsageInStreaming": false
          }
        }
      ]
    }
  }
}
```

> **注意**：上面示例只填了 `id` / `name` / `compat`（schema 只要求 `id` 必填）。生产环境建议补 `contextWindow` 和 `maxTokens`——两者缺省时会回退到默认值（`contextWindow` 默认 128000、`maxTokens` 默认 16384），**不会失效**，但可能与模型真实能力不符：`contextWindow` 不准会让自动压缩（compaction）的触发时机偏离（按 128k 算阈值，实际可能 32k 就该压）；`maxTokens` 不准会让输出长度限制偏离真实容量。

3. **收尾固定提醒**：配置是初稿，**请实测验证**（文档说支持 ≠ 真的支持）——在 pi 里换这个 provider 直接跑一次；报错就把信息贴回来，我对照调整 compat；有问题随时继续问。

## 附录：不支持 Function Calling 的取舍

Pi Agent 在**有工具时**才往请求里带 `tools` 参数（`if (activeTools && activeTools.length > 0)` 才发 `params.tools`）；pi-coding-agent 默认运行时通常有内置工具，所以效果接近「默认带」。网关不认 `tools` 就报错。清空工具集（`session.setActiveToolsByName([])`，pi-coding-agent 的 AgentSession 方法，非 pi-agent-core）能让请求不带 tools，但 Agent 会退化成纯聊天：内置/自定义工具、工具拦截、ReAct 循环全部失效。要保留工具能力只能自己实现 prompt-based 工具循环（工具描述写进系统提示词、解析模型输出的特定格式 JSON），工程量大且不稳定。**接之前如实告诉用户这个代价**。
