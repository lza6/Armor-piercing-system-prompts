For director/ledger validator commands below, prefer `--summary --result-file <unique-run-local-result.json>`. Read eligibility, selected receipts and errors from the saved full result as needed. Before payment, read the complete fresh authorized outbound object and verify its hash; the summary cannot authorize a call.

> Stage-specific reference. Read for new production/preflight/creative changes, not routine status or delivery. All inline paths and commands in this document are relative to the owning skill directory (one level above this file), unless explicitly absolute. Markdown links resolve from this file.

# PopBoom Skill

Use the PopBoom MCP server named `PopBoom` for PopBoom video generation and TikTok open-platform workflows. The canonical connection is Streamable HTTP at `https://tkvideo.zbycorp.com:5002/mcp`; use only the existing Codex secure MCP credential configuration and never print or persist the bearer value.

Before using PopBoom tools, read `popboom-codex-skills.md` in this skill directory for the exported tool guide and examples.

## Mandatory Runtime And Batch Contract

Before any new PopBoom preflight, upload, paid submission, or exact unpaid-package resume, read `references/batch-execution-contract.md` in full.

- The contract controls the canonical HTTPS Streamable HTTP endpoint, trusted local-reference Hook, 15-second capability routing, fixed model-asset validation, persistent ledger, balance gate, SHA-256 upload dedupe, the long-term user concurrency policy, barrier waves, quality-gated one-color releases, and round-robin polling. Load `assets/execution-policy.json`; persist configured concurrency as `12`, source `user`, and `user_explicit` evidence with allowed value `12`. Barrier batches retain ordered waves. A quality-gated streaming parent uses one validated one-job ledger per color and enforces the same aggregate limit across all child releases. Never serialize this user setting as `default`.
- Keep configured concurrency 12 immutable when throttling. On `HTTP 429`, `too_many_requests`, or a concurrency-limit signal, append a fingerprint-bound `rate_limit_events` entry, set only `effective_max_inflight` to `1`, and keep the batch-scoped override latched until a genuinely new `run_id`. Every job must persist an explicit safe `next_action` and `resubmit_allowed: false`; a job with a `record_id` is poll/download/report-only and must never return to `generate_video`.
- Bind each color's exact timeline version, compiled prompt fingerprint, zero-human-identity-pixel three-view reference, fixed-model asset ID plus identity-image SHA/version, and exact ordered outbound MCP request before submission.
- Every `submission_started` paid authorization must explicitly declare ledger `workflow_kind` as exactly `zibuyu_apparel` or `generic_popboom`, and every job's `job_kind` must equal it. Omission is allowed only for historical poll/report ledgers and never authorizes a paid call. Any planned or paid job carrying an `apparel_three_view` reference is unconditionally a `zibuyu_apparel` job; labeling it `generic_popboom` is a rejected downgrade and never bypasses the director receipt or exact batch-compile gate.
- For default new Zibuyu paid work, set ledger `batch_compile_sha256` to the SHA-256 of the exact persisted batch-compile file bytes. Persist each job's exact schema `1.4` director receipt with `quality_contract_id: zibuyu_ugc_quality_v4`, `serializer_id: canonical_prompt_v7`, `deadline_contract_id: zibuyu_three_layer_deadlines_v1`, `three_layer_deadlines_sha256`, `market_prompt_contract_id: zibuyu_market_prompt_v1`, `market_prompt_profile_id`, `market_prompt_profile_sha256`, `generation_controls_sha256`, `voiceover_review_sha256`, `market`, `voiceover_language`, the existing variant/timeline/research/quality/canonical-beat/compiled-text hashes, and both validator booleans true. Accept only the closed market tuple `US + 美1/美2/美3 + en-US + us_champion_v1` or `DE + 德1/德2/德3 + de-DE + de_champion_v1`. Persist registry-matching `asset_identity_sha256` and `asset_identity_version`, and an exact `outbound_request` plus its SHA-256. The paid fingerprint and validator-returned outbound authorization must bind the full receipt, three-layer deadline contract, market profile, model-first reference order, all image SHA values, and the complete outbound payload. Historical v3/v6 receipts remain read-only.
- Keep schema `1.3` / `canonical_prompt_v5` historical jobs readable for polling, download, delivery, and audit. They are not generally eligible for a new paid call. The only exception is `legacy_v5_exact_resume_v1`: create a new resume run, pass the exact prior planned ledger through `--legacy-source-ledger`, bind explicit user approval plus the prior ledger, batch, director-receipt, and full paid-package hashes, and assert that prompt, reference order, model identity, settings, receipt, and outbound request are unchanged. The source job must have been previously valid and eligible but never submitted. Any job with a `record_id` is permanently poll/download/report-only and must never be migrated, re-receipted, or returned to `generate_video`.
- When the exact compile contains `streaming_release`, require the sibling director validator to load and validate its immutable all-color plan before accepting the receipt. Streaming supports 2-12 planned colors. Use one complete release compile and one ledger job for that color, and require both `streaming_release.child_run_id` and ledger `run_id` to equal `<batch_run_id>:<variant_id>`; never submit from the draft plan alone.
- Run `scripts/validate_ledger.py` after preflight, before every paid submission batch, after every submission response, and after terminal updates. Persist revision 0 as preflight-only; for every later revision pass the immediately previous snapshot with `--previous-ledger`. Before a Zibuyu paid call, also pass the exact persisted file as `--batch-compile <absolute-path-to-batch-compile.json>`; the validator must load the current sibling director validator and independently verify the file, selected variant, timeline, prompt hashes, receipt, and apparel reference owner. For streaming, it also scans every sibling release and rejects parent-plan hash drift or cross-child concurrency/rate-latch violations. Before paid calls, validate the exact released `submission_started` set and require `history_verified: true`, `wave_plan.paid_submission_allowed: true`, and matching `authorized_submission_job_keys`; this set may contain up to 12 jobs normally and only 1 after a rate-limit latch. Stop on any nonzero or `valid: false` result.
- If older text in this skill or `popboom-codex-skills.md` conflicts with the contract, the contract wins.
- Keep `mcp_provisional` historical and poll/report-only. For new 15-second submissions whose live schema omits 15, use `mcp_observed` only while `assets/runtime-capabilities.json` contains a validator-accepted successful record and `valid_for_new_submission: true`. The current baseline is successful `record_id: 200704` from 2026-07-30; it supersedes the 2026-07-20 negative observation for routing, without rewriting that older evidence.


