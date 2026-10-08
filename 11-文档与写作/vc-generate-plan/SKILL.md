---
name: vc-generate-plan
description: Create or update implementation plans in the repo's SIMPLE or COMPLEX format. Use when turning an idea, PRD, or approved direction into a saved plan artifact.
trigger_keywords: plan, create plan, write plan, generate spec, plan artifact
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "1.0.0"
---

# Generate Plan

> Output style: write the plan answer-first with tables/bullets and a one-line TL;DR per the canonical rule — `process/development-protocols/communication-standards.md`.

Use this skill to produce the authoritative implementation plan artifact set for the project's work.

This skill is the canonical planning contract for the repo. Planning discipline previously spread across `vc-plan` now belongs here plus the `plan-agent` prompt.

Normal output is one plan file.

For large multi-phase programs, this skill instead defines how to create an umbrella plan plus
phase-plan set under one feature folder. See `process/development-protocols/phase-programs.md`.

Optional input: a feature idea plus `simple` or `complex` when the user already knows the intended depth.

## Workflow

1. Read `references/generate-plan.md` for the full plan contract.
2. Run `date +%d-%m-%y` before choosing the filename.
3. If complexity is not obvious, ask whether the plan is `SIMPLE` or `COMPLEX`.
4. Save the plan inside a task folder: `process/general-plans/active/{slug}_{date}/{slug}_PLAN_{date}.md` (or `process/features/{feature}/active/{slug}_{date}/{slug}_PLAN_{date}.md`). Create the `{slug}_{date}/` subfolder first. Per **task-folder artefact colocation**, every artefact this plan produces — the plan, any `{slug}_SPEC_{date}.md`, reports, and references — lives INSIDE this same task folder; never write to the deprecated sibling `reports/` or `references/` dirs.
5. Read `process/context/all-context.md` when present to choose relevant context docs.
6. For complex plans, read `.claude/skills/vc-generate-plan/references/example-complex-prd.md` before writing.
7. Include automated and manual verification gates from `process/context/tests/all-tests.md`.
8. For new or newly touched direct `*_PLAN_*.md` plans, include explicit sections for `Touchpoints`, `Public Contracts`, `Blast Radius`, `Verification Evidence`, `Test Infra Improvement Notes`, and `Resume and Execution Handoff`.
9. Keep resume/dependency notes Markdown-structured for now; do not invent a second machine-only schema.
10. If the work is a large multi-phase program, create or update a feature folder plan set:
   - one umbrella/orchestration plan
   - one direct plan file per phase
   - one durable report destination per phase
11. Validate the generated artifact:
   ```bash
   node .claude/skills/vc-generate-plan/scripts/validate-plan-artifact.mjs <plan-path>
   ```

## Important Rules

- For standard work, create exactly one plan file.
- For a phase program, create one umbrella plan plus one direct plan file per phase.
- Prefer `process/features/{feature}/active/{slug}_{date}/` task folder when the topic maps to an existing feature folder.
- Keep phase status honest: code-only completion is `CODE DONE`, not `VERIFIED`.
- Make execution trust explicit inside the plan: what code or data can change, what contracts are exposed, what proof is required, and how EXECUTE should resume after compaction.
- End with the next instruction for RIPER-5 or Cursor Plan mode.
- Treat validation failures as blockers before presenting the plan as ready.
- Fold red-team questions, dependency mapping, verification gates, and ambiguity checks into the generated plan itself instead of relying on a parallel plan-owner workflow.
- Do not hide a large program inside one giant plan if execution will actually happen phase by phase.
- Preserve the older complex-plan behavior by keeping pre-phase research and proof gates inside each
  phase plan; the new protocol changes the artifact shape, not the rigor.

## Required Plan Sections

For new or newly touched direct `*_PLAN_*.md` files, include all of the following sections:

- `Touchpoints` — files, packages, or services that will be changed or read
- `Public Contracts` — interfaces, APIs, schemas, or behaviors visible to other packages or callers
- `Blast Radius` — the scope of change: how many files, which packages, and what risk class
- `Verification Evidence` — table with columns `| Gate / Scenario | Strategy | Proves SPEC criterion |`; each row maps a test gate to the SPEC acceptance criterion it proves and the strategy (Fully-Automated / Hybrid / Agent-Probe)
- `Test Infra Improvement Notes` — placeholder at plan-write time ("(none identified yet)"); updated with test infrastructure gaps found during vc-test-coverage-plan and EVL
- `Resume and Execution Handoff` — required sub-fields:
  1. selected plan file path
  2. last completed phase or step
  3. validate-contract status (written / skipped with reason / pending)
  4. supporting context files loaded
  5. next step for a fresh agent picking up mid-execution
- `Validate Contract` — written by vc-validate-agent after VALIDATE runs; leave a placeholder
  heading during PLAN (`## Validate Contract\n\n(placeholder — vc-validate-agent writes this section before EXECUTE)`)

Use Markdown-structured sections, not a second machine-only schema. Markdown sections are
stable across all agents (Claude, Codex, future systems) without requiring a parser.
