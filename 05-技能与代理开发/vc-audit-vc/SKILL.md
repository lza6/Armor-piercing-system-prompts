---
name: vc-audit-vc
description: >-
  Audit agent harness health: Claude/Codex agent parity, skill registry
  consistency, README.md sync, and protocol file wiring. Use when agents,
  skills, README.md, or development-protocol files move, split, or drift.
trigger_keywords: harness audit, agent parity, skill audit, guide sync
layer: contract
---

# Audit VC (Version Control Harness Health)

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Use this skill to verify that the agent harness layer is internally consistent
and correctly wired across Claude, Codex, README.md, and protocol files.

For context routing, grouping, and discoverability audits, use the `audit-context` skill instead.

## Workflow

1. Run the Claude/Codex agent parity validator:
   ```bash
   node .claude/skills/vc-audit-vc/scripts/validate-agent-parity.mjs
   ```
2. Run the shared skill discovery validator:
   ```bash
   node .claude/skills/vc-audit-vc/scripts/validate-skills.mjs
   ```
3. Run the README.md sync validator:
   ```bash
   node .claude/skills/vc-audit-vc/scripts/validate-guide-sync.mjs
   ```
4. Run the protocol wiring validator:
   ```bash
   node .claude/skills/vc-audit-vc/scripts/validate-protocol-wiring.mjs
   ```
5. Run the seed/scaffold consistency validator:
   ```bash
   node .claude/skills/vc-audit-vc/scripts/validate-seeds.mjs
   ```
6. Run the kit portability validator:
   ```bash
   node .claude/skills/vc-audit-vc/scripts/validate-kit-portability.mjs
   ```
7. Run the skill invocation wiring validator:
   ```bash
   node .claude/skills/vc-audit-vc/scripts/validate-skill-invocation-wiring.mjs
   ```
8. Run the agent frontmatter validator:
   ```bash
   node .claude/skills/vc-audit-vc/scripts/validate-agent-frontmatter.mjs
   ```
9. If any script reports failures, inspect the referenced files and patch the smallest
   relevant surface.
10. Re-run the failed validators until they pass.

## Rules

- Treat `.claude/agents/` as canonical for agent definitions; `.codex/agents/` mirrors them.
- Treat `.claude/skills/` as canonical for skills; `.agents/skills/` is the Codex discovery symlink.
- When updating agents, mirror Claude markdown and Codex TOML surfaces together.
- Treat `process/_seeds/` as an optional legacy scaffold surface in the live repo. Its absence is a warning-only audit result unless the user is explicitly auditing export-kit scaffolding.
- Treat validator warnings as audit findings unless the user asks for a strict cleanup.
- For context routing and discoverability audits, delegate to `audit-context`.
