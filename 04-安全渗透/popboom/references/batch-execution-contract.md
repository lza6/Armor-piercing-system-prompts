# PopBoom Batch Execution Contract

This contract is mandatory for PopBoom submission, resume, and monitoring. It overrides conflicting endpoint, duration, model-selection, upload, ledger, or polling instructions in older skill text and tool guides.

## Canonical MCP And Capability Snapshot

- Server name: `PopBoom`
- Transport: `streamable_http`
- Endpoint: `https://tkvideo.zbycorp.com:5002/mcp`
- Authentication: existing Codex secure configuration only. Never print, copy, persist, or request the bearer value.
- HTTP `/sse`, placeholder hosts, and direct developer-client scripts are legacy paths and must not be used for production. The plugin `PreToolUse` Hook is a local argument-rewrite layer for the canonical MCP call, not a second PopBoom client.

At the start of each batch, perform one non-paid MCP initialization/tool-schema preflight and persist only the server version, tool names, and relevant schema summary. Required tools are `upload_images`, `generate_video`, `check_task`, `query_balance`, and `list_custom_portraits`. For local references, the installed plugin Hook matching `mcp__PopBoom__upload_images` must also be trusted and active. `download_video` is conditional and is needed only when a successful status lacks a usable `video_url` or the user requests a local mirror.

An MCP inventory entry is not proof that a tool is callable in the current turn. If PopBoom is listed but any required tool is absent from the callable set, use the platform tool-search capability to load each exact required tool before classifying MCP as unavailable. Persist this as non-paid preflight discovery, never as an upload or generation attempt.

Persist `transport: streamable_http`. Hash the normalized live `generate_video` schema into `schema_sha256`. For a new 15-second submission, use `mcp_declared` when that schema explicitly declares 15; when the schema description omits 15, use `mcp_observed` only if the machine baseline contains a validator-accepted successful record with `valid_for_new_submission: true`. `mcp_provisional` is retained only so historical accepted jobs can be polled, downloaded, or reported; it must never authorize a `planned` or `submission_started` job. Never write a 10- or 30-second request into a 15-second job.

The machine-readable baseline is `../assets/runtime-capabilities.json`. Preserve its evidence chronology: the verified 2026-07-11 server snapshot was `sora2-codex 1.27.1` with 18 tools; six independent 15-second attempts failed on 2026-07-20; then the current MCP endpoint accepted `duration: 15` on 2026-07-30 as `record_id: 200704`, task `cgt-20260730170321-rsrlh`, charged 84 points at 480p/9:16, and reached `succeeded`. This latest accepted terminal record supersedes the older negative observation for new routing while `valid_for_new_submission` remains true. Do not infer a current server version from the historical snapshot.

The persistent user execution policy is `../assets/execution-policy.json`. It fixes configured PopBoom concurrency at `12`, source `user`, evidence kind `user_explicit`, and barrier waves of at most 12 jobs. Every ledger must snapshot that policy exactly; never replace it with an unevidenced `default` value. A quality-gated streaming parent preserves this policy by using one immutable one-job release ledger per ready color and enforcing the same aggregate ceiling across all active child ledgers.

## Unified 15-Second Routing

Never silently downgrade, split, stretch, or relabel a 15-second request as 10 seconds.

Use this decision order:

1. If the live schema declares 15 seconds, use `mcp_declared` with explicit `model: "sd2"`, `duration: 15`, `resolution: "720p"`, and `ratio: "9:16"` unless the user requested other supported settings.
2. If the live schema omits 15 but the machine baseline has `mode: mcp_observed`, `valid_for_new_submission: true`, and a validator-accepted successful record, use `mcp_observed`. Bind the ledger evidence ID, observed duration, accepted `record_id`, accepted task ID, and terminal status exactly to that baseline.
3. If an `mcp_observed` call returns an explicit invalid-parameter rejection without a `record_id`, persist the failure, disable observed routing for subsequent new submissions in that run, and route those jobs to the PopBoom UI with explicit 15s, 720p, and 9:16. Do not retry the rejected MCP call. Any response with a `record_id` is accepted work and becomes poll/download/report-only.
4. If neither `mcp_declared`, valid `mcp_observed`, nor an operable UI route exists, stop before submission and report the exact capability blocker.

Historical ledgers may retain `route: mcp_provisional` only after a request was already accepted or reached a reconciliation/poll/report state. They are never eligible for a new `generate_video` action.

For an explicitly requested 5- or 10-second job, MCP may be used when the live schema supports it.

## Fixed Personal Model Registry

For the personal `zibuyu-top-tiktok-operations-specialist` tenant, use the machine-readable registry at `../assets/fixed-model-registry.json`, mirrored below for auditability:

| Preset | Asset ID | Identity version | Identity image SHA-256 | Market | Language | Last live verification |
| --- | --- | --- | --- | --- | --- | --- |
| 德1 | `asset-20260703091345-n2q24` | `de1-20260804-v1` | `9e3ba3779e005e1344072f066714b2a099e5315059c19fdcce92a53ca916367b` | Germany | German | 2026-08-04 |
| 德2 | `asset-20260703091441-5rfz2` | `de2-20260804-v1` | `9af5518095e676fb6ab7abc046a1dbc7826b3e2c14a15f775d418f022b9548bc` | Germany | German | 2026-08-04 |
| 德3 | `asset-20260819170940-ltnxb` | `de3-20260819-v1` | `4322f8f0c6e4cf45cbac52dd94304472f597e98073915b80cce5fb52897afdd5` | Germany | German | 2026-08-19 |
| 美1 | `asset-20260810141830-tbcnd` | `mei1-20260819-v1` | `f0834f80d842acca9f8df7bd874c3bf50ad5f935136a8ea0b071c2570b0e9d26` | United States | American English | 2026-08-19 |
| 美2 | `asset-20260819170810-kbchw` | `mei2-20260819-v1` | `3af80c418d0e7b6660e6eaa84a65bb7882ddd893eb0ca34c5693be5681c7e533` | United States | American English | 2026-08-19 |
| 美3 | `asset-20260819170855-8927v` | `mei3-20260819-v1` | `227e0c8ae6ee1e41e0c3098ff9ea5d9abd56e008f05c6c319a8b1854b2f89825` | United States | American English | 2026-08-19 |

At batch preflight, validate the configured asset ID directly with the available portrait query and require exactly one accessible asset in the current tenant. For every new Zibuyu apparel job, require the registry-pinned `identity_version` and `identity_image_sha256`, copy both into the job as `asset_identity_version` and `asset_identity_sha256`, and verify the live portrait bytes against that digest before payment. Do not rely on Chinese-name keyword search: live name queries for these presets can return zero. If an ID is missing, duplicated, belongs to another tenant, resolves to the wrong model, or lacks/mismatches its identity byte pin, stop before paid submission and report the registry problem. Historical records remain poll/report/audit-compatible; any future model replacement requires a new registry version and digest before new payment.

Pass the model as `asset://<asset_id>` in `ref_image_urls` unless the live schema exposes a dedicated virtual-human field. For fixed-model apparel, reference order is strict: `ref_image_urls[0]` is the validated fixed-model asset, and garment/product URLs begin at `ref_image_urls[1]`. The prompt role map must match this exact order. A request with garment-first/model-last ordering is invalid for paid submission even when the UI shows the correct portrait thumbnail. Model photos must never enter `upload_images` or apparel reference material. A team or other tenant must maintain its own registry and must not reuse these personal-tenant IDs without live validation.

The PopBoom `/sd2` web build observed on 2026-08-03 appended the selected portrait after uploaded garment references. That UI serialization produced a rendered identity mismatch for a legacy 美1 preset despite showing the selected portrait thumbnail. Treat that UI route as blocked for fixed-model apparel until its outbound request can be verified model-first; use a validator-accepted MCP route that preserves the strict order.

Model-first ordering did not fully solve exact identity in the subsequent 2026-08-03 test `record_id: 204446`: hair moved toward the approved curls, but face and skin tone remained a lighter-skinned hybrid. Under the current user-approved workflow, every new fixed-model Zibuyu apparel request requires an apparel artifact and binding with `human_identity_pixels_absent: true` plus a hash-bound `zero_human_identity_pixels_v1` full-resolution cue audit, while fixed-model `asset://` remains first in `ref_image_urls` and garment references remain second onward. Do not require or generate a separate identity-conditioned first-frame image for the default workflow. Treat identity accuracy as rendered-frame QA: if the output is a hybrid or wrong identity, record `identity_mismatch`, stop the batch, and do not submit more colors without explicit user authorization. Prompt prohibitions may not be used to excuse a garment reference that still contains human identity pixels.

## Persistent Ledger

Create or resume `$CODEX_HOME/zibuyu-runs/<run_id>/ledger.json`. Resolve `CODEX_HOME` from the environment; when unset, use the current user's `.codex` directory. Use one run directory per logical production batch. Create missing parent directories, write atomically when possible, and never store credentials or image base64 data.

Minimum logical schema:

```yaml
schema_version: "1.0"
ledger_revision: non-negative integer
previous_ledger_sha256: lowercase SHA-256 | null
run_id: string
sku_family_id: string
batch_compile_id: string
workflow_kind: zibuyu_apparel | generic_popboom | omitted for historical poll/report only
batch_compile_sha256: lowercase SHA-256 of exact persisted bytes  # required only for zibuyu_apparel
endpoint: string
transport: streamable_http
server_name: string
server_version: string
capability_snapshot:
  captured_at: timestamp
  server_version: string
  tool_names: [string]
  generate_video:
    schema_sha256: string
    declared_durations: [integer]
    declared_resolutions: [string]
    declared_ratios: [string]
  download_video_available: boolean
route: mcp_declared | mcp_observed | mcp_provisional (historical only) | ui | blocked
route_evidence:
  kind: declared | accepted_observed | provisional_observed (historical only) | ui_operable | blocked
  evidence_server_version: string | null
  observation_id: string | null
  observed_duration_seconds: number | null
  accepted_record_id: integer | null
  accepted_task_id: string | null
  accepted_terminal_status: succeeded | null
  verified_at: timestamp
wave_control:
  policy_id: popboom-user-concurrency-12-v1
  max_inflight: 12
  max_inflight_source: user
  max_inflight_updated_at: timestamp
  max_inflight_evidence:
    kind: user_explicit
    allowed_max_inflight: 12
    observed_at: timestamp
    evidence_id: string
    endpoint: string
    server_version: null
    source_run_id: string | null
    source_ledger_sha256: string | null
    observed_peak_inflight: integer | null
    successful_jobs: integer | null
    rate_limit_errors: integer | null
  effective_max_inflight: 12 | 1
  max_jobs_per_wave: 12
  submission_strategy: barrier_waves
  fill_wave_without_wait: true
  next_wave_release: previous_wave_terminal
  planned_job_count: integer
  submission_waves:
    - wave_index: integer
      job_keys: [string]
  rate_limit_events:
    - evidence_id: string
      batch_run_id: string
      trigger_job_key: string
      trigger_request_fingerprint: string
      error_code: string | null
      error_message: string | null
      observed_at: timestamp
      submission_outcome: confirmed_not_accepted | accepted | unknown
      record_id: integer | null
  rate_limit_override: null | object
    kind: rate_limit_response
    scope: batch
    batch_run_id: string
    allowed_max_inflight: 1
    trigger_job_key: string
    trigger_request_fingerprint: string
    error_code: string | null
    error_message: string | null
    observed_at: timestamp
    evidence_id: string
balance_gate:
  queried_at: timestamp
  balance: number | null
  estimated_worst_case_cost: number | null
  unit: string
  estimate_status: known | unknown
  passed: boolean
created_at: timestamp
updated_at: timestamp
models:
  - name: string
    asset_id: string
    validation_status: valid | invalid | unknown
    validated_at: timestamp
artifacts:
  - artifact_id: string
    variant_id: string
    local_path: path
    filename: string
    sha256: string
    bytes: integer
    mtime: timestamp
    upload_transport: local_file_hook | url_reuse | popboom_ui | legacy_inline_base64
    content_variant: original | derivative
    source_sha256: string | null
    transformation_reason: string | null
    transformation_authorized: boolean
    human_identity_pixels_absent: boolean  # required true for new apparel_three_view artifacts
    identity_cue_audit:  # required on new apparel_three_view artifacts
      audit_version: zero_human_identity_pixels_v1
      audited_sha256: exact artifact sha256
      inspection_method: full_resolution_visual_inspection
      reviewed_at: timestamp
      passed: true
      skin_present: false
      face_present: false
      hair_present: false
      neck_chest_collarbone_present: false
      shoulders_arms_wrists_present: false
      hands_fingers_nails_present: false
      tattoos_jewelry_present: false
      person_specific_body_shape_present: false
    identity_cue_audit_sha256: canonical SHA-256 of identity_cue_audit
    upload_status: pending | uploading | uploaded | failed
    uploaded_url: string | null
    uploaded_at: timestamp | null
    url_expiry: timestamp | null
    url_status: unknown | usable | missing | expired | unreachable
    last_verified_at: timestamp | null
jobs:
  - job_key: string
    job_kind: zibuyu_apparel | generic_popboom | omitted for historical poll/report only
    variant_id: string
    color_name: string
    model_name: string
    model_preset: string
    asset_id: string
    asset_identity_version: string  # required for new Zibuyu apparel
    asset_identity_sha256: lowercase SHA-256  # required for new Zibuyu apparel
    batch_compile_id: string
    timeline_id: string
    timeline_version: integer
    compiled_prompt: string
    prompt_sha256: string
    director_receipt:  # required only for zibuyu_apparel
      variant_id: string
      timeline_id: string
      timeline_version: positive integer
      batch_compile_sha256: lowercase SHA-256
      schema_version: "1.4"  # default new work; strict legacy exact resume may retain 1.3
      quality_contract_id: zibuyu_ugc_quality_v4
      serializer_id: canonical_prompt_v7
      deadline_contract_id: zibuyu_three_layer_deadlines_v1
      three_layer_deadlines_sha256: lowercase SHA-256
      quality_plan_sha256: lowercase SHA-256
      research_bundle_sha256: lowercase SHA-256
      canonical_beats_sha256: lowercase SHA-256
      compiled_text_sha256: lowercase SHA-256
      market_prompt_contract_id: zibuyu_market_prompt_v1
      market_prompt_profile_id: us_champion_v1 | de_champion_v1
      market_prompt_profile_sha256: lowercase SHA-256
      generation_controls_sha256: lowercase SHA-256
      voiceover_review_sha256: lowercase SHA-256
      market: US | DE
      voiceover_language: en-US | de-DE
      director_valid: true
      eligible_for_new_submission: true
    legacy_v5_exact_resume:  # allowed only on a new run resuming one exact unsubmitted v5 source job
      contract_id: legacy_v5_exact_resume_v1
      user_explicit: true
      authorization_scope: exact_existing_compile_no_rewrite
      authorization_id: string
      authorized_at: timestamp
      source_run_id: string
      source_ledger_sha256: lowercase SHA-256 of canonical prior ledger
      source_job_key: lowercase SHA-256
      batch_compile_sha256: lowercase SHA-256
      director_receipt_sha256: lowercase SHA-256 of exact prior v5 receipt
      paid_package_sha256: lowercase SHA-256 of exact prompt, references/order, identity, settings, receipt, and outbound request
      prior_director_validated: true
      prior_submission_eligible: true
      unchanged_assertions:
        compiled_prompt: true
        reference_bindings_and_order: true
        model_identity: true
        settings: true
        director_receipt: true
        outbound_request: true
    reference_bindings:
      - reference_id: string
        artifact_id: string
        variant_id: string
        role: apparel_three_view
        url: string
        sha256: string
        human_identity_pixels_absent: true
        identity_cue_audit: exact batch-reference audit copied to the bound artifact
        identity_cue_audit_sha256: exact canonical audit SHA-256
    generation_model: sd2
    outbound_request:  # exact MCP request; product_info is required
      server_name: PopBoom
      tool_name: generate_video
      arguments:
        prompt: exact compiled_prompt
        ref_image_urls: [asset://validated-model first, garment URLs in binding order]
        model: sd2
        duration: exact job duration
        resolution: exact job resolution
        ratio: exact job ratio
        product_info: exact compact JSON string {"brand":"拓展平台","sku":"<batch item number>"}
    outbound_request_sha256: lowercase SHA-256 of exact sorted compact outbound_request JSON
    request_fingerprint: string
    resolution: string
    duration: integer
    ratio: string
    route: string
    submission_transport: mcp_streamable_http | popboom_ui
    state: planned | submission_started | submission_unknown | submitted | queued | running | succeeded | failed | pending_timeout | blocked
    record_id: integer | null
    task_id: string | null
    submission_started_at: timestamp | null
    submitted_at: timestamp | null
    last_polled_at: timestamp | null
    next_poll_at: timestamp | null
    poll_attempts: integer
    video_url: string | null
    video_url_status: unknown | usable | missing | unusable | expired
    download:
      reason: not_required | missing_url | local_mirror
      state: not_required | pending | succeeded | failed
      attempts: 0 | 1
      started_at: timestamp | null
      completed_at: timestamp | null
      local_path: path | null
      sha256: string | null
      bytes: integer | null
      error_code: string | null
    error_code: string | null
    error_message: string | null
    next_action: generate_video | reconcile_submission | check_task | poll | wait | download_video | report | complete | none
    resubmit_allowed: false
    submission_reported: boolean
    completion_reported: boolean
```

