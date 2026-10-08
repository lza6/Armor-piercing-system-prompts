# Ideas

Ideas bridge compiled knowledge and committed delivery. They preserve a rough
proposal while it is researched and shaped without creating a Project early.

## Object Model

- **Concept** — reusable, evidence-backed knowledge in `wiki/concepts/`.
- **Idea** — something that might be built, tested, published, or otherwise
  delivered; one record under `inventory/ideas/`.
- **Project** — explicitly committed work under `output/projects/<slug>/`.

The usual path is Concept → Idea → Project, with many-to-many links. Direct
Projects remain valid.

## Storage And Record

Each Idea has one stable slug and one primary topic wiki:

```text
inventory/ideas/
├── _index.md
└── <slug>.md
```

Use `related_wikis` for cross-topic relevance. A hub-wide catalog reads active
topics' Idea indexes through `wikis.json`; never create a hub-level Ideas store.
For a combined cross-topic view of Ideas plus direct and promoted Projects, use
the read-only [Portfolio workflow](portfolio.md).

```yaml
---
title: "Local Podcast Search"
kind: idea
status: active
priority: p1
created: YYYY-MM-DD
updated: YYYY-MM-DD
aliases: [podcast search]
tags: [idea, local-first]
summary: "Explore cited local search across personal podcast transcripts."
next_action: "Choose and approve a shaped version."
related_wikis: [home-ai]
sources:
  - wiki/concepts/hybrid-search.md
---
```

Required fields are normal inventory fields: `title`, `kind`, `status`,
`priority`, `created`, `updated`, `tags`, and `summary`. Recommended fields are
`aliases`, `next_action`, `owner`, `related_wikis`, and `sources`. Promotion may
add `approved_at`, `promoted`, and `project: output/projects/<slug>`.

Preserve the original seed plus the opportunity, target, supporting
Concepts/evidence, assumptions/counterevidence, alternatives, current shape,
scope/no-gos, risks, questions, success criteria, intended deliverable, and
decision notes. Evidence stays in `raw/`; reusable synthesis stays in `wiki/`.
An Idea links to both but is not factual evidence.

## Lifecycle

Use `proposed`, `active`, `blocked`, `superseded`, or `archived`. Generic
inventory also permits `ingested`, but it is rarely meaningful for Ideas.

Derive maturity instead of storing `stage` or percentage complete:

- **captured** — record exists;
- **researching** — linked raw research exists;
- **shaped** — `approved_at` and Approved Decision Snapshot exist;
- **project** — linked non-empty `WHY.md` exists;
- **delivered** — the linked Project records an outcome or deliverable.

## Capture, Research, And Shape

Before capture, search Idea titles/aliases, Concepts, inventory, and Project
slugs. Link related Ideas; never merge. Durable writes require one owning topic.

Development reuses normal research for gaps, writes evidence to `raw/`, compiles
reusable knowledge when useful, and updates the brief without implying commitment.

Shaping challenges the premise and offers minimal, ideal, and optionally
lateral versions. Define target, appetite, scope, no-gos, risks, success
criteria, and deliverable, then stop for explicit choice. Only explicit user
approval adds `approved_at` and an Approved Decision Snapshot.

## Promotion

Promotion requires approval in the current request or an existing
`approved_at` plus Approved Decision Snapshot. It is idempotent:

1. Reuse an already-linked valid Project and reject slug collisions.
2. Create `output/projects/<slug>/WHY.md` from the approved rationale.
3. Freeze the snapshot into `BRIEF.md`; both files link back to the Idea.
4. Add the Project link and promotion date to the Idea; rebuild indexes/log.

The Idea remains catalog/lineage. The Project owns changing plans,
implementation decisions, code, data, deliverables, and outcomes. Idea and
Project archive are independent.

## Fuzzy Interface And Invariants

Users need not know object names, paths, slugs, or commands. Resolve aliases and
current conversation context:

- "I have an idea" or "park this" → capture.
- "what ideas do I have" → catalog.
- "research this idea" or "does it hold water" → develop.
- "poke holes" or "smallest version" → shape.
- "use the narrow version and make it a project" → approve/promote.
- "plan/build/ship it" after promotion → use the linked Project.

Clarify only when ambiguity changes state: "make this real" may mean shape,
promote, or implement; "kill this" may mean Idea, Project, or both. Never
promote automatically, merge by similarity, duplicate project state, or create
a separate hub Ideas tree.
