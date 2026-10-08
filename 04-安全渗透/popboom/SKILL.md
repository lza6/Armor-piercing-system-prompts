---
name: popboom
description: Use when the user wants PopBoom video generation, especially evidence-bound advanced Seedance video generation with clothing-three-view references, fixed custom models, schema 1.4 market-bound Amazon/review receipts, strict legacy-v5 resume handling, task submission, status polling, completion reporting, or a resumed Zibuyu apparel run that must also deliver captions with exactly 5 hashtags.
---

# PopBoom execution router

Choose the route before loading an execution manual. Paths are relative to this skill directory.

| Current action | Required reading |
| --- | --- |
| Query existing records, reconcile an interrupted submission, download or report | [Poll and reconcile](references/poll-and-reconcile.md). For a Zibuyu run, first run `../zibuyu-top-tiktok-operations-specialist/scripts/inspect_run.py <run-directory>` and use its per-job paths. |
| New preflight, upload, paid submission, repair or exact legacy unpaid resume | [Production execution](references/production-execution.md) and the full [batch contract](references/batch-execution-contract.md); these retain all existing paid gates. |
| Zibuyu final QA/caption delivery | [Resume and delivery](../zibuyu-top-tiktok-operations-specialist/references/resume-and-delivery.md). |

An existing record is query/download/report-only forever. An unknown submission must be reconciled without a repeat paid call. Route classification and status summaries never authorize payment, mark QA passed, or authorize publishing.

For new paid work, preserve exact current director receipts, full persisted compile validation, immutable model/reference identity, model-first ordering, product_info, balance, concurrency 12 and batch-latched rate limits. Run `scripts/validate_ledger.py` with the exact previous revision and batch file as required by the full contract. Prefer `--summary --result-file <run-local-result.json>` for routine review; payment must read the fresh full result and use only its exact authorized outbound request. Never build a paid request from a summary.

For Zibuyu, do not end with links alone: finish actual-media QA and the caption/five-tag delivery gate for every job. Do not regenerate accepted work to repair captions. Only after whole-batch acceptance and delivery may the parent publishing handoff begin; it retains separate final-table authorization. Production-only scope ends at delivery.
