---
name: vc-setup
description: Interactive harness setup for any project. Detects stack, scaffolds process dirs, deep-scans the codebase, populates context. Works on fresh and existing projects — always asks before reorganizing.
trigger_keywords: seed, harness setup, bootstrap, new project, scaffold, setup
layer: helper
metadata:
  author: vibecode
  version: "3.3.0"
  # version tracks SKILL.md doc changes independently from kit version in vc-manifest.json
---

# VibeCo Agent Harness Setup

> **CRITICAL — Dual-File Synchronization:** This document and `references/vc-setup.md` must be edited together. Any change to Merge Mode logic, safe-inference rules, or migration flow MUST appear in both files in the same commit. Dual-file drift creates inconsistent user behavior.

> **Output style:** Use BLUF (answer first), plain language, no unexplained jargon, TL;DR on long responses. Full rules in `process/development-protocols/communication-standards.md` once installed — on first run that file may not exist yet, so follow this inline rule instead.

Interactive setup for the agent development harness. Works on fresh projects and existing projects with pre-existing `.claude/` or `process/` configs.

The skill adapts its flow based on what it finds:
- **New projects** (no existing process/ or context): detect, ask the user about their project, scaffold, study, validate.
- **Existing projects** (has process/, context files, or CLAUDE.md): detect, study what exists first, present findings and ask what to keep vs change, scaffold with approval, re-study to fill gaps, validate.

In both cases, the skill asks questions and waits for approval at every major step. It never silently reorganizes files or overwrites good content.

CLAUDE.md and AGENTS.md are managed protocol files (orchestrator, RIPER-5 methodology, routing). They contain zero project-specific content and should NOT be adapted. Project-specific information lives in `process/context/all-context.md`, which is populated during the STUDY phase.

## Prerequisites

- The target repo should have a project manifest. Detection order:
  1. `package.json` — Node/Bun/Deno projects (JS/TS)
  2. `pyproject.toml` or `requirements.txt` — Python projects
  3. `go.mod` — Go projects
  4. `Gemfile` — Ruby projects
  5. `Cargo.toml` — Rust projects
  6. None found — ask the user: "What language/runtime does this project use? I'll adapt the setup to match."
- That's it. The skill handles the rest.

## Workflow

Read `references/vc-setup.md` for detailed phase instructions, detection heuristics, interactive question templates, parallel subagent delegation strategy, and validation checks.

### Phase 0: BOOTSTRAP (handled by install.sh)

The `install.sh` script handles fetching and installing harness files before vc-setup runs. For existing projects, it backs up old `.claude/`, `.codex/`, `.agents/` to `.vibecode-backup/`, then does a clean install of all kit files. User's `.claude/settings.json` is restored after install.

**What install.sh DOES create under `process/`:** `process/_seeds/`, `process/development-protocols/`, and `process/context/generated-skills-catalog.json`. These are kit-installed files, not user content.

**What install.sh does NOT create:** `process/general-plans/`, `process/features/`, `process/context/all-context.md`, or any context group directories. Those are vc-setup's job, created during the SCAFFOLD and STUDY phases.

**If harness files are already present** (`.claude/agents/` and `.claude/skills/` exist with 12+ agents and 20+ skills), skip Phase 0 and proceed directly to Phase 1 DETECT.

**If harness files are NOT present**, tell the user to run the installer first:
```
curl -fsSL https://raw.githubusercontent.com/withkynam/vibecode-pro-max-kit/main/install.sh | bash
```

Then re-run vc-setup.

### Phase 1: DETECT

Gather information about the target project before making any changes.

