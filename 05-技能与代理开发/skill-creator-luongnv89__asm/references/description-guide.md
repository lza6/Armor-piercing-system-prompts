# Writing a good description: pushy + negative triggers

A description has two jobs: pull in the queries that _should_ trigger the skill, and push away the queries from adjacent domains that _shouldn't_. Most authors do the first part well and forget the second, producing false-positive triggers — a Tailwind skill running on a Vue project, a Python skill firing on shell-script questions.

**The fix is a "Don't use for ..." clause.** Name adjacent domains that share keywords or intent but are the wrong fit.

- Positive only: `Creates React components using Tailwind CSS.`
- With negatives: `Creates React components using Tailwind CSS. Use whenever the user asks for a new React component, UI element, or styled layout. Don't use for Vue, Svelte, vanilla CSS, or plain HTML projects.`

Write positive and negative halves as one continuous sentence or two back-to-back sentences — not a structured list. `scripts/quick_validate.py` warns (non-fatal) when the negative-trigger clause appears missing.

## One trigger per branch

Every word in the description costs context load on every turn, so name each distinct triggering branch once and stop. Two triggers that describe the _same_ branch in different words ("build features test-first … asks for TDD") are that branch written twice — keep one. List only genuinely distinct branches, plus a short "when another skill needs…" clause if another skill must reach this one.

## Description length budget

Three limits, in order of which one bites first:

1. **250 chars** — Claude Code's `/skills` listing cap. Anything beyond is **truncated tail-first**, chopping the negative-trigger clause. This is the limit that actually shapes triggering behavior.
2. **~2% of context window** (~16k chars total, ~109 chars overhead per skill) — the shared `available_skills` budget. When it overflows, extra skills become **invisible to the agent**.
3. **1024 chars** — the API spec ceiling, hard error.

**Rule: target ≤250 characters.** Treat 1024 as a hard error, not a goal. Lead with verbs, drop hedge words ("helps", "allows you to"), collapse synonyms, keep the negative half to two or three adjacent domains.

## Bad examples (each fails one job)

```yaml
# Too vague — pulls nothing in
description: Helps with projects.

# Capability described, but no trigger phrases a user would say
description: Creates sophisticated multi-page documentation systems.

# Implementation-speak — users don't phrase requests like this
description: Implements the Project entity model with hierarchical relationships.
```

For eval-driven tuning of an existing description, see `references/description-optimization.md`.
