# Quality-Gated Streaming Contract

Use this contract for same-SKU multi-color apparel batches when the workflow should submit each color as soon as its own reference and director package pass, without waiting for every other color. New work uses `zibuyu_quality_gated_streaming_v2`; this contract changes scheduling only. It never relaxes three-view fidelity, evidence provenance, market-profile, canonical-timeline, prompt, anatomy, action, differentiation, PopBoom, rendered-quality, or delivery gates.

Historical `zibuyu_quality_gated_streaming_v1` plans remain immutable and read/poll/report-compatible. Never add v2 market fields to a locked v1 plan in place. Move unreleased colors that require v1.4/v6 to a new v2 parent; accepted child `record_id` values remain in the original parent and are never resubmitted.

## Quality Invariant

A color may enter a paid PopBoom call only after all of these are true:

1. Every intended color is known and grouped from source images.
2. One immutable all-color plan has locked the shared evidence, SKU signature, template, timeline grid, per-color creative deltas, captions, hashtags, and pairwise differentiation.
3. The current color's three-view image passed both the complete source-versus-output audit and the zero-human-identity-pixel audit, recorded as `human_identity_pixels_absent: true`.
4. The current color has one complete schema `1.4` `single_compile` object using `zibuyu_ugc_quality_v4`, `zibuyu_market_prompt_v1`, `zibuyu_three_layer_deadlines_v1`, and `canonical_prompt_v7`, with the exact planned market profile/generation controls, immutable global/asset deadline blueprint, complete per-beat shot deadlines, `Reference 1 / @Image1` fixed-model identity first, and garment references at `Reference 2 / @Image2+`.
5. The release exactly implements its planned color delta and non-CTA timeline signatures.
6. `validate_streaming_plan.py`, `validate_batch_compile.py`, and `validate_ledger.py` all return `valid: true`; the director and ledger validators also authorize the exact paid job.

Do not use streaming when color grouping, same-SKU construction, target market/model, source evidence, or the intended color list is uncertain. Fall back to the all-references barrier workflow until those facts are stable.

## Immutable Plan And Release Layout

Use one parent run directory:

```text
$CODEX_HOME/zibuyu-runs/<run_id>/
  batch-plan.json
  releases/
    <variant_id>/
      batch-compile.json
      ledger.json
      snapshots/
      artifacts/
      videos/
```

`batch-plan.json` is immutable after the first paid submission. It contains:

- `schema_version: "1.1"`;
- `contract_id: zibuyu_quality_gated_streaming_v2`;
- stable parent run, batch-compile, and SKU-family IDs;
- `plan_revision: 0` and `plan_locked: true`;
- `market_prompt_contract_id: zibuyu_market_prompt_v1`, the exact batch key including `market_prompt_profile_id`, and the full locked `shared_core`;
- `shared_core` must already contain a locked `amazon_product_review_evidence_v1` research bundle plus non-empty pain-solution map, claims allowlist, and claims registry;
- canonical `shared_core_sha256` and `differentiation_plan_sha256`;
- all planned variants, each with source images, one identical color-excluded garment signature, market-specific `generation_controls`, immutable `deadline_blueprint`, creative delta, caption, exactly five hashtags, planned non-CTA spoken-intent IDs, and planned non-CTA `framing|core_action|product_point` signatures;
- every required quality gate set to its strict value.

Streaming is limited to 2-12 planned colors. Use the barrier wave workflow above 12 colors.

Run:

```text
python scripts/validate_streaming_plan.py <absolute-path-to-batch-plan.json>
```

Require `valid: true` before the first color release. Store the exact-file `streaming_plan_sha256` in every release's `streaming_release.plan_sha256`.

Use these exact plan fields; do not improvise aliases:

