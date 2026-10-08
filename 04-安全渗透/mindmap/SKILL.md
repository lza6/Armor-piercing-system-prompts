---
name: mindmap
description: "Generate a text-based attack surface mindmap. Shows tech stack → vuln class → endpoint relationships. Usage: /mindmap <target>"
disable-model-invocation: false
---
Generate attack surface mindmap for: $ARGUMENTS

## Process
1. Read brain data: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py brief $ARGUMENTS`
2. Read recon data from recon/ directory
3. Read intel data: `uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py suggest <tech-stack>`
4. Generate a tree-format mindmap:

```
target.com
├── Tech Stack
│   ├── Next.js 14 → SSRF (Server Actions), Open Redirect
│   ├── GraphQL → Introspection, IDOR via node(), Mutation Auth
│   └── PostgreSQL → SQL Injection
├── Auth
│   ├── Okta SSO → SAML bypass, OAuth redirect_uri
│   └── JWT → Secret brute-force, Algorithm confusion
├── API Surface
│   ├── /api/v2/users/{id}/* → IDOR (P1)
│   │   ├── /orders — TESTED: exhausted
│   │   ├── /export — UNTESTED
│   │   └── /settings — UNTESTED
│   ├── /api/v2/payments/* → Race conditions, price manipulation (P1)
│   └── /graphql → Auth bypass on mutations (P1)
├── File Handling
│   └── /upload → Extension bypass, SVG XSS (P2)
└── Findings
    ├── [CONFIRMED] IDOR on /api/v2/users/{id}/orders
    └── [EXHAUSTED] XSS on /search — CloudFront blocks all payloads
```

5. Mark each endpoint as TESTED, UNTESTED, CONFIRMED, or EXHAUSTED from brain data
6. Suggest: "Start with UNTESTED P1 endpoints. Run /hunt $ARGUMENTS --vuln-class <suggested>"

## Top-Tier Mindmap Standard

The mindmap should expose attack decisions at a glance.

- Group by trust boundary first: unauth, user, tenant, admin, integration, internal, CI/CD, AI/tool.
- Mark every node with one of: `P1`, `P2`, `Kill`, `Confirmed`, `Partial`, `Exhausted`, `Chain`.
- Draw capability edges, not just URL hierarchy: export reads data, webhook sends server-side request, template renders attacker input, OAuth callback grants token.
- Surface blind spots explicitly: "no second-account test", "no browser verification", "no sibling replay", "no chain attempt".
- End with the top three routes where one more test could change severity or reportability.
