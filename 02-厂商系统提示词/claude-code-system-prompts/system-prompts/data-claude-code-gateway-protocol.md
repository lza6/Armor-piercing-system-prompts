<!--
name: "Data: Claude Code gateway protocol"
description: "Markdown reference documenting the Claude Code gateway wire contract, including OAuth 2.0 device flow, RFC 8414 discovery, Messages API inference, managed settings, model discovery, OTLP telemetry, error envelopes, TLS certificate pinning, and proxying to Bedrock, Vertex, and Foundry"
ccVersion: "2.1.265"
-->
# Claude Code gateway protocol

This is the wire contract the Claude Code CLI uses to talk to this gateway:
sign-in, inference, managed settings, and telemetry. It's served from the
gateway itself so it always matches the version you're running.

> **Stability:** this protocol exists to give you a more stable target than
> proxying raw CLI traffic. Auth is standard OAuth 2.0, inference is the
> Messages API, and headers are the lowest common denominator across
> backends. We keep it backwards compatible within reason to support older
> clients, but not forever — expect changes, managed settings in particular,
> with notice.

A developer points Claude Code at your gateway's base URL via `/login` and
the client does the rest. All paths below are relative to that base URL, and
the client does not follow cross-origin redirects.

## Flow

1. Client fetches `GET {base}/.well-known/oauth-authorization-server`.
2. On first contact, client fingerprints your TLS certificate and asks the
   user to trust it.
3. Client runs the RFC 8628 device flow: `POST device_authorization_endpoint`
   -> user approves in a browser at `verification_uri` -> client polls
   `token_endpoint` until it gets a bearer token.
4. Client sends `Authorization: Bearer <token>` on every subsequent request.
5. Client uses fixed paths under `{base}` for inference (`/v1/messages`),
   policy (`/managed/settings`), model discovery (`/v1/models`), and
   telemetry (`/v1/{metrics,logs,traces}`).
6. Before the token expires, client silently calls `token_endpoint` with
   `grant_type=refresh_token`. If you didn't issue a refresh token, the user
   is sent back through the browser flow instead.

## Discovery — required

`GET /.well-known/oauth-authorization-server` (unauthenticated)

RFC 8414 authorization server metadata. The client reads
`device_authorization_endpoint` and `token_endpoint` and ignores the rest;
both must be same-origin with `{base}`. `authorization_endpoint` is
intentionally absent.

    {
      "issuer": "https://gw.corp.example.com",
      "device_authorization_endpoint": "https://gw.corp.example.com/oauth/device_authorization",
      "token_endpoint": "https://gw.corp.example.com/oauth/token",
      "grant_types_supported": ["urn:ietf:params:oauth:grant-type:device_code", "refresh_token"]
    }

## Device authorization — required

`POST {device_authorization_endpoint}` (unauthenticated; answer directly, the
client follows no redirect here)

RFC 8628 §3.2. The client opens `verification_uri_complete` in the user's
browser and polls `token_endpoint` every `interval` seconds.

    {
      "device_code": "AbK9-s3n4C8H...",
      "user_code": "WDJB-MJHT",
      "verification_uri": "https://gw.corp.example.com/device",
      "verification_uri_complete": "https://gw.corp.example.com/device?user_code=WDJB-MJHT",
      "expires_in": 600,
      "interval": 5
    }

`device_code` should be >=256 bits, opaque, single-use. `user_code` should
use a base-20 charset (RFC 8628 §6.1).

The request body is form-encoded and may carry one extension parameter next
to RFC 8628 §3.1's: `surface`, a stable identifier of the client application
that is signing in. Claude Code sends `surface=claude_code`. Record it if you
attribute sessions by client; otherwise ignore it, as OAuth servers do for
any parameter they do not recognize (RFC 6749 §3.1, §3.2). Claude Code also sends `User-Agent: claude-code/<version>` on
the metadata, device, token and refresh requests.

## Verification page — required

`GET/POST {verification_uri}` (browser-facing; the client never calls this)

Accept the user code, authenticate the user against your IdP, and mark the
matching `device_code` approved so the next token poll succeeds. Apply a
per-IP rate limit (RFC 8628 §5.1) and don't auto-submit a pre-filled code
(§5.4).

## Token — required

`POST {token_endpoint}` (unauthenticated,
`application/x-www-form-urlencoded`; answer directly, the client follows no
redirect here)

**Device grant** (`grant_type=urn:ietf:params:oauth:grant-type:device_code`):

| Status | Body | Client reaction |
|---|---|---|
| 200 | `{"access_token","token_type":"Bearer","expires_in","refresh_token"?}` | Login complete. `refresh_token` is optional; omit it and the client re-runs the device flow on expiry. |
| 400 | `{"error":"authorization_pending"}` | Keep polling. |
| 400/429 | `{"error":"slow_down"}` | Add 5s to the poll interval. |
| 400 | `{"error":"access_denied"}` | Stop. |
| 400 | `{"error":"expired_token"}` | Stop. |

