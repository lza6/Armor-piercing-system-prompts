---
name: vc-generate-spec
description: Create or update a product-discovery SPEC (requirements doc) for user review. Use when turning RESEARCH findings plus user intent into a reviewable requirements artifact before INNOVATE/PLAN.
argument-hint: "[feature idea or task slug]"
trigger_keywords: spec, requirements doc, product discovery, what and why, requirements artifact
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "1.0.0"
---

# Generate SPEC

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Use this skill to produce the authoritative **product-discovery SPEC** artifact for a task — a plain-language requirements document written for **user review**, not for an engineer.

A SPEC captures **what the user wants and why** so the user can read it, recognize their own intent, and confirm "yes, build that" before any approach or code is chosen. It is the bridge between RESEARCH (the facts) and INNOVATE (the how). PLAN cannot start until a SPEC exists for non-trivial work.

**SPEC consumes:** RESEARCH findings + the user's stated intent/brainstorm. It does **NOT** consume a chosen approach or a Decision Summary — no approach exists yet at SPEC time.

Normal output is one SPEC file: `{slug}_SPEC_{dd-mm-yy}.md`.

For a phase program, the program-level (umbrella) SPEC is written once during the outer loop and governs every inner phase. Use `templates/program-spec-template.md` for that case. The inner loop never writes a SPEC.

## Workflow

1. Read `references/spec-contract.md` for the full section-by-section writing rules.
2. Run `date +%d-%m-%y` before choosing the filename.
3. Confirm the inputs are present: RESEARCH findings + the user's stated intent. If neither is present, emit `SPEC_INTENT_BLOCKED` instead of writing.
4. Read `process/context/all-context.md` first, then load the relevant context group. When the work touches testing/verification, read `process/context/tests/all-tests.md` so acceptance-criteria scenarios are grounded in the real test-context chain.
5. Save the SPEC inside the task folder alongside the plan: `process/features/{feature}/active/{slug}_{date}/{slug}_SPEC_{date}.md` (or `process/general-plans/active/{slug}_{date}/{slug}_SPEC_{date}.md`). For a phase program, the umbrella task folder holds `{program-slug}_SPEC_{date}.md`. Per task-folder artefact colocation, never write to the deprecated sibling `reports/` or `references/` dirs.
6. Write the SPEC sections in canonical order (see `references/spec-contract.md`):
   - `## Summary`
   - `## User Stories / Jobs To Be Done`
   - `## What The User Wants (Behavioral Outcomes)`
   - `## Flow / State Diagram` (ASCII)
   - `## Acceptance Criteria (Testable Outcomes)` — each criterion carries `proven by:` + `strategy:`
   - `## Out Of Scope`
   - `## Constraints`
   - `## Open Questions`
   - `## Background / Research Findings`
7. Keep the document oriented for **user review**: plain language, no file paths, no library names, no schema or code. Anything engineer-only moves to Background or a later phase.
8. Each acceptance criterion MUST be provable by comprehensive tests, with a fully-automated E2E/integration gate wherever the behavior is automatable. Agent-Probe / Known-Gap stand only as an explicitly-justified residual. Vacuous-green is banned.
9. If `## Open Questions` is non-empty at finalize (interactive session): emit `SPEC_INTENT_BLOCKED` and stop. Under /goal: record each as a backlog note and continue.
10. Once Open Questions is empty/"None" (or backlogged under /goal): save the file and the agent emits `PHASE_COMPLETE: SPEC`.

## Important Rules

- The SPEC is written for a human reviewer. If a sentence only makes sense to an engineer, rewrite it or move it to Background.
- Do NOT choose an implementation approach (INNOVATE's job, downstream), write implementation steps (PLAN), make schema decisions, or include code/library references.
- Do NOT modify any file except the SPEC file being created.
- Acceptance-criteria scenarios must be grounded in the RESEARCH test-context-discovery, not invented here.
- Once INNOVATE/PLAN begins the SPEC is frozen — later scope gaps go to the phase report `## SPEC Gaps` heading and a backlog note, never an edit to the SPEC.
- For a phase program, write the umbrella SPEC once; the inner loop reads it and never writes a per-phase SPEC.

## Required SPEC Sections

Every SPEC file must include all of the following, in this order:

- `## Summary` — one plain-language paragraph: what changes for the user and why
- `## User Stories / Jobs To Be Done` — the heart of the doc; "As a [user], I want [X], so that [Y]" or JTBD
- `## What The User Wants (Behavioral Outcomes)` — observable behavior only, no implementation
- `## Flow / State Diagram` — at least one ASCII flow/state diagram
- `## Acceptance Criteria (Testable Outcomes)` — observable outcomes, each with `proven by:` + `strategy:`; max 20
- `## Out Of Scope` — at least one item
- `## Constraints` — user-stated, system/process, and technical limits from research
- `## Open Questions` — each with an owner; empty/"None" required to finalize (interactive)
- `## Background / Research Findings` — key facts from RESEARCH that shaped the requirements

Use Markdown-structured sections, not a second machine-only schema. Markdown sections are stable across all agents (Claude, Codex, future systems) without requiring a parser.
