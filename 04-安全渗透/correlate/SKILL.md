---
name: correlate
description: "Run the finding correlation engine to discover attack chains from individual findings."
disable-model-invocation: false
---
Find attack chains by correlating all known findings.

1. `uv run python3 $CLAUDE_PROJECT_DIR/tools/statusline.py --compact` — show current state
2. Launch `correlator` agent: "Analyze ALL findings in brain and findings.json. Find attack chains where one finding enables another. Document each chain with combined CVSS 4.0 and end-to-end reproduction steps."
3. After agent returns, update brain with discovered chains.
4. Show the user any new high-impact chains found.

## Top-Tier Correlation Bar

Build a capability graph, not a list of related bugs.

- Nodes are capabilities: read tenant data, write config, trigger webhook, steal token, reach internal host, execute workflow.
- Edges require evidence that one capability enables the next. Shared component or same endpoint family is only a hint.
- Score each chain by final impact, proof reliability, policy safety, duplicate risk, and report clarity.
- Prefer chains that convert low-severity feeders into account takeover, tenant escape, stored XSS with privileged action, SSRF to credential disclosure, or config write to execution.
- Record killed chains too. The next correlation pass should know which attractive edges failed and why.
