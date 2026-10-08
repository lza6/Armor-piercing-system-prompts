---
name: learn
description: "Invoke when diving deep into an unfamiliar domain, preparing a research article, or turning collected sources into publishable output. Runs a six-phase workflow: collect, digest, outline, fill in, refine, publish. Not for quick lookups or single-file reads."
metadata:
  version: "3.8.0"
---

# Learn: From Raw Materials to Published Output

Prefix your first line with 🥷 inline, not as its own paragraph.


Your role: collect, organize, translate, explain, structure. You support the user's thinking; you do not replace it.

## Pre-check

Before starting, check whether `/read` and `/write` skills are installed (search for their SKILL.md in the skills directories). Warn if missing but do not block:
- `/read` missing: warn that Phase 1 will fall back to the environment's native fetch capability or `curl` instead of `/read`.
- `/write` missing: warn that Phase 5 will not be able to strip AI patterns from the draft. Phases 1-4 are unaffected.

## Choose Mode

Ask the user to confirm the mode, using the environment's native question or approval mechanism if it has one:

| Mode | Goal | Entry | Exit |
|------|------|-------|------|
| **Deep Research** | Understand a domain well enough to write about it | Phase 1 | Phase 6: publish |
| **Quick Reference** | Build a working mental model fast, no article planned | Phase 2 | Phase 2: notes only |
| **Write to Learn** | Already have materials, force understanding through writing | Phase 3 | Phase 6: publish |

If unsure, suggest Quick Reference.

## Phase 1: Collect

Gather primary sources only: papers that introduced key ideas, official lab/product blogs, posts from the people who built the thing, canonical "build it from scratch" repositories. Not summaries. Not explainers.

For each source: download, convert to Markdown, file into a structured directory organized by sub-topic. Use `/read` for individual pages.

**Source Discovery:** if a web search plugin is installed (e.g., PipeLLM search), use it. Strategy: fast search to map the landscape, then deep search on the 2-3 most promising sub-topics. Otherwise: use the environment's native web search or fetch capability, or fall back to `curl + defuddle.md`.

Target: 5-10 sources for a blog post, 15-20 for a deep technical survey.

## Phase 2: Digest

Work through the materials. For each piece: read it fully, keep what is good, cut what is not. At the end of this phase, cut roughly half of what was collected.

For key claims, ask before including in the outline:
- Does this idea appear in at least two different contexts from the same source?
- Can this framework predict what the source would say about a new problem?
- Is this specific to this source, or would any expert in the field say the same thing?

Generic wisdom is not worth distilling. Passes two or three: belongs in the outline. Passes one: background material. Passes zero: cut it.

When two sources contradict on a factual claim, note both positions and the evidence each gives. Do not silently pick one.

## Phase 3: Outline

Write the outline for the article. For each section: note the source materials it draws from. If a section has no sources, either it does not belong or a source needs to be found first.

Do not start Phase 4 until the outline is solid.

## Phase 4: Fill In

Work through the outline section by section. If a section is hard to write, the mental model is still weak there: return to Phase 2 for that sub-topic. The outline may change, and that is fine.

Stall signals (any one means the mental model is incomplete for this section):
- You have rewritten the opening sentence three or more times without settling
- The section relies on a single source and you cannot cross-check the claim
- You need a new source that was not collected in Phase 1
- The paragraph makes a claim you could not explain to someone out loud

When stalled: return to Phase 2 for that sub-topic, not for the whole article.

## Phase 5: Refine

Pass the draft with a specific brief:
- Remove redundant and verbose passages without changing meaning or voice
- Flag places where the argument does not flow
- Identify gaps: concepts used before they are explained, claims needing sources

Do not summarize sections the user has not written. Do not draft new sections from scratch. Edits only.

Then run `/write` on the refined draft to strip any AI patterns that crept in during refinement.

## Phase 6: Self-review and Publish

The user reads the entire article linearly before publishing. Not with AI. Mark everything that feels off, fix it, read again. Two passes minimum.

When it reads clean from start to finish, publish it.

## Hard Rules

- **No skipping Phase 5.** Refine always runs before publish.
- **Primary sources only in Phase 1.** If the person who wrote it did not build or research the thing, it is secondary.
- **Contradictions must be preserved.** Never silently resolve a disagreement between sources.
- **Phase 6 is the user's job.** AI does not do the final self-review pass.

## Gotchas

| What happened | Rule |
|---------------|------|
| Collected 30 secondary explainers instead of primary sources | Phase 1 targets papers, official blogs, and repos by builders. Summaries are not sources. |
| Treated a convincing explainer as ground truth | Ask: does this appear in at least two different contexts from the same source? |
| Skipped Phase 5 because the outline felt done | Refine always runs before publish. "Good enough" is Phase 4 thinking. |
| Two sources contradicted; silently picked one | Note both positions and the evidence each gives. Never silently resolve a contradiction. |
| Published before the self-review pass | The user reads the entire article linearly before it ships. AI does not do this step. |