## Tool Access Gate

Before starting video generation, confirm which PopBoom execution path is available and apply duration routing before preferring MCP:

- If the MCP inventory lists PopBoom but a required tool is not callable in the current turn, use the platform tool-search capability to load that exact PopBoom tool before declaring it unavailable. This is a non-paid discovery step. Continue only after the callable schema is exposed; report a blocker only when exact tool search also fails.
- For a duration explicitly declared by the normalized live schema, prefer exposed PopBoom MCP tools, especially `query_balance`, `list_custom_portraits`, `upload_images`, `generate_video`, and `check_task`, and record `route: mcp_declared`. For local reference images, also require the trusted plugin Hook described below. Treat `download_video` as conditional when a successful task lacks a usable URL or a local mirror is required.
- For a 15-second request omitted by the live schema, use MCP with `route: mcp_observed` only when the machine baseline still marks the successful accepted observation valid for new submissions and the ledger validator accepts the exact evidence binding. Never use `mcp_provisional` for a new call and never substitute 10 or 30 seconds.
- For every fixed-model apparel request sent through MCP, serialize `ref_image_urls` in strict role order: item 1 must be `asset://<validated_fixed_model_asset_id>` and item 2 onward must be garment/product references. The compiled prompt must describe the same order (`Reference 1 / @Image1` is fixed-model identity only; `Reference 2 / @Image2` onward is garment identity only). Never place the garment before the fixed-model asset. This order is a paid-submission gate, not a prompt preference.
- Model-first ordering reduces the known identity-mismatch risk but does not guarantee exact identity. The 2026-08-03 legacy 美1 test `record_id: 204446` used the validated asset first and the garment second, yet rendered a hybrid face instead of the then-approved portrait. Under the current user-approved workflow, do not require a separate first-frame image; treat identity accuracy as a rendered-frame QA gate and stop the batch on `identity_mismatch`.
- For every new fixed-model Zibuyu apparel request, require a garment artifact/binding with `human_identity_pixels_absent: true` plus the same hash-bound `zero_human_identity_pixels_v1` cue audit on the batch reference, ledger binding, and uploaded artifact, with every human cue false. Retain the fixed-model asset first in `ref_image_urls`, keep garment references second onward, and never upload model portraits/photos as reference material. If any garment-reference binding, SHA, cue-audit, or model identity pin is missing, stop before video payment.
- Never hand-compose the paid call from ledger prose. Run the ledger validator and use only the exact `outbound_request.arguments` returned for the authorized job in `wave_plan.authorized_outbound_requests`; recompute its `outbound_request_sha256`, then send that object verbatim to the returned PopBoom `generate_video` tool. Do not reorder URLs, replace the prompt, or add parameters after validation.
- If an `mcp_observed` call returns an explicit invalid-parameter rejection without a `record_id`, disable that route for subsequent new submissions in the run and use the PopBoom browser/Windows UI workflow. If any `record_id` is returned, treat the request as accepted and poll/download/report only; never retry it.
- For any other duration absent from the live declaration, use the PopBoom browser/Windows UI workflow through available browser or computer-use tools.

