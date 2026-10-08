---
name: vc-autopilot
description: "Emit and validate the provisional goal block for Autopilot Mode. Owns the 9-field format and resume detection from a pasted goal block."
argument-hint: "[task description or pasted goal block for resume detection]"
trigger_keywords: autopilot, run autopilot, full autopilot, autonomous mode, goal block, provisional goal, AUTOPILOT_ACTIVATED, resume autopilot
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "1.0.0"
---

# vc-autopilot

Contract skill that owns the **provisional goal block artifact** for Autopilot Mode sessions. This skill is invoked by the orchestrator to emit, validate, and resume from the goal block. The canonical protocol lives in `process/development-protocols/autopilot.md`.

---

## When To Invoke

Invoke this skill when:

- The orchestrator has just received an autopilot trigger phrase and needs to emit the provisional goal block after the consolidated clarification round.
- The orchestrator detects a pasted goal block at session start and needs to determine whether it is a resume scenario (goal block already present → no new clarification round).
- The `validate-autopilot-goal-block.mjs` D1 validator needs to be run against an artifact.

---

## Skill Artifact

The **provisional goal block** — a structured text block of ≤ 4000 characters with exactly 9 named fields. The canonical field spec lives in `process/development-protocols/autopilot.md §Provisional Goal Block Format`. This skill does not redefine the spec — it references it.

Required fields (exact string anchors — do not rename or abbreviate):

1. `SESSION GOAL:`
2. `ENTRY PHASE:`
3. `REMAINING PHASES:`
4. `CLARIFICATIONS LOCKED:`
5. `EXECUTE CONSENT:` — must contain the literal text `standing-granted`
6. `DECISION POLICY:`
7. `HARD STOPS:`
8. `TEST GATES:`
9. `START:`

---

## Emission Procedure

Step-by-step for the orchestrator:

1. Complete the consolidated clarification round (`process/development-protocols/autopilot.md §Consolidated Clarification Round`).
2. Determine `ENTRY PHASE` from on-disk artifact detection (autopilot.md §Trigger-Anywhere Detection Flow).
3. Build the `REMAINING PHASES` checklist: for each phase not yet complete in canonical RIPER-5 order, add a `[ ]` checkbox line with the phase name and planned execution strategy.
4. Fill in all 9 fields using the locked clarification answers.
5. Count total characters. If > 4000: compress DECISION POLICY and CLARIFICATIONS LOCKED to summaries and reference autopilot.md for full detail.
6. Print the block to chat as a fenced code block.
7. Write the block to disk: `{task-folder}/{slug}_AUTOPILOT_GOAL_{dd-mm-yy}.md` (header: "Emitted: [datetime]. Provisional block. V7 will emit (UPDATE) variant.").
8. Emit `AUTOPILOT_ACTIVATED: [task] — entry phase: [phase] — goal block emitted`.

---

## Resume Detection

How the orchestrator recognizes a pasted goal block at session start:

- If the user message contains all 9 field names as headings/labels, treat it as an autopilot resume.
- On resume: skip the consolidated clarification round entirely. Read `ENTRY PHASE` and `REMAINING PHASES` from the pasted block. Read `CLARIFICATIONS LOCKED` as the already-locked decisions. Read `DECISION POLICY` and `HARD STOPS` as the standing policy.
- Emit `[MODE: AUTOPILOT | <ENTRY PHASE>]` and begin the run from `START:`.
- Do NOT issue a new clarification round (SPEC AC-14).

---

## V7 UPDATE Variant

When VALIDATE V7 completes during an autopilot run, the orchestrator:

1. Reads the real gate commands from the new validate-contract.
2. Copies the provisional block.
3. Prefixes `SESSION GOAL:` with `(UPDATE) `.
4. Replaces `TEST GATES: TBD — populated after VALIDATE` with the actual gate commands.
5. Updates `START:` to reflect post-VALIDATE state.
6. Prints the updated block to chat.
7. Appends an `## (UPDATE) [YYYY-MM-DD]` section to the disk file (never overwrites the original).

---

## Validator

Run after writing any goal block artifact to confirm it passes D1 checks:

```bash
node .claude/skills/vc-autopilot/scripts/validate-autopilot-goal-block.mjs <artifact-path>
```

Exit 0 = PASS (all 9 required fields present, EXECUTE CONSENT contains `standing-granted`, total ≤ 4000 chars).
Exit 1 = FAIL (one or more checks failed — error printed to stdout).
WARN printed (exit 0) when `TEST GATES:` contains `TBD` (reminder that V7 UPDATE is pending).

See `scripts/validate-autopilot-goal-block.mjs` and `fixtures/` for the D1 validator and pass/fail fixture pair.
