# Your First Project

Use this when you have a raw idea and want the agent to set up the project
properly before coding.

## Step 1: Open The Project

```bash
cd path/to/your-new-project
```

## Step 2: Invoke The System

Codex:

```text
Use $vibe-code-common-sense.
```

Claude Code:

```text
/vc-sense:vc-sense
```

## Step 3: Paste The Idea

```text
I have a new project idea. Do not build product features yet.

First create:
- README.md
- AGENTS.md
- docs/PROJECT_BLUEPRINT.md
- docs/PRODUCT_TIMELINE.md
- docs/ROADMAP.md
- docs/ARCHITECTURE.md
- docs/DEVELOPMENT_CYCLE.md
- docs/DECISIONS.md

Turn my idea into phases and small PR-sized checklist items.
Make docs/PRODUCT_TIMELINE.md the source of truth.

My idea is:
[paste idea here]
```

## Step 4: Review The Rails

Before building features, read:

- `docs/PROJECT_BLUEPRINT.md`
- `docs/PRODUCT_TIMELINE.md`
- `AGENTS.md`

If those are wrong, fix them first.