If neither MCP tools nor a usable UI path is available, stop before submitting the job, report the exact blocker, and do not claim that the video was submitted or generated.

## Local Reference Upload

Use the plugin's trusted `PreToolUse` Hook for every local apparel reference sent to `mcp__PopBoom__upload_images`.

1. Before staging a new apparel three-view, require its visual-QC receipt and the batch reference, ledger binding, and artifact records to declare `human_identity_pixels_absent: true` and carry one identical hash-bound `zero_human_identity_pixels_v1` audit for the exact image SHA-256. Every cue must be explicitly false: skin, face, hair, neck/chest/collarbone, shoulders/arms/wrists, hands/fingers/nails, tattoos/jewelry, and person-specific body shape. Any visible cue or missing/stale audit fails the upload gate. Then copy the approved image byte-for-byte into the current run's `references` or `artifacts` directory under `$CODEX_HOME/zibuyu-runs/<run_id>`. Do not decode, resize, recompress, or re-encode it while staging.
2. Compute and persist its original SHA-256, byte size, mtime, filename, and `upload_transport: local_file_hook` before upload.
3. Pass `image_data` as an absolute percent-encoded `local-file:///...` URI and pass the matching base filename. The trusted Hook reads the file locally and replaces that short URI with Base64 immediately before the existing PopBoom MCP call.
4. Never have the model or shell generate, echo, paste, or save Base64 in chat text, a ledger, or an intermediate file. The Hook-created argument is host-managed, but a Codex CLI diagnostic tool-call view may still render it; treat that as a host display limitation, never copy it, and use a fresh Codex Desktop task for normal runs. Never pass a raw Windows path as `image_data`.
5. Preserve the original image by default. Do not create a 160px/320px upload copy or silently reduce dimensions. If the original exceeds the documented 20 MB image limit or PopBoom returns an explicit size rejection, stop before paid generation and report the exact failure. A transformed derivative requires explicit user authorization and separate provenance.
6. If the Hook is untrusted, skipped, absent, or fails validation, classify the upload as `local_upload_hook_unavailable`, stop before `generate_video`, and report the Hook problem. Do not fall back to terminal Base64.
7. Reuse an existing accessible URL only when its ledger SHA-256 exactly matches the current original file; record that as `upload_transport: url_reuse`.

## Advanced Video Generation UI Workflow

Use this workflow when the user asks to generate a PopBoom video from a Seedance UGC script, product/clothing images, or a clothing-three-view output.

Required inputs before starting:

- A complete script/prompt from `$seedance-ugc-cn-director`.
- For default new Zibuyu apparel, persist the exact schema-1.4 machine receipt returned by the current director validator as the job's fingerprint-bound `director_receipt`, including the market/profile/control/voiceover-review bindings and both eligibility booleans true. Keep the exact validated batch-compile JSON on disk so `validate_ledger.py --batch-compile` can independently reproduce the product/review provenance, market tuple, pain/claim/proof bindings, structured generation controls, bilingual voiceover review, and prompt hashes immediately before payment. A legacy-v5 exact resume additionally requires the exact prior planned ledger through `--legacy-source-ledger`; link-only, copied prose, a ledger-local self-assertion, or an unbound historical schema is not enough.
- A white-background clothing three-view reference image from `$clothing-three-view` when the product is apparel.
- Uploaded reference material must contain product/garment imagery only; never upload model portraits/photos, including 德1, 德2, 德3, 美1, 美2, 美3, or their source images. A validated `asset://` fixed-model entry is not an uploaded material and must occupy `ref_image_urls[0]` for MCP fixed-model apparel requests. The default workflow does not use `first_frame_url`.
- One virtual-human model name: 美1, 美2, 美3, 德1, 德2, or 德3. If the user has not named one, ask for the model name before generation.
- One batch item number (`货号`) for the videos being generated. If the user has not provided the batch item number, ask for it before any PopBoom paid generation. Do not ask for the brand department: it is always fixed to `拓展平台`.