**Refresh grant** (`grant_type=refresh_token`): return a fresh
`{"access_token","token_type","expires_in","refresh_token"}` on 200. Return
`401 {"error":"invalid_grant"}` to force re-login — this is your
deprovisioning hook.

## Messages — required

`POST /v1/messages` and `POST /v1/messages/count_tokens` (bearer)

The Anthropic Messages API (https://platform.claude.com/docs/en/api/messages),
unchanged. Proxy to your upstream and stream the response back. Enforce your
model allowlist here, returning `400 invalid_request_error` for a denied
model. Don't buffer SSE on the `stream: true` path. The client always sets
`Content-Length`, so you may reject chunked-without-CL (`411`) and cap body
size (`413`). The client doesn't assume server-side tools are available. The
client also sends `x-app` and `x-stainless-*` headers — pass them through or
drop them, but don't reject the request because of them.

## Managed settings — optional

`GET /managed/settings` (bearer)

The authenticated user's Claude Code `managed-settings.json`; see
https://code.claude.com/docs/en/settings for the key reference. The client
polls about once an hour; support `ETag`/`If-None-Match` -> `304` to keep
that cheap. Return `404` for "no managed policy"; `200 {}` means "this user
has an empty policy" — they're not the same. **This is the endpoint most
likely to change.**

## Models — optional

`GET /v1/models` (bearer)

Anthropic models-list shape: `{"data":[{"id","display_name"},...]}`. Use
Anthropic-style IDs (`claude-{family}-{major}-{minor}`) — the client's
model-family logic keys on that shape. The client only calls this when
`CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY` is set on the client, which you
can push via the `env` block in `/managed/settings`. Return `404` to fall
back to the client's built-in list.

## Telemetry — optional

`POST /v1/metrics`, `/v1/logs`, `/v1/traces` (bearer)

OTLP/HTTP (protobuf or JSON). When connected to a gateway the client sends
telemetry here and ignores `OTEL_EXPORTER_OTLP_*` env vars, with one
exception: when your own `/managed/settings` document names an OTLP endpoint
on another host (https, or http to loopback, at a collector's
`…/v1/<signal>` path) and nothing has overridden it, the
client exports there as an ordinary managed install (set the protocol and
headers in the same document; where it does not, the developer's apply) and
never sends the gateway bearer to it. Return `200`
whether you forward or discard — `404` makes the client's exporter log an
error on every flush.

## Errors

OAuth endpoints use `{"error":"...","error_description":"..."}`
(RFC 6749/8628). Bearer-authenticated endpoints use the Anthropic envelope so
the SDK surfaces the message to the user:

    {"type":"error","error":{"type":"authentication_error","message":"..."}}

| HTTP | error.type | Use for |
|---|---|---|
| 400 | `invalid_request_error` | Denied model, malformed body, policy violation |
| 401 | `authentication_error` | Missing/expired/invalid bearer; client prompts re-login |
| 403 | `permission_error` | Authenticated but not allowed |
| 413 | `request_too_large` | Body over your cap |
| 429 | `rate_limit_error` | Throttling; include `Retry-After` |
| 429 | `billing_error` | The user's own cap on your gateway is reached; see Usage-limit headers below |
| 501 | `not_supported` | Endpoint not available on this backend |
| 529 | `overloaded_error` | Upstream at capacity; client backs off and retries |
| 5xx | `api_error` | Anything else |

## Usage-limit headers — optional

If you enforce a per-user spend or usage cap, report the caller's standing
against it on each successful `POST /v1/messages` response and Claude Code
(2.1.225 and later, when signed in to a gateway) shows its usual "You've used
NN% of your usage credits · resets …" notice past 75% and again past 95%.
These are the same `anthropic-ratelimit-unified-*` headers api.anthropic.com
sends its subscribers, so strip the upstream's own `anthropic-ratelimit-*`
response headers first — otherwise your org-wide quota reaches users as if it
were theirs. Send none of these for a user with no cap.

| Header | Value |
|---|---|
| `anthropic-ratelimit-unified-status` | `allowed`, or `allowed_warning` once past a threshold |
| `anthropic-ratelimit-unified-representative-claim` | `overage` — the window-agnostic claim; `5h`/`7d` mean rolling 5-hour/7-day windows the client does time math on, so don't borrow them for a calendar budget |
| `anthropic-ratelimit-unified-overage-status` | Same value as `-status` |
| `anthropic-ratelimit-unified-overage-utilization` | Fraction of the cap used, two decimals, kept below `1` while you're still allowing requests (`0.82`) |
| `anthropic-ratelimit-unified-overage-surpassed-threshold` | `0.75` or `0.95` once utilization passes it — this header is what triggers the client's notice; omit it below 75% |
| `anthropic-ratelimit-unified-reset`, `-overage-reset` | When the cap resets, Unix seconds |

When the cap is reached, reject `POST /v1/messages` before proxying:

    HTTP/1.1 429
    retry-after: 37800
    x-should-retry: false
    anthropic-ratelimit-unified-status: rejected
    anthropic-ratelimit-unified-reset: 1786147200
    anthropic-ratelimit-unified-overage-reset: 1786147200
    anthropic-ratelimit-unified-overage-utilization: 1
    anthropic-ratelimit-unified-overage-surpassed-threshold: 1
    anthropic-ratelimit-unified-overage-period: daily
    anthropic-ratelimit-unified-overage-disabled-reason: org_spend_cap_reached

    {"type":"error","error":{"type":"billing_error","message":"spend limit reached (daily; resets 2026-08-08 00:00 UTC) — request an increase at https://go.corp.example.com/claude-limits"}}

Leave `representative-claim` and `overage-status` off the 429. With them the
client composes its own "You've hit your limit" line and drops your message;
without them it prints `error.message` as-is (older clients too, behind a
generic "API Error" prefix), so put the period, the reset time, and what the
user should do next in that one sentence. `retry-after` is seconds until the
reset; `x-should-retry: false` keeps the SDK from retrying into the block. If
you can't read your counter and choose to fail closed, send the 429 with
`x-should-retry: false`,
`anthropic-ratelimit-unified-overage-disabled-reason: fetch_error`, and a
message, nothing else. This gateway sends exactly the shapes above for caps
set through its admin API (`overage-period` is `daily`, `weekly`, or
`monthly`); when several caps apply it describes the fullest one, or once
blocked the one that resets last.

## Bearer token

Your `access_token` is opaque to the client — it stores it, sends it, and
refreshes it before `expires_in`, but never inspects the payload. Encode the
user's identity and groups in the token (or in server-side state keyed by it)
so you can apply per-user RBAC at `/v1/messages` and per-group policy at
`/managed/settings`. The same token must work across every
bearer-authenticated endpoint.

## TLS

`https://` is required; `http://` is accepted only for loopback during
development. The client pins the SHA-256 fingerprint of your TLS leaf
certificate per-hostname after the user confirms it on first connect, and
re-prompts on mismatch — rotating your certificate costs every user one
confirmation prompt.

## Client guarantees

- OAuth endpoint paths come from your discovery document; the client never
  hard-codes `/oauth/token`.
- Fixed-path endpoints are resolved against `{base}`, never a redirect.
- Every request body carries `Content-Length`.
- The OTLP exporter is locked to `{base}/v1/{signal}` regardless of the
  user's environment; only your own `/managed/settings` document can name a
  different collector, and the gateway bearer is never sent to it.
- `404` from `/v1/models` or `/managed/settings` is a clean "not
  implemented", with no retry storm.

## Proxying to Bedrock, Vertex, or Foundry

Proxying to `api.anthropic.com` is pass-through. Proxying to a cloud
provider's Claude endpoint needs translation:

- **Model IDs.** The client sends Anthropic-style IDs like
  `claude-sonnet-4-5`; translate to the upstream's form (Bedrock model ID or
  inference-profile ARN; Vertex `@`-versioned ID), or advertise
  upstream-native IDs from `/v1/models`.
- **`anthropic-beta`.** Bedrock rejects some betas in the *header*; move them
  into the request body as `"anthropic_beta": [...]`. Vertex and Foundry
  accept the header.
- **Streaming.** Bedrock's native stream is AWS binary event-stream, not SSE;
  decode and re-emit Anthropic-shaped `text/event-stream`. The provider SDKs
  handle this, but their stream iterators drop the upstream's `ping` events
  (and Bedrock sends none) — emit your own `event: ping` during silent gaps
  so long thinking pauses don't trip client or proxy idle timeouts.
- **`count_tokens`.** Bedrock has no count-tokens API. Return
  `501 not_supported`; the client counts with a one-token request (the
  session's model unless `ANTHROPIC_SMALL_FAST_MODEL` or
  `ANTHROPIC_DEFAULT_HAIKU_MODEL` is set).
- **Headers.** Forward `content-type`, `accept`, `accept-encoding`,
  `anthropic-version`, `anthropic-beta`, `user-agent`, and `x-stainless-*`;
  strip the client's `Authorization` and apply the upstream's own
  credentials. On the response, strip hop-by-hop headers
  (`content-encoding`, `content-length`, `transfer-encoding`, `connection`).
- **Errors.** Upstream error messages can carry your cloud account
  IDs/ARNs/project IDs — log them for the operator and return a generic
  message, keeping `error.type`. The exception is a 400/413 in Anthropic's
  own error envelope (e.g. `prompt is too long: …`): relay that
  `error.message`, the client's recovery (auto-compact etc.) keys on it.

## References

RFC 6749 (OAuth 2.0), RFC 8414 (AS metadata), RFC 8628 (device grant),
Anthropic Messages API, Claude Code settings reference, OTLP spec.
