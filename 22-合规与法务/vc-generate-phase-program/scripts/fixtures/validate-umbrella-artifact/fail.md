---
node_type: memory
type: plan
phase: umbrella
---

# Test Umbrella (fixture — FAIL: missing '## Program Status Table')

This umbrella is missing the REQUIRED `## Program Status Table` section, so the
validator must exit 1.

## Program Goal Charter

Deliver the fixture program.

## Stable Program Goal

Short stable /goal block.

## Current Execution State

Phase 1 pending.

## Phase Ordering

| Phase | Plan | Depends on |
|---|---|---|
| 1 — Foundation | phase-01-foundation | — |

### Join Conditions

- none

## Validate Contract

Gate: PASS (fixture) — but the missing Program Status Table must still fail validation.
