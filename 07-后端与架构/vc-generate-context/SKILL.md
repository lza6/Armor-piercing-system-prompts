---
name: vc-generate-context
description: Generate or update the project's authoritative repository context at process/context/all-context.md. Use when repo context is missing, stale, or contradicted by code.
trigger_keywords: generate context, update context, refresh context, missing context
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "1.0.0"
---

# Generate Context

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Use this skill to maintain `process/context/all-context.md`, the broad portable project knowledge layer shared by Codex and Claude. Use `process/context/all-context.md` as the context router before reading grouped docs.

Optional input: a package, app, feature, context group, or architectural area to refresh first.

## Workflow

1. Read `references/generate-context.md` for the full context contract.
2. Determine mode:
   - Full scan when `process/context/all-context.md` is missing.
   - Delta update when it exists.
3. Read `process/context/all-context.md` when present to identify relevant grouped context files.
4. Inspect current repo state, active plans, feature folders, package scripts, tooling, important architecture files, and relevant `process/context/**/*.md` docs.
5. Produce `process/context/all-context.md` PLUS `process/context/{group}/all-{group}.md` for each detected or approved group (see Invocation Modes below).
6. Include scan timestamp, repo HEAD if available, changes since last update, open questions, and source references.
7. Validate the generated context:
   ```bash
   node .claude/skills/vc-generate-context/scripts/validate-all-context.mjs
   ```
8. If routing or grouped context changed, also run:
   ```bash
   node .claude/skills/vc-audit-context/scripts/validate-context-discovery.mjs
   ```

## Rules

- Treat `process/context/` as durable cross-agent knowledge.
- Treat `process/context/all-context.md` as the durable routing protocol; do not replace it with generated prose.
- Do not store agent-specific mechanics here unless they affect project workflow.
- Do not rewrite grouped context docs; if they are stale or mis-grouped, flag `audit-context`.
- Prefer concise, factual, path-specific documentation.
- Use `pnpm` terminology for package management.
- Treat validation failures as blockers before presenting context as refreshed.

## Invocation Modes

| Mode | Trigger | Group-detection | Output |
|---|---|---|---|
| `setup-delegation` | Called by vc-setup passing an approved-groups list | Skip re-detection; use the list provided | `all-context.md` + `all-{group}.md` for each approved group |
| `standalone-full` | Direct invocation with no groups list | Self-detect groups via the detection table in `references/generate-context.md` | `all-context.md` + `all-{group}.md` for all detected groups |
| `delta` | Invoked when context already exists; update context | Self-detect; create MISSING groups only; warn on unrecognized; never delete | `all-context.md` updated + any new `all-{group}.md` files created |

See `references/generate-context.md` for per-mode instructions, the Context Group Detection Table, and delta-mode group-creation rules.
