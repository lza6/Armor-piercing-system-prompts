# Provider Capability Matrix (Runtime-Realistic)

## Policy First

- T0 runtime policy is currently **Claude Opus only**.
- Provider comparison below is for worker-terminal dispatch planning (mainly T1).

## How to Check Active Provider

```bash
jq -r '.T1.provider' .vnx-data/state/panes.json
```

## Capability Matrix

| Capability | claude_code | codex_cli | gemini_cli |
|---|---|---|---|
| Native skill invocation command | Yes | No guaranteed native slash-skill command | No guaranteed native slash-skill command |
| Fallback via inline role/context preamble | Supported (optional) | Supported (primary pattern) | Supported (primary pattern) |
| Context reset command | `/clear` | `/new` | `/clear` |
| Planning mode support | Yes | Limited (`/plan`, verify per version) | No |
| Thinking mode support | Yes | No | No |
| MCP support in worker runtime | Yes (provider/config dependent) | No | No |

## Dispatch Guidance by Provider

### Claude Code

- Use standard skill-first dispatch with `Workflow: [[@.claude/skills/<skill>/SKILL.md]]`.
- `Mode: planning` and `Mode: thinking` are available when appropriate.

### Codex CLI

- Do not assume native slash-skill invocation.
- Use explicit `Role`, `Workflow`, and `Context` fields in Manager Block.
- Avoid `Mode: thinking`; planning support must be validated in-session.
- Omit Claude-specific `Requires-Model` values unless confirmed compatible.

### Gemini CLI

- Do not assume native slash-skill invocation.
- Use explicit `Role`, `Workflow`, and `Context` fields in Manager Block.
- Avoid `Mode: thinking` and `Mode: planning`.
- Omit `Requires-Model` fields (single-model runtime behavior).

## MCP Routing Rule

If a task requires MCP-dependent capabilities, set `Requires-MCP: true` and route to T3.
