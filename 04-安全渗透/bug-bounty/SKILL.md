---
name: bug-bounty
description: >-
  Bug bounty workflow using Deep Eye for recon/scan and high-signal report
  writing (HackerOne/Bugcrowd). Use for bug bounty, VDP, HackerOne, Bugcrowd,
  bounty report, /bug-bounty. Only in-scope program assets.
---

# Deep Eye — Bug Bounty Skill

Policy first. Deep Eye accelerates surface coverage; **impact + PoC** win bounties.

## Preconditions

1. Program policy read (scope, OOS, rate limits, safe harbor).
2. In-scope only — no third-party collateral.
3. Local config; never commit API keys or session cookies.

## ROI module pack

Enable in `vulnerability_scanner.enabled_checks`:

```yaml
enabled_checks:
  - idor
  - api_bola_deep
  - jwt_deep
  - oauth_testing
  - graphql_deep
  - ssrf_cloud
  - cloud_misconfig
  - cors_csp
  - open_redirect_deep
  - stored_xss
  - sql_injection
  - xss
  - ssrf
  - mass_assignment
```

Optional: `ai_triage.enabled`, `bug_bounty.enabled` (Markdown under `reports/bounty/`).

## Commands

```bash
python deep_eye.py --setup
python deep_eye.py -u https://IN_SCOPE -v --formats json,html
python deep_eye.py -u https://IN_SCOPE --scope-nl "only /api/* host target.com"
python deep_eye.py -u https://IN_SCOPE --retest-new reports/prior.json
```

OpenAPI: `openapi.enabled: true` + `source`.

## Hunt order

1. Authz — `idor`, `api_bola_deep`
2. Token/auth — `jwt_deep`, `oauth_testing`, `login_replay`
3. SSRF/cloud — `ssrf_cloud`, `cloud_misconfig`
4. GraphQL — `graphql_deep`
5. Stored XSS chains — `stored_xss`
6. Secrets — only if actionable (`secret_scanning`)

## Report template

```markdown
## Summary
## Steps to reproduce
## PoC
## Impact
## Remediation
## Environment
```

Finding keys: `type`, `severity`, `url`, `parameter`, `payload`, `evidence`, `remediation`.

## Rules

Respect rate limits; redact PII; check duplicates; show delta impact on partial dupes.
