# Vibe Code Common Sense

Vibe Code Common Sense is a project setup system for responsible AI coding.

It starts from one blunt rule:

```text
Treat every AI coding agent as dumb until the project proves otherwise.
```

Agents are useful, but a new project needs rails before velocity: a blueprint,
small PR-sized tasks, verification rules, decision records, and a progress
tracker that survives across sessions.

## Why This Exists

At the start of a project, you are vulnerable:

- the idea is still loose;
- the architecture is undecided;
- the roadmap is unclear;
- the agent does not know your standards;
- the agent can confidently generate AI slop.

Vibe Code Common Sense turns that first hour into a repeatable setup workflow.

## What It Creates

```text
README.md
AGENTS.md
docs/PROJECT_BLUEPRINT.md
docs/PRODUCT_TIMELINE.md
docs/ROADMAP.md
docs/ARCHITECTURE.md
docs/DEVELOPMENT_CYCLE.md
docs/DECISIONS.md
```

The most important file is `docs/PRODUCT_TIMELINE.md`. It becomes the source of
truth for what has shipped and what comes next.

## How Work Moves

```text
idea -> blueprint -> product timeline -> small PR -> verification -> tracker update
```

Every future PR should:

1. read `docs/PRODUCT_TIMELINE.md`;
2. build the first unchecked item;
3. open one focused PR;
4. mark the item `[x]` with the PR number;
5. show the next unchecked item.

## Start Here

1. [What It Does](start/what-it-does.md)
2. [Install](start/install.md)
3. [Your First Project](start/your-first-project.md)
4. [Daily Workflow](start/daily-workflow.md)
