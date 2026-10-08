---
name: plan:{program-slug}-umbrella
description: "{Program name} — umbrella/orchestration plan for the {N}-phase program"
date: {dd-mm-yy}
metadata:
  node_type: memory
  type: plan
  feature: {feature-name}
  phase: umbrella
---

# {Program Name} — Umbrella Plan

**Date:** {dd-mm-yy}
**Complexity:** COMPLEX
**Status:** ⏳ PLANNED

- Program type: PHASE PROGRAM ({N} phases, sequential with gated joins)
- Date: {dd-mm-yy}
- Feature folder: `process/features/{feature}/`

---

## Program Goal Charter

```
{Program Name} — Program Goal Charter

North star:
- {one sentence stating the real end goal}

Definition of done (an unattended agent must be able to do all of these):
1. {concrete capability 1}
2. {concrete capability 2}
3. {concrete capability 3}

What "verified" means (program level):
- {exact bar for promoting work to VERIFIED — gate surface, evidence, coverage}
- validate-contract gates must be recorded alongside phase gates and regression evidence for a
  phase to reach VERIFIED. A phase without a validate-contract (or documented skip reason)
  cannot be marked VERIFIED.

Scope tiers → phase mapping:
- Tier 1 {name} → Phase {N}
- Tier 2 {name} → Phase {N}
- Tier 3 {name} → Phase {N}
- This program retires Tiers {1-N}.

Explicitly out of scope (deferred tier):
- {items intentionally not addressed by this program}

Hard safety constraints (non-negotiable, per phase):
- {program-specific irreversible/destructive boundaries}
- Commit each phase's execution changes before starting the next phase.
  Keep process/plan/context commits separate from execution commits.
```

---

## Stable Program Goal (copy-paste this to start autonomous execution)

```
SESSION GOAL: {feature} — {Program Name}
Ref: process/features/{feature}/active/{program-slug}-umbrella_{dd-mm-yy}/{program-slug}-umbrella_PLAN_{dd-mm-yy}.md

TARGET: Complete ALL phases until:
- {exit condition 1 — e.g. all validators exit 0}
- {exit condition 2 — e.g. all phase exit gates green}
- Test tiers: automated (iterate-until-green) / hybrid (fix-if-in-blast-radius) / agent-probe (record-judgment)

AUTONOMY: Before ANY subagent spawn, read:
1. Umbrella ## Current Execution State → loop step + validate-contract status
2. Phase plan ## Phase Loop Progress → first unchecked box = next subagent to spawn

PER-PHASE LOOP (7-step inner loop `R → I → P → PVL → E → EVL → UP`, never skip, never reorder; SKIPS SPEC — SPEC runs once in the outer program loop):
  1. RESEARCH → 2. INNOVATE → 3. PLAN-SUPPLEMENT → 4. PVL → 5. EXECUTE → 6. EVL → 7. UPDATE-PROCESS
- PLAN-SUPPLEMENT: plan-agent writes research/innovate gaps into phase plan (or marks "n/a — clean")
- PVL NEVER skipped; contract must follow example-validate-output.md full format;
  partial contract (missing Plan updates applied / Execute-agent instructions / Test gates) =
  blocked same as placeholder
- Every subagent FIRST ACTION: run vc-context-discovery (load context group files +
  process/context/tests/all-tests.md routing chain) AND vc-plan-discovery (same-feature full
  depth active/backlog/completed/reports/refs + other features active-only + general-plans active)
- Every phase-END: invoke vc-agent-strategy-compare for next step strategy recommendation

Report via phase reports. No approval between phases unless hard stop hit.

HARD STOPS (pause, wait for user):
- Irreversible/outward-facing action without explicit validate-contract instruction
- Net gate = BLOCKED with no backlog resolution path
- Plan file marks "pause required" or agent count > 100
- Validate-contract is placeholder and vc-validate-agent cannot run

SAFETY (never override):
- {program-specific safety rule 1}
- {program-specific safety rule 2}
- Commit each phase before advancing; process and execution commits separate

TEST GATES (every phase exit):
  {test gate command 1}
  {test gate command 2}

VALIDATE CONTRACT: Per-phase contracts written by vc-validate-agent into each phase plan before EXECUTE.

START: Phase {N}, loop step RESEARCH (pending). Spawn vc-research-agent for Phase {N}.
```