Default settings:

- Platform: TikTok.
- Generation model: Seedance 2.0.
- Product information: required for every PopBoom video generation. Fill 品牌事业部/brand as `拓展平台` and 货号/sku as the current user-provided batch item number. For MCP `generate_video`, pass `product_info` as the compact JSON string `{"brand":"拓展平台","sku":"<货号>"}`. If the item number is missing, stop and ask for it before generation.
- Voice: leave empty unless the user explicitly asks to choose a voice.
- Aspect ratio: 9:16.
- Generate audio: on.
- Web search: off.
- Watermark: off.
- Duration: 15s.
- The current MCP tool description declares 5/10/30/60/120 seconds and omits 15, while the runtime accepted and completed the 2026-07-30 15-second baseline. Use `mcp_observed` only through the validated evidence route above; otherwise use the UI route below.
- Quality: 720p. Use another resolution only when the user explicitly requests it.
- The 720p rule applies to every fixed custom model. Never upgrade US presets to 1080p because of UI memory, model preset behavior, or perceived output quality.
- For MCP `generate_video`, explicitly pass `resolution: "720p"` unless the user has requested another resolution. Do not omit `resolution`.

UI steps:

1. Open the PopBoom platform.
2. In the left sidebar, choose 做视频, then choose 高级视频生成.
3. Keep any unspecified fields at the platform defaults listed above.
4. Paste the exact validated `renderings.prompt.compiled_text` from `$seedance-ugc-cn-director` into 提示词 verbatim. Recompute `compiled_text_sha256` and `research_bundle_sha256`, require them to equal the batch and director receipt, then recompute the receipt-aware `request_fingerprint` immediately before submission. Never paste raw Amazon reviews, research notes, only the voiceover/script section, or a paraphrase.
5. Upload the clothing three-view image from `$clothing-three-view` into 参考素材. Do not upload any model photo here; select the person only through 虚拟人像.
6. Open 虚拟人像, switch to 自制模特库, and select the requested custom model:
   - 德1 selects the PopBoom custom model named 德1.
   - 德2 selects the PopBoom custom model named 德2.
   - 德3 selects the PopBoom custom model named 德3.
   - 美1 selects the PopBoom custom model named 美1.
   - 美2 selects the PopBoom custom model named 美2.
   - 美3 selects the PopBoom custom model named 美3.
   - Before payment, verify the serialized request order, not just the visible thumbnail. The 2026-08-03 `/sd2` web build serialized uploaded garments before the selected `asset://` portrait. That order produced a confirmed 美1 identity mismatch and is noncompliant for fixed-model apparel. If the UI cannot prove `asset://<fixed_model_id>` is first, do not submit through UI; use the validated MCP route with model-first ordering.
7. Set 时长 to 15s.
8. Set 清晰度 to 720p unless the user explicitly requests another resolution.
9. After selecting any fixed preset, re-check that 清晰度 still shows 720p. This is mandatory for US presets because the UI or a previous run may retain 1080p.
10. Fill 产品信息 with 品牌事业部 `拓展平台` and the current batch 货号. If the platform serializes this as MCP `product_info`, it must be exactly `{"brand":"拓展平台","sku":"<货号>"}` with the user-provided item number.
11. Confirm 生成音频 is on, 联网搜索 is off, and 水印 is off.
12. Click 开始生成.

## Submission And Completion Reporting

After submitting a PopBoom generation request:

- Immediately tell the user that the video generation request has been submitted. Include the `record_id` when available.
- Poll or monitor the task until PopBoom reports a terminal status.
- When the task reaches `succeeded` or the UI shows completion, tell the user that the PopBoom platform generation task is complete. Include the `record_id`, `video_url`, and status when available.
- For a quality-gated fixed-model batch, extract and visually inspect representative beginning, middle, and ending frames before releasing another color. Verify face identity, skin tone, hair, and single-person continuity against the selected custom portrait, then verify garment color and construction. A wrong identity is terminal QA failure `identity_mismatch`: preserve the completed record as failed evidence, set `resubmit_allowed: false` for that accepted request, and block every remaining color until a corrected reference route is compiled and separately authorized or already covered by the user's quality-gated test instruction.
- For a generic standalone PopBoom task with no Zibuyu batch context, stop after the terminal status/link report.
- For a Zibuyu apparel task, do not stop at status/links. Continue through the Zibuyu Parent Workflow Handoff Gate below.
- If a generic standalone PopBoom task returns `failed`, report the failed status and visible error, then stop. For a Zibuyu job, stop paid execution for that job but continue through the parent delivery gate so its failure reason and copy package are still delivered.

For a quality-gated streaming parent, immediately resume the next unfinished color after reporting the current submission. Poll accepted child ledgers round-robin while later references/director packages are prepared. Before the first paid child, require full-plan worst-case balance coverage; before every later child, recheck current balance. A rate/concurrency signal in any child latches the parent scheduler to effective concurrency `1` for all remaining releases.

Task completion for a generic standalone use of this skill means PopBoom accepted the generation request and later reported that rendering finished. Zibuyu plugin completion additionally requires the copy delivery package.

## Zibuyu Parent Workflow Handoff Gate

Treat a run as Zibuyu-owned when any of these is true:

- the user invoked `zibuyu-top-tiktok-operations-specialist` or its plugin;
- the run lives under `$CODEX_HOME/zibuyu-runs` and has `batch-compile.json`;
- the release is nested under a Zibuyu parent run containing immutable `batch-plan.json`;
- the prompt asks to resume a Zibuyu revision, ledger, color batch, or fixed-model apparel submission.

For every such run, read `../zibuyu-top-tiktok-operations-specialist/SKILL.md` and rejoin its rendered-quality and Copy Delivery Pack gates after polling. This applies even when the current request names only PopBoom, asks only to resume revision 0, or begins after the creative compile already finished.

After the final terminal ledger update and validator pass, run:

```text
python ../zibuyu-top-tiktok-operations-specialist/scripts/validate_delivery_pack.py <absolute-path-to-batch-compile.json> <absolute-path-to-ledger.json>
```

Use a verified Python runtime. On Windows, load the bundled workspace dependencies and call the returned absolute Python path instead of assuming the `python.exe` app alias works. If Python is unavailable, keep `next_action: report` and report the delivery gate as blocked; do not fall back to a link-only completion.

Require `valid: true` and one returned item per ledger job. Deliver each item's status, `record_id` when present, usable video URL or verified local path, failure reason when failed, creative-quality verdict, and `copy_ready_caption`. The copy-ready paragraph must contain the caption followed by exactly 5 hashtags in the model market language.

For Zibuyu apparel delivery, choose exactly five unique hashtags based on the garment, market, TikTok Shop context, occasion, outfit style, and buyer search intent. Do not force a fixed brand hashtag. Do not use `#Imily Bela`; replace it with a relevant non-brand tag so the total remains exactly five. For 德1/德2/德3 German outputs, keep the caption/action language aligned with the German account learning from the parent workflow: product-visible result hook, profile/shop/comment intent, simple either/or comment trigger when appropriate, and German lower-left/profile/shop CTA wording.

Do not set `completion_reported: true` after a link-only update. In a Zibuyu run, it becomes true only after the complete status/link, quality verdict, caption, and exactly-5-hashtag package has been delivered. If links were already reported but copy was omitted, keep or restore `next_action: report` and repair only the delivery handoff; never resubmit an accepted `record_id`.

After the whole Zibuyu batch is accepted and fully delivered, return to the parent skill's [Post-Production Publishing](../../zibuyu-top-tiktok-operations-specialist/references/post-production-publishing.md) phase for integrated tasks. Do not load the publishing skill, query channels/products, or schedule one completed release while the rest of the production batch is unfinished. Explicit production-only requests end at delivery, and publishing still requires its own exact final-table authorization.

Never upload model portraits/photos into 参考素材. 德1, 德2, 德3, 美1, 美2, and 美3 source photos are only for model identity/preset recognition and must be selected through 自制模特库, not uploaded as reference material.

Always fill 产品信息 for PopBoom video generation: 品牌事业部 is fixed to `拓展平台`, and 货号 is the current batch item number provided by the user. If 货号 is missing, ask for it before generation. Do not fill 音色 during this workflow unless the user specifically asks for it.
