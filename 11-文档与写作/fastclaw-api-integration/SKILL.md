# FastClaw API Integration

Use this skill when integrating an upstream application with FastClaw as an
agent runtime.

The canonical reference is `docs/upstream-api.md` in the FastClaw repository.
Follow that document over guesses from existing dashboard code.

## Integration Boundary

Use `/v1/*` for upstream applications:

- `POST /v1/chat/completions` for end-user chat.
- `GET /v1/agents` to list agents accessible to the API key.
- `POST /v1/users` to provision upstream end-users, or lazy-provision via chat.
- `GET /v1/usage` for billing dashboards.
- `PUT /v1/quota`, `GET /v1/quota`, `DELETE /v1/quota` for paid-plan limits.

Use `/api/*` or the `fastclaw` CLI only for operator/admin workflows such as
creating agents, configuring providers, installing skills, managing channels,
and runtime/project administration.

## Required Inputs

Before writing integration code, identify:

- FastClaw base URL.
- API key, stored server-side only.
- Agent ID.
- Upstream stable user ID field.
- Upstream conversation/session ID field.
- Whether usage/quota billing must be wired.
- Whether attachments/images must be supported.

Do not expose FastClaw API keys to browsers or mobile clients. Route calls
through the upstream backend.

## Chat Contract

Call:

```http
POST /v1/chat/completions
Authorization: Bearer <FASTCLAW_API_KEY>
Content-Type: application/json
X-Fastclaw-Session-Key: <deterministic-session-key>
```

Body:

```json
{
  "agent_id": "agt_...",
  "stream": true,
  "user": "upstream-user-id",
  "messages": [
    { "role": "user", "content": "..." }
  ],
  "params": {}
}
```

Rules:

- `agent_id` selects the FastClaw agent. Body wins over
  `X-Fastclaw-Agent-ID`.
- `user` is the upstream stable user ID. Body wins over
  `X-Fastclaw-End-User`.
- `X-Fastclaw-Session-Key` controls conversation history. Use a deterministic
  key such as `<app>:<user-id>:<conversation-id>`.
- `params` is per-turn structured context. It is shown to the agent but not
  persisted.
- Use `images` or `imageUrls` for image URLs/data URLs intended for vision
  models.
- Use `attachments` for general files, with optional `name`.

## User Provisioning

Either explicitly provision:

```http
POST /v1/users
Authorization: Bearer <FASTCLAW_API_KEY>
Content-Type: application/json

{
  "external_id": "upstream-user-id",
  "display_name": "Optional Display Name"
}
```

Or skip provisioning and pass `user` on every chat call. FastClaw will lazy
create the app-user for that API key.

Store the returned `user_id` if the upstream app needs usage/quota lookups.

## Usage And Quota

Query usage:

```http
GET /v1/usage?user_id=u_...&days=30
Authorization: Bearer <FASTCLAW_API_KEY>
```

Set quota after subscription changes:

```http
PUT /v1/quota
Authorization: Bearer <FASTCLAW_API_KEY>
Content-Type: application/json

{
  "user_id": "u_...",
  "monthly_token_limit": 5000000,
  "monthly_request_limit": 10000,
  "reset_day": 1
}
```

## Implementation Checklist

1. Add server-side FastClaw client configuration:
   - base URL
   - API key
   - agent ID
2. Add or reuse upstream conversation IDs.
3. Map upstream user IDs to FastClaw `user` or `/v1/users`.
4. Implement streaming SSE parsing for `/v1/chat/completions`.
5. Persist or derive `X-Fastclaw-Session-Key` per conversation.
6. Add attachment support only if the product UI needs it.
7. Add `/v1/usage` and `/v1/quota` only if billing/paid limits are required.
8. Handle OpenAI-style error objects:
   - `400` invalid request
   - `401` auth failure
   - `404` inaccessible agent
   - `429` rate/quota limit
   - `503` subsystem disabled

## Do Not

- Do not call dashboard `/api/chat/stream` for upstream app chat unless you are
  embedding FastClaw's own dashboard semantics.
- Do not put FastClaw API keys in frontend code.
- Do not use email/display name as the stable FastClaw `user` value.
- Do not reuse one session key across unrelated conversations.
- Do not configure agents/providers/skills through `/v1`; use dashboard,
  `/api/*`, or CLI for admin workflows.