---

## Phase Sequence

| Phase | Plan file | Scope summary | Depends on |
|---|---|---|---|
| 0 (pre-program) | this file | Confirm folder structure, baseline audit, create sub-phase plans | — |
| 1 — {Name} | `process/features/{feature}/active/{program-slug}_{dd-mm-yy}/phase-01-{slug}_PLAN_{dd-mm-yy}.md` | {scope summary} | Phase 0 |
| 2 — {Name} | `process/features/{feature}/active/{program-slug}_{dd-mm-yy}/phase-02-{slug}_PLAN_{dd-mm-yy}.md` | {scope summary} | Phase 1 |
| 3 — {Name} | `process/features/{feature}/active/{program-slug}_{dd-mm-yy}/phase-03-{slug}_PLAN_{dd-mm-yy}.md` | {scope summary} | Phase 1 + Phase 2 |

### Join Conditions

- Phase 1 MUST NOT start until Phase 0 exit gate passes.
- Phase 2 MUST NOT start until Phase 1 exit gate passes.
- Phase 3 MUST NOT start until Phase 1 AND Phase 2 exit gates both pass.

---

## Per-Phase Entry / Exit Gates

| Phase | Entry | Exit gate |
|---|---|---|
| 0 | Program start | Phase plan files created; baseline validators recorded |
| 1 | Phase 0 complete | {exit condition for phase 1} |
| 2 | Phase 1 exit met | {exit condition for phase 2} |
| 3 | Phases 1+2 exits met | {exit condition for phase 3} |

---

## Per-Phase Loop

Each phase executes the canonical 7-step inner loop `R → I → P → PVL → E → EVL → UP`. This inner
loop SKIPS SPEC — SPEC runs once in the outer program loop, not per phase. The 7 steps map to:

1. **RESEARCH** — spawn research-agent: load context, read prior phase reports, check plan drift, document findings
2. **INNOVATE** — spawn innovate-agent: decide approach; write Decision Summary (chosen approach + rejected alternatives)
3. **PLAN-SUPPLEMENT** — spawn plan-agent: if research/innovate found gaps/pre-conditions not in checklist, add them; otherwise mark "n/a — research clean" and tick step 3
4. **PVL** — spawn vc-validate-agent: full V1-V7; validate-contract written per `.claude/skills/vc-validate-findings/references/example-validate-output.md` format (Status / Gate / Plan updates applied / Execute-agent instructions / Test gates / High-risk pack / Backlog artifacts / Known gaps / Accepted by)
5. **EXECUTE** — spawn vc-execute-agent per approved plan and validate-contract
6. **EVL** — spawn vc-tester: run phase test gates to green; register follow-up stubs; write EVL HANDOFF SUMMARY
7. **UPDATE-PROCESS** — write phase report to durable report path, rewrite umbrella `## Current Execution State` section (overwrite, not append — git history is the audit log)

**PVL is NEVER skipped.** A placeholder `## Validate Contract` = blocked. Do not spawn execute-agent while the Validate Contract section reads "(placeholder — vc-validate-agent writes this section before EXECUTE)".

---

## Autonomous Execution Rules (During /goal)

During /goal execution of a phase program:
- Agent self-decides at all V5 gates — no user approval needed between phases
- CONDITIONAL net gate: proceed autonomously, fixes applied in-flight, gaps on record
- BLOCKED net gate: document items in backlog, continue with remaining phase plans; backlog is always a valid resolution — always find a path forward
- Hard stops (must pause for user approval):
  - Irreversible/outward-facing action without explicit contract instruction (push to remote, deploy to production, schema migration on live DB)
  - Plan file explicitly marks "pause required" at a step
- Agent writes phase reports, updates phase plans, creates new sub-plans as needed — all autonomously
- The phase report is the communication channel for conflicts, errors, and learnings — not inline questions

---

## Global Constraints

