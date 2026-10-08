---
name: vc-plan-discovery
description: "Discover related plans for the current task: same feature folder full depth, other features active-only, general-plans active. Like vc-context-discovery but for plan artifacts."
argument-hint: "[feature folder name or task description]"
trigger_keywords: related plans, what was tried, plan history, feature backlog, plan discovery
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "1.1.0"
---

# vc-plan-discovery

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

## Invocation

**Primary method** — run the auto-discovery script. It applies the Scope Rules below
(same-feature full depth; other features active-only; general-plans active always) and
extracts ONLY the leading YAML frontmatter block of each `.md` file (no whole-file reads):

```bash
node .claude/skills/vc-plan-discovery/scripts/discover-plans.mjs [--feature <name>] [--json]
```

It groups output into Active Plans / Backlog / Completed / Reports / References (each line:
`- [name]: description (path)`) and prints a trailing `Found N active, N backlog, N
completed, N reports, N references` line. It never throws on a missing root and exits 0
unless given a bad flag. Use `--json` for a machine-readable object. Prefer this over the
manual scan below — it is deterministic and avoids loading huge files into context.

Per **task-folder artefact colocation**, the script scans one level into each
`{slug}_{date}/` task folder and surfaces co-located `_PLAN_`/`_REPORT_`/`_REF_` artefacts;
the sibling `reports/`/`references/` dirs are deprecated and only hold legacy artefacts.

The manual scan in **Scope Rules** below is the FALLBACK for when the script fails.

## Purpose

Surface all plans relevant to the current task so agents have full plan context — what was tried, what is in progress, what is deferred, and what reports and references exist — before doing any phase work.

## Scope Rules

- **Same feature folder** (from task context or argument): read ALL of `active/`, `backlog/`, `completed/`, plus any legacy sibling `reports/`, `references/` — surface every file with frontmatter
- **Other feature folders**: read `active/` only — surface plans whose frontmatter `description` or `feature` field matches the task domain
- **`general-plans/active/`**: always scan
- **`general-plans/completed/`, legacy `reports/`, `references/`**: scan only when same-feature folder is not identified

Per **task-folder artefact colocation**, expect every current artefact (plan, spec, reports, references) INSIDE its `{slug}_{date}/` task folder — scan one level into each task folder. The sibling `reports/`/`references/` dirs are deprecated and only hold legacy artefacts.

## Frontmatter Reading

Read frontmatter fields: `name`, `description`, `type`, `feature`, `phase`

Route by `description` field for relevance matching (same approach as vc-context-discovery).

Skip files without frontmatter or with incomplete frontmatter — log as "no frontmatter, skipped".

Output: grouped list by folder — Active Plans / Backlog / Completed / Reports / References — with name + description per file.

## When To Invoke

- First action alongside `vc-context-discovery` at the start of every loop step (research / validate / execute / update-process)
- Any time an agent needs to know: what plans exist for this feature, what was tried before, what is deferred, what references exist

## Output Format

```
### Active Plans
- [name]: description (path)

### Backlog
- [name]: description (path)

### Completed
- [name]: description (path)

### Reports
- [name]: description (path)

### References
- [name]: description (path)

Found N active, N backlog, N completed, N reports, N references
```
