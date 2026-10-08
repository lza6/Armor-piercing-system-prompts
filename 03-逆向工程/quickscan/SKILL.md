---
name: quickscan
description: "Run a quick security scan on a target. Consults the Brain first, validates scope, runs passive recon + vuln scan in parallel."
disable-model-invocation: false
---

ALL agents dispatched by this command MUST use `model: "inherit"` in the Agent tool call.

Run a quick security assessment on: $ARGUMENTS

Workflow:
1. **Brain**: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py brief $ARGUMENTS` — check what we already know. Note exhausted areas.
2. **Scope**: `uv run python3 $CLAUDE_PROJECT_DIR/tools/scope_check.py $ARGUMENTS` — if out of scope, STOP.
3. Launch IN PARALLEL (skip areas the brain marks EXHAUSTED):
   - `recon` agent with passive-only depth, passing brain context about known subdomains/tech
   - `config-auditor` agent for headers, CSP, CORS, TLS, cookies
4. Record results: for each new finding, run `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> <status> <technique> <details>`
5. Log session: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py log "quickscan completed on $ARGUMENTS"`
6. Summarize: separate NEW findings from KNOWN, recommend next steps.

## Top-Tier Quickscan Loop

Quickscan should answer "is there obvious money or obvious risk here in 30 minutes?"

1. Spend the first five minutes on scope, policy headers, brain, and live host sanity.
2. Spend the next ten on high-signal passive recon: JS routes, exposed APIs, auth flows, cloud/storage names, source maps, security headers, and known vendor panels.
3. Spend ten on two targeted probes only: the best config/information leak candidate and the best auth/tenant-boundary candidate.
4. Spend five on triage: new, known, killed, or needs full hunt.

Never report from quickscan alone unless the proof is already complete. Promote strong leads to `/hunt`, `/validate`, or `/chain`.
