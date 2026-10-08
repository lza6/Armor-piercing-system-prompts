---
name: find-skills
description: |
  Run this BEFORE any package install (pip / npm / apt / brew / cargo / gem / go install) you would otherwise execute via the exec tool — including when the user asks for a deliverable that needs third-party libraries (PDF generation, image processing, scrapers, dashboards, charts, OCR, audio/video, …). It searches the open agent-skill ecosystem (skills.sh) for an installable skill that already does the job; only fall back to direct package install if find-skills returns no credible match. Also covers explicit "find a skill for X" / "how do I do X" / "is there a skill that can..." requests, and any time the user expresses interest in extending agent capabilities.
metadata:
  fastclaw:
    requires:
      bins: [npx]
---

# Find Skills

This skill helps you discover and install skills from the open agent skills ecosystem.

## When to Use This Skill

Use this skill when the user:

- Asks "how do I do X" where X might be a common task with an existing skill
- Says "find a skill for X" or "is there a skill for X"
- Asks "can you do X" where X is a specialized capability
- Expresses interest in extending agent capabilities
- Wants to search for tools, templates, or workflows
- Mentions they wish they had help with a specific domain (design, testing, deployment, etc.)

## What is the Skills CLI?

The Skills CLI (`npx skills`) is the package manager for the open agent skills ecosystem. Skills are modular packages that extend agent capabilities with specialized knowledge, workflows, and tools.

**Key commands:**

- `npx skills find [query]` - Search for skills interactively or by keyword
- `npx skills add <package>` - Install a skill from GitHub or other sources
- `npx skills check` - Check for skill updates
- `npx skills update` - Update all installed skills

**Browse skills at:** https://skills.sh/

## How to Help Users Find Skills

### Step 1: Understand What They Need

When a user asks for help with something, identify:

1. The domain (e.g., React, testing, design, deployment)
2. The specific task (e.g., writing tests, creating animations, reviewing PRs)
3. Whether this is a common enough task that a skill likely exists

### Step 2: Check the Leaderboard First