The ledger root remains schema `1.0`; the director receipt is versioned independently. `prompt_sha256` is the lowercase hexadecimal SHA-256 of the exact UTF-8 `compiled_prompt`. Derive `job_key` from `run_id + variant_id + model_preset + prompt_sha256 + settings_hash`. Derive `request_fingerprint` from a deterministic UTF-8 JSON object whose keys are sorted and whose separators are compact. For default new `job_kind: zibuyu_apparel` work, sign the entire `outbound_request` object, its independent SHA-256, the complete ordered reference bindings, fixed identity SHA/version, and every normalized schema-1.4/v7 `director_receipt` field shown above, including deadline, market/profile/control, and voiceover-review bindings. The validator-returned `authorized_outbound_requests` entry must expose the receipt SHA-256 and exact market binding next to the unchanged MCP request. A strict v5 resume additionally signs the complete `legacy_v5_exact_resume` object. Changing any MCP argument, URL order, image byte digest, product_info brand/SKU, deadline hash, market/language/profile value, evidence receipt, owner, version, authorization, or verdict must invalidate the paid fingerprint. Generic standalone jobs retain the historical signature. One color can have several jobs when model or prompt version differs; do not reduce the ledger to one row per color.

Every new PopBoom video generation must carry product information. The 品牌事业部/brand value is fixed to `拓展平台` and must not be requested from the user. The 货号/sku is batch-scoped user input; if it is missing, stop before paid submission and ask for it. For MCP `generate_video`, serialize it as compact JSON in `product_info`, exactly `{"brand":"拓展平台","sku":"<货号>"}`. Do not submit a paid request with omitted product_info, an empty sku, a non-JSON string, extra product_info keys, or any brand other than `拓展平台`.

