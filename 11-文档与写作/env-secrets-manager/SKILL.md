---
name: env-secrets-manager
description: >
  Use this skill when the user wants to manage environment variables, secrets,
  API keys, .env files, deployment configuration, secret rotation, credential
  hygiene, local development setup, or secret leakage prevention. Trigger phrases
  include ".env," "environment variables," "secrets manager," "API key safety,"
  "rotate secrets," "deployment env," "credential leak," and "secret hygiene."
---

# Environment Secrets Manager

## Mandatory Content Standards

- Write in active voice.
- Use short sentences and plain language.
- Address the user as "you" and "your."
- Use `.marketing-os/product-context.md` when it exists. If it is blank or missing, continue because secret hygiene can run without marketing context.
- Never print, copy, transform, or expose real secrets in the response.
- Redact secrets as `***REDACTED***`.
- Do not suggest committing `.env` files that contain credentials.
- Prefer least privilege, rotation, environment separation, and documented ownership.
- Do not use em dashes, hashtags, emojis, or filler closings.
- End every full deliverable with one specific next step.

## Objective

Use this skill to keep credentials, API keys, tokens, webhook secrets, deployment variables, and local `.env` files organized and safe.

This skill supports marketing automation projects because they often connect CRMs, email tools, ad platforms, analytics, webhooks, and CLIs.

Next step: inventory required environment variables without revealing values.

## Secret Inventory

| Variable | Service | Environment | Required | Scope | Owner | Rotation Cadence | Notes |
|----------|---------|-------------|----------|-------|-------|------------------|-------|

Classify each variable.

| Type | Examples | Handling |
|------|----------|----------|
| Public config | Public API base URL, analytics public key | Can appear in client when intended |
| Server secret | API key, private token, webhook secret | Server only |
| Build secret | Package token, deploy token | CI or deploy platform only |
| Local secret | Developer sandbox key | Local `.env`, never committed |
| Rotating credential | OAuth refresh token, signing secret | Store in managed secret system |

Next step: separate public config from true secrets.

## File Strategy

Recommended files:

```text
.env.example
.env.local
.env.development
.env.test
.env.production
```

Rules:

- Commit `.env.example` with variable names and safe placeholder values.
- Do not commit real `.env` files.
- Add `.env*` to `.gitignore`, except `.env.example` if the project needs it.
- Keep production secrets in the deployment platform or managed secrets store.
- Use different keys for development, staging, and production.

Next step: create or update `.env.example` without real values.

## Rotation Workflow

1. Identify affected service.
2. Revoke or rotate the exposed credential.
3. Update local, CI, staging, and production secret stores.
4. Restart dependent services.
5. Verify functionality.
6. Search recent commits and logs for exposed values.
7. Document the incident and prevention step.

Do not paste the exposed secret into tickets, chat, commits, or documentation.

Next step: rotate the highest-risk exposed key first.

## Common Marketing Stack Variables

| Service Type | Example Variables |
|--------------|-------------------|
| CRM | `HUBSPOT_ACCESS_TOKEN`, `SALESFORCE_CLIENT_SECRET` |
| Email | `MAILCHIMP_API_KEY`, `SENDGRID_API_KEY` |
| Analytics | `GA4_PROPERTY_ID`, `POSTHOG_PROJECT_API_KEY` |
| Ads | `META_ACCESS_TOKEN`, `GOOGLE_ADS_DEVELOPER_TOKEN` |
| Webhooks | `WEBHOOK_SIGNING_SECRET`, `STRIPE_WEBHOOK_SECRET` |
| GitHub | `GITHUB_TOKEN` |
| AI | `OPENAI_API_KEY`, `ANTHROPIC_API_KEY` |

Next step: map each variable to the tool or CLI that needs it.

## Output Format

| Priority | Secret Area | Risk | Fix | Owner | Verification |
|----------|-------------|------|-----|-------|--------------|

Then provide current risk summary, `.env.example` plan, rotation plan, deployment checklist, CI checklist, and local developer setup instructions.

End with the safest next action to reduce credential risk.

