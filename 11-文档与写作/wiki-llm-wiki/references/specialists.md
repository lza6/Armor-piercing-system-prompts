# Personal Specialist Skills

Specialists are user-owned, Agent Skills-compatible instruction packages that
apply a bounded evidence or review method during wiki research and analysis.
They are not simulated credentials. Prefer names such as
`research-methodologist` or `financial-stewardship-reviewer` over `phd`, `cfo`,
or `doctor`.

## Local storage and sharing boundary

The personal reusable library lives under the locally resolved hub:

```text
HUB/.skills/
├── _index.md
├── registry.json
└── <specialist-name>/
    ├── SKILL.md
    └── references/       # optional Markdown only
```

`registry.json` is the source of truth for topic allowlists:

```json
{
  "schema_version": 1,
  "topics": {
    "meta-llm-wiki": ["research-methodologist"]
  }
}
```

There is no global default. A specialist must be explicitly enabled for each
active topic. Definitions stay at hub scope and are never copied into a topic
wiki. Local `.wiki/` projects can consume a hub specialist only through an
explicit user request; v1 does not persist local-project allowlists.

"Personal" means user-owned, stored in the selected hub, and absent from the
public plugin. It does not mean encrypted or strictly local: a synced hub is
still synced, and instructions enter the selected model's context when loaded.
Store methods, source hierarchies, templates, and preferences here—not
credentials, case facts, health records, customer data, or source corpora.

## Instruction-only package contract

V1 packages may contain only `SKILL.md` and optional Markdown files under
`references/`. Symlinks, executable bits, scripts, binaries, and extra
directories fail validation. This intentionally narrows the trusted
instruction surface.

`SKILL.md` uses standard `name` and `description` frontmatter. llm-wiki
extensions live under `metadata`:

```yaml
---
name: research-methodologist
description: Reviews study design, causal claims, evidence quality, and reproducibility.
metadata:
  llm-wiki-kind: specialist
  version: "0.1.0"
  risk-tier: medium
  reviewed: "2026-08-17"
---
```

Every specialist must define these level-one sections:

1. `Mandate`
2. `Non-claims`
3. `Use when`
4. `Do not use when`
5. `Required inputs`
6. `Method`
7. `Output contract`
8. `Stop and escalation rules`
9. `Tool posture`
10. `Evaluation cases`
11. `Provenance and maintenance`

An `allowed-tools` field is prohibited. The parent wiki workflow and runtime
sandbox own tool authority; loading a specialist never grants tools, writes,
network access, or permission to spawn agents.

## Deterministic management

Use the bundled helper relative to the installed plugin root, or
`scripts/llm-wiki` in a source checkout:

```bash
llm-wiki specialist init
llm-wiki specialist create research-methodologist \
  --description "Reviews study design, causal claims, evidence quality, and reproducibility."
llm-wiki specialist validate [<name>]
llm-wiki specialist refresh
llm-wiki specialist list [--wiki <topic>] [--json]
llm-wiki specialist show <name> [--json]
llm-wiki specialist enable <name> --wiki <topic>
llm-wiki specialist disable <name> --wiki <topic>
```

`create` writes a complete scaffold, not a finished expert method. Replace its
TODOs and generic procedure, add specific positive and negative evaluation
cases, validate it, then enable it. Run `refresh` after manual edits so
`.skills/_index.md` reflects current descriptions and versions. These writes
append hub activity-log entries; enable/disable also append the affected topic
log.

Hub lint recognizes `.skills/` and fails on invalid packages or allowlists.
It does not require a specialist library for users who do not want one.

## Selection protocol

For research or analysis:

1. Resolve the topic and read its indexes normally.
2. Read `.skills/_index.md`, then obtain the topic allowlist with
   `specialist list --wiki <topic> --json`. Do not scan every package.
3. Classify the task by domain, decision type, risk, jurisdiction, freshness,
   and evidence needs.
4. Select zero to three enabled specialists; default to one. Explicit
   `--specialist <name>` still requires that name to be enabled. Honor
   `--no-specialists` without reading package bodies.
5. Run `specialist validate <name>` for every selected method and stop on any
   finding. Load only the selected `SKILL.md`, plus a referenced Markdown file
   only when its instructions require it.
6. Give every selected specialist the same bounded evidence packet: exact
   question, intended use, as-of date, jurisdiction when relevant, selected
   wiki/raw evidence, missing inputs, and output schema.
7. Apply a single specialist inline by default. Use isolated agents only when
   independent judgment, incompatible contexts, or cross-domain analysis
   justifies the coordination cost.
8. Synthesize by claim and evidence strength. Preserve disagreements and
   unknowns; majority vote is not a truth mechanism.
9. Verify citations, dates, unsupported claims, and stop-rule compliance.
10. Record each selected specialist's name, version, and content SHA-256 in the
    research session, raw note, or output provenance.

High-stakes medical, legal, tax, accounting, and financial methods must browse
current authoritative primary sources, expose jurisdiction and as-of date, and
name the qualified human review required. A specialist can structure evidence;
it cannot diagnose, prescribe, sign, attest, file, or assume fiduciary duty.

## Candidate discovery

`/wiki:specialist suggest` is agentic because useful candidates depend on the
actual portfolio. It must remain index-first:

1. Read the hub `_index.md` and active entries in `wikis.json`.
2. Read each active topic's root `_index.md` only.
3. Rank recurring decision patterns, evidence weaknesses, high-stakes review
   needs, and repeated output types—not merely topic names.
4. Open category indexes or a small sample of articles/outputs only for the
   strongest candidate clusters.
5. Compare against the existing specialist index and avoid duplicates.
6. Return a short ranked list with evidence of recurrence, bounded mandate,
   non-trigger cases, risk tier, source hierarchy, and the first evaluation
   cases. Do not create or enable packages automatically.

## Maintenance and eval gate

Before enabling a specialist broadly, compare it against the no-specialist
baseline on positive selection, non-selection, stale evidence, conflicting
evidence, insufficient data, and escalation cases. Keep it only if it improves
evidence use, reasoning process, or decision usefulness. Review source
hierarchies and jurisdiction-specific references on the declared cadence.
Retain the recorded version/hash in durable outputs so older analyses remain
auditable.