Every job entering `submission_started` must have an explicit ledger `workflow_kind` from the closed enum `zibuyu_apparel | generic_popboom`, and its `job_kind` must exactly match. A missing kind is tolerated only while polling/reporting historical accepted work; it cannot produce `paid_submission_allowed: true`. Any planned or paid job whose structured references include `role: apparel_three_view` must use `workflow_kind: zibuyu_apparel` and `job_kind: zibuyu_apparel`; `generic_popboom` is never a downgrade route for apparel and must be rejected before payment.

Every `reference_bindings` entry must resolve to one ledger artifact with the same artifact ID, variant ID, URL, and SHA-256. The apparel reference and compiled prompt must belong to the same variant and timeline version.

Persist every mutation as a new ledger revision. Revision `0` has `previous_ledger_sha256: null` and is preflight-only: it can validate but can never authorize a paid call. For revision `N > 0`, retain revision `N-1` as an append-only snapshot, hash its canonical UTF-8 JSON form with sorted keys and compact separators, store that digest in `previous_ledger_sha256`, and pass the snapshot through `--previous-ledger`. The validator must verify a one-step revision, identical run/batch identity, immutable wave plan and paid fingerprints, append-only rate-limit events, and immutable accepted record IDs before it can return `history_verified: true` or authorize payment.

Use an anchored `planned` revision to select up to the available capacity from the released wave; that state never authorizes payment by itself. Immediately before the paid calls, create the next revision by changing exactly that selected set to `submission_started` and validate it against the retained planned snapshot. Issue those calls together only when this exact revision returns `history_verified: true`, `wave_plan.paid_submission_allowed: true`, and `authorized_submission_job_keys` exactly equal the selected set. The set may contain up to 12 jobs under the normal policy and at most 1 after a rate-limit latch. Persist every response as a new revision, including `record_id` when accepted, and continue the released wave without inserting polling work. Keep `resubmit_allowed: false` for every job. On timeout or ambiguous network failure, set `submission_unknown` with `next_action: reconcile_submission`; reconcile by ledger/server history before any retry and never blindly resubmit a possibly accepted paid request. Any job that has a `record_id` is permanently poll/download/report-only within that run: retain its record ID and request fingerprint, and use `check_task`, never `generate_video`.

## Quality-Gated Streaming Releases

Use this path for new work only with an immutable plan that passed `../../seedance-ugc-cn-director/scripts/validate_streaming_plan.py` under `zibuyu_quality_gated_streaming_v2`. Locked v1 parents remain read/poll/report-only.

- Keep one parent directory with immutable `batch-plan.json` and one child release directory per color.
- Streaming parents are limited to 2-12 colors. Above 12, use normal barrier waves; never rely on unvalidated aggregate child concurrency.
- Each new child contains one complete schema `1.4` `single_compile` file and one one-job ledger. The compile's `streaming_release` block stores the absolute plan path, exact plan SHA-256, and current variant ID. Do not migrate an accepted legacy child into this schema; retain its original ledger for polling/reporting.
- Set `streaming_release.child_run_id` and the child ledger `run_id` to the exact same `<batch_run_id>:<variant_id>` value; the ledger validator rejects drift.
- Keep every released child at the exact `releases/<variant_id>/batch-compile.json` and `ledger.json` paths. Before a paid child, the ledger validator scans all sibling releases, rejects a changed parent plan path/hash, duplicate/relabelled variants, missing or mismatched sibling ledgers, aggregate inflight above 12, and more than one active/submission-started child after any sibling rate-limit event.
- The sibling director validator loads that plan during normal compile validation. Therefore `validate_ledger.py --batch-compile <child-release-file>` still receives one exact file, one exact `batch_compile_sha256`, and one normal director receipt; no paid-call gate is bypassed.
- Before the first child payment, query balance and require coverage for the worst-case cost of every color in the immutable parent plan. Recheck current balance before each later child.
- Submit a child immediately after its own reference, upload, director, model, balance, history, and exact-job authorization pass. Its single-job `submission_waves` plan remains immutable.
- Across all nonterminal child ledgers, never exceed configured concurrency `12`. Poll accepted children round-robin while later colors are prepared.
- A rate/concurrency signal from any child latches the parent effective limit to `1` for all unreleased children. Preserve the triggering event in that child and the parent coordination state; never retry an accepted record.
- Never edit `batch-plan.json` after the first paid child or edit a child compile/receipt/fingerprint after its payment starts. If a later color cannot satisfy the locked plan, block it or create a new parent run for unreleased work.

Barrier batches that use one multi-job ledger continue to follow the existing wave rules below.

