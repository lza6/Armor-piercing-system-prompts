# Project Knowledge Checkpoints Export

A checkpoint is a portable snapshot of what a teammate needs to understand,
build, operate, or import one project. It is a curated materialized view across
active topic wikis—not a topic, raw-source, or session dump.

## Bundle and operations

```text
docs/knowledge/<project-slug>/
├── index.md
├── project-knowledge.md
├── sources.md
├── checkpoint.json
└── privacy-report.json
```

- `index.md`: portable entry point and import/verification guidance.
- `project-knowledge.md`: full synthesis.
- `sources.md`: readable evidence ledger with stable source IDs.
- `checkpoint.json`: scope, project revision, selected inputs/reasons/hashes,
  exclusions, gaps, audience, generated hashes, and checkpoint identity.
- `privacy-report.json`: deterministic file-hash and privacy attestation. The
  agent never hand-writes it, and it never contains matched excerpts.

Operations are `create` (discover/synthesize), `refresh` (review diffs),
`verify` (read-only checks), and `import` (pinned evidence, not truth).

Create/refresh are dry-run first. `--apply` grants only the previewed file
write; commit, push, publication, release, and import remain separate.

## Selection and synthesis

Start from a bounded project packet: exact repo/revision, goal, audience,
question, aliases, README, supplied `WHY.md`/`BRIEF.md`, ADRs, architecture and
operations/security docs, and explicit seeds. Do not scan `.git`, dependencies,
builds, caches, env files, or arbitrary user directories.

Read HUB/indexes and selected compiled/Idea/Project/plan/output material; one
support hop is allowed. Archives are explicit; raw is excluded. Record
topic/path/hash/words, reason, confidence, access/reuse, exclusions/gaps.

## Comprehensive coverage contract

Default to comprehensive, not executive-summary-only. Retain tables,
requirements, alternatives, phases, workflows, rights, risks, and gaps. Every
input needs positive `words` and a mapped section; `coverage` stores the total
and omissions. Seal rejects knowledge below 20% of selected
words. The floor is not a quality score.

`project-knowledge.md` keeps this order:

1. purpose, audience, current state;
2. project/system map;
3. decisions and rejected alternatives;
4. facts, assumptions, and evidence strength;
5. requirements, constraints, non-goals;
6. development and operating workflows;
7. risks, failure modes, privacy/security boundaries;
8. terminology and important entities;
9. disagreement, unknowns, gaps;
10. next experiments/actions; and
11. scope, trust, refresh instructions.

Use stable source IDs for material claims. Distinguish source facts, user
decisions, hypotheses, and recommendations. Never concatenate articles or
promise completeness beyond the selection method.

## Mandatory privacy gate

Privacy has two layers; neither may be skipped.

### Semantic minimization

Classify every input before synthesis. `unknown` fails closed; an input above
the declared audience requires explicit exact-source approval. Exclude by
default:

- raw bodies, copied paywalled/licensed text, and unverified private evidence;
- sessions, transcripts, event logs, prompts, feedback, and chat text;
- home paths, usernames, hostnames, local IPs, mounts, and temp paths;
- credentials, keys, tokens, cookies, connection strings, and secrets;
- private-adapter code, requests, receipts, artifacts, and data-plane paths;
- customer, health, legal, financial, or other personal records; and
- internal URLs, repos, issues, organization detail, or identifiers outside
  the audience.

Summarize only what recipients need. A scanner cannot recognize every
confidential decision, so semantic review remains mandatory even after a pass.

### Deterministic seal

Generate in private staging, then run the bundled helper:

```bash
llm-wiki checkpoint seal <bundle> --audience private|team|public --json
llm-wiki checkpoint verify <bundle> --json
```

Seal enforces the fixed UTF-8 Markdown/JSON bundle, computes file hashes and
checkpoint ID, scans names/content, and writes `privacy-report.json`. It covers
likely secrets, credentials, identities, local paths/network data,
session/transcript metadata, and prompt/conversation markers. Reports contain
only category, safe/opaque relative path, line, message, and stable finding ID.

There is no `--no-scan` or global bypass. Intentional retention requires every
exact current finding/source ID plus a one-line reason:

```bash
llm-wiki checkpoint seal <bundle> --audience team \
  --allow-finding privacy-<id> \
  --override-reason "Public attribution handle"
```

The scan reruns and reports `overridden`, not `passed`. Never infer an override
from general approval. IDs are content/line-bound; changed or stale values need
review again. Source overrides are likewise exact and attested.

Blocked staging must not reach the final repo. After a passed/overridden seal,
copy the complete five files and verify the destination. On failure remove the
new copy or restore its backup. Never commit staging.

## Audience and manifest

Audience is `private`, `team`, or `public`. Default `private` only for
local/untracked or authoritatively private destinations. Public repos require
explicit `public`; unknown visibility stops before final copy.

`checkpoint.json` uses `llm-wiki/project-knowledge-checkpoint/v1`: sealer ID,
files/hashes, time/audience, project/revision, scope/policy, inputs with
topic/path/hash/words/reason/access, coverage, overrides, exclusions, and gaps.

Paths are bundle-, repo-, or wiki-relative. Never serialize a hub/home root,
transcript pointer, adapter root, or temporary path.

## Refresh

Verify first, rerun stored scope, preserve logical source IDs, and show:

- sources/claims added, removed, or changed;
- decisions/constraints changed;
- gaps opened/closed; and
- privacy findings/overrides changed.

Attested hash differences reveal human edits. Never overwrite silently; offer
staged regeneration, manual merge, or a new path, then seal and verify again.

## Import

Verify before reading/importing. Reject invalid hashes, missing attestation, or
`blocked`; show `overridden` IDs/categories and require recipient acceptance.
For repos, isolate the bundle at the exact producing revision.

Ingest all five files as one bounded collection; preserve checkpoint ID,
audience, source IDs, trust, and staleness; dedupe by ID then path/hash.
Imported synthesis never overwrites Projects, articles, or prior snapshots.