```yaml
quality_gates:
  all_source_colors_grouped: true
  shared_evidence_locked: true
  planned_differentiation_passed: true
  per_variant_three_view_qc_required: true
  per_variant_human_identity_pixel_qc_required: true
  per_variant_director_validation_required: true
  identity_first_reference_order_required: true
  schema_version_required: "1.4"
  quality_contract_id_required: zibuyu_ugc_quality_v4
  market_prompt_contract_id_required: zibuyu_market_prompt_v1
  serializer_id_required: canonical_prompt_v7
  deadline_contract_id_required: zibuyu_three_layer_deadlines_v1
  three_layer_deadlines_required: true
  rendered_quality_review_required: true
```

Every `planned_variants[]` object must contain `variant_id`, `color_name`, `source_images`, the full color-excluded `garment_signature`, the complete `generation_controls`, the immutable `deadline_blueprint`, `caption`, exactly five `hashtags`, `planned_non_cta_spoken_intent_ids`, `planned_non_cta_visual_signatures`, and this complete `creative_delta`:

```text
hook_angle_id, hook_text_original, color_terms, opening_visual_id,
scene_id, scene_palette, styling_signature, proof_focus_id,
pain_focus_id, signature_action_id, spoken_intent_ids,
caption_angle_id, difference_summary
```

Store each planned visual signature as the canonical normalized lowercase `framing|core_action|product_point` string that the final timeline will implement. Use `validate_streaming_plan.py`; hand-written summaries or differently punctuated aliases do not authorize a release.

The `deadline_blueprint` must contain the exact current deadline `contract_id`, the complete global layer, the complete asset layer, and the required shot-field/forbidden-outcome policy. Global and asset layers are immutable after plan validation. A release may populate its per-beat shot deadline values only from that color's canonical timeline and may not weaken the field set or mandatory failure exclusions. Read `three-layer-deadline-contract.md` before creating the plan.

The batch key accepts only `US + 美1/美2/美3 + en-US + us_champion_v1` or `DE + 德1/德2/德3 + de-DE + de_champion_v1`. Each planned `generation_controls` object uses the closed schema in `market-prompt-compiler-contract.md`: prompt shell, delivery, camera, music, verdict, commerce CTA, and fixed-model-stats-only screen text. The immutable plan hashes these controls inside the differentiation payload so a release cannot silently switch from US to DE logic or from hybrid proof to continuous live speech.

## Shared Precompile

Complete these once for the parent plan while three-view work is in progress when parallel-agent capacity is available:

- product/Amazon evidence collection and conflict handling;
- recent TikTok research or the labeled local-knowledge fallback;
- garment construction baseline from reliable product photos;
- claims allowlist, pain-solution map, template, and timeline grid;
- all planned color hooks, scenes, styling, proof focus, action signatures, captions, and hashtags;
- the complete pairwise differentiation matrix.

The main agent must read the owning skills and contracts before delegating. A parallel creative lane may draft the shared plan, but it may not generate references, authorize paid work, mutate ledgers, or submit PopBoom jobs. The main agent must independently inspect the artifacts and run every deterministic gate.

Plan at least four non-CTA intent IDs and four non-CTA visual signatures for every default 15-second variant. Every pair must retain the existing three-dimension attention/visual-world/proof-action rule, two non-CTA intent differences, two non-CTA visual-signature differences, non-color-only hooks/captions, and at least two pairwise-unique hashtags per side.

## Per-Color Release

After one color's three-view passes garment QC and the zero-human-identity-pixel audit:

1. Create `releases/<variant_id>/batch-compile.json` as one complete schema `1.4` `single_compile` object.
2. Copy the exact locked `market_prompt_contract_id`, `batch_compile_id`, `sku_family_id`, `batch_key`, and `shared_core` from `batch-plan.json`.
3. Copy that color's planned source images, garment signature, generation controls, deadline blueprint, creative delta, caption, and hashtags exactly. Materialize the release's global and asset deadline layers byte-for-byte from the blueprint, then add one complete shot deadline for every canonical beat.
4. Add:

```yaml
streaming_release:
  contract_id: zibuyu_quality_gated_streaming_v2
  plan_path: absolute path to batch-plan.json
  plan_sha256: exact file SHA-256 returned by validate_streaming_plan.py
  variant_id: current variant ID
  child_run_id: <batch_run_id>:<variant_id>
```