## Balance Gate And Upload Dedupe

Call `query_balance` once before the paid batch and persist the result in `balance_gate`. Estimate the worst-case batch cost from the current settings and number of planned jobs. The observed 2026-07-30 15-second 480p text-to-video call cost 84 points. For any other 15-second setting without an explicit quote, use at least three times the documented 5-second cost at the selected resolution and apply every documented image-generation multiplier; prefer any higher live platform quote. If the balance is insufficient or the estimate remains unknown, set `passed: false` and stop before the first paid submission rather than create an accidental partial batch.

For every local apparel reference:

1. Stage the QC-approved image byte-for-byte under the current run directory. Never resize, recompress, decode/re-encode, or otherwise transform it during staging.
2. Compute SHA-256, byte size, and mtime from the staged original. Set `content_variant: original`, null transformation fields, and `upload_transport: local_file_hook` for a new MCP upload.
3. Call `upload_images` with the matching filename and an absolute percent-encoded `local-file:///...` URI in `image_data`. The trusted plugin Hook must rewrite that URI immediately before the canonical remote MCP call. The model and shell must never generate, echo, copy, or persist Base64. A Codex CLI diagnostic tool-call view may still render the host-rewritten argument; treat that as a host display limitation, do not copy it into artifacts, and use a fresh Codex Desktop task for normal production.
4. Reuse a ledger URL only when the same SHA-256 is still accessible and unexpired; use `upload_transport: url_reuse` and retain `content_variant: original`.
5. Deduplicate identical content across colors, prompts, and models.
6. Upload only missing unique files, at most 10 images per `upload_images` call, and always include a filename.
7. Persist successful URLs immediately; on partial failure retry failed items only. If the Hook is unavailable or PopBoom rejects the original size, stop before paid generation and retain the exact error.
8. Do not create a reduced-resolution fallback automatically. A derivative requires the original `source_sha256`, a non-empty transformation reason, and explicit user authorization; persist it as `content_variant: derivative` instead of replacing the original artifact record.

`legacy_inline_base64` is historical provenance only. It may remain on a reusable completed artifact or a poll/report ledger, but it must not authorize a new local upload. `popboom_ui` is allowed only when the UI actually uploaded the original file.

The upload batch size is not the render concurrency limit.

## Wave Submission And Round-Robin Polling

Use the long-term policy in `../assets/execution-policy.json` for every new batch. Persist the configured values unchanged as `max_inflight: 12`, `max_inflight_source: user`, and `max_inflight_evidence.kind: user_explicit` with `allowed_max_inflight: 12`. Never serialize this user choice as source `default`.

Build `submission_waves` from the ordered `jobs` list before the first paid request:

- when `planned_job_count <= 12`, create exactly one wave containing every job and submit that released wave continuously without polling waits between submissions;
- when `planned_job_count > 12`, partition in job order into full 12-job waves plus one final remainder wave;
- every job key appears exactly once, and no wave contains more than 12 jobs;
- release wave 2 or later only after every job in the preceding waves is truly terminal. `submission_unknown` and `pending_timeout` do not release the next wave.

Keep the configured 12/user/user-explicit fields immutable when throttling. Normally start a new `run_id` with `effective_max_inflight: 12`, an empty `rate_limit_events` array, and `rate_limit_override: null`. Treat `429`, `HTTP_429`, `rate_limit`, `rate_limited`, `too_many_requests`, `concurrency_limit`, and `concurrency_limited` in either the error code or message as a limit signal. Append a batch-scoped event that snapshots the triggering job key, request fingerprint, error, observation time, acceptance outcome, and record ID. The event log is append-only for that `run_id`; task recovery or cleared transient error fields never remove it. Before the next paid submission, persist `effective_max_inflight: 1` plus a batch-scoped `rate_limit_response` override matching the newest event. Keep that latch for the rest of the run and reset it only by starting a genuinely new `run_id`. Already accepted jobs keep their record ID and request fingerprint and are poll-only; never submit them again.

When a `generate_video` limit response unambiguously proves that no task was accepted, record `submission_outcome: confirmed_not_accepted` and the job may return to `planned` for a one-at-a-time retry. If acceptance is ambiguous, use `submission_unknown` and reconcile first. Do not mark retryable limit work `blocked`, because `blocked` is terminal for wave release and would silently drop it.

The scheduler must use `effective_max_inflight`, not the configured base value. Before each paid call, require the validator to return `valid: true`, `wave_plan.paid_submission_allowed: true`, and at least one available submission slot. Preserve per-color prompt/reference binding and never reuse one color's prompt fingerprint for another color.

Poll active jobs round-robin rather than blocking on one record:

- queued: after 15 seconds, then 30 seconds, then at most every 60 seconds;
- running: about every 15 seconds;
- add +/-20 percent jitter to avoid synchronized bursts;
- transient network, 429, and 5xx errors: retry up to three times with increasing backoff, without changing `record_id`;
- after 15 minutes without a terminal state: mark `pending_timeout`, retain the record for a later resume, and do not resubmit.

On `succeeded`, use `video_url` from `check_task` when present and record `video_url_status: usable`. Set `download.reason/state` to `not_required` in that case. Call `download_video` at most once only when that URL is missing or a local mirror is required, and persist its reason, attempt count, result, and local artifact metadata. Aggregate routine polling updates; immediately report each submission with its `record_id` when available, then report terminal completion/failure without flooding the user with unchanged polls.

## Submission Binding Gate

Every paid job must bind all of the following before submission:

```text
batch_compile_id, variant_id, timeline_id, timeline_version,
three_view_path or uploaded_url, compiled_prompt, prompt_sha256,
model_preset, asset_id, asset_identity_version, asset_identity_sha256,
outbound_request, outbound_request_sha256,
duration, resolution, ratio
```

Block a job if the prompt belongs to another color, the reference does not match the variant, the timeline version is stale, the fixed asset is unverified, or a multi-color batch reuses an identical prompt fingerprint.

For default new Zibuyu apparel submission, declare `workflow_kind: zibuyu_apparel` on the ledger and `job_kind: zibuyu_apparel` on every job. Persist the SHA-256 of the exact batch-compile file bytes as ledger `batch_compile_sha256`, then copy the exact current `validate_batch_compile.py` receipt into the job: schema `1.4`, `quality_contract_id: zibuyu_ugc_quality_v4`, `serializer_id: canonical_prompt_v7`, `deadline_contract_id: zibuyu_three_layer_deadlines_v1`, `three_layer_deadlines_sha256`, `market_prompt_contract_id: zibuyu_market_prompt_v1`, the exact profile/control/voiceover-review hashes and market/language fields, all existing variant/timeline/research/quality/canonical-beat/compiled-text hashes, and both validation booleans true. The only valid closed tuples are `US + 美1/美2/美3 + en-US + us_champion_v1` or `DE + 德1/德2/德3 + de-DE + de_champion_v1`. The result must cover source/ASIN/variant-locked product evidence, direct-review sampling, conflicts, pain-to-solution mapping, claims evidence, exact allowlisted/spoken claim IDs and target-language terms, one-to-one claim/part/framing/action/evidence bindings, structured generation controls, the complete global/asset/per-shot deadline hierarchy, target-language plus Chinese voiceover review, minimum sequential product-action coverage, relaxed posture, complete left/right hand plans, anatomy-safe coordinated two-hand exceptions, deterministic `@Image1` fixed-identity/`@Image2` garment-role mapping, and `human_identity_pixels_absent: true` on the garment reference.

`validate_ledger.py` must reject the Zibuyu job before paid authorization when `--batch-compile` is absent; when the exact file, research bundle, profile, generation-controls, deadline, or voiceover-review hash differs; when the current sibling director validator does not return a matching valid and eligible schema-1.4/v7 receipt; when the market/model/language/profile tuple is invalid; when the job's variant, color, timeline ID/version, compiled prompt, prompt hashes, receipt, or apparel reference owner does not exactly match the selected variant in that file; when the global, asset, or shot deadline markers are absent from the prompt; when the garment artifact/binding does not independently declare `human_identity_pixels_absent: true`; when the selected asset ID lacks a matching registry identity SHA/version; when the exact `outbound_request` differs from the job prompt/settings, omits required product_info, uses any product_info brand other than `拓展平台`, has an empty sku, or fails model-first order; when the normalized paid fingerprint does not include the current receipt and complete outbound payload; or when the prompt lacks deterministic evidence-bound motion-rich structure. An A-color or US-market receipt/prompt/reference cannot authorize a B-color or DE-market job.

Schema `1.3` / `canonical_prompt_v5` is historical read/poll/download/report/audit-compatible but is not generally eligible for a new paid call. `legacy_v5_exact_resume_v1` is the sole exception: the user must explicitly approve continuing the exact existing unsubmitted package; create a new resume run; pass both the exact v5 batch compile and the exact original planned ledger; require the source receipt to prove prior `director_valid: true` and `eligible_for_new_submission: true`; and bind the canonical source-ledger SHA-256, original job key, batch hash, director-receipt hash, complete paid-package hash, and all unchanged assertions. The validator independently requires the source job to remain `planned` with null `record_id`, task ID, and submission timestamps; requires the current prompt, reference order, identity, settings, receipt, outbound request, and related hashes to match that source package exactly; and signs the resume authorization into the paid fingerprint. Do not edit an old run's fingerprint in place. Any job that already has a `record_id` stays in its original schema and is poll/download/report-only forever: never migrate it, regenerate its receipt, or return it to `generate_video`.

