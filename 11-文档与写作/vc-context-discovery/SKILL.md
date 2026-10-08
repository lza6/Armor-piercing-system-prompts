---
name: vc-context-discovery
description: "Discover and load all relevant context for the current task. Lists feature group nested files with full paths, loads process/context/ files by domain routing. Called at the start of every agent session."
argument-hint: "[task domain or feature name]"
trigger_keywords: context discovery, load context, feature folder files, context routing discovery
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "1.1.0"
---

# vc-context-discovery

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Discover and load all relevant context for the current task. This skill lists feature group nested files with full paths and loads `process/context/` files by domain routing table. It is the canonical context-loading entrypoint for every agent session.

## When To Invoke

At the start of every agent session (research, innovate, plan, validate, execute, update-process, fast-mode). Also at the start of any skill that needs repo context before operating.

## Frontmatter Schemas

Document the canonical schemas for three file types used across the repo. These schemas enable frontmatter-aware routing and filtering.

**Context file frontmatter schema:**
```yaml
name: context:{slug}
description: "one-line scope — used by vc-context-discovery for routing"
keywords: comma, separated, task, vocabulary, terms   # drives grep-first keyword routing
related: [context:{other-slug}, context:{another-slug}]  # sibling/cross-group links (optional)
date: dd-mm-yy
```