Before running a CLI search, check the [skills.sh leaderboard](https://skills.sh/) to see if a well-known skill already exists for the domain. The leaderboard ranks skills by total installs, surfacing the most popular and battle-tested options.

For example, top skills for web development include:
- `vercel-labs/agent-skills` — React, Next.js, web design (100K+ installs each)
- `anthropics/skills` — Frontend design, document processing (100K+ installs)

### Step 3: Search for Skills

If the leaderboard doesn't cover the user's need, run the find command. The CLI emits ANSI colour codes and a banner that obscure the actual results — strip them with `sed` so the output is parseable:

```bash
npx skills find <query> 2>&1 | sed -E 's/\x1b\[[0-9;]*[a-zA-Z]//g'
```

You'll get clean lines like:

```
owner/repo@skill-name 1234 installs
└ https://skills.sh/owner/repo/skill-name
```

Multi-word queries are accepted as positional args (e.g. `npx skills find pdf resume`). Don't pipe through `head -N` — full output is small (5-10 results) and truncating risks dropping the best match. If you need to limit results, prefer narrowing the query.

For example:

- User asks "how do I make my React app faster?" → `npx skills find react performance | sed -E 's/\x1b\[[0-9;]*[a-zA-Z]//g'`
- User asks "create a PDF resume" → `npx skills find pdf resume | sed -E 's/\x1b\[[0-9;]*[a-zA-Z]//g'`
- User asks "make a slide deck" → `npx skills find pptx | sed -E 's/\x1b\[[0-9;]*[a-zA-Z]//g'`

### Step 4: Pick a Skill to Install

**Default bias**: an installable skill is almost always better than reimplementing the workflow with raw `pip install <lib>` + your own glue code. The skill author already wrote the SKILL.md prompt, the error handling, and the LLM-friendly arg shape — you'd be reinventing all of that from scratch in the chat. So the bar for "good enough to install" is low; the bar for "skip and reinvent" is high.

Pick the top result that is:

- **On-topic** for what the user actually asked (skim the SKILL.md first line; reject obvious mismatches like a "pptx" skill when the user asked for "pdf").
- **Not obviously abandoned** (the URL resolves; the SKILL.md isn't a stub).
- **Not near-zero installs** (a brand-new skill with 0-5 installs and an unknown author is the only case worth declining — pick the next result down).

Install counts in the high tens / low hundreds are FINE. skills.sh is a young ecosystem; "100+ installs" already means real users have used it. Don't insist on 1K+ — that filters out 95% of the catalogue and leaves you reinventing wheels for no reason.

Official sources (`vercel-labs`, `anthropics`, `microsoft`, `github`) are nice signal when they appear, but plenty of useful community skills come from individual authors. Don't auto-reject based on author obscurity alone.

### Step 5: Tell the User What You're Installing, Then Install It

Surface the chosen skill in one short line so the user sees what just got pulled in, then install it via Step 6 in the SAME response — don't ask for permission first. The user asked for the deliverable, not for an interview about which skill to pick.

Example phrasings (both popular and niche):

```
Pulling in `anthropics/skills@pptx` (93.8K installs) to handle this — it's
purpose-built for PowerPoint generation. Installing now…
```

```
Going with `marswangyang/roger@resume-latex-pdf-generator` (20 installs) —
it's the closest match for "PDF resume" on skills.sh. Installing now…
```

The 20-install case is FINE — it's a niche request, the skill is on-topic, that's all that matters. Don't preemptively apologise for low install counts; they're load-bearing only if the skill is also obviously abandoned or off-topic.

### Step 6: Install AND Use the Skill

Install with `-g -y` — the FastClaw sandbox bind-mounts the global install location to the chatter's host skill bucket, so a `-g` install lands in `~/.fastclaw/users/<uid>/skills/<name>/` on host and is visible to the next chat turn.

```bash
npx skills add <owner/repo@skill> -g -y
```

`-g` = global (user-level skill dir, which is bind-mounted), `-y` = skip prompts.

### Step 7 (CRITICAL): Read the skill, then USE it — don't reimplement

After install, the skill directory contains its own SKILL.md, scripts, and docs. Your next move is:

1. `ls ~/.agents/skills/<name>/` — see what's there.
2. `cat ~/.agents/skills/<name>/SKILL.md` (and any other `.md`) — learn the entry points and arg shapes.
3. Run the skill's scripts as documented.

DO NOT, after installing a skill, fall back to writing your own script that imports the same underlying library (e.g. installing `pptx` skill then running `npm install -g pptxgenjs && node my-own-script.js`). The whole point of installing a skill is to use the skill author's pre-built workflow — reimplementing means you wasted the install round-trip AND lose the skill's prompt engineering / error handling.

If the skill doesn't fit your task after reading its docs, uninstall it (`npx skills remove …`) and try a different one or fall through to ad-hoc code with full justification to the user. Don't silently ignore the skill you just installed.

### Anti-pattern: `npm install -g <library>` for arbitrary deps

If you need a bare npm/pip library (NOT a skill), do NOT use `-g`. Global installs in the sandbox pollute the global namespace AND vanish on container eviction. Instead, work inside `/workspace/`:

```bash
cd /workspace && npm init -y && npm install <pkg>
# OR for python:
cd /workspace && python -m venv .venv && .venv/bin/pip install <pkg>
```

`-g` is reserved for `npx skills add` (which is special — see Step 6).

## Common Skill Categories

When searching, consider these common categories:

| Category        | Example Queries                          |
| --------------- | ---------------------------------------- |
| Web Development | react, nextjs, typescript, css, tailwind |
| Testing         | testing, jest, playwright, e2e           |
| DevOps          | deploy, docker, kubernetes, ci-cd        |
| Documentation   | docs, readme, changelog, api-docs        |
| Code Quality    | review, lint, refactor, best-practices   |
| Design          | ui, ux, design-system, accessibility     |
| Productivity    | workflow, automation, git                |

## Tips for Effective Searches

1. **Use specific keywords**: "react testing" is better than just "testing"
2. **Try alternative terms**: If "deploy" doesn't work, try "deployment" or "ci-cd"
3. **Check popular sources**: Many skills come from `vercel-labs/agent-skills` or `ComposioHQ/awesome-claude-skills`

## When No Skills Are Found

If no relevant skills exist:

1. Acknowledge that no existing skill was found
2. Offer to help with the task directly using your general capabilities
3. Suggest the user could create their own skill with `npx skills init`

Example:

```
I searched for skills related to "xyz" but didn't find any matches.
I can still help you with this task directly! Would you like me to proceed?

If this is something you do often, you could create your own skill:
npx skills init my-xyz-skill
```
