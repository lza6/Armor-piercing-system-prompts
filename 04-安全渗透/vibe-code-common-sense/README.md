
![Vibe Code Common Sense hero illustration](./assets/vibe-code-common-sense-hero.png)

# Vibe Code Common Sense

My biggest lesson from Agentic Coding (read vibe coding) is :

> Treat every AI agent as dumb until the project proves otherwise.

Agents can move fast, but at the start of a project you are vulnerable. 

The idea is still loose, the architecture is undecided, the roadmap is unclear, and the
agent will happily generate confident slop if you do not give it rails.

Vibe Code Common Sense is a reusable coding agent skill (Codex skill/ Claude Code Plugin) for guiding AI-built projects
responsibly. 

It helps turn a raw idea into a professional, trackable project with a blueprint, roadmap, PR workflow, agent instructions, and a living
progress tracker.


## What It Creates

- `README.md`
- `AGENTS.md`
- `docs/PROJECT_BLUEPRINT.md`
- `docs/PRODUCT_TIMELINE.md`
- `docs/ROADMAP.md`
- `docs/ARCHITECTURE.md`
- `docs/DEVELOPMENT_CYCLE.md`
- decision records

## Core Rule

Every PR updates `docs/PRODUCT_TIMELINE.md`.

That file is the project tracker. Completed work is marked `[x]` with the PR
number, and the next unchecked item tells the agent what to build next.

## How To Use It

Use Vibe Code Common Sense before the agent writes product code.

The first run should create rails:

- project explanation;
- agent instructions;
- roadmap;
- architecture notes;
- development workflow;
- PR-sized progress tracker.

After that, every build should come from the first unchecked item in
`docs/PRODUCT_TIMELINE.md`.

## Codex Install

In Codex, ask:

```text
Install the skill from GitHub:
https://github.com/Njengah/vibe-code-common-sense
```

Then open a new project folder and say:

```text
Use $vibe-code-common-sense.

I have a new project idea. Do not build product features yet.
First create the professional project rails and progress tracker.

My idea is:
[paste your idea here]
```

## Claude Code Install

This repo is also packaged as a Claude Code plugin with
`.claude-plugin/plugin.json`.

In Claude Code, add the marketplace:

```text
/plugin marketplace add Njengah/vibe-code-common-sense
```

Then install the plugin:

```text
/plugin install vc-sense@vibe-code-common-sense
```

Reload plugins so the new skill is available in the current session:

```text
/reload-plugins
```

Then run it in a new project:

```text
/vc-sense:vc-sense

I have a new project idea. Do not build product features yet.
First create the professional project rails and progress tracker.

My idea is:
[paste your idea here]
```

If your Claude Code setup uses local plugin directories instead, clone this repo
and run Claude Code with the plugin directory:

```bash
git clone https://github.com/Njengah/vibe-code-common-sense.git
claude --plugin-dir ./vibe-code-common-sense
```

## Any Agent

If your agent does not support skills or plugins, paste this instruction:

```text
Read and follow this project setup system:
https://github.com/Njengah/vibe-code-common-sense/blob/main/SKILL.md

Do not build product features yet. First create the project rails:
README.md, AGENTS.md, docs/PROJECT_BLUEPRINT.md, docs/PRODUCT_TIMELINE.md,
docs/ROADMAP.md, docs/ARCHITECTURE.md, docs/DEVELOPMENT_CYCLE.md, and
docs/DECISIONS.md.
```

## What Good Output Looks Like

After the first run, the project should have:

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

The most important file is `docs/PRODUCT_TIMELINE.md`.

Every future PR should:

1. read `docs/PRODUCT_TIMELINE.md`;
2. build the first unchecked item;
3. open a focused PR;
4. mark that item `[x]` with the PR number;
5. show the next unchecked item.

## Compatibility

- Codex reads `SKILL.md`, `references/`, and optional `agents/openai.yaml`.
- Claude Code reads the plugin manifest in `.claude-plugin/plugin.json`, plus
  the short `/vc-sense:vc-sense` skill command.
- Other agents can use the system manually by reading `SKILL.md`.

The Claude plugin wrapper uses `disable-model-invocation: true`, so Claude Code
users invoke it explicitly. The shared root `SKILL.md` stays Codex-compatible and
does not include Claude-only frontmatter.

---
## Documentation

Docs: [https://njengah.github.io/vibe-code-common-sense/](https://njengah.github.io/vibe-code-common-sense/)

--- 
