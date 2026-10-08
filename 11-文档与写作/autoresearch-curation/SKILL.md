---
name: autoresearch-curation
description: Curate and expand the awesome-autoresearch repository. Use when adding new autoresearch cases, collecting discussion evidence from X/Reddit/HN/blogs, promoting discussion items into main categories, refreshing README counts, or running periodic evidence sweeps.
---

# Autoresearch Curation

Use this skill to maintain `awesome-autoresearch` as a **strict, high-signal list of direct autoresearch use cases**.

## Goal

Keep the repository focused on two questions:

1. Where is autoresearch actually being used in public?
2. Which autoresearch patterns transfer across domains?

This skill is for **curation**, not broad AI trend collection.

## Source of truth

Read these files before making changes:

- `README.md`
- `CONTRIBUTING.md`
- every file under `categories/`

`README.md` is the homepage aggregate, not the primary editing surface.
Update category files first, then refresh `README.md` from the current category files.
If available, use `scripts/build-readme.py` instead of hand-editing the aggregate.

## Hard inclusion rules

Only include items that satisfy at least one of these:

- explicitly mention `autoresearch`
- explicitly cite Karpathy's autoresearch
- clearly show a `modify → verify → keep/discard → repeat` loop

And all of these:

- source is public and citable
- description is concrete
- entry stays **one sentence**
- item is **strictly autoresearch-relevant**, not a generic research agent

Reject:

- generic agents
- vague AI commentary
- private or uncitable claims
- things that need a paragraph to justify inclusion

## Category model

Use **main category pages** for stronger evidence such as:

- public repos
- project pages
- substantial write-ups
- clear README evidence of the loop

Use **`categories/related-practices-discussions.md`** for:

- X threads
- Reddit discussions
- Hacker News discussions
- interviews
- blog mentions

when they show credible real practice signals but do not yet have a strong standalone repo or case page.

## Working strategy

### 1. Search broadly, classify narrowly

Use cross-platform searches, but keep inclusion strict.

Preferred evidence channels:

- GitHub
- X / Twitter
- Reddit
- Hacker News
- independent blogs / write-ups

### 2. Keep X queries simple

Prefer medium-complexity searches such as:

- `autoresearch trading`
- `autoresearch benchmark`
- `autoresearch debugging`
- `Karpathy autoresearch robotics`

Avoid very long advanced-search expressions when the adapter is unstable.

### 3. Chinese + English

Search in both languages when useful.

Useful Chinese patterns:

- `autoresearch 回滚`
- `autoresearch 验证器`
- `autoresearch benchmark`
- `Karpathy autoresearch 工程`

But keep Chinese queries narrow to avoid noisy generic matches.

## Promotion workflow

Use this exact ladder:

1. **Discussion lead found**
   - Add to `categories/related-practices-discussions.md` if it is credible and directly autoresearch-related.
2. **Evidence chain search**
   - Look for repo, README, case page, blog post, or project page.
3. **Promotion test**
   - Promote only if public evidence clearly shows a real autoresearch loop or explicit autoresearch framing.
4. **Promote**
   - Move it into the best-fit main category.
5. **Deduplicate**
   - Remove the weaker discussion-only item if the main case now covers it.
6. **Refresh counts**
   - Update `README.md` counts if category totals changed.

## Entry-writing rules

### Main categories

Format:

```md
- [Name](URL) - Domain: one-sentence description of the autoresearch use case.
```

Rules:

- one sentence only
- must mention scenario + loop/value
- prefer concrete verbs like `applies`, `adapts`, `uses`, `iterates`, `keeps`
- avoid hype

### Discussions page

Format:

```md
- [Name or thread title](URL) - Source/platform: one-sentence description of the autoresearch-related practice or discussion.
```

Rules:

- keep it factual
- describe the practice signal, not your opinion
- if it is mostly about transfer of the pattern, say that clearly

## Periodic maintenance loop

When invoked for a recurring sweep:

1. Read the current category files.
2. Search for 3-10 new public leads.
3. Filter aggressively.
4. Add only high-signal entries.
5. Attempt promotion for the strongest discussion leads.
6. Remove duplicates.
7. Recount category totals.
8. Refresh `README.md` so the homepage aggregate matches the current category files and counts.
9. Summarize:
   - what was added
   - what was promoted
   - what remains discussion-only
   - what needs stronger evidence

## Suggested commands

Count entries:

```bash
python - <<'PY'
from pathlib import Path
for p in sorted(Path('categories').glob('*.md')):
    cnt=sum(1 for line in p.read_text().splitlines() if line.startswith('- ['))
    print(f'{p}:{cnt}')
PY
```

Example searches:

```bash
bb-browser site twitter/search 'autoresearch benchmark' --json
bb-browser site twitter/search 'autoresearch debugging' --json
bb-browser site twitter/search 'autoresearch robotics' --json
bb-browser site google/search 'site:reddit.com autoresearch real codebase OR autoresearch debugging' | sed -n '1,220p'
bb-browser site google/search 'site:news.ycombinator.com autoresearch OR "Karpathy autoresearch"' | sed -n '1,220p'
bb-browser site google/search 'site:github.com "autoresearch" robotics' | sed -n '1,220p'
opencli gh api repos/<owner>/<repo>/readme
```

## Quality bar

**Promote slowly. Add discussions faster.**

If evidence is good but not strong enough for a main case, keep it in discussions.
Precision beats coverage.

## Deliverable checklist

Before finishing, verify:

- entries are one sentence
- no generic agents slipped in
- promoted items have stronger evidence than discussion-only items
- discussions page remains useful as a map of emerging practice
- README homepage aggregate matches the current category files

## Recommended invocation phrases

This skill should be used for prompts like:

- "继续搜集 awesome-autoresearch"
- "做一轮 autoresearch 证据巡检"
- "把 discussions 里强条目升格"
- "更新 autoresearch awesome list"
- "定期维护这个仓库"