5. Bind the passed three-view absolute path, the SHA-256 recomputed from its readable local bytes, `human_identity_pixels_absent: true`, exact `@Image2`-or-higher interface tag, and expanded reference-role contract. `@Image1` remains reserved for the fixed model.
6. Finalize the canonical timeline from the locked planned signatures and the passed three-view evidence.
7. Compile and validate:

```text
python scripts/validate_batch_compile.py <release-batch-compile.json> --compile --output <release-batch-compile.json>
python scripts/validate_batch_compile.py <release-batch-compile.json>
```

The existing validator loads the immutable plan and blocks any market-contract/profile, shared-core, color, source, SKU-signature, generation-control, global/asset deadline, shot-deadline policy, creative-delta, caption, hashtag, intent, visual-signature, schema, serializer, identity-first reference order, zero-human-pixel reference-QC, or quality drift. Continue only with `valid: true` and `eligible_for_new_submission: true`. Historical locked plans remain readable for audit/poll/report; do not mutate them to authorize a current release.

Create one immutable PopBoom ledger for this release with `run_id` exactly equal to `<batch_run_id>:<variant_id>`. Its exact `batch_compile_sha256` and director receipt bind the plan-backed release into the request fingerprint. The ledger validator rejects a child-run mismatch. Run the normal ledger history gate before the paid call.

## Streaming Scheduler

- Submit a release immediately after its own three-view, director, upload, model, balance, history, and paid-authorization gates pass. The parent plan validator caps the entire streaming batch at 12 one-job children, so all active children together cannot exceed configured concurrency `12`.
- Poll submitted releases asynchronously while generating or finalizing later colors.
- Treat every release ledger as a one-job PopBoom wave. Across the parent run, never exceed the configured user concurrency of `12`.
- Before the first paid release, verify that the balance covers the worst-case cost of the entire parent plan. Recheck current balance before each later paid release.
- If any child receives a rate/concurrency signal, latch the parent run's effective maximum to `1` for all remaining releases. Accepted records remain poll-only.
- Immediately report each accepted `record_id`; do not wait for the remaining colors to submit.

## Freeze And Repair Rules

- After the first paid release, never edit `batch-plan.json`.
- Never edit a released color's compile, prompt, reference, receipt, fingerprint, or accepted record ID.
- If a later three-view reveals that the planned SKU signature or proof plan is wrong, block that color. Do not weaken its validator and do not change the already submitted color.
- To change the locked all-color plan after payment has started, create a new parent `run_id` for the unreleased colors and rerun the balance and differentiation gates.
- A failed three-view regenerates through `image_gen.imagegen`; a failed director package returns to the canonical timeline; a failed PopBoom job preserves its visible error. No paid retake occurs without explicit user authorization.

## Final Quality And Delivery Gate

Streaming changes time-to-first-result, not completion criteria. After every planned color is terminal:

- run the normal delivery-pack validator for every release ledger;
- inspect every available video using the existing garment identity/material, claim-to-part, action-to-voiceover, action coverage, posture, two-arm/two-hand anatomy, continuity, lip-sync, lighting, proof, and CTA checks;
- record one rendered-quality verdict and one primary failure variable per release;
- rerun actual pairwise differentiation across all rendered colors and mark it verified only when each pair contains a non-color audible difference and a non-color visible difference;
- deliver status/location, quality verdict, caption, and exactly five hashtags for every planned color.

Never describe platform `succeeded` as creative-quality acceptance.

If an unreleased color must move to a new parent run after the original plan was frozen, write a separate `parent-closure.json` beside the immutable plan with `status: partial_migrated`, the migrated variant IDs, destination parent run IDs, reason, and timestamp. Do not mark the original parent complete. Run per-release delivery gates for its submitted colors, and claim full all-color completion only after every migrated color also reaches terminal quality review in the linked run.