- `keywords` — strongly recommended, non-empty. Comma-separated task-vocabulary terms an
  agent would use to describe a task this doc serves (e.g. `session, token, refresh, jwt,
  login`). This is the match surface for `discover-context.mjs --match`; weak/absent
  keywords are why a relevant doc never gets routed to. Lint WARNS when empty (so existing
  projects don't break on sync) — backfill at UPDATE-PROCESS.
- `related` — OPTIONAL list of `context:{slug}` values for sibling docs that are usually
  needed together (the markdown-native equivalent of cross-links). Every slug listed MUST
  resolve to a real `context:` doc; dangling links fail lint. Discovery follows these after
  the primary match so a task touching two domains loads both.

**Plan file frontmatter schema:**
```yaml
name: plan:{slug}
description: "one-line scope and feature"
date: dd-mm-yy
feature: {feature-folder-name}
phase: "{phase-id}"  # optional, for phase program plans only
```

**Report file frontmatter schema:**
```yaml
name: report:{slug}
description: "one-line scope"
date: dd-mm-yy
metadata:
  node_type: memory
  type: report
  feature: {feature-folder-name}
  phase: {phase-id}
```

## Invocation

**Primary method** — run the auto-discovery script. It lists all nested files under
`process/context/`, `process/development-protocols/`, `process/general-plans/active/`,
and (with `--feature`) the feature folder, extracting ONLY the leading YAML frontmatter
block of each `.md` file (no whole-file reads):

```bash
node .claude/skills/vc-context-discovery/scripts/discover-context.mjs [--feature <name>] [--json]
```

The script groups output into: context files with frontmatter, protocol files, feature
files by subfolder, active general plans, and files-without-frontmatter (path only). It
never throws on a missing root and exits 0 unless given a bad flag. Use `--json` for a
machine-readable object. Prefer this over manually reading each file — it is deterministic
and avoids loading huge files into context.

**Keyword-first routing (deterministic, not judgment).** When the task vocabulary does not
obviously map to a routing-table row, do not guess — let the index do the matching:

```bash
node .claude/skills/vc-context-discovery/scripts/discover-context.mjs --match "update the user ORM model"
```

This tokenizes the task and ranks context docs by overlap with their frontmatter `keywords`,
then appends any `related:` siblings of the top hits. Read the ranked docs in order. This is
the fallback that fixes "the right doc existed but the agent walked past it."

**Routing-table generation (drift-proof index).** The "Current Root Entry Points" and
"Current Context Groups" tables in `all-context.md` are GENERATED from frontmatter, not
hand-authored — they live between `<!-- GENERATED:routing -->` / `<!-- /GENERATED:routing -->`
markers. Rebuild them after any context-org change:

```bash
node .claude/skills/vc-context-discovery/scripts/discover-context.mjs --emit-routing   # rewrites the block
node .claude/skills/vc-context-discovery/scripts/discover-context.mjs --check-routing  # lint: block in sync?
```

The hand-authored Task Routing Table (task-type → file) stays editorial and is NOT generated.

Per **task-folder artefact colocation**, the script surfaces each task's plan, spec,
reports, and references INSIDE its own `{slug}_{date}/` task folder; the sibling
`reports/`/`references/` dirs are deprecated and only hold legacy artefacts.

## Workflow (manual FALLBACK)

Use these steps only if the script above fails or is unavailable.

**Step 1.** Run `find process/context/ -type f | sort` and record all available context files.

**Step 2.** Run `find process/development-protocols/ -type f | sort` and record all protocol files.

**Step 3.** Read `process/context/all-context.md` to get the routing table and current feature list.

**Step 4.** If a feature name was provided as the argument, run `find process/features/{feature}/ -type f | sort` to list ALL artifacts across all subfolders (`active/`, `completed/`, `backlog/`, plus any legacy `reports/`, `references/`). Surface full file paths — not just folder names. Per **task-folder artefact colocation**, expect each task's plan, spec, reports, and references INSIDE its own `{slug}_{date}/` task folder; the sibling `reports/`/`references/` dirs are deprecated and only hold legacy artefacts.

**Step 5.** If no feature name was provided, run `find process/general-plans/active/ -type f | sort` to surface any active plans relevant to the current task. Note: plan files are inside `{slug}_{date}/` task subfolders — look one level deep for `*_PLAN_*.md` files.

**Step 6.** From the routing table in `all-context.md`, identify the context group files relevant to the current task domain (e.g. `tests`, `container`, `infra`, `skills`, `uxui`, `workflows`). Do NOT read every context file — only the ones the routing table says apply to this domain. If no row obviously matches, run `discover-context.mjs --match "<task>"` and use its ranked keyword hits instead of guessing.

**Step 7.** Read the domain context file(s) identified in step 6. Each `all-{group}.md` is a router — after reading it, follow its routing table to load the deeper domain file(s) for the task. Then load any `related:` siblings of the docs you read — a task spanning two domains (e.g. auth + tests) needs both, and `related` is how the cross-domain link is declared.

**Step 8.** Frontmatter extraction: For each file in the discovered set, if the file has YAML frontmatter (a `---` block at top), extract the `name`, `description`, and `date` fields. Surface in output as `"[path] (name: X — description: Y)"`. Files without frontmatter: surface path only (no error, do not skip them).

**Step 9.** Report: full file listing for the feature folder (if applicable), active context files loaded, and any open gaps in context that would block the task.

## Output Format

```
Context file paths with frontmatter:
  process/context/skills/skill-apps.md (context:skill-apps — Skill app runtime, vite architecture, ctx-gateway, deployment)
  process/context/tests/all-tests.md (context:all-tests — Test routing, runner split, debugging procedures)

Plan files with frontmatter:
  process/features/{feature}/completed/{slug}_{date}/{slug}_PLAN_{date}.md (plan:{plan-name} — short description...)

Files without frontmatter (path only):
  process/context/ui/design.md
```

## Context Envelope

At session start, every inner-loop agent (research / plan / execute / update-process) emits a
**Context Envelope** — a 10-field table capturing the orientation an agent needs to act. All 10
fields are required (use best-effort values; `TBD — [reason]` when not yet determinable). The fields
MUST appear in the EXACT canonical C-2 order below — identical order in this SKILL and in all four
inner-loop agents:

| # | Field | Value |
|---|---|---|
| 1 | `feature` | feature folder name (or `TBD`) |
| 2 | `phase` | current RIPER phase (`RESEARCH` / `INNOVATE` / `PLAN` / `PVL` / `EXECUTE` / `EVL` / `UPDATE-PROCESS`) |
| 3 | `session-goal` | one-line goal from the `/goal` block |
| 4 | `branch` | current git branch |
| 5 | `worktree` | worktree path (or `main`) |
| 6 | `context-group` | relevant `process/context/` group (or `none`) |
| 7 | `blast-radius-packages` | packages/paths in scope (comma-separated or `TBD`) |
| 8 | `active-plan` | selected plan file path (or `none`) |
| 9 | `test-runner` | test runner(s); multi-runner uses pipe-delimited DISPLAY format `bun test \| vitest` |
| 10 | `validate-contract` | validate-contract path (or `none`) |

**Canonical order (memorize):** `feature → phase → session-goal → branch → worktree → context-group
→ blast-radius-packages → active-plan → test-runner → validate-contract`.

**`test-runner` multi-runner rule:** the pipe-delimited `bun test | vitest` value is a DISPLAY
convention only. The phase-loop workflow template
(`.claude/skills/vc-generate-phase-program/templates/phase-loop-workflow-template.js`) expands it into
SEQUENTIAL test steps (`bun test` THEN `vitest`) — a literal `bun test | vitest` shell pipe is NEVER
emitted or run. See `03-session-start.md` for the matching field table.

## Frontmatter-Aware Routing

After collecting file paths, read YAML frontmatter from each file where present.

Surface the following fields alongside path: `name`, `description`, `keywords`, `related`, `date`, `type`, `feature`, `phase`.

Use the `description` and `keywords` fields for routing decisions instead of filename inference.

Group plan files by their `feature` field value.

Filter and sort by `type` field (`context` / `plan` / `report` / `references`).

Discover all nested files under feature group subdirs: `active/`, `completed/`, `backlog/`, `reports/`, `references/`.

Files without frontmatter: surface path only (no error), do not skip them.

## Important Rules

- This skill is **INDEPENDENT** — it does not invoke other skills.
- Read `all-context.md` as a router, then load the deeper file(s). Do not treat `all-context.md` as sufficient on its own.
- Always produce the full `find` output, not a summary. The exact file paths are the output.
- Never hardcode context file paths — always discover via the `find` command and the routing table in `all-context.md`.
