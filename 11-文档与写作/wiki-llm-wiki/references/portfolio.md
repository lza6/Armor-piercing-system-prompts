# Portfolio

Portfolio is a live, read-only view over canonical Ideas and Projects. It
answers "what could we build?" and "what have we committed to?" without a
catch-all topic or copied records.

## Authority And Scope

- Ideas stay in `inventory/ideas/<slug>.md` in one primary topic.
- Projects stay in `output/projects/<slug>/` with a non-empty `WHY.md`.
- Concepts stay in `wiki/concepts/`; they are supporting knowledge, not
  portfolio rows unless explicitly captured as Ideas.
- Portfolio is derived chat output. Never create `HUB/portfolio/`, `HUB/ideas/`,
  a `project-ideas` topic, duplicate records, or a portfolio manifest.

Scope order differs from topic-writing commands: `--local` selects `.wiki/`,
`--wiki <name>` selects one active topic, and no flag selects all active topics
from `HUB/wikis.json`. Skip the synthetic hub entry and archived topics unless
`--include-archived` is explicit.

## Index-First Collection

For every selected topic:

1. Read its `_index.md`, then `inventory/ideas/_index.md` when present.
2. Use Idea index fields first. Open a record only for missing requested fields,
   stale-index checks, filters, project linkage, or maturity verification.
3. Enumerate active Projects through `output/projects/*/WHY.md`; folder state is
   authoritative. Read only the heading and first paragraph by default.

Do not scan `raw/`, Concept/article bodies, loose outputs, or Project members
for a normal list.

## Cross-References And Maturity

Link an Idea and Project only when the Idea's `project:` field resolves to that
Project. Label it `promoted Idea`; all other Projects are `direct`. Never infer
lineage from similar slugs or titles. Report plausible missing links for review
without editing either object.

Derive Idea maturity normally: `captured` when the record exists;
`researching` from linked research; `shaped` from approval plus an Approved
Decision Snapshot; `project` from a resolving Project link and `WHY.md`; and
`delivered` from a linked outcome/deliverable. Never store `stage`.

## Output And Filters

Show counts for Ideas, active Projects, promoted Projects, and direct Projects,
then render separate tables:

```markdown
| Topic | Idea | Status | Maturity | Priority | Next Action | Project |
| Topic | Project | Origin | Goal |
```

Support filters for kind, topic, Idea status/maturity/priority, tag, owner,
linked/unlinked, archive inclusion, and limit. Business/product/service/plugin/
experiment are Idea tags, not object types. If tags are absent, report
under-classification rather than guessing.

## Legacy Candidates

Direct Projects do not need synthetic Ideas. If asked to find untracked
opportunities, separately inspect topic/Concept/output/Project indexes, preview
1-3 candidate Ideas with proposed owning topics, and wait for approval before
capture. Never bulk-promote Concepts.

The portfolio query performs no wiki or activity-log write. Rerunning it reads
current canonical state, so no saved cache can drift.
