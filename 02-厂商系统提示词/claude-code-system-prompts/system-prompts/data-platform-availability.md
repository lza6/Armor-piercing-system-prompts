<!--
name: "Data: Platform availability"
description: "Feature availability matrix across Claude API provider platforms (first-party, Claude Platform on AWS, Bedrock, Vertex, and Foundry)"
ccVersion: "2.1.257"
-->
# Platform Availability

Which features work on which provider platform. **This table is the single source of truth in this skill** - per-feature sections elsewhere point here instead of restating availability. When writing code for a third-party platform (Bedrock, Vertex, Foundry) or Claude Platform on AWS, check this table first; a feature not supported there means use the first-party Claude API surface or a different approach.

Columns: **1P** = first-party Claude API, **P-AWS** = Claude Platform on AWS (Anthropic-operated, same-day parity), **Bedrock** = Amazon Bedrock, **Vertex** = Google Cloud Vertex AI, **Foundry** = Microsoft Foundry. Yes = GA, beta = beta, No = not supported.

| Feature | 1P | P-AWS | Bedrock | Vertex | Foundry | Notes |
|---|---|---|---|---|---|---|
| Messages, streaming, tool use | Yes | Yes | Yes | Yes | Yes | Core API |
| PDF input | Yes | Yes | Yes | Yes | beta | |
| Structured outputs / strict tool use | Yes | Yes | Yes | Yes | beta | |
| Adaptive thinking / effort | Yes | Yes | Yes | Yes | beta | |
| Extended thinking | Yes | Yes | Yes | Yes | beta | |
| Prompt caching (5m, 1h) | Yes | Yes | Yes | Yes | Yes | |
| Automatic prompt caching | Yes | Yes | Yes | Yes | Yes | The legacy Bedrock integration (Opus 4.6 and earlier) rejects top-level `cache_control` with a 400 - explicit breakpoints only there |
| Token counting | Yes | Yes | Yes | Yes | beta | |
| Citations | Yes | Yes | Yes | Yes | beta | |
| Search results content blocks | Yes | Yes | Yes | Yes | beta | |
| Fine-grained tool streaming | Yes | Yes | Yes | Yes | Yes | |
| Compaction | beta | beta | beta | beta | beta | |
| Context editing | beta | beta | beta | beta | beta | |
| Context windows (1M) | Yes | Yes | Yes | Yes | beta | |
| `inference_geo` (data residency) | Yes | Yes | No | No | No | |
| **Server-side tools** | | | | | | |
| &nbsp;&nbsp;Web search | Yes | Yes | No | Yes | beta | Vertex: basic `web_search_20250305` only (no `_20260209` dynamic filtering) |
| &nbsp;&nbsp;Web fetch | Yes | Yes | No | No | beta | |
| &nbsp;&nbsp;Code execution | Yes | Yes | No | No | beta | |
| &nbsp;&nbsp;Tool search | Yes | Yes | Yes | Yes | beta | Bedrock: InvokeModel API only, not Converse |
| &nbsp;&nbsp;Advisor tool | beta | beta | No | No | No | |
| **Client-implemented tools** | | | | | | |
| &nbsp;&nbsp;Bash, text editor, memory | Yes | Yes | Yes | Yes | beta | |
| &nbsp;&nbsp;Computer use | beta | beta | beta | beta | beta | |
| **Agentic / orchestration** | | | | | | |
| &nbsp;&nbsp;Agent Skills (Messages API) | Yes | Yes | No | No | beta | |
| &nbsp;&nbsp;Programmatic tool calling | Yes | Yes | No | No | beta | |
| &nbsp;&nbsp;MCP connector | beta | beta | No | No | beta | |
| &nbsp;&nbsp;Managed Agents | beta | beta | No | No | No | Foundry: No (inferred; not in Foundry docs either way) |
| &nbsp;&nbsp;Self-hosted sandboxes | beta | beta | No | No | No | P-AWS: worker authenticates with IAM/SigV4 or an AWS-Console API key + `AnthropicSelfHostedEnvironmentAccess` (Console environment keys don't work there); sessions on self-hosted environments cannot attach memory stores; `GET /v1/environments/{id}/work` list endpoint not supported, other work endpoints OK |
| **API endpoints** | | | | | | |
| &nbsp;&nbsp;Message Batches | Yes | Yes | No | No | No | |
| &nbsp;&nbsp;Files API | Yes | Yes | No | No | beta | |
| &nbsp;&nbsp;Models API | Yes | Yes | No | No | No | |
| **Other** | | | | | | |
| &nbsp;&nbsp;Mid-conversation system messages | Yes | Yes | Yes | Yes | No | {{OPUS_NAME}}, {{PREV_OPUS_NAME}}, {{PREV_FABLE_NAME}}, {{FABLE_NAME}}, {{PREV_MYTHOS_NAME}}, {{MYTHOS_NAME}}; not {{SONNET_NAME}}. Bedrock: InvokeModel passthrough, not ARN-versioned models |
| &nbsp;&nbsp;Turn-scoped (`clear_at`) system messages | beta | beta | beta | beta | No | Same models as mid-conversation system messages; beta `mid-conversation-system-clear-at-2026-08-21` (on Bedrock/Vertex pass the value as a beta) |
| &nbsp;&nbsp;Per-message `effort` (system message `output_config`) | beta | No | No | No | No | {{FABLE_NAME}}, {{MYTHOS_NAME}}, {{OPUS_NAME}}; beta `mid-conversation-output-config-2026-07-01`; Claude API at launch (Bedrock/Vertex/Foundry unconfirmed; {{OPUS_NAME}} excluded on Bedrock) |
| &nbsp;&nbsp;`thinking.display: "updates"` | beta | beta | beta | beta | beta | {{FABLE_NAME}}, {{MYTHOS_NAME}}, {{PREV_FABLE_NAME}}; beta `thinking-display-updates-2026-08-18` (pass the beta value per platform); without it `"updates"` is rejected as an unknown `display` value |
| &nbsp;&nbsp;Thinking block-binding controls | beta | beta | per model | per model | No | `thinking.block_binding` + `input_transformations`; beta `thinking-binding-controls-2026-08-01` (on Bedrock via the `anthropic_beta` body field); the controls beta arrives per model on Bedrock/Vertex - until then the header is rejected; the history-editing enforcement itself follows the account-age rule in `shared/model-migration.md` -> Migrating to {{FABLE_NAME}} from {{PREV_FABLE_NAME}} |
| &nbsp;&nbsp;Server-side `fallbacks` | beta | beta | No | No | No | `"default"` -> beta `server-side-fallback-2026-07-01`; array form -> beta `server-side-fallback-2026-06-01` |
| &nbsp;&nbsp;Fast mode | beta | No | No | No | No | Research preview, beta `fast-mode-2026-02-01`, first-party API only |
| &nbsp;&nbsp;Cache diagnostics | beta | No | No | No | No | First-party API only |
| &nbsp;&nbsp;Task budgets | beta | beta | No | No | No | Beta header `task-budgets-2026-03-13`; 3P availability not documented - assume unsupported |

<!--
GROUNDING (reviewer-only; stripped at runtime by processSkillMarkdown).
All paths are under docker_eval/resources/cdp-skill/public-docs/.

Primary source: build-with-claude/overview.mdx <PlatformAvailability> props
(claudeApi->1P, claudePlatformAws->P-AWS, bedrock->Bedrock, vertexAi->Vertex,
azureAi->Foundry; *Beta suffix->beta; prop absent->No). Per-row citations:

  Context windows          ov:44
  Adaptive thinking        ov:45
  Batch / Message Batches  ov:46; bed:360; vtx:381; fdy:507
  Citations                ov:47
  inference_geo            ov:48
  Effort                   ov:49
  Extended thinking        ov:50
  PDF input                ov:51
  Search results           ov:52
  Structured outputs       ov:53
  Advisor tool             ov:63
  Code execution           ov:64
  Web fetch                ov:65
  Web search               ov:66; agents-and-tools/tool-use/web-search-tool.mdx:41
  Bash/text-editor/memory  ov:72,75,74
  Computer use             ov:73
  Agent Skills             ov:83
  Fine-grained streaming   ov:84
  MCP connector            ov:85; agents-and-tools/mcp-connector.mdx:36
  Programmatic tool call   ov:86
  Tool search              ov:87; agents-and-tools/tool-use/tool-search-tool.mdx:24-30
  Compaction               ov:95
  Context editing          ov:96
  Automatic caching        ov:97
  Prompt caching 5m/1h     ov:98,99
  Token counting           ov:100
  Files API                ov:108; build-with-claude/files.mdx:17
  Managed Agents           managed-agents/overview.mdx:11,70-72; bed:360; vtx:381
  Self-hosted sandboxes    build-with-claude/claude-platform-on-aws.mdx:525,547
  Mid-convo system msgs    build-with-claude/mid-conversation-system-messages.mdx:15
  Fast mode                build-with-claude/fast-mode.mdx:23
  Cache diagnostics        build-with-claude/cache-diagnostics.mdx:15,1379
  Task budgets             build-with-claude/task-budgets.mdx:15
  Models API               bed:360; vtx:381; fdy:506

  ov  = build-with-claude/overview.mdx
  bed = build-with-claude/claude-in-amazon-bedrock.mdx
  vtx = build-with-claude/claude-on-vertex-ai.mdx
  fdy = build-with-claude/claude-in-microsoft-foundry.mdx
-->
