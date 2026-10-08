---
name: vc-audit-context
description: Audit project context routing, shared-skill discoverability, and Claude/Codex wiring. Use when context docs or skill surfaces move, split, or drift.
trigger_keywords: audit context, context gaps, context routing audit, discoverability
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "1.0.0"
---

# Audit Context

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Use this skill to verify that the project's durable context layer is discoverable and organized.

Optional input: a context group, agent, skill, or folder scope to prioritize during the audit.

## Workflow

0. Run `find process/context/ -type f | sort` to get the full file listing before routing.
   This ensures no context file is silently skipped when the router is incomplete or drifted.
1. Read `process/context/all-context.md` for the context routing protocol.
2. Read `references/audit-context.md` for the full audit process.
3. Run the context discovery validator:
   ```bash
   node .claude/skills/vc-audit-context/scripts/validate-context-discovery.mjs
   ```
3a. Run the protocol discovery frontmatter validator (enforces discovery frontmatter on every
   `process/development-protocols/**/*.md`, recursive incl. `vc-system-behavior/`; `note.md` is the
   only intentional exclusion):
   ```bash
   node .claude/skills/vc-audit-context/scripts/validate-protocol-discovery.mjs
   ```
4. Run the shared skill routing coverage validator:
   ```bash
   node .claude/skills/vc-audit-context/scripts/validate-skill-routing.mjs
   ```
5. Run the skill cross-reference validator:
   ```bash
   node .claude/skills/vc-audit-context/scripts/validate-skill-cross-refs.mjs
   ```
6. Run the skill dependency/confusable analysis:
   ```bash
   node .claude/skills/vc-audit-context/scripts/validate-skill-dependencies.mjs
   node .claude/skills/vc-audit-context/scripts/validate-confusable-skills.mjs
   ```
7. Regenerate or check the machine-readable skill catalog:
   ```bash
   node .claude/skills/vc-audit-context/scripts/generate-skills-catalog.mjs --write
   node .claude/skills/vc-audit-context/scripts/generate-skills-catalog.mjs --check
   ```
8. Validate that every SKILL.md carries `trigger_keywords` + a valid `layer`
   (`contract`|`helper`) and that the catalog is in sync:
   ```bash
   node .claude/skills/vc-audit-context/scripts/validate-skill-keywords.mjs
   ```
9. If any script reports failures, inspect the referenced files and patch the smallest
   relevant surface.
10. Re-run the failed validators until they pass.

For agent/skill harness validation (agent parity, skill frontmatter, README.md sync, protocol wiring), use the `audit-vc` skill.

## Context Bootstrap (when process/context/ doesn't exist or needs full init)

Use when initializing a new project's context layer from scratch:

1. Run `vc-scout` in parallel across major source directories (skip `.git`, `node_modules`, `.claude`, caches) to gather codebase summaries.
2. Create `process/context/all-context.md` (routing table, architecture, conventions) and group `all-{group}.md` entrypoints for any durable domains identified.
3. **Parallel reader strategy for existing context files** — before updating, spawn subagents proportional to file count: 1-3 files read directly; 4-6 files use 2-3 reader agents; 7+ files use 4-5 reader agents (max 5), distributing by LOC.
4. After generating or updating context files, run `find process/context -name '*.md' -print0 | xargs -0 wc -l | sort -rn` — files over 800 LOC should be split into a context group or the user asked.
5. Finish by running the discovery validator (step 3 above) before declaring done.

## Rules

- Treat `.claude/skills/` as canonical; `.agents/skills/` is the Codex discovery symlink.
- Treat `.claude/skills/vc-audit-context/references/skill-routing-policy.json` as the explicit allowlist for intentionally non-routed shared skills.
- Do not move large context files without updating `process/context/all-context.md`.
- Do not delete compatibility wrappers unless no current reference points to them.
- Keep context groups durable-domain based, not one group per temporary feature.
- When updating agents, mirror Claude markdown and Codex TOML surfaces together.
- Treat validator warnings as audit findings unless the user asks for a strict cleanup.
- Prefer validator-backed routing truth over adding more soft prose.
- Treat process/context/generated-skills-catalog.json as the machine-readable catalog owned by `audit-context`.
