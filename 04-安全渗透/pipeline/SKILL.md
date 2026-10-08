---
name: pipeline
description: "Prepare the battlefield — recon, scanning, and surface ranking. Stops before hunting. Run /hunt or /autopilot after. Usage: /pipeline or /pipeline <target>"
disable-model-invocation: false
---
Prepare the battlefield for: $ARGUMENTS

This command runs recon, scanning, and surface ranking — everything needed BEFORE hunting.
It does NOT hunt, validate, or report. Use `/hunt` or `/autopilot` for that.

## Phase 0: SETUP

1. Read `scope.yaml` — resolve and verify targets
   - If `$ARGUMENTS` is empty: `uv run python3 $CLAUDE_PROJECT_DIR/tools/scope_check.py --list`
   - If `$ARGUMENTS` is a domain: `uv run python3 $CLAUDE_PROJECT_DIR/tools/scope_check.py $ARGUMENTS`
2. Read `policy.md` — extract policy preamble for all agent dispatches
3. Brain init or brief:
   - If no brain exists: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py init`
   - If brain exists: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py brief <target>`

## Phase 1: RECON

4. Dispatch `recon` agent (model: inherit) with policy preamble and scope
5. After recon: dispatch `config-auditor` agent (model: inherit) for header/TLS/cookie review
6. After config: dispatch `js-analyzer` agent (model: inherit) for JavaScript analysis
7. Brain update: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> recon "<results summary>"`

## Phase 2: SCANNING (parallel, max 3)

8. Dispatch in parallel (all model: inherit, all with policy preamble):
   - `vuln-scanner` agent with nuclei on discovered hosts
   - `waf-profiler` agent on primary targets
9. Brain update with scan results

## Phase 3: RANK

10. Dispatch `recon-ranker` agent (model: inherit) with recon data + brain knowledge
11. Output P1/P2/Kill list

## Complete

```
Battlefield ready.

P1 targets: [list]
P2 targets: [list]
Kill list:  [list]

Next steps:
  /hunt <target>     — manual hunting on a specific target
  /autopilot         — autonomous hunting across all P1 targets
  /surface           — re-rank surface with current brain knowledge
```

Sync brain: `uv run python3 $CLAUDE_PROJECT_DIR/tools/global_brain.py sync-from-local`

## Top-Tier Pipeline Standard

The pipeline prepares a battlefield, not a folder of scan files.

1. Scope first: every generated target must be in-scope or tagged `out-of-scope` with reason.
2. Normalize assets into stable inventories: hosts, endpoints, JS files, APIs, auth flows, cloud buckets, repos, mobile packages, and third-party integrations.
3. Rank during collection. Do not wait until the end to identify crown jewels.
4. Preserve raw evidence and parsed summaries. A hunter should be able to replay the exact source of every target.
5. End with `P1`, `P2`, and `Kill` lists plus the best first vuln class for each P1. If no P1 exists, say why and recommend monitoring or a different program.
