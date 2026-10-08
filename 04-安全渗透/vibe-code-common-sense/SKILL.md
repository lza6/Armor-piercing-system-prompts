---
name: vibe-code-common-sense
description: "Use when the user explicitly asks to apply Vibe Code Common Sense, create project rails, scaffold AI-agent project guidance, build a project blueprint, create AGENTS.md instructions, or set up docs/PRODUCT_TIMELINE.md progress tracking for an AI-built software project. Do not use for ordinary feature work in an already-tracked project unless the user asks to update the project guidance system."
---

# Vibe Code Common Sense

Treat every AI coding agent as dumb until the project proves otherwise.

Agents can move fast, but a new project is vulnerable: the idea is loose, the
architecture is undecided, the roadmap is unclear, and an agent will confidently
fill the gaps unless the project has rails.

Use this skill to turn a raw idea into a guided open-source project with:

- a clear product blueprint;
- an explicit roadmap;
- a PR-sized product timeline;
- agent instructions;
- verification-before-completion rules;
- decision records for architecture choices;
- progress tracking that survives across sessions and agents.

## Core Workflow

1. Read the current repo state before proposing structure.
2. If the project is new, create the starter docs from `references/`.
3. If the project exists, preserve current direction and add only missing rails.
4. Convert the idea into phases, then PR-sized checklist items.
5. Make `docs/PRODUCT_TIMELINE.md` the source of truth for progress.
6. Require every PR to mark exactly the completed item `[x]` with its PR number.
7. Choose next work from the first unchecked item unless the user overrides it.
8. Verify changes before claiming completion.

## Files To Create Or Maintain

- `README.md`: project summary, status, setup, docs links.
- `AGENTS.md`: rules for AI agents working in the repo.
- `docs/PROJECT_BLUEPRINT.md`: problem, users, goals, non-goals, MVP.
- `docs/PRODUCT_TIMELINE.md`: phase checklist and PR tracking.
- `docs/ROADMAP.md`: milestone-level product direction.
- `docs/ARCHITECTURE.md`: system shape and boundaries.
- `docs/DEVELOPMENT_CYCLE.md`: branch, PR, test, and release workflow.
- `docs/DECISIONS.md` or `docs/adr/`: decisions that should not be re-litigated.

## Reference Templates

Load only the needed reference:

- `references/readme-template.md`: use for root `README.md`.
- `references/project-blueprint-template.md`: use for `docs/PROJECT_BLUEPRINT.md`.
- `references/product-timeline-template.md`: use for `docs/PRODUCT_TIMELINE.md`.
- `references/agents-template.md`: use for root `AGENTS.md`.
- `references/roadmap-template.md`: use for `docs/ROADMAP.md`.
- `references/architecture-template.md`: use for `docs/ARCHITECTURE.md`.
- `references/decisions-template.md`: use for `docs/DECISIONS.md`.
- `references/pr-workflow.md`: use for `docs/DEVELOPMENT_CYCLE.md` or PR rules.
- `references/claude-code-install.md`: use when installing or configuring this
  skill for Claude Code.

## Operating Rules

- Keep PRs small enough to review.
- Do not invent architecture silently; add or update a decision note.
- Do not skip tracker updates.
- Do not mark a task complete before verification.
- Do not continue building from memory after a merge; sync main and read the
  tracker again.
- If the tracker and user request conflict, tell the user exactly what conflicts
  and ask only if the safe path is unclear.
- Only create branches, push commits, open PRs, or add PR numbers when the user
  has asked for a PR workflow and the repository has a configured remote.

## Completion Standard

Before calling work ready:

- local tests or documented checks have passed;
- `docs/PRODUCT_TIMELINE.md` is updated when a tracked item is completed;
- the next unchecked item is visible;
- the summary includes verification evidence;
- the repo is not left with unintended changes.
