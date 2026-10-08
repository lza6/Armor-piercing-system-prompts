---
name: plan:{program-slug}-phase-NN-{slug}
description: "{Program name} — Phase NN: {phase title}"
date: {dd-mm-yy}
metadata:
  node_type: memory
  type: plan
  feature: {feature-name}
  phase: phase-NN
---

# Phase NN — {Phase Title}

**Program:** {program-slug}
**Umbrella plan:** process/features/{feature}/active/{program-slug}-umbrella_{dd-mm-yy}/{program-slug}-umbrella_PLAN_{dd-mm-yy}.md
**Phase status:** ⏳ PLANNED
**Report destination:** process/features/{feature}/active/{program-slug}_{dd-mm-yy}/phase-NN-{slug}_REPORT_{dd-mm-yy}.md (flat in the program task folder)

---

## Purpose

{One paragraph describing what this phase achieves and why it exists in the program sequence.}

---

## Entry Gate

- Phase NN-1 complete (all checklist items done, validators green)
- {any additional prerequisite for this specific phase}

---

## Blast Radius

- {file or folder 1 — modified or created in this phase}
- {file or folder 2}
- {file or folder 3}

---

## Implementation Checklist

### Step A — {Step name}

- [ ] A1. {atomic action 1}
- [ ] A2. {atomic action 2}
- [ ] A3. {atomic action 3}

### Step B — {Step name}

- [ ] B1. {atomic action 1}
- [ ] B2. {atomic action 2}

### Step C — {Step name}

- [ ] C1. {atomic action 1}
- [ ] C2. {atomic action 2}

---

## Exit Gate

```bash
# {verification command 1}
{command}
# Expected: {expected output}

# {verification command 2}
{command}
# Expected: {expected output}
```

- {done-means check 1 — e.g. "all checklist items checked"}
- {done-means check 2 — e.g. "validators exit 0"}
- Phase report written to report destination above

---

## Blockers That Would Justify BLOCKED Status

- {blocker 1 — e.g. "upstream phase exit gate not yet passed"}
- {blocker 2 — e.g. "required dependency not available"}
- {blocker 3 — e.g. "validate-contract cannot be written due to missing prerequisite"}

---

## Phase Loop Progress

Orchestrator reads this before deciding which subagent to spawn next. The canonical 7-step inner loop
`R → I → P → PVL → E → EVL → UP` SKIPS SPEC (SPEC runs once in the outer program loop).

- [ ] 1. RESEARCH — research-agent: prior phase reports read; test context loaded; plan drift checked
- [ ] 2. INNOVATE — innovate-agent: approach decided; Decision Summary written
- [ ] 3. PLAN-SUPPLEMENT — plan-agent: existing phase plan updated; Inner Loop Refresh Note if sections changed (or "n/a — research clean")
- [ ] 4. PVL — vc-validate-agent: full V1-V7; validate-contract written per `.claude/skills/vc-validate-findings/references/example-validate-output.md` (Status / Gate / Plan updates applied / Execute-agent instructions / Test gates / High-risk pack / Backlog artifacts / Known gaps / Accepted by)
- [ ] 5. EXECUTE — all checklist items done; per-section test gates run and green (or gaps documented)
- [ ] 6. EVL — all EVL gates green; follow-up stubs registered; EVL HANDOFF SUMMARY written
- [ ] 7. UPDATE PROCESS — phase report written, umbrella state updated, commit done

**Validate-contract required before execute.** If step 4 (PVL) is unchecked or `## Validate Contract`
reads "(placeholder — vc-validate-agent writes this section before EXECUTE)", orchestrator must
spawn vc-validate-agent first. A partial contract missing Plan updates applied / Execute-agent
instructions / Test gates sections is treated as a placeholder.

---

## Touchpoints

- {file or folder modified or created — list each one}

---

## Public Contracts

- {unchanged behavior 1 — e.g. existing CLI interface unchanged}
- {unchanged behavior 2 — or "none" if no external contract}

---

## Verification Evidence

```bash
# {verification command — run after phase complete}
{command}
# Expected: {expected output}
```

---

## Resume and Execution Handoff

- Selected plan file path: `process/features/{feature}/active/{program-slug}_{dd-mm-yy}/phase-NN-{slug}_PLAN_{dd-mm-yy}.md`
- Last completed step: {step name or "not started"}
- Validate-contract status: pending
- Next step: {e.g. "Spawn vc-research-agent for RESEARCH (Step 1)"}

---

## Validate Contract

(placeholder — vc-validate-agent writes this section before EXECUTE)