1. **Non-JS projects:** if the detected manifest is NOT `package.json` (e.g. `go.mod`, `pyproject.toml`, `Cargo.toml`, `Gemfile`), SKIP the Package Manager / Framework / Test-Setup detection steps below and jump to Manifest Detection in `references/vc-setup.md` §DETECT Phase.
2. Read `package.json` to detect the package manager (`packageManager` field, lockfile presence), framework (dependencies), and test commands (scripts).
3. Detect monorepo structure: `workspaces` in `package.json`, `pnpm-workspace.yaml`, `apps/`, `packages/` directories.
4. Scan for existing `process/`, `docs/`, `.github/` directories and any context files.
5. **Classify the project** as one of:
   - **New**: no existing `process/` directory, no `all-context.md`, no meaningful prior setup.
   - **Existing**: has `process/`, `all-context.md`, CLAUDE.md with project content, or other prior context.

   **Classification corner cases (full 7-row table in `references/vc-setup.md` §Project Classification):**
   - `process/` contains ONLY kit-installed files (`_seeds/`, `development-protocols/`, `context/generated-skills-catalog.json`) and no user content → **New / Flow A** (install.sh ran but the user hasn't set up yet).
   - `all-context.md` exists but its non-comment body is all placeholder/stub (`<!-- STUDY: -->`) → **Flow A**, continue to STUDY (do NOT treat as existing project).
5. Present a detection summary to the user and wait for confirmation before proceeding.

**After detection, the workflow branches based on project type.** See the two flows below.

---

### Flow A: New Project (no existing process/ or context)

For projects where the harness is being set up for the first time.

**Step 1: ASK** -- Before scaffolding or scanning anything, have a real conversation with the user about their project. Do not guess when you can ask. Do not ask a fixed list of questions and move on — this is an open-ended discovery conversation that continues until you have a thorough understanding.

Start with the basics, then follow up based on their answers:

**Round 1 — Project identity:**
- "What is this project? Give me a brief description in your own words."
- "Who uses it? Who is the target audience?"

**Round 2 — Architecture and scope** (adapt based on Round 1 answers):
- "What are the main product areas or features?"
- "How is the codebase organized? Any key services, packages, or modules I should know about?"
- "What are the most important or complex parts of the codebase?"

**Round 3 — Workflow and conventions** (adapt based on what you've learned):
- "Do you work solo or with a team?"
- "Any coding conventions, naming patterns, or architectural decisions that are important?"
- "How do you handle testing? CI/CD? Deployments?"
- "Any external services, APIs, or integrations that are central to the project?"

**Round 4 — Follow-ups** (ask as many as needed until everything is clear):
- Follow up on anything vague or interesting from previous answers.
- "You mentioned [X] — can you tell me more about how that works?"
- "Are there any pain points, tech debt, or things you want agents to be careful about?"
- "Anything else that is important context for working on this codebase?"

**Keep asking follow-ups until you genuinely understand the project.** If the user gives a short answer, probe deeper. If they mention something complex, ask for details. The goal is that after this conversation, you could explain the project to another developer — what it does, how it's built, what matters, and what to watch out for. Only move on when both you and the user are satisfied.

**Step 2: SCAFFOLD** -- Create the `process/` directory structure from seed templates. (See Phase 2 details below.)

**Step 3: STUDY** -- Deep-scan the codebase and populate context files with real content, enriched by the user's answers from Step 1. (See Phase 3 details below.)

**Step 4: VALIDATE** -- Verify everything is wired correctly. (See Phase 4 details below.)

---

### Flow B: Existing Project (has process/, context files, or prior setup)

For projects that already have some form of process/ directory, context files, or CLAUDE.md content.

**Step 1: STUDY EXISTING** -- Before proposing any changes, read and understand what is already there:

- Read all files under `process/context/` (especially `all-context.md`).
- Read all files under `process/general-plans/` and `process/features/`.
- Read the existing CLAUDE.md if it contains project-specific content (beyond the managed protocol).
- Read any existing `all-tests.md`, feature `_GUIDE.md` files, context group entrypoints.
- Build a complete picture of what the user already has.

**Step 2: PRESENT and ASK** -- Show the user what you found and propose changes. Format:

```
Here is what I found in your existing setup:

LOOKS GOOD (I recommend keeping these as-is):
- [list files/sections that have good content]

COULD BE IMPROVED (I can update these):
- [list files/sections that are stale, placeholder, or incomplete, with brief reason]

MISSING (I recommend adding these):
- [list files/directories that the harness expects but do not exist yet]

LAYOUT CHANGES (reorganization I would suggest):
- [list any directory moves or renames, if applicable]
- [if none needed, say "Your layout looks standard, no reorganization needed."]
```

Wait for the user to approve each category. The user may say "keep everything" or "go ahead with improvements" or selectively approve. Respect their choices.

**Then have the same discovery conversation as Flow A**, regardless of how much existing context you found. Existing context files may be stale, incomplete, or written by someone else. The user's own words are always more valuable than old docs:

- Start with: "I've read your existing context. Let me verify my understanding and fill in the gaps."
- Summarize what you learned from existing files, then ask: "Is this still accurate? What's changed?"
- Ask about anything the existing context doesn't cover (see Flow A Round 1-4 for question areas).
- Follow up on anything unclear. Keep asking until you thoroughly understand the project as it is today.

The combination of existing context + fresh user input produces the best results.

**Step 3: SCAFFOLD** -- Apply only the changes the user approved. Migrate old layouts if needed (using the migration table). Never touch files the user said to keep. (See Phase 2 details below.)

**Step 4: STUDY** -- Deep-scan and update/create context with real content. For existing files, merge intelligently -- fill gaps and update stale sections without replacing good user-written content. (See Phase 3 details below.)

**Step 5: VALIDATE** -- Verify everything is wired correctly. (See Phase 4 details below.)

---

### Phase 2: SCAFFOLD (details)

Create the `process/` directory with seed files and instructional content.

1. Determine the scaffold mode:
   - **Fresh**: no existing `process/` directory -- create everything from `process/_seeds/`.
   - **Merge**: existing `process/` with a different layout -- preserve content, migrate old layout, add missing directories, seed empty folders.
   - **Refresh**: existing harness `process/` -- update protocol docs, seed missing files, preserve user-created content.

**For Merge and Refresh modes, show the user what you plan to change before doing it.** List every file you will create, move, or overwrite. Wait for approval.

**Merge mode includes automatic layout migration.** Before creating new directories, detect and migrate old layouts:

| Old Layout | Migration Action |
|------------|-----------------|
| `process/plans/` exists, `process/general-plans/` does not | Move `process/plans/*` to `process/general-plans/active/`, then remove empty `process/plans/` |
| `process/reports/` exists at top level | Move `process/reports/*` to `process/general-plans/reports/`, then remove empty `process/reports/` |
| `process/skills/` exists at top level | Move `process/skills/*` to `process/general-plans/backlog/`, then remove empty `process/skills/` |
| Example PRDs at old locations (e.g. `process/context/example-*.md` or under `process/context/planning/` or under `process/development-protocols/references/`) | Move to `.claude/skills/vc-generate-plan/references/` |
| process/context/backlog.md at top of context/ | Move to `process/general-plans/backlog/backlog.md` |
| Flat `*_PLAN_*.md` files directly in `process/general-plans/active/` or `process/features/*/active/` (old pre-v3 layout) | Create a `{slug}_{date}/` task subfolder and move the plan file inside it. Completed plans go to `completed/{slug}_{date}/` instead. |
| `process/general-plans/reports/`, `process/general-plans/references/`, or `process/features/*/reports/`, `process/features/*/references/` sibling dirs | Auto-migrate every **safe** legacy artifact into the relevant task folder, then remove the legacy dir if it becomes empty. Leave only unresolved artifacts behind and print them for manual follow-up. |
| Feature folder missing `active/` subdirectory (e.g. `process/features/{name}/` exists but has no `active/`, `completed/`, or `backlog/` under it) | Create `active/`, `completed/`, `backlog/` under that feature folder, seed each from the `_feature-template/` `_GUIDE.md`, and print every creation. |

#### Merge Mode auto-migrate flow (orphaned-dir detection → display → confirm → execute)

When run on an existing project (Flow B), Merge Mode detects orphaned legacy dirs and offers a one-step migration:

1. **Detect** orphaned dirs: scan for `process/reports/`, `process/references/`, and any feature-level `process/features/*/reports/`, `process/features/*/references/` sibling dirs (plus pre-v3 `process/plans/`, `process/skills/`, `process/_seeds/` leftovers).
2. **Display** a migration summary table to the user — each row: source path, inferred destination task folder, file count, risk level (safe vs needs-review).
3. **Confirm** — present one-button confirmation: "Migrate and clean up all stale artifacts". The user approves under the LAYOUT CHANGES section before any move executes. vc-setup never moves files silently.
4. **Execute** — move safe artifacts into the inferred `completed/{slug}_{date}/` (or `active/{slug}_{date}/`) task folder, then delete the now-empty source dir. Print a result summary ("moved X files, deleted Y dirs").

##### Safe-Inference Rules for Pre-Create

- **Safe** — a plan slug exists in `process/general-plans/active/` or `process/features/{feature}/active/` and exactly one matching `{slug}_{date}/` task folder can be inferred → auto-create `completed/{slug}_{date}/` as the destination and migrate the artifact.
- **NOT safe** — no matching plan slug is found, filenames are ambiguous, multiple task folders match, or there is a conflict with an existing dir → do NOT create any destination folder; show the fallback message below and pause for manual review.

##### Fallback Display Message

When orphaned dirs are detected but no plan slug can be inferred from project files (no safe inference possible), vc-setup displays:

> "Found orphaned dir [path] but cannot safely infer destination task folder. Manual migration required. See reference docs for migration guide."

vc-setup will NOT auto-create ghost folders (false folders) when inference fails. The user must review and manually place the files. Conservative behavior: when in doubt, leave the artifact in place and surface it — never guess a destination.

**Migration rules:**
- Never overwrite existing files at the destination. If a file with the same name exists, keep both (rename the migrated copy with a `-migrated` suffix).
- Print every move action to the user so they can verify.
- After all moves, remove empty source directories.
- If `process/plans/` contains files matching date-based patterns (e.g., `2026-05-22-*.md`, `*_PLAN_*.md`), classify completed plans (look for "COMPLETE" or "DONE" in the file) and move them to `completed/` instead of `active/`.
- **Safe legacy artifact migration contract:** for legacy `reports/` / `references/` sibling dirs, move an artifact automatically only when exactly one destination task folder can be inferred in the same scope (`process/general-plans/` or one `process/features/{feature}/`). Safe inference signals are:
  - the artifact basename already starts with one task slug and only one matching `{slug}_{date}/` folder exists
  - the artifact path or basename contains an exact `{slug}_{date}` token matching one task folder
  - there is exactly one task folder total in that scope
  - exactly one task-folder plan references the legacy artifact path or basename
- Treat an artifact as **unsafe** and leave it in place when multiple task folders match, when multiple plans reference it, when it is clearly cross-task/shared, or when no task folder exists yet in that scope.
- Preserve the original filename by default when moving legacy artifacts. Only add a `-migrated` suffix when the destination already has a same-name file.
- After migrating all safe artifacts from a legacy sibling `reports/` or `references/` dir, delete that dir if and only if it is empty. The target end-state is that each feature folder and `process/general-plans/` keep only `active/`, `completed/`, and `backlog/` unless unresolved legacy artifacts still block cleanup.

Seed and template handling:
1. Seeds live in `process/_seeds/` (read-only during setup -- never modified by the scaffold process):
   - Files with `.seed` extension: copy with `.seed` removed, replace `{{project_name}}` with the detected project name.
   - Files without `.seed` extension: copy verbatim.
   - Context group seed folders use `-seed` suffix (e.g., `tests-seed/`, `planning-seed/`). When copying to real locations, drop the `-seed` suffix.
2. Copy development protocol docs from `process/development-protocols/` (these are managed system files, not seeds -- they live in the real directory, not `_seeds/`).
3. Place `_GUIDE.md` files in empty process folders to explain what goes there.
4. Retain `.seed` originals alongside populated files: after copying and filling seed files, also copy the original `.seed` files to the target `process/` directory as structural reference companions. These `.seed` files serve as reference for what sections are expected, and future `vc-update` can diff against them to detect structural drift.
5. Use `_all-group-template.md.seed` as the base when creating new context group entrypoints during the STUDY phase.
6. Use `_feature-template/_GUIDE.md.seed` as the base when creating new feature folder guides during the STUDY phase. The `_feature-template/` includes 3 subdirectories (`active/`, `completed/`, `backlog/`) with their own `_GUIDE.md` files. Do NOT create `reports/` or `references/` sibling dirs for new repos — these are deprecated; new artifacts go inside task folders under `active/` or `completed/`.
7. See `references/vc-setup.md` for the full target directory tree and placeholder list.

**After scaffolding, show a summary of what was created/changed.** Example: "Created 12 directories, 8 seed files, 6 protocol docs. No existing files were modified."

### Phase 3: STUDY

Perform deep codebase analysis and populate context files with real, researched content.

This is the core value -- instead of leaving placeholder text, the STUDY phase actively reads the codebase and writes ready-to-use context.

1. **Architecture and stack analysis**: Scan source directories, detect frameworks with versions, map import aliases, catalog environment variables, identify key patterns and conventions.
2. **Test setup analysis**: Identify test runners, config files, test directories, and test commands per package/workspace.
3. **Context group detection and per-group file authoring**: Invoke the `vc-generate-context` skill in `setup-delegation` mode. Pass: (a) the approved-groups list from the context-group-detector subagent (Round 1 Subagent C), and (b) mode = `setup-delegation`. This skill will produce all `process/context/{group}/all-{group}.md` files for the approved groups. See `.claude/skills/vc-generate-context/SKILL.md` for the Invocation Modes reference and `.claude/skills/vc-generate-context/references/generate-context.md` for the detection table and per-mode instructions.
4. **Feature area detection**: Identify major product areas from route groups, packages, and existing docs. Create feature folders for areas meeting the threshold (3+ source files, distinct product area).
5. **Populate all-context.md**: Write real repository structure, technology stack details, key patterns, environment configuration, and routing tables -- not placeholders. Incorporate what the user told you in the ASK step. Note: per-group context-file authoring (`process/context/{group}/all-{group}.md`) is delegated to `vc-generate-context` (step 3 above); `all-context.md` itself is Subagent E's responsibility and is authored here in vc-setup.
6. **Populate all-tests.md**: Write actual test runner names, real test commands, and per-package breakdowns.
7. **Migration intelligence** (when existing process/ content is found): Read existing content, identify gaps vs fresh scan, fill only gaps while preserving user-written content.

**For existing projects (Flow B):** Before writing, compare your scan results against existing content. If the existing content is more detailed than what you scanned, keep it. Only replace placeholder or stale sections. Add missing sections with scanned data. Never silently overwrite good content.

**After the STUDY phase, show a summary of what was populated.** Example: "Populated all-context.md (8 sections with real content), all-tests.md (3 test runners, 12 commands), created 2 context groups (database/, container/), created 3 feature folders."

See `references/vc-setup.md` for the full STUDY phase checklist, parallel subagent delegation strategy, context group detection table, and feature detection heuristics.

### Phase 4: VALIDATE

Verify the setup is complete, correct, and populated with real content.

1. Check all expected directories exist under `process/`.
2. Verify agent parity: agent names in `.claude/agents/` should match `.codex/agents/`.
3. Check that `.agents/skills` symlink exists and resolves.
4. Verify STUDY phase output quality:
   - `all-context.md` has no remaining `{{placeholder}}` text (except `{{project_name}}` if seed was just created)
   - `all-context.md` has a populated Repository Structure section with real directory tree
   - `all-context.md` has a populated Technology Stack section with specific versions
   - `all-tests.md` has actual test commands (not placeholder text)
   - Context groups created have corresponding entries in the routing tables
   - Feature folders created have `_GUIDE.md` files with real scope descriptions
4. **Placeholder scan (required):** grep the populated `all-*.md` context files for remaining `<!-- STUDY:` or `(pending` markers. If any remain, STUDY is incomplete — return to STUDY and populate them before declaring VALIDATE done.
   ```bash
   grep -rn -e '<!-- STUDY:' -e '(pending' process/context/ && echo 'INCOMPLETE — populate before VALIDATE'
   ```
   A zero-match exit (no output, exit 1 from grep) means the scan is clean and VALIDATE may proceed.
6. **Catalog generate-on-install safety check:** If `process/context/generated-skills-catalog.json` is absent after setup, generate it now:
   ```bash
   node .claude/skills/vc-audit-context/scripts/generate-skills-catalog.mjs --write
   ```
   This file is required for `discover-skills.mjs` (Routing Step 0) to work correctly. Fresh installs that copy this file from the kit manifest include do not need this step, but if the file is missing for any reason (missing manifest include, partial install), generate it explicitly. Note: `generate-skills-catalog.mjs` (and its shared utils) works on non-git projects — it falls back to `process.cwd()` when `git rev-parse` is unavailable.
7. Report any issues found.
8. Suggest running validation scripts if they exist in the target repo:
   - `node .claude/skills/vc-generate-context/scripts/validate-all-context.mjs`
   - `node .claude/skills/vc-audit-context/scripts/validate-context-discovery.mjs`

   **Validator cwd discipline:** Run every validator from the project root: `cd {project_root} && node .claude/skills/...`. The scripts resolve the project via `git rev-parse --show-toplevel`; running from a parent directory resolves the wrong root and produces misleading results.

   **Expected validator warnings on fresh projects:** `validate-context-discovery.mjs` may report missing context group directories such as process/context/uxui/ (if project has UI/UX context group) or other groups. This is EXPECTED for projects that do not have content in those domains — do not create empty group directories just to silence the warning. Create a context group only when the project genuinely has substantial content for that domain (see Context Group Detection Table in the STUDY phase).

**Present the final summary** to the user: what was set up, what is ready to use, and recommended next steps (review context, start using the harness).

## Interaction Principles

These principles apply throughout the entire setup flow:

- **Never reorganize without asking.** Show what you found, propose changes, wait for approval.
- **Study before scaffold for existing projects.** Understand what is already there before proposing changes.
- **Have a real conversation, not a checklist.** Do not ask a fixed list of questions and move on. Ask, listen, follow up, ask more. If an answer is vague, probe deeper. If the user mentions something interesting, explore it. Continue until you genuinely understand the project — what it does, how it's built, what matters, and what to watch out for.
- **Show summaries at each step.** After DETECT, show findings. After ASK, summarize your understanding and confirm it's correct. After SCAFFOLD, show what was created. After STUDY, show what was populated.
- **Preserve the user's existing good content.** If they have a well-written CLAUDE.md, detailed context files, or a working process/ layout, merge intelligently -- do not replace with generic scans.
- **One step at a time.** Complete each phase, show the result, and get confirmation before moving to the next phase.
- **Verify your understanding before acting.** After the discovery conversation, summarize what you learned back to the user: "Here's what I understand about your project: [summary]. Is this accurate? Anything I'm missing?" Only proceed when they confirm.

## Rules

- CLAUDE.md and AGENTS.md are managed protocol files. Do NOT adapt or modify them.
- Do not modify RIPER-5 methodology sections, phase transition rules, or key principles.
- Do not modify tool restriction lists in agent prompts.
- Do not modify the status reporting format (DONE, DONE_WITH_CONCERNS, BLOCKED, NEEDS_CONTEXT).
- Always wait for user confirmation after the DETECT phase before making changes.
- Always ask the user about their project before populating context files. Do not rely solely on code scans.
- In Merge mode, never overwrite existing user content.
- In Flow B (existing projects), always present what you found and get approval before modifying anything.
- Project-specific information (tech stack, features, conventions) belongs in `process/context/all-context.md`, not in CLAUDE.md.
- In STUDY phase, write real researched content, not placeholder text. Every section should contain actual project-specific information discovered by scanning the codebase AND informed by the user's answers.
- For large repos (monorepos, 5+ source directories), spawn parallel subagents to maximize throughput and avoid context window exhaustion -- see reference doc for delegation strategy.
