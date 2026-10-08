---
name: seedance-ugc-cn-director
description: Generate Chinese director plans for conversion-oriented and anatomically stable Seedance 2.0 UGC short-video ads from product images, clothing three-view references, product descriptions, or product pages. Use for apparel/TikTok/PopBoom workflows needing Amazon product-and-review evidence analysis, buyer-pain-to-claim mapping, market-specific US/Germany prompt compilation, claim-matched close-ups, a recognition-to-proof-to-conviction creator performance arc, continuous phone/prop and endpoint handoffs, motivated natural kinetics, expressive connected voice delivery, role-locked references, garment-first fidelity, canonical timeline compilation, deterministic pre-submission validation, one-variable repair guidance, and fixed PopBoom model presets 美1, 美2, 美3, 德1, 德2, 德3.
---

# Seedance UGC director router

Use the canonical timeline as the single source for scripts, B-roll, spoken lines and final prompts. Keep product evidence, proof actions, model/garment identity and market fidelity intact.

- For new direction, timeline edits, reference repair, prompt compilation or any package intended for new payment, read [Directing workflow](references/directing-workflow.md), then all contracts and the selected market profile it requires. This first optimization changes loading by stage; it does not relax creative or validation requirements.
- For query/download/QA/delivery of an existing video, use the parent [Resume and delivery](../zibuyu-top-tiktok-operations-specialist/references/resume-and-delivery.md). Do not compile a new prompt or regenerate receipts for an accepted record.
- For caption-only requests, use the parent's caption-only route. If compiled copy already exists, recover it through `validate_delivery_pack.py`. If new copy is needed, deliver a separate copy supplement bound to the existing video; never edit an accepted compile, immutable parent plan, paid prompt or receipt to repair copy. URL-only requests need verified garment/video or user-supplied evidence, not a fabricated production ledger.
- For plugin maintenance, inspect only the relevant implementation; do not start a production task.

Paths in the workflow are relative to this skill directory. New work is schema 1.4 / `canonical_prompt_v7`; historical v5/v6 remain query/download/QA/report-compatible. Their existence never authorizes a new or repeated paid call.

Use `scripts/validate_batch_compile.py <batch.json> --compile --output <batch.json> --summary --result-file <compile-result.json>`, then independently validate the persisted file with `--summary --result-file <validation-result.json>`. Require success and new-submission eligibility before a new paid handoff; the ledger validator independently rechecks the exact file before payment. The full result file contains all receipts; the summary is a navigation aid only.

Keep market-language voiceover plus line-by-line Chinese review translations available to the user, with review translations excluded from generated audio. Deliver captions with exactly five unique relevant non-brand hashtags; forbid both `#Imily Bela` and `#ImilyBela`.
