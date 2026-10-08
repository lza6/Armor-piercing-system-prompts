# VibeCo Agent Harness Setup - Reference

vc-setup version 3.3.0 — (independent of kit version; see SKILL.md for documentation discipline)

> **CRITICAL — Dual-File Synchronization:** This document and `.claude/skills/vc-setup/SKILL.md` must be edited together. Any change to Merge Mode logic, safe-inference rules, or migration flow MUST appear in both files in the same commit. Dual-file drift creates inconsistent user behavior.

Detailed instructions for each phase of the vc-setup skill, including interactive flows for new and existing projects.

## Project Classification

After running DETECT, classify the project before choosing a flow:

| Signal | Classification |
|--------|---------------|
| No `process/` directory, no `all-context.md`, no meaningful CLAUDE.md content | **New project** -- use Flow A |
| `process/` exists but contains ONLY kit-installed files (`_seeds/`, `development-protocols/`, `context/generated-skills-catalog.json`) and no user content (`all-context.md` absent, `general-plans/` absent, `features/` absent or empty) | **New project** -- use Flow A (install.sh ran but vc-setup has not yet run; this is a fresh install) |
| Has `process/` directory with user content (e.g. `all-context.md`, `general-plans/` with plans, `features/` with entries) | **Existing project** -- use Flow B |
| Has `all-context.md` with real (non-placeholder) content | **Existing project** -- use Flow B |
| Has `all-context.md` but its non-comment body is entirely placeholder/stub (e.g. contains `<!-- STUDY:` markers throughout, all sections say "pending", no real stack or routing content) | **New project** -- use Flow A (SCAFFOLD ran but STUDY was interrupted; treat as fresh and continue from STUDY) |
| Has CLAUDE.md with project-specific sections (beyond managed protocol) | **Existing project** -- use Flow B |
| Has `.vibecode-backup/` (just ran install.sh over an existing setup) | **Existing project** -- use Flow B |

**Classification is based on USER content, not kit-installed files.** `_seeds/`, `development-protocols/`, and `context/generated-skills-catalog.json` are always present after install.sh and do not indicate an existing project setup. Check for user-created files before routing to Flow B.

When in doubt, treat as existing. It is always safer to study first and ask before changing things.

---

## Flow A: New Project

For projects where the harness is being set up for the first time. The full sequence is:

```
DETECT -> present summary -> ASK user about project -> SCAFFOLD -> STUDY -> present summary -> VALIDATE
```

### Step 1: DETECT

Run all detection checks (see DETECT Phase section below). Present findings to the user. Wait for confirmation.

### Step 2: ASK — Discovery Conversation

This is not a checklist — it is an open-ended conversation that continues until you thoroughly understand the project. The user's own words are the most valuable input for producing useful context files.

**How the conversation works:**

Start broad, then go deeper based on answers. Ask follow-ups on anything vague or interesting. Do not move on after a fixed number of questions — keep going until both you and the user are satisfied that nothing important is missing.

**Round 1 — Project identity (always start here):**
- "What is this project? Give me a brief description in your own words."
- "Who uses it? Who is the target audience?"
- Purpose: populates the project description in all-context.md. A human description is always better than what a code scan infers.

**Round 2 — Architecture and scope (adapt based on Round 1):**
- "What are the main product areas or features?"
- "How is the codebase organized? Any key services, packages, or modules I should know about?"
- "What are the most important or complex parts of the codebase?"
- Purpose: guides feature folder creation, architecture sections, and STUDY focus areas.

