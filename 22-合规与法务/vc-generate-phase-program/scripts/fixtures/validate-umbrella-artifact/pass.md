---
node_type: memory
type: plan
phase: umbrella
---

# Test Umbrella (fixture — PASS)

A flat-named umbrella with every REQUIRED structural section present and one
RECOMMENDED section ("Durable Report Destinations") deliberately omitted, so the
validator exits 0 with a NON-zero warnings count.

## Program Goal Charter

Deliver the fixture program.

## Stable Program Goal

Short stable /goal block (well under 4000 chars).

## Current Execution State

Phase 1 complete; Phase 2 pending.

## Phase Ordering

| Phase | Plan | Depends on |
|---|---|---|
| 1 — Foundation | phase-01-foundation | — |
| 2 — Expansion | phase-02-expansion | Phase 1 |

### Join Conditions

- Phase 2 starts after Phase 1 is green.

## Program Status Table

| Phase | Status |
|---|---|
| 1 | ✅ COMPLETE |
| 2 | ⏳ PLANNED |

## Per-Phase Loop

Standard 7-step inner loop (R → I → P → PVL → E → EVL → UP).

## Global Constraints

- `git mv` only; never `rm`.

## Validate Contract

Gate: PASS (fixture).