- {non-negotiable constraint 1 — e.g. never lower validator checks}
- {non-negotiable constraint 2 — e.g. never widen an allowlist without user approval}
- After every phase that touches agent files, run parity validator and confirm it exits 0 before declaring phase DONE.
- All new skill SKILL.md files must include YAML frontmatter (name, description, argument-hint, metadata.author, metadata.version).
- Commit each phase's execution changes before starting the next phase. Keep process/plan/context commits separate from execution commits.

---

## Durable Report Destinations

| Phase | Report path (inside task folder) |
|---|---|
| 0 (pre-program) | `process/features/{feature}/active/{program-slug}-umbrella_{dd-mm-yy}/phase-00-{slug}_REPORT_{dd-mm-yy}.md` |
| 1 — {Name} | `process/features/{feature}/active/{program-slug}_{dd-mm-yy}/phase-01-{slug}_REPORT_{dd-mm-yy}.md` |
| 2 — {Name} | `process/features/{feature}/active/{program-slug}_{dd-mm-yy}/phase-02-{slug}_REPORT_{dd-mm-yy}.md` |
| 3 — {Name} | `process/features/{feature}/active/{program-slug}_{dd-mm-yy}/phase-03-{slug}_REPORT_{dd-mm-yy}.md` |

---

## Program Status Table

| Phase | Status |
|---|---|
| 0 — Pre-program (plan creation) | ⏳ PLANNED |
| 01 — {Name} | ⏳ PLANNED |
| 02 — {Name} | ⏳ PLANNED |
| 03 — {Name} | ⏳ PLANNED |

Status values: ⏳ PLANNED | 🔨 CODE DONE | 🧪 TESTING | ✅ VERIFIED | 🚧 BLOCKED | ✅ COMPLETE

---

## Touchpoints

- {file or folder modified or created in phase 1}
- {file or folder modified or created in phase 2}
- {file or folder modified or created in phase 3}

---

## Public Contracts

- {unchanged behavior 1 — e.g. existing CLI interface unchanged}
- {unchanged behavior 2 — e.g. external API surface unchanged}

---

## Blast Radius

Files directly modified or created:

- {exact file paths modified or created, one per line}
- {group of files, e.g. "9 new files under .claude/skills/*/SKILL.md"}

---

## Verification Evidence

```bash
# {verification command 1 — e.g. validator exit 0 check}
{command}
# Expected: {expected output}

# {verification command 2}
{command}
# Expected: {expected output}
```

---

## Resume and Execution Handoff

- Selected plan file path: `process/features/{feature}/active/{umbrella-plan-filename}.md`
- Last completed phase: Phase 0 (this umbrella plan file = Phase 0 artifact)
- Validate-contract status: pending (vc-validate-agent writes per-phase)
- Next step for a fresh agent: Read this umbrella plan, read the Phase 1 plan, then run Phase 1 research subagent before any EXECUTE work.
- Current phase: {phase name}
- Next action: {e.g. "Spawn vc-research-agent for Phase 1"}
- Execute-agent start instruction: Read this file. Read Phase 1 plan. Run research subagent first.

---

## Current Execution State

Last updated: {dd-mm-yy}
Completed phases: {list, e.g. "Phase 0 (Planning)"}
Current phase: {phase name or "NONE — program complete"}
Current loop step: {RESEARCH | INNOVATE | PLAN-SUPPLEMENT | PVL | EXECUTE | EVL | UPDATE-PROCESS | DONE}
Validate-contract status: {pending / written / all phases validated}
Program Net Gate: {PASS | FAIL | PENDING}
Latest validator run: {date} — {results, e.g. "0 failures / 0 warnings"}

Loop step values: RESEARCH | INNOVATE | PLAN-SUPPLEMENT | PVL | EXECUTE | EVL | UPDATE-PROCESS
Orchestrator rule: read "Current loop step" and "validate-contract status" before spawning any subagent. Never spawn execute-agent when loop step is RESEARCH, INNOVATE, PLAN-SUPPLEMENT, or PVL.

Note: The Stable Program Goal above is fixed. This section is the only part that changes — update-process-agent rewrites it after every phase closeout (overwrite, not append — git history is the audit log).

---

## Validate Contract

(placeholder — vc-validate-agent writes this section before EXECUTE)