**Round 3 — Workflow and conventions (adapt based on what you've learned):**
- "Do you work solo or with a team?"
- "Any coding conventions, naming patterns, or architectural decisions that are important?"
- "How do you handle testing? CI/CD? Deployments?"
- "Any external services, APIs, or integrations that are central to the project?"
- Purpose: populates Key Patterns and Conventions, testing context, and infrastructure sections.

**Round 4+ — Follow-ups (as many rounds as needed):**
- Follow up on anything vague: "You mentioned [X] — can you tell me more about how that works?"
- Probe pain points: "Are there any pain points, tech debt, or things you want agents to be careful about?"
- Catch-all: "Anything else that is important context for working on this codebase?"
- Domain-specific: If the project involves a specialized domain (finance, healthcare, gaming, etc.), ask about domain concepts that agents need to understand.
- Purpose: captures the nuances that make context files genuinely useful vs generic.

**When to stop asking:** When you can confidently explain the project to another developer — what it does, how it's built, what matters, what to watch out for. Summarize your understanding back to the user and ask them to confirm: "Here's what I understand about your project: [summary]. Is this accurate? Anything I'm missing?"

**How to use the answers:**

- Store the user's answers in working memory. Do not write them to files yet.
- During the STUDY phase, incorporate the answers into the relevant sections of all-context.md.
- The user's description of the project should appear near the top of all-context.md, not buried in a scan output.
- Feature areas the user mentions should be cross-referenced with the automated feature detection. If the user names a feature area that the scan does not detect, ask the user where the relevant code lives rather than silently skipping it.
- Pain points and gotchas the user mentions should appear in a dedicated section so agents are warned.

### Steps 3-5: SCAFFOLD, STUDY, VALIDATE

Proceed with the standard phases documented below. All scaffolding uses Fresh mode (create everything from seeds). The STUDY phase should reference the user's answers when populating context files.

---

## Flow B: Existing Project

For projects that already have process/, context files, or prior setup. The full sequence is:

```
DETECT -> STUDY EXISTING -> PRESENT & ASK -> (optionally ASK project questions) -> SCAFFOLD (approved changes only) -> STUDY (gap-fill) -> present summary -> VALIDATE
```

### Step 1: DETECT

Run all detection checks (see DETECT Phase section below). Note what existing setup was found. Present findings to the user. Wait for confirmation.

### Step 2: STUDY EXISTING

Before proposing any changes, build a complete picture of what is already there. This step is read-only.

**Read these files/directories:**

- `process/context/all-context.md` -- assess quality: does it have real content or placeholders?
- `process/context/tests/all-tests.md` -- assess quality: real test commands or template text?
- All `process/context/*/all-*.md` -- list existing context groups
- All `process/features/*/` -- list existing feature folders, read their `_GUIDE.md` files
- All `process/general-plans/active/*.md` -- note active plans (do not touch these)
- `process/development-protocols/` -- check if protocol docs exist and their version
- Any `CLAUDE.md` content beyond the managed protocol (some users add project-specific sections)

**Cross-check against the expected target directory tree.** Any directory from the following list that does not exist on disk is classified as MISSING:

- `process/general-plans/active/`, `process/general-plans/completed/`, `process/general-plans/backlog/`
- `process/features/`
- `process/context/planning/`
- `process/context/tests/`

Record MISSING directories in the PRESENT & ASK findings under "MISSING" so the user can approve their creation during SCAFFOLD.

**Produce an internal assessment:**

For each file/directory found (or expected), classify it as:
- **Good**: has real, detailed, useful content. Keep as-is.
- **Stale**: has content but it is outdated, incomplete, or has unfilled placeholders. Candidate for update.
- **Placeholder**: seed template text with no real content. Candidate for replacement.
- **Missing**: expected by the harness but does not exist. Candidate for creation.

### Step 3: PRESENT and ASK

Show the user your findings in this format:

```
Here is what I found in your existing setup:

LOOKS GOOD (I recommend keeping these as-is):
- process/context/all-context.md -- has detailed architecture and stack info
- process/features/auth/ -- well-documented with 3 active plans
- [etc.]

COULD BE IMPROVED (I can update these):
- process/context/tests/all-tests.md -- has placeholder text, no real test commands
- process/features/billing/_GUIDE.md -- empty, no scope description
- [etc., with brief reason for each]

MISSING (I recommend adding these):
- process/context/database/ -- you have Prisma with 15+ models but no database context group
- process/features/api/ -- the API layer has 20+ routes, warrants its own feature folder
- [etc., with evidence for each recommendation]

LAYOUT CHANGES (reorganization I would suggest):
- process/plans/ -> process/general-plans/active/ (old layout, 4 plan files to migrate)
- [etc.]
- [if none needed: "Your layout matches the harness standard. No reorganization needed."]
```

**Wait for the user to respond.** They may:
- Approve everything: "looks good, go ahead"
- Selectively approve: "update the stale ones but don't add the new feature folders"
- Reject changes: "leave everything as-is, just validate"
- Ask questions: "why do you think billing needs improvement?"

Respect their choices. Only proceed with approved changes.

**Then have the full discovery conversation from Flow A (Step 2: ASK),** regardless of how much existing context you found. Existing context files may be stale, incomplete, or written by someone else. The user's live answers are always more current and more valuable.

Start with: "I've read your existing context. Let me verify my understanding and fill in the gaps." Then:
- Summarize what you learned from existing files.
- Ask: "Is this still accurate? What's changed since these docs were written?"
- Work through the same Round 1-4 question areas from Flow A, skipping anything the existing context already covers well and the user confirms is current.
- Follow up on anything unclear or contradictory between the existing docs and what you see in the code.
- Keep asking until you thoroughly understand the project as it is today.

The combination of existing context + fresh user input produces the best results. Neither alone is sufficient.

### Steps 4-6: SCAFFOLD, STUDY, VALIDATE

- **SCAFFOLD**: Apply only the changes the user approved. Use Merge or Refresh mode as appropriate. Show what will be changed before doing it.
- **STUDY**: Deep-scan and populate/update files. For files the user said to keep, do not overwrite. For stale/placeholder files, replace with real content. For new files, populate from scratch. Always merge intelligently -- see Migration Intelligence section below.
- **VALIDATE**: Standard validation, then present the final summary.

---

## DETECT Phase

> **Non-JS projects:** If the manifest detected is NOT `package.json` (i.e. the project is Python, Go, Ruby, or Rust), skip the Package Manager Detection, Framework Detection, and Test Setup Detection subsections below — those read `package.json` and are JS-only. Use the runtime-specific heuristics in each manifest entry instead. Derive the project name from the runtime manifest (`module` line in `go.mod`, `name` field in `Cargo.toml` or `pyproject.toml`, directory name for Ruby/Gemfile projects) rather than `package.json`.

### Project Manifest Detection

Before reading `package.json`, check which project manifest is present. Use this fallback chain:

1. `package.json` → Node/Bun/Deno project (JS/TS); proceed with full JS detection below.
2. `pyproject.toml` or `requirements.txt` → Python project; adapt detection heuristics (skip package manager / framework checks, detect test runner from `pytest`/`unittest`, detect ORM from `sqlalchemy`/`django`).
3. `go.mod` → Go project; detect module name from `module` directive, infer test runner (`go test`); skip Package Manager / Framework / Test Setup subsections below.
4. `Gemfile` → Ruby project; detect framework (`rails`/`sinatra`), test runner (`rspec`/`minitest`); skip Package Manager / Framework / Test Setup subsections below.
5. `Cargo.toml` → Rust project; detect workspace members, test runner (`cargo test`); skip Package Manager / Framework / Test Setup subsections below.
6. None found → Ask the user: "I couldn't find a project manifest (package.json, pyproject.toml, go.mod, Gemfile, Cargo.toml). What language/runtime does this project use? I'll adapt the setup to match."

### Package Manager Detection

Read `package.json` and check these signals in order:

1. `packageManager` field (e.g., `"pnpm@9.0.0"`, `"yarn@4.0.0"`, `"npm@10.0.0"`)
2. Lockfile presence:
   - `pnpm-lock.yaml` -> pnpm
   - `yarn.lock` -> yarn
   - `package-lock.json` -> npm
   - `bun.lockb` or `bun.lock` -> bun
3. Default to `npm` if no signal found

### Framework Detection

Read `dependencies` and `devDependencies` in `package.json`:

- `next` -> Next.js (check version for App Router vs Pages Router)
- `nuxt` -> Nuxt
- `@angular/core` -> Angular
- `svelte` or `@sveltejs/kit` -> Svelte/SvelteKit
- `vue` -> Vue
- `remix` or `@remix-run/react` -> Remix
- `astro` -> Astro
- `express` or `fastify` or `hono` or `koa` -> Node.js server
- None of the above -> generic Node.js/TypeScript project

### Test Setup Detection

Read `scripts` in `package.json`:

- Look for `test`, `test:unit`, `test:e2e`, `test:integration` scripts
- Detect test runner from devDependencies: `vitest`, `jest`, `mocha`, `@playwright/test`, `cypress`
- Record test commands for use in seed files

### Monorepo Detection

Check these signals:

- `workspaces` field in `package.json` (npm/yarn workspaces)
- `pnpm-workspace.yaml` file exists
- `apps/` directory exists
- `packages/` directory exists
- `lerna.json` exists
- `turbo.json` or `nx.json` exists

### Existing Setup Scan

Check for:

- `process/` directory (existing harness)
- `process/context/all-context.md` (existing context -- read first few lines to check if real or placeholder)
- `process/development-protocols/` (existing protocol docs)
- `process/features/` (existing feature folders)
- `docs/` directory (existing documentation)
- `.github/` directory (CI/CD)
- `README.md` content (project description)
- `.vibecode-backup/` (indicates install.sh just ran over an existing setup)

### Detection Summary Format

Present to user:

```
Project Detection Summary:
- Project name: {from package.json name field}
- Package manager: {detected_pm}
- Framework: {framework} {version} ({details like "App Router", "Pages Router"})
- Runtime: {node/bun/deno} {version from engines or .node-version}
- Monorepo: {yes/no} ({workspace count} workspaces)
- Database: {ORM} + {DB type} (or "none detected")
- Auth: {provider} (or "none detected")
- Test runner: {runners with per-package breakdown}
- Test commands: {commands}
- Existing setup: {none / partial / full harness} ({details})
- Setup flow: {Flow A (new project) / Flow B (existing project)}

Detected context groups: {list of groups to create}
Detected feature areas: {list of features to create}

[For Flow A]: I'll ask you a few questions about your project before setting things up.
[For Flow B]: I'll study your existing setup first, then show you what I found and recommend.

Proceed? (y/n)
```

## SCAFFOLD Phase

### Managed vs User Files

CLAUDE.md, AGENTS.md, and agent prompts are **managed protocol files**. They are copied as-is from the harness source and should NOT be adapted per-project. They contain zero project-specific content.

Project-specific information lives in `process/context/all-context.md`, populated during the STUDY phase.

### Seed Location

All seed templates live in `process/_seeds/` (read-only during setup -- never modified by the scaffold process). The SCAFFOLD phase reads from `_seeds/` and copies to the real working directories.

Context group seed folders use `-seed` suffix (e.g., `tests-seed/`, `planning-seed/`). When copying to real locations, drop the `-seed` suffix.

### Seed Directory Tree

```
process/_seeds/
  _GUIDE.md                              -- explains the process/ directory
  context/
    all-context.md.seed                  -- root context router template
    _all-group-template.md.seed          -- template for new context group entrypoints
    tests-seed/
      all-tests.md.seed                  -- testing context template
    planning-seed/
      all-planning.md.seed               -- planning context template
  features/
    _GUIDE.md                            -- feature folders guide (verbatim)
    _feature-template/
      _GUIDE.md.seed                     -- template for new feature folder guides
      active/
        _GUIDE.md                        -- active plans guide (task-folder convention)
      completed/
        _GUIDE.md                        -- completed plans guide
      backlog/
        _GUIDE.md                        -- backlog guide
      (no reports/ or references/ — these are deprecated sibling dirs; artifacts colocate inside task folders)
  general-plans/
    active/
      _GUIDE.md                          -- active plans guide (task-folder convention)
    completed/
      _GUIDE.md                          -- completed plans guide
    backlog/
      _GUIDE.md                          -- backlog guide
    (no reports/ or references/ — deprecated; artifacts go inside task folders under active/ or completed/)
```

### Target Directory Tree (after SCAFFOLD)

```
process/
  _seeds/                                -- read-only seed templates (copied from above)
  development-protocols/
    all-development-protocols.md
    orchestration.md
    implementation-standards.md
    plan-lifecycle.md
    phase-programs.md
    context-maintenance.md
  context/
    all-context.md
    all-context.md.seed              -- structural reference companion
    _all-group-template.md.seed      -- template for new context group entrypoints
    planning/
      all-planning.md
      all-planning.md.seed           -- structural reference companion
    tests/
      all-tests.md
      all-tests.md.seed              -- structural reference companion
  general-plans/
    active/
      _GUIDE.md                      -- documents task-folder convention
    completed/
      _GUIDE.md
    backlog/
      _GUIDE.md
    (reports/ and references/ are NOT created for new repos — deprecated sibling dirs)
  features/
    _GUIDE.md
    _feature-template/
      _GUIDE.md.seed                 -- template for new feature folder guides
      active/
        _GUIDE.md                    -- documents task-folder convention
      completed/
        _GUIDE.md
      backlog/
        _GUIDE.md
      (reports/ and references/ are NOT created for new repos — deprecated sibling dirs)
```

### Migration Mode Decision Logic

| Signal | Mode |
|--------|------|
| No `process/` directory exists | Fresh |
| `process/` exists but no `development-protocols/` | Merge |
| `process/` exists with `development-protocols/` | Refresh |
| `process/context/all-context.md` exists | Merge or Refresh (preserve it) |

### Fresh Mode

Create everything from `process/_seeds/` (read-only source, never modified during setup):

1. Create all directories in the target tree
2. Copy all files from `process/_seeds/` to their real locations
3. Process `.seed` files: copy with `.seed` removed, replace `{{project_name}}` with detected project name
4. Copy non-seed files verbatim (example PRDs, development protocols, `_GUIDE.md` files)
5. Context group seed folders: copy from `-seed` suffix dirs to real dirs (e.g., `tests-seed/` -> `tests/`, `planning-seed/` -> `planning/`)
6. Retain `.seed` originals: copy the original `.seed` files alongside the populated real files in the target `process/` directory. These serve as structural reference companions -- agents and future `vc-update` runs can diff populated files against their `.seed` originals to detect structural drift or missing sections

### Merge Mode

Preserve existing content, migrate old layouts, fill gaps (read from `process/_seeds/`, never modify seeds):

**Step 0 -- Layout Migration (before creating anything new):**

Detect old directory layouts and reorganize them into the harness standard structure. **Show every planned move to the user and wait for approval before executing.**

| Old Layout | Migration Action |
|------------|-----------------|
| `process/plans/` exists, no `process/general-plans/` | Create `process/general-plans/active/` and `process/general-plans/completed/`. For each file in `process/plans/`: scan for "COMPLETE", "DONE", or checkmark markers -- move matches to `completed/`, move the rest to `active/`. Remove empty `process/plans/`. |
| `process/reports/` exists at top level | Move `process/reports/*` to `process/general-plans/reports/`. Remove empty `process/reports/`. |
| `process/skills/` exists at top level | Move `process/skills/*` to `process/general-plans/backlog/`. Remove empty `process/skills/`. |
| Example PRDs at old locations (under `process/context/`, `process/context/planning/`, or `process/development-protocols/references/`) | Move to `.claude/skills/vc-generate-plan/references/`. |
| process/context/backlog.md | Move to `process/general-plans/backlog/backlog.md` |
| Flat `*_PLAN_*.md` file directly in `process/general-plans/active/` or `process/features/*/active/` (pre-v3.0.0 layout) | Create a `{slug}_{date}/` task subfolder and move the plan file inside it. Scan the plan for "COMPLETE"/"DONE" markers; if found, create under `completed/{slug}_{date}/` instead. Never overwrite if a task folder with the same name exists — add `-migrated` suffix. |
| `process/general-plans/reports/`, `process/general-plans/references/`, or `process/features/*/reports/`, `process/features/*/references/` sibling directories | Auto-migrate every **safe** legacy artifact into the relevant task folder in the same scope. Delete the legacy dir if it becomes empty. Leave only unresolved artifacts behind, list them in LAYOUT CHANGES, and do NOT create new sibling `reports/` or `references/` dirs during scaffold. |

**Migration rules:**
- Never overwrite existing files at the destination. If a same-name file exists, keep both (rename the migrated copy with a `-migrated` suffix).
- Print every move action so the user can verify.
- After all moves, remove empty source directories.
- If a plan file contains phase patterns (`phase-00-`, `phase-01-`, etc.) and a master plan file exists alongside them, keep them grouped in the same destination directory.
- **Safe legacy artifact migration contract:** for sibling `reports/` / `references/` dirs, a move is automatic only when exactly one destination task folder can be inferred in the same scope. Safe inference signals are:
  - the artifact basename starts with one task slug and only one matching `{slug}_{date}/` folder exists
  - the artifact basename or path contains an exact `{slug}_{date}` token matching one task folder
  - the scope has exactly one task folder total
  - exactly one task-folder plan references that legacy artifact path or basename
- Leave an artifact in place when multiple task folders match, multiple plans reference it, the file is clearly shared across tasks, or no task folder exists yet in that scope.
- Preserve the original filename when moving unless there is a collision; in that case append `-migrated`.
- The intended end-state after successful safe migration is that each feature folder and `process/general-plans/` contain only `active/`, `completed/`, and `backlog/`. A remaining legacy `reports/` or `references/` dir means unresolved artifacts still need manual review.

#### Migration Flow (orphaned-dir detection → display → confirm → execute)

This mirrors the Merge Mode auto-migrate flow in `SKILL.md` — same rationale, same rules, same fallback logic. The two docs MUST stay consistent (dual-file synchronization).

1. **Detect** orphaned dirs: scan for `process/reports/`, `process/references/`, and feature-level `process/features/*/reports/`, `process/features/*/references/` sibling dirs (plus pre-v3 `process/plans/`, `process/skills/` leftovers).
2. **Display** a migration summary table — each row: source path, inferred destination task folder, file count, risk level (safe vs needs-review). Surface under the LAYOUT CHANGES section of PRESENT & ASK.
3. **Confirm** — one-button confirmation: "Migrate and clean up all stale artifacts". The user approves before any move executes. Never move files silently.
4. **Execute** — move safe artifacts into the inferred `completed/{slug}_{date}/` (or `active/{slug}_{date}/`) task folder, then delete the now-empty source dir. Print a result summary ("moved X files, deleted Y dirs").

##### Safe-Inference Rules for Pre-Create

- **Safe** — a plan slug exists in `process/general-plans/active/` or `process/features/{feature}/active/` and exactly one matching `{slug}_{date}/` task folder can be inferred → auto-create `completed/{slug}_{date}/` as the destination and migrate the artifact.
- **NOT safe** — no matching plan slug found, ambiguous filenames, multiple task folders match, or conflict with an existing dir → do NOT create any destination folder; show the fallback message below and pause for manual review.

##### Fallback Display Message

When orphaned dirs are detected but no plan slug can be inferred from project files (no safe inference possible), vc-setup displays:

> "Found orphaned dir [path] but cannot safely infer destination task folder. Manual migration required. See reference docs for migration guide."

vc-setup will NOT auto-create ghost folders (false folders) when inference fails. The user must review and manually place the files. Conservative behavior: when in doubt, leave the artifact in place and surface it — never guess a destination.

**Step 1-6 -- Standard merge (after migration):**

1. Create only missing directories
2. Add `_GUIDE.md` to empty directories that lack them (source: `process/_seeds/`)
3. Copy seed files only where the target file does not exist
4. Copy development protocols only where the target file does not exist
5. Retain `.seed` originals alongside populated files (same as Fresh mode step 6)
6. Never overwrite existing files

### Refresh Mode

Update protocols, preserve user content (read from `process/_seeds/`, never modify seeds):

1. Create any missing target-tree working directories (sourcing each `_GUIDE.md` from `process/_seeds/` where applicable):
   - `process/general-plans/active/`, `process/general-plans/completed/`, `process/general-plans/backlog/`
   - `process/features/`
   - `process/context/planning/`
   - `process/context/tests/`
   Skip directories that already exist.
2. Overwrite development protocol files from `process/development-protocols/` (these are managed system files that live in the real directory, not in `_seeds/`)
3. Add missing seed files: for each `.seed` file in `process/_seeds/`, if the corresponding target (same path with the `.seed` extension removed) does NOT exist, copy it with `.seed` removed and `{{project_name}}` replaced with the detected project name. Never overwrite an existing target file.
4. Add missing `_GUIDE.md` files from `process/_seeds/`
5. Update `.seed` companion files to latest versions from `process/_seeds/` (these are structural references, not user content)
6. Preserve all user-created plans, reports, references, and context docs

### Placeholder Reference

The only placeholder used in seed templates is:

| Placeholder | Source |
|-------------|--------|
| `{{project_name}}` | `package.json` name field |

All other content is populated by the STUDY phase using real codebase analysis, not string replacement.

## STUDY Phase

The STUDY phase is the core value of vc-setup v3. It transforms scaffolded seed files into ready-to-use context by actively scanning the codebase.

### Incorporating User Answers

If the user answered project questions (from the ASK step in Flow A, or the PRESENT & ASK step in Flow B), incorporate their answers into context files:

- **Project description**: The user's own words go into the top of all-context.md. This is more valuable than an auto-generated description.
- **Product areas/features**: Cross-reference with automated feature detection. If the user named a feature the scan missed, ask where the code lives. If the scan found a feature the user did not mention, include it but note it was auto-detected.
- **Conventions and workflows**: Add to the Key Patterns and Conventions section. Team knowledge that is invisible in code is exactly what makes context files useful.
- **Team/branching info**: If provided, include in a Workflow or Conventions section.

### Parallel Subagent Delegation Strategy

The STUDY phase is the most resource-intensive part of vc-setup. The executing agent SHOULD spawn parallel subagents to maximize throughput and avoid context window exhaustion.

**Round 0 -- Migration Gap Analysis (only when existing process/ is found):**

Spawn a single subagent that reads all existing `process/` content and produces a gap analysis. Round 1 subagents receive this gap analysis to avoid duplicating existing content.

**Round 1 -- Parallel Research (read-only, no file writes):**

Spawn 4 parallel subagents. Each produces a structured findings report as its output.

| Subagent | Task | Output |
|----------|------|--------|
| A: Architecture and Stack Scanner | Source directory scan, tech stack detection with versions, import alias mapping, env var cataloging, key patterns and conventions | Structured findings: stack details, directory tree, aliases, entry points, patterns |
| B: Test and Quality Scanner | Test runners, config files, test directories, test commands per package/workspace | Structured findings: runners, configs, commands, known quirks |
| C: Context Group Detector | Scan for database, auth, CI/CD, container, UI, workflow signals using the detection table below | List of recommended context groups with evidence for each |
| D: Feature Area Detector | Route groups, package names, README features, doc structure | List of recommended feature folders with evidence for each |

Wait for all Round 1 subagents to complete. Present the combined detection summary to the user. User confirms or adjusts the detected groups and features before Round 2.

**Round 2 -- Parallel Writers (file writes, non-overlapping targets):**

Spawn up to 4 parallel subagents. Each writes to a distinct set of files with no overlap.

| Subagent | Consumes | Writes to |
|----------|----------|-----------|
| E: all-context.md Writer | Findings from A + C + user answers | `process/context/all-context.md` |
| F: all-tests.md Writer | Findings from B | `process/context/tests/all-tests.md` |
| G: vc-generate-context (delegation) | Findings from C (approved-groups list) | DELEGATION: invoke `vc-generate-context` skill in `setup-delegation` mode, passing the approved-groups list from Subagent C. Context-group detection and per-group file authoring is owned by `vc-generate-context` — see `.claude/skills/vc-generate-context/references/generate-context.md` for the detection table and per-mode instructions. |
| H: Feature Folder Scaffolder | Findings from D + user answers | `process/features/{feature}/` dirs + `_GUIDE.md` files |

Wait for all Round 2 subagents to complete. Proceed to VALIDATE.

**For SMALL projects (fewer than 5 source directories, no monorepo):**

Skip parallel delegation. A single agent can handle the entire STUDY phase sequentially, following the same checklists below in order: codebase analysis, context groups, feature areas, then file population.

### Deep Codebase Analysis Checklist

The agent (or Subagent A) must scan and document:

**Source directory scan:**
- List all top-level source directories (src/, lib/, app/, pages/, etc.)
- For monorepos: list all workspace packages with their purpose
- Identify entry points (main files, route handlers, server starts)

**Tech stack specifics:**
- Framework + version (e.g., "Next.js 15 with App Router", not just "Next.js")
- Runtime (Node, Bun, Deno) + version from `package.json` engines or `.node-version`
- Database (ORM + DB type from dependencies + config files)
- UI library + CSS approach from dependencies
- Auth solution from dependencies
- API layer (REST/GraphQL/tRPC) from dependencies + route files

**Import alias mapping:**
- Read `tsconfig.json` / `jsconfig.json` paths field
- Read `vite.config` resolve.alias if present
- Document every alias -> actual path

**Environment variables:**
- Scan `.env.example`, `.env.local.example`, or env schema files (e.g., `env.ts`, `env.mjs`)
- Group by category (auth, database, API keys, URLs)
- List variable names only, never values

**Key patterns:**
- Error handling convention (throws vs result pattern)
- State management (Redux, Zustand, Jotai, context)
- API patterns (REST conventions, middleware chains, RPC)
- Naming conventions (file naming, function naming from existing code)

### Context Group Detection Table

Context Group Detection Table: see `.claude/skills/vc-generate-context/references/generate-context.md` section "Context Group Detection Table" for the canonical detection signals, group names, and creation rules. That file is the single source of truth for group detection — do not duplicate the table here.

### Feature Detection Heuristics

Scan these locations for feature identification:

- Route directories (`app/*`, `pages/*`) -- each major route group is a potential feature
- Package names in monorepos -- each package with business logic is a potential feature
- README sections listing features
- Existing `docs/` or wiki content

**Cross-reference with user answers.** If the user named product areas in the ASK step, those are strong signals for feature folders even if the automated scan would not have created them. Ask the user where the code lives if you cannot find it.

**Feature folder creation threshold:**
- Create a feature folder only when the feature has 3+ source files AND is a distinct product area (not a utility)
- For monorepos: each app or business-logic package is a candidate
- For single apps: each major route group with its own components/API is a candidate

**Each feature folder gets:**
- `active/`, `completed/`, `backlog/` subdirectories (do NOT create `reports/` or `references/` — these are deprecated sibling dirs; new artifacts go inside task folders)
- `_GUIDE.md` explaining the feature scope, key files, and current state

### all-context.md Population Instructions

After scanning (or receiving Round 1 findings), write real content into `all-context.md`. Replace the seed template sections with actual project data:

1. **Title**: Replace `{{project_name}}` with the actual project name
2. **Project description**: Use the user's own words from the ASK step if available. Supplement with what the code scan reveals. The user's description should be prominent, not buried.
3. **Quick Start section**: Keep the generic routing instructions from the template
4. **Current Root Entry Points table**: Populate with actual context files that were created
5. **Current Context Groups table**: Populate with groups created by the context group detection step
6. **Task Routing table**: Fill based on what context groups exist, mapping task types to the relevant entry points
7. **Repository Structure**: Write actual directory tree output (2-3 levels deep, showing key directories and files)
8. **Technology Stack**: Write specific framework names, versions, and combinations discovered during analysis
9. **Key Patterns and Conventions**: Document actual patterns found in the codebase (error handling, state management, API patterns, naming conventions, import aliases). Include conventions the user mentioned in the ASK step.
10. **Environment and Configuration**: List actual env var groups found (names only, never values)
11. **Context Group Lifecycle**: Keep the generic instructions from the template
12. **Scan Metadata** (add at bottom):
    ```
    ## Scan Metadata

    - Generated: {ISO timestamp}
    - HEAD: {git rev-parse HEAD, if available}
    - Mode: {fresh/merge/refresh}
    - Package manager: {detected_pm}
    ```

### all-tests.md Population Instructions

After scanning test setup (or receiving Round 1 findings from Subagent B), write real content into `all-tests.md`:

1. **Title**: Replace `{{project_name}}` with the actual project name
2. **Quick Decision Guide**: Replace placeholder with actual test runner name(s) and when to use each. For monorepos with multiple runners, list each runner with its scope.
3. **Commands section**: List actual test commands per package/workspace in a table or code blocks
4. **Debugging Quick Reference**: Note any test config quirks found (e.g., "uses jsdom environment", "needs .env.test", "requires running database")
5. **Known Gaps**: Leave empty but remove any placeholder comments

### Migration Intelligence (for Merge/Refresh modes)

When existing `process/` content is found, follow these rules:

1. **Read every existing `.md` file under `process/context/`**
2. **For each file**:
   - Parse section headings
   - Identify which sections have real content vs placeholders/TODOs
   - Mark sections that need updating
3. **Compare existing content against fresh scan results**:
   - If existing content is MORE detailed than scan results -> preserve it
   - If existing content is a placeholder or stale -> replace with scanned data
   - If section is missing entirely -> add it with scanned data
4. **For context groups**:
   - Map existing groups to detected groups
   - Preserve groups that exist even if not detected (user may have created them intentionally)
   - Add only MISSING groups
5. **For feature folders**:
   - Preserve all existing feature folders and their contents
   - Add only MISSING feature folders that meet the detection threshold
6. **For all-context.md routing tables**:
   - Merge: add new entries, preserve existing entries
   - Update entries whose paths changed

## VALIDATE Phase

### Directory Existence Checks

Verify all directories from the target tree exist:

```bash
ls -d process/development-protocols/ process/context/ process/context/planning/ process/context/tests/ process/general-plans/active/ process/general-plans/completed/ process/general-plans/backlog/ process/features/
# process/general-plans/reports/ and process/general-plans/references/ are deprecated sibling dirs — do NOT create them for new repos; skip if absent on existing repos
```

### STUDY Output Quality Checks

Verify the STUDY phase produced real content:

1. **No remaining placeholders**: Grep `all-context.md` for `{{` -- only `{{project_name}}` is acceptable (and only if the seed was just created without a project name)
2. **Repository Structure populated**: `all-context.md` contains a code block under "Repository Structure" with actual directory names
3. **Technology Stack populated**: `all-context.md` contains specific framework names and versions under "Technology Stack"
4. **Test commands populated**: `all-tests.md` contains actual test commands (not `{{test_commands}}` or placeholder text)
5. **Context group routing**: Every context group directory under `process/context/` has a corresponding entry in the "Current Context Groups" table in `all-context.md`
6. **Feature folder guides**: Every feature folder under `process/features/` (excluding the root `_GUIDE.md`) has a `_GUIDE.md` file with a real scope description
7. **User input incorporated**: If the user provided project description or conventions in the ASK step, verify they appear in the relevant sections of all-context.md

### Agent Parity Check

Verify that agent names match between Claude and Codex surfaces:

```bash
ls .claude/agents/*.md | sed 's|.claude/agents/||;s|\.md$||' | sort > /tmp/claude-agents.txt
ls .codex/agents/*.toml | sed 's|.codex/agents/||;s|\.toml$||' | sort > /tmp/codex-agents.txt
diff /tmp/claude-agents.txt /tmp/codex-agents.txt
```

### Skill Discovery Check

Verify the symlink resolves:

```bash
ls -la .agents/skills
ls .agents/skills/vc-setup/SKILL.md
```

### Post-Setup Summary

Present a final summary to the user:

```
Setup complete!

What was created/updated:
- [list of directories created]
- [list of files populated with real content]
- [list of context groups created]
- [list of feature folders created]

What was preserved (existing project only):
- [list of files/dirs that were kept as-is per user approval]

Recommended next steps:
1. Review process/context/all-context.md and refine any sections that need more detail
2. Review detected context groups and feature folders -- add or remove as needed
3. Run the vc-audit-context skill to validate context discovery wiring
4. Start using the harness: describe a feature request to trigger the RIPER-5 workflow
```