Generic standalone PopBoom jobs with explicit `generic_popboom` kind and no `apparel_three_view` reference retain the historical fingerprint and do not require a director receipt or batch file. The moment an `apparel_three_view` reference is present, the job is apparel and must pass the Zibuyu director gate regardless of its claimed kind. Paste the exact validated `renderings.prompt.compiled_text` verbatim; never substitute raw Amazon reviews, the voiceover-only script, or a paraphrase. Historical terminal `canonical_prompt_v4`, ordinary v5, and schema `1.0`/`1.1`/`1.2` director rows remain polling/reporting/audit-only and do not need the new fields for those actions, but cannot authorize a repair, rewrite, or new paid call. Never submit a prompt whose product explanation is a motionless hold or whose spoken garment part is not the framed/action target.

## Executable Ledger Gate

Run the bundled standard-library validator against the persistent ledger after preflight, before each paid submission, after every submission response, and after each terminal-state update:

```text
python scripts/validate_ledger.py <absolute-path-to-ledger.json> --previous-ledger <absolute-path-to-revision-N-minus-1.json> --batch-compile <absolute-path-to-batch-compile.json>
```

For `legacy_v5_exact_resume_v1`, append the exact source package:

```text
--legacy-source-ledger <absolute-path-to-original-planned-v5-ledger.json>
```

For revision `0`, omit `--previous-ledger`; it is validation-only and must return `history_verified: false` plus `wave_plan.paid_submission_allowed: false`. Historical polling/reporting and explicit `generic_popboom` jobs may omit `--batch-compile`; a Zibuyu `submission_started` revision may not. `--legacy-source-ledger` is required only for the strict v5 exact-resume path and never authorizes an accepted source record. For a non-personal/team registry, add `--model-registry <absolute-path-to-model-presets.json>`. Continue to the paid calls only when the validator exits `0`, returns `valid: true`, returns `history_verified: true`, returns `wave_plan.paid_submission_allowed: true`, returns the exact intended keys in `wave_plan.authorized_submission_job_keys`, and returns one matching entry in `wave_plan.authorized_outbound_requests` for each new Zibuyu job. For v7, that authorization entry must contain the matching `director_receipt_sha256`, `market_prompt_binding`, and deadline-bound receipt fields; a legacy resume must contain `legacy_v5_exact_resume_sha256`. Send only the returned `outbound_request.arguments` object verbatim to the returned server/tool; the adjacent authorization fields are audit evidence, not extra MCP arguments. Do not reconstruct, reorder, supplement, or manually override the outbound request. Recompute and compare `outbound_request_sha256` immediately before the MCP call. The balance, route, model, reference, idempotency, state, identity-byte, and external director-evidence checks must also pass. A prose claim or ledger-local receipt never replaces validation against the exact persisted batch file and, for legacy resume, the exact source ledger. Persist the validator's `primary_error` and `errors` on failure. If the script, previous snapshot, batch file, legacy source when required, policy asset, registry, sibling director validator, or Python runtime is unavailable, stop before paid submission; never replace the deterministic gate with an unchecked retry.

When several errors exist, retain them all and choose `primary_error` deterministically by category: schema/required fields; SKU/variant/reference/idempotency binding; timeline/projection; differentiation; endpoint/capability/model/balance; submission reconciliation; polling/download/delivery. Sort ties by error code and path.

## Zibuyu Delivery Handoff Invariant

When the run is part of the Zibuyu apparel workflow, terminal PopBoom status is not the final deliverable. Treat a run as Zibuyu-owned when its run directory contains or is paired with `batch-compile.json`, when it is a child release under a parent `batch-plan.json`, when the user invoked the Zibuyu plugin, or when the request resumes a Zibuyu ledger, color batch, or fixed-model apparel batch.

After the terminal ledger is persisted, rejoin the parent `zibuyu-top-tiktok-operations-specialist` workflow and run its delivery-pack validator:

```text
python ../zibuyu-top-tiktok-operations-specialist/scripts/validate_delivery_pack.py <absolute-path-to-batch-compile.json> <absolute-path-to-ledger.json>
```

Resolve a working Python runtime before this command. On Windows, load the bundled workspace dependencies and use the returned absolute Python executable rather than an unverified app-execution alias. If no runtime is available, preserve `next_action: report` and block the delivery gate; never downgrade to a link-only completion.

The final user handoff is complete only after every terminal job includes its status, record ID when present, usable video URL or verified local path, failure reason when failed, rendered-quality verdict, one copy-ready caption paragraph, and exactly five unique hashtags. Use the validator's `copy_ready_caption` verbatim. A response containing only status or links is incomplete.

Set `completion_reported: true` only after that full delivery pack has been shown to the user. If execution stops after reporting links, retain `completion_reported: false` and `next_action: report` so the next resume repairs the handoff. Do not resubmit, rerender, reupload, or spend credits merely to repair omitted copy.
