---
name: zibuyu-top-tiktok-operations-specialist
description: Produce evidence-bound US or German TikTok apparel videos from product images, a source URL, SKU, and fixed model; complete three-views, direction, PopBoom generation, actual-video QA, and copy delivery before handing the fully accepted batch to scheduled publishing. Also supports production-only requests and resumes without repeating accepted work.
---

# Zibuyu apparel workflow

Produce and accept US/DE apparel videos, deliver each color's complete copy, then prepare publishing only within the user's requested scope.

## Choose the current stage

Paths below are relative to this skill directory. Use absolute paths when executing scripts from another working directory. Load only the current route; load a later route when entering that stage. Existing references to this SKILL.md enter this router, not automatically the new-production contract.

| Request / evidence | First action and required reading |
| --- | --- |
| Existing run, record, interrupted batch, status or delivery | Run `python scripts/inspect_run.py <absolute-run-directory>`, then read [Resume and delivery](references/resume-and-delivery.md). Use the returned ledger/compile paths and per-job next actions. |
| Only captions/tags for an existing video, including URL-only input | Use the caption-only route in [Resume and delivery](references/resume-and-delivery.md). Do not create a production ledger for a copy request. |
| New production or an unsubmitted package requiring preparation | Read [Production workflow](references/production-workflow.md). Activate three-view, director and PopBoom skills only as their stages begin. |
| Reference, script or rendered failure | Inspect the existing run first; preserve completed artifacts. Read the relevant production stage and the owning skill. A new paid repair requires its own authorization. |
| Accepted, fully delivered batch or independently selected completed videos for publishing | Read [Post-production publishing](references/post-production-publishing.md), then the bundled publishing skill. |
| Existing publishing IDs, saved-time check, timezone discrepancy or schedule audit | Read the bundled [post-schedule audit](../zibuyu-popboom-auto-publish/references/post-schedule-audit.md); reconcile the existing IDs without new generation or duplicate publication. |
| Plugin audit or implementation | Inspect only the source/contracts/scripts needed for the requested change; this does not start production. |

`inspect_run` is a read-only navigation snapshot, not validation, QA, authorization, or a live platform query. Unknown/missing evidence stays blocked. A `new_planning` next action is not permission to generate.

## Invariants across routes

- Lock user SKU, product/source identity, exact color, market and language. 美1/美2/美3 map to US/en-US; 德1/德2/德3 map to DE/de-DE. Keep market evidence separate. Defaults: TikTok, Seedance 2.0, 15s, 9:16, 720p, audio on, watermark off, product_info brand `拓展平台` and user SKU.
- Preserve fixed-model identity and hash-bound pure-white garment three-views with zero human identity pixels. The model asset is first; garment references follow. Never upload a model portrait as garment material.
- New work uses schema 1.4 / canonical_prompt_v7 and all current evidence, action, market, timeline and ledger validators. Historical packages retain their original version for query/download/QA/delivery. Strict v5 exact resume is handled only by the full PopBoom production contract.
- A `record_id` is permanently query/download/report-only. `submission_unknown` or an interrupted dispatch without acceptance evidence means reconcile first. Never automatically resubmit or use a timeout as paid permission.
- Platform success is separate from downloaded original-media inspection, technical checks, visual/product/identity QA and complete copy delivery. Record the actual verdict; `keep` requires inspected video evidence. Identity mismatch blocks further releases.
- Deliver every color's status/video or failure, QA verdict, and one market-language paragraph containing the caption plus exactly five unique relevant non-brand hashtags. Neither `#Imily Bela` nor `#ImilyBela` is allowed. Script reviews include every spoken line with its Chinese translation; Chinese review text is excluded from generated audio.
- Preserve existing batch isolation and authorization boundaries. Preparation, generation, retakes and publishing have distinct scopes. Publishing begins only after the whole intended batch is accepted and delivered, unless independently requested for existing videos; production-only ends at delivery.
- A created publishing task and an audited schedule are separate facts. Compare the platform's queried identity/copy/time with the approved values after each submission; missing timezone evidence or a mismatch holds remaining submissions. Only the whole intended set passing this audit supports a claim that scheduling is complete.

## Compact machine results

Use `--summary --result-file <run-local-result.json>` with the director and ledger validators. Full results remain on disk. Read detailed errors only when needed. Before payment, load the complete fresh ledger result, verify its exact authorized set and outbound-request hashes, then send its exact returned arguments. A summary never authorizes a call. Delivery output remains complete for the user.


Publishing defaults: follow `../zibuyu-popboom-auto-publish/references/confirmed-publishing-contract.md`. Explicit user manual acceptance and same-scope publishing authorization are sufficient; do not repeatedly ask about caption mapping or account-local-to-Beijing conversion.
