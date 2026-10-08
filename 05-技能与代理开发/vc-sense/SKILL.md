---
name: vc-sense
description: "Use when the user explicitly asks to apply Vibe Code Common Sense, create project rails, scaffold AI-agent project guidance, build a project blueprint, create AGENTS.md instructions, or set up docs/PRODUCT_TIMELINE.md progress tracking for an AI-built software project. Do not use for ordinary feature work in an already-tracked project unless the user asks to update the project guidance system."
disable-model-invocation: true
---

# Vibe Code Common Sense

This Claude Code plugin skill uses the shared project setup system in the plugin
root.

Read the shared instructions first:

- `${CLAUDE_PLUGIN_ROOT}/SKILL.md`

Load only the needed template:

- `${CLAUDE_PLUGIN_ROOT}/references/readme-template.md`: use for root `README.md`.
- `${CLAUDE_PLUGIN_ROOT}/references/project-blueprint-template.md`: use for `docs/PROJECT_BLUEPRINT.md`.
- `${CLAUDE_PLUGIN_ROOT}/references/product-timeline-template.md`: use for `docs/PRODUCT_TIMELINE.md`.
- `${CLAUDE_PLUGIN_ROOT}/references/agents-template.md`: use for root `AGENTS.md`.
- `${CLAUDE_PLUGIN_ROOT}/references/roadmap-template.md`: use for `docs/ROADMAP.md`.
- `${CLAUDE_PLUGIN_ROOT}/references/architecture-template.md`: use for `docs/ARCHITECTURE.md`.
- `${CLAUDE_PLUGIN_ROOT}/references/decisions-template.md`: use for `docs/DECISIONS.md`.
- `${CLAUDE_PLUGIN_ROOT}/references/pr-workflow.md`: use for `docs/DEVELOPMENT_CYCLE.md` or PR rules.

Operate from the same core rule:

> Treat every AI coding agent as dumb until the project proves otherwise.

Create project rails before product features, make `docs/PRODUCT_TIMELINE.md`
the source of truth, and require every future PR to update that tracker.
