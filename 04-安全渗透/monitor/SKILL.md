---
name: monitor
description: "Monitor targets for changes. Usage: /monitor baseline (first run), /monitor check (detect changes), /monitor scope (check platform for scope updates)"
disable-model-invocation: false
---
Run the monitoring agent: $ARGUMENTS

Parse the mode from arguments:
- If empty or "baseline": Run monitor agent with prompt "Baseline mode. Capture the current state of all in-scope targets. Use recon/ data if available for a richer baseline."
- If "check": Run monitor agent with prompt "Check mode. Compare current state against baselines in monitor/. Report all changes and recommend which agents to re-run."
- If "scope": Run monitor agent with prompt "Scope mode. Use MCP get_program_scope to check if the platform has updated the program scope. Diff against local scope.yaml and report new assets."

After the agent completes:
1. If changes were found, show a summary and suggest next actions
2. If scope changes were found, ask if the user wants to run /pipeline on new assets
3. Log to brain: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py log "Monitor {mode}: {summary}"`

## Top-Tier Monitoring Bar

Treat changes as vulnerability triggers.

- New host: run scope check, tech fingerprint, and surface ranking.
- New endpoint: diff auth requirement, methods, content types, and response schemas against siblings.
- New JS bundle: extract routes, API clients, feature flags, source maps, and changed sinks.
- New policy/scope text: update safe-harbor headers, prohibited actions, asset tiers, payout exclusions, and credential rules.
- Fixed finding: verify remediation without bypass attempts unless authorized, then record the patch pattern for future dupcheck.

Prioritize changes touching auth, billing, admin, integrations, uploads, exports, webhooks, OAuth, AI/tool execution, and tenant boundaries.
