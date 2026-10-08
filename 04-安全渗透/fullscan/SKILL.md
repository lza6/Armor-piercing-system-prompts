---
name: fullscan
description: "Full security assessment with brain coordination. Multi-phase, skips known-exhausted areas, builds on prior knowledge."
disable-model-invocation: false
---

ALL agents dispatched by this command MUST use `model: "inherit"` in the Agent tool call.

Full security assessment on: $ARGUMENTS

## Phase 0: Brain Briefing
1. `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py init` (if first run)
2. `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py brief $ARGUMENTS`
3. `uv run python3 $CLAUDE_PROJECT_DIR/tools/scope_check.py $ARGUMENTS`

## Phase 1: Recon (skip already-discovered assets)
Launch `recon` agent with brain context. Focus on discovering NEW subdomains and services beyond what's already known.

## Phase 2: Scanning (skip exhausted areas)
Launch IN PARALLEL, each with brain context about what's been scanned before:
- `vuln-scanner` — skip known false positives, focus new hosts
- `config-auditor` — check if previously noted misconfigs are fixed
- `js-analyzer` — focus on new/changed JS files

Record all results to the brain after each agent returns.

## Phase 3: Targeted Testing (brain-guided)
Based on Phase 2 + brain knowledge, selectively launch ONLY agents targeting UNTESTED or ACTIVE vectors:
- `xss-hunter` ONLY on endpoints not marked exhausted
- `api-audit` ONLY on newly discovered or untested endpoints  
- `auth-tester` ONLY if new auth flows found

Record all results to the brain.

## Phase 4: Reporting
1. `uv run python3 $CLAUDE_PROJECT_DIR/tools/dedup_findings.py --scan-dir scans/ --db findings.json --stats`
2. `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py status`
3. Launch `poc-builder` for each confirmed finding
4. Launch `report-writer` with full brain context
5. `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py log "Full scan completed on $ARGUMENTS"`

## Top-Tier Fullscan Standard

A full scan must leave a defensible coverage ledger.

1. Start with the crown jewels: auth, tenant data, billing, admin, integrations, uploads, webhooks, exports, AI/tool surfaces, and source/repo exposure.
2. For each phase, record both positives and negatives. "Scanned" is invalid unless it names tools, targets, timestamps, and skipped areas.
3. Do not let scanner output dominate. Use scanners to find leads, then spend human-grade effort on state changes, authorization boundaries, and parser differentials.
4. Before reporting, run `/correlate` or `/chain` on all confirmed and partial findings. Single low bugs should not escape the scan if a plausible chain remains.
5. Final output must contain: confirmed findings, killed false positives, exhausted high-value paths, untested residual risk, and the next best command.
