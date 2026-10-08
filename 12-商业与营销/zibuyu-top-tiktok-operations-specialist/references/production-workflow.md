For director/ledger validator commands below, prefer `--summary --result-file <unique-run-local-result.json>`. Read eligibility, selected receipts and errors from the saved full result as needed. Before payment, read the complete fresh authorized outbound object and verify its hash; the summary cannot authorize a call.

> Stage-specific reference. Read for new production/preflight/creative changes, not routine status or delivery. All inline paths and commands in this document are relative to the owning skill directory (one level above this file), unless explicitly absolute. Markdown links resolve from this file.

# Zibuyu Top TikTok Operations Specialist

## Mission

Turn minimal user input into high-conversion TikTok/PopBoom apparel videos.

The user should only need to provide:

- product or garment images;
- one model name: `美1`, `美2`, `美3`, `德1`, `德2`, or `德3`;
- optional overrides such as platform, duration, language, aspect ratio, or resolution.

Default to the established Zibuyu workflow when details are missing:

- Platform: TikTok / TikTok Shop.
- Video model: Seedance 2.0 through PopBoom advanced video generation.
- Aspect ratio: 9:16.
- Duration: 15s.
- Quality: 720p. Use another resolution only when the user explicitly requests it.
- Resolution is not model-specific: all fixed presets use 720p unless the user explicitly requests another resolution. Never use 1080p for a US preset by default.
- Product information field: required for every PopBoom video generation. 品牌事业部/brand is always fixed to `拓展平台`; do not ask the user for it. 货号/sku must be provided by the user for each batch. If the user forgets to provide the batch item number, ask for it before PopBoom generation and stop before any paid submission.
- Voice field: leave empty unless explicitly requested.
- Generate audio: on.
- PopBoom internal web search: off. This only controls PopBoom's generation UI setting; it does not waive Codex-side recent TikTok apparel-commerce research before scriptwriting.
- Watermark: off.

## Required Bundled Skills

Before executing the workflow, use the bundled skills in this order:

1. `clothing-three-view` for apparel reference image generation.
2. `seedance-ugc-cn-director` for conversion-oriented script and prompt writing.
3. `popboom` for video submission, model selection, task polling, and completion reporting.

The fourth phase is `zibuyu-popboom-auto-publish`, loaded only after the whole production batch passes actual-video acceptance and complete copy delivery. Follow [Post-Production Publishing](../references/post-production-publishing.md) at that point. Do not load publishing tools, look up channels/products, or build schedules during production. Publishing inputs supplied early may be saved without resolving them. An explicit production-only, manual-delivery, or no-publishing request ends after production delivery.

Read each bundled skill's `SKILL.md` before relying on it. When a bundled skill references required files, follow that skill's routing instructions.

## Bundled Skill Exposure And Fallback

This plugin owns five bundled skills: `zibuyu-top-tiktok-operations-specialist`, `clothing-three-view`, `seedance-ugc-cn-director`, `popboom`, and `zibuyu-popboom-auto-publish`. The first four implement production; the publishing skill is a later, conditional phase. The standalone `$zibuyu-popboom-auto-publish` remains available for existing videos.

In some existing Codex threads, the runtime skill list may only show one plugin-prefixed skill, usually `zibuyu-top-tiktok-operations-specialist:popboom`, because plugin skills are loaded lazily or the thread started before the plugin cache was refreshed. Treat that as a discovery limitation, not as permission to skip the missing bundled skills.

When executing this plugin workflow:

- Load the relevant production skill files from this plugin directory first, even if the current thread's visible Skills list only exposes one of them. Defer the publishing skill until the post-production gate passes.
- Use standalone copies under `C:\Users\35868\.codex\skills` only as a fallback or synchronization target when the bundled copy is inaccessible.
- Do not replace the `clothing-three-view -> seedance-ugc-cn-director -> popboom` route with a single PopBoom prompt.
- If a bundled skill is truly inaccessible from both the plugin directory and standalone directory, stop at that step and report the exact missing file or tool access blocker.
- After plugin source updates, refresh the plugin cache/version and use a new thread for testing so the visible Skills list can reload all bundled metadata.

## Tool Access Gate

Before starting a full production run, confirm that the required execution path is available:

- Three-view generation requires an image generation or image editing capability.
- Recent TikTok apparel-commerce learning requires browsing when live/current evidence is available; if browsing is unavailable, continue with local platform knowledge and clearly label it as not live-ranked.
- Amazon product/review analysis prefers the user's signed-in Chrome session when the bundled Chrome control skill and its required browser tool are exposed. Otherwise use only publicly reachable Amazon content. Never bypass login/CAPTCHA, and never replace blocked direct reviews with AI summaries, search snippets, reseller pages, or a similar ASIN.
- PopBoom video generation requires either PopBoom MCP tools (`query_balance`, `list_custom_portraits`, `upload_images`, `generate_video`, and `check_task`; `download_video` is conditional) or an operable PopBoom browser/Windows UI path through available browser/computer-use tools. When the MCP inventory lists PopBoom but a required tool is not callable in the current turn, first use platform tool search to load that exact tool; do not misclassify lazy loading as server failure. Local MCP reference upload additionally requires the trusted bundled `PreToolUse` Hook; never print Base64 through the terminal as a substitute.

For any 15-second job, first use `mcp_declared` when the normalized live schema explicitly allows `15`. If the live description omits `15`, use `mcp_observed` only while the bundled runtime baseline contains a validator-accepted successful 15-second record with `valid_for_new_submission: true`; the current baseline is completed `record_id: 200704` from 2026-07-30. Re-check the live schema each run, bind the exact observation into the ledger, and never substitute 10s or 30s. `mcp_provisional` remains historical poll/report-only. An explicit invalid-parameter response without a `record_id` disables `mcp_observed` for subsequent new submissions in that run and triggers UI fallback; any response containing a `record_id` is accepted work and may only be polled, downloaded, or reported.

If PopBoom MCP tools are not exposed, or both `mcp_declared` and valid `mcp_observed` routing are unavailable, use the browser/Windows UI workflow from the bundled `popboom` skill. If neither MCP nor UI can submit 15s, fall back to the Manual Upload Handoff below instead of dropping the prepared creative work. Do not claim the video was submitted or generated unless Codex actually submitted it through PopBoom or the user later reports completion.

## Manual Upload Handoff Fallback

Use this fallback whenever a quality-ready 15-second Zibuyu job cannot be submitted by Codex because neither `mcp_declared` nor validator-approved `mcp_observed` is available, PopBoom MCP tools are missing, the browser/Windows UI path is unavailable, or the user asks to upload manually.

- Keep the same quality-first workflow before handoff: three-view generation/QC, selling-point analysis, buyer-pain analysis, evidence binding, proof-action planning, market-profile selection, voiceover conversion logic, schema `1.4` compile, and validation remain required. Manual upload is only an execution fallback, not a shortcut.
- For each ready color, send the user the exact identity-neutral three-view reference image as a local path or rendered image, plus the exact paste-ready PopBoom prompt from the validated `renderings.prompt.compiled_text` / `canonical_prompt_v7`. Historical v6/v5 packages remain read/poll/report-only. The only unsubmitted v5 exception is a validator-confirmed `legacy_v5_exact_resume` that the user explicitly approved and whose prompt, reference order, settings, and hashes are byte-for-byte unchanged. Do not paraphrase, summarize, or substitute the voiceover review block for the paste-ready prompt.
- Include a compact manual settings checklist: Seedance 2.0, 9:16, 15s, 720p unless overridden, generate audio on, watermark off, product information filled with 品牌事业部 `拓展平台` and the current batch 货号, voice field empty unless requested, upload only an identity-neutral clothing three-view as garment reference material, choose the requested fixed custom model, and re-check 720p after model selection. If the user has not provided 货号, ask for it before generation. If the UI cannot prove model-first reference ordering, keep the job at `manual_upload_needed` instead of paying.
- Include the fixed model asset ID and market-language voiceover lines with Chinese translations so the user can verify both model choice and spoken conversion copy before uploading.
- For fixed-model apparel generation, bind the selected `asset://` model as reference 1 and identity-neutral garment imagery as reference 2 onward. The garment reference must contain zero visible human-identity pixels: no realistic skin, neck/chest, arms, hands, fingers, nails, body-shape cues, tattoos, jewelry, face, or hair. Any such cue fails reference QC and blocks downstream use. A visible portrait thumbnail is not proof of outbound binding; require model-first request serialization and rendered-frame identity QA before releasing the next color. Do not require or generate a separate identity-conditioned first-frame image for the default workflow.
- If a rendered video produces a hybrid or wrong identity, stop the batch, record `identity_mismatch`, and do not submit another color or repeat the same route without explicit user authorization.
- Clearly label the job status as `manual_upload_needed` or equivalent. Do not set submission/completion reported flags, do not fabricate a `record_id`, and do not treat the workflow as complete until the user provides a PopBoom result or asks for only the manual package.
- If the user later provides the generated video or link, resume at rendered-quality review and the Copy Delivery Pack Gate; do not rerun the paid job unless the user explicitly authorizes a new version.

## Workflow State, Routing, And Sync Discipline

Borrow the GitHub-style specialist workflow discipline for every batch production run: resolve context first, route each problem to the right bundled skill, keep artifact state aligned, and track PopBoom task completion.

For a barrier run, create or resume the persistent ledger at `$CODEX_HOME/zibuyu-runs/<run_id>/ledger.json`. For quality-gated streaming, keep immutable `$CODEX_HOME/zibuyu-runs/<run_id>/batch-plan.json` and one persistent one-job ledger under `releases/<variant_id>/ledger.json` for each color. Working notes are summaries only. Track one job per paid request fingerprint, not one row per color.

- color / variant name;
- source product image paths used for that color;
- three-view image path, original SHA-256/byte size, unchanged staging path, upload transport, and three-view quality-check status;
- three-view identity-cue audit with `human_identity_pixels_absent: true`, the exact audited image SHA-256, every human cue explicitly false, and a matching canonical audit SHA-256 before upload or submission;
- selected fixed-model identity version and immutable identity-image SHA-256;
- selected fixed-model asset ID, identity version, immutable identity-image SHA-256, and model-first reference ordering proof;
- validator-accepted ordered `outbound_request` and its fingerprint; use this exact object as the only source for the paid MCP call;
- TikTok research status, including whether it was live recent-half-month research or local platform knowledge;
- Seedance script version;
- PopBoom execution path, record_id, task status, video URL or local file path;
- batch schema/quality contract, market contract/profile, prompt serializer, quality-plan hash, research-bundle hash, canonical-beats hash, market-profile hash, generation-controls hash, voiceover-review hash, and pre-submission validation status;
- rendered quality status (`keep`, `fix_in_post`, `reroll`, `rewrite`, `stop_or_rescope`, or `unverified`), one `primary_failure_variable`, and concise evidence when the video can be inspected;
- submission reported, completion reported, and any PopBoom failed status/error message.

Use the ledger to resume efficiently: do not rerun a completed three-view image, script, or video unless the user asks for a new version or a dependency changed. If a color is still pending or failed, continue from the earliest failed step.

Any resumed run under `$CODEX_HOME/zibuyu-runs` that has a sibling `batch-compile.json`, or is a child of a parent `batch-plan.json`, remains an end-to-end Zibuyu workflow, even when the user asks only to resume revision 0, submit through PopBoom, poll records, or return completed links. A direct component-skill invocation must rejoin this parent workflow at rendered-quality review and the Copy Delivery Pack Gate below. Never narrow a Zibuyu resume into a link-only PopBoom handoff.

Classify failures before repairing:

- `reference failure`: wrong/missing three-view, non-white background, wrong crop, color mismatch, or inconsistent front/side/back. Route to `clothing-three-view`.
- `script/prompt failure`: shallow feature-first selling-point analysis, weak or non-pain-based hook, sparse/overloaded or flat equal-stress voiceover, unsupported claims, wrong language, three-view layout/identity bleed, missing exact reference tag, actionless product explanation, stiff attention posture, disconnected equal-energy acting tasks, repeated symmetric hip resets, unmotivated distance changes, silent phone/prop hand switches or disappearing objects, no emotional rise, slow fashion-film pacing, idle explanatory beats, constant-speed limb motion, a garment claim without a matching part close-up/action/endpoint, unclear hand trajectories, simultaneous action overload, an outward-wear garment defaulted to bedroom/closet/bathroom/home without a valid scene strategy, unstable scene instructions, or CTA wording that invites stickers/icons. Route to `seedance-ugc-cn-director` before PopBoom submission.
- `generation failure`: PopBoom job error, upload/model-setting issue, polling failure, failed task status, or missing video URL after completion. Route through `popboom` and report the exact failed status or blocker.
- `tool/access failure`: missing MCP tools, blocked browser/UI path, or missing token. Stop at the blocked step and report the exact blocker.

For repeated pre-submission script/prompt failures, cluster by behavior before rewriting prompts. If three colors have the same script problem, repair the shared script/prompt pattern once and apply the corrected pattern to the remaining colors instead of patching each color differently.

Quality-first acceleration rules:

- Default to quality-first production, not a lightweight shortcut mode. Never speed up by weakening product selling-point analysis, buyer-pain analysis, evidence binding, proof-frame planning, voiceover conversion logic, three-view QC, schema validation, ledger validation, or rendered-quality review.
- Speed up only by reducing duplicate work, idle waiting, and unnecessary user-facing process reports. Keep internal artifacts compact when the user has not asked to inspect them, but keep the underlying evidence/action/quality gates complete.
- Start compatible lanes as soon as prerequisites are stable: source grouping, shared evidence/research, per-color three-view generation, draft timeline planning, caption/hashtag preparation, PopBoom polling, and later-color preparation may overlap when the main agent can still inspect and authorize every gate.
- Reuse shared product/category/TikTok research, pain-solution maps, stability templates, validation outputs, uploaded assets, and completed ledgers for the same SKU family when the evidence still applies. Do not rerun completed work unless the dependency changed or the user requests a new version.
- During PopBoom polling or UI waits, continue preparing pending color releases, delivery copy, and validation repairs that do not require the unfinished video. Do not wait idly when useful downstream work is unblocked.
- User updates should be concise state changes and blockers, not full internal reasoning dumps. Still expose reviewable voiceover lines with Chinese translations, final prompt/package details, task statuses, quality verdicts, and copy-ready captions when those are deliverables.

Pain-first selling-point rule:

- Always start apparel selling-point analysis from the customer's real wearing anxiety, purchase hesitation, or review-derived complaint, not from a visible feature inventory. First ask what the buyer is afraid this garment will do to her body, age impression, price impression, comfort expectation, coverage, occasion fit, or styling effort.
- Convert visible features only after they answer that anxiety. Color, neckline, fabric texture, hem, sleeve, length, print, pockets, buttons, crochet, or layered details are not selling points by themselves; they become sellable only when they solve a buyer problem such as looking slimmer, more proportional, less boxy, less childish, less cheap, less clingy, less revealing, easier to style, or more occasion-appropriate.
- For every candidate, write the chain `buyer pain -> desired wearing result -> garment evidence -> proof action -> visible endpoint -> spoken conversion line`. If the chain cannot be completed, downgrade the feature to background styling detail or exclude it from the 15-second script.
- Do not avoid high-conversion phrases such as slimming, waist-defining, leg-lengthening, non-clingy, secure, polished, or not childish when the garment evidence can visibly support a qualified version. Phrase them as visual wearing results, not absolute body-change promises.

Occasion-first scene rule:

- Treat the location as a selling point and audience-interest source, not decoration. Before choosing the creative form, state where the buyer would wear the garment, what occasion-specific styling uncertainty she has, and the complete outfit answer the video will demonstrate.
- Read `../seedance-ugc-cn-director/references/three-layer-deadline-contract.md` and `../seedance-ugc-cn-director/references/occasion-scene-routing.md`; persist the required global and asset deadline layers plus `generation_controls.scene_strategy` before the canonical timeline. Every v7 beat carries a matching machine `scene_id` and one complete shot deadline.
- For outward-wear garments, default private interiors to excluded. Café, beach/boardwalk, commercial street, sidewalk, office/commute, travel, entryway, and other real-use scenes should be considered from garment evidence and buyer occasion before bedroom, closet, bathroom mirror, or generic home.
- A private interior is allowed as a short proof-only opening or a specifically justified fit/detail/multi-styling primary scene. Convenience, natural light, generic UGC feel, and generation stability are not sufficient justifications.
- Route the video form from the scene and proof need: destination outfit share, GRWM departure, friend-filmed walkthrough, fixed-phone fit proof, mirror-to-destination match cut, travel pack-and-wear, multi-styling switch, or home try-on. Do not reuse one video form across unrelated garments by habit.
- Apply US and Germany differently: US may lean more boldly into café, beach, commercial-street, market, event-arrival, and travel interest; Germany uses equally concrete real-use scenes with calmer, plausible daily-life treatment and no forced landmarks or tourist signaling.

## Dual-Market Prompt Compiler Routing

Select the market profile before choosing a creative template or writing a timeline. Read `../seedance-ugc-cn-director/references/market-prompt-compiler-contract.md`, then load exactly one market profile: `../seedance-ugc-cn-director/references/us-winning-prompt-profile.md` for `美1`/`美2`/`美3`, or `../seedance-ugc-cn-director/references/de-winning-prompt-profile.md` for `德1`/`德2`/`德3`. The United States and Germany profiles share the evidence/action skeleton but are separate compilers; never translate one market's finished prompt into the other language and call it localized.

- New work uses `schema_version: "1.4"`, `quality_contract_id: zibuyu_ugc_quality_v4`, `market_prompt_contract_id: zibuyu_market_prompt_v1`, `deadline_contract_id: zibuyu_three_layer_deadlines_v1`, and `canonical_prompt_v7`. Historical `zibuyu_ugc_quality_v3` / `canonical_prompt_v6` packages are read-only.
- Resolve fixed presets deterministically: `美1`/`美2`/`美3 -> US + en-US + us_champion_v1`; `德1`/`德2`/`德3 -> DE + de-DE + de_champion_v1`. Reject a conflicting market, language, or profile instead of silently choosing one field.
- Preserve the common chain `buyer pain -> core sellable wearing result -> evidence-bound claim -> exact part -> matching frame/action -> visible endpoint -> matching spoken line`. Evidence, identity, safety, and unsupported-claim rules always override corpus-derived market defaults.
- Treat the sales-champion corpus as descriptive structure, not causal proof. Do not load the raw workbook, full source prompts, row-level statistics, or Chinese review translations into the runtime generation prompt.
- Keep historical schema `1.3` / `canonical_prompt_v5` artifacts available for audit, accepted-job polling, download, and reporting. A `record_id` is an immutable accepted job and must never be migrated or resubmitted. The narrow `legacy_v5_exact_resume` path applies only to an already validated but unsubmitted package after explicit user approval and exact prompt/reference/settings/hash verification.

Batch efficiency rules:

- Group all product images by color before starting creative work.
- Reuse one recent-half-month TikTok apparel-commerce research pass for the same product category, country, platform, and model preset; then adapt hooks, styling, caption, and hashtags per color.
- For a stable same-SKU multi-color batch, lock one immutable all-color evidence/differentiation plan, then release each color only after that color's three-view and full director package pass. Do not wait for unrelated colors after the current release is eligible.
- Fall back to the all-references barrier workflow when the intended color list, source grouping, same-SKU construction, market/model, or shared evidence is uncertain.
- Reuse the selected template's stability rules and timeline grid, not complete shot content. Every color pair must pass the bundled differentiation gate before submission.

## German Account Learning Overlay

Apply these rules whenever the selected model is `德1`, `德2`, or `德3`, the market is Germany, the language is German, or the user asks for German TikTok/TikTok Shop apparel output.

The July 2026 `nanettefei5` TikTok Studio export showed that recommendation traffic can arrive before conversion signals mature: 51,859 active-period views with only 0.43% profile-view rate, 0.93% total engagement rate, and recent 28-day comments at zero. Recent videos also had a rising returning-viewer share but weak profile visits. Treat this as a standing creative lesson for German scripts and prompts: do not only chase views; build every video to improve profile clicks, comments, and purchase-intent actions.

- Keep the first 1-2 seconds product-visible and result-led. Open on the garment already worn, with the buyer hesitation or wearing result visible; avoid generic walk-in, empty room, slow fashion posing, or plain product introductions.
- For German basics, shirts, blouses, V-neck tops, black cardigans, and summer/everyday pieces, prioritize hooks around `Basic but not boring`, `loose but still shaped`, `quickly dressed but polished`, `V-neck/fit detail`, `button/pocket/detail proof`, and `brunch/daily/vacation outfit use` when the garment evidence supports those angles.
- Add a comment trigger to the content plan or caption whenever it does not weaken the sale. Prefer simple German either/or prompts such as `Schwarz oder Hellblau?`, `Offen oder geschlossen tragen?`, `Urlaub oder Alltag?`, or `Jeans oder Shorts?`.
- Keep the final spoken CTA as a lower-left-link prompt, but make the caption and delivery pack also support profile/shop movement with German wording such as `unten links`, `im Profil`, or `welcher Look passt besser` when appropriate.
- Judge candidate scripts by profile-view intent, commentability, and product proof before raw view potential. A high-view/low-engagement structure is not a winning template unless the hook, proof, and CTA are rewritten to create action.
- For German Zibuyu apparel delivery, hashtags must be exactly five total. Choose all five from garment, German TikTok Shop, occasion, outfit context, style, and buyer search intent; do not force a fixed brand hashtag. Do not use `#Imily Bela`; replace it with a relevant non-brand tag so the total remains exactly five.

## Quality-Gated Streaming Default

Read `../seedance-ugc-cn-director/references/streaming-quality-contract.md` for same-SKU multi-color production. It changes scheduling only; every existing evidence, three-view, schema `1.4`, market-profile, canonical timeline, action/anatomy, prompt, paid-call, rendered-quality, and delivery requirement remains mandatory.

- Persist immutable `batch-plan.json` before the first paid release. It contains all intended colors, the locked shared core, every color's creative delta, planned non-CTA intent/visual signatures, captions, hashtags, and the inputs/hash for a complete recomputed pairwise differentiation result.
- When parallel-agent capacity is available, the main agent may delegate only shared research/plan drafting while it owns reference generation. The main agent must first read the owning skills/contracts, independently inspect every returned artifact, and run all deterministic gates. A delegated lane may never authorize payment, submit PopBoom work, or mutate an accepted ledger.
- Give each color an independent release directory containing one complete `single_compile` object and one one-job PopBoom ledger. Its release file hash and director receipt remain immutable after submission.
- Use streaming only for 2-12 planned colors. Above 12, use the barrier wave workflow so aggregate concurrency stays executable rather than implicit.
- Submit the current color only after `../seedance-ugc-cn-director/scripts/validate_streaming_plan.py`, `../seedance-ugc-cn-director/scripts/validate_batch_compile.py`, and `../popboom/scripts/validate_ledger.py` all pass and the ledger authorizes exactly that paid job.
- Poll accepted releases while generating, checking, or finalizing later colors. Across all releases, honor the same parent concurrency ceiling and rate-limit latch.
- Never edit the immutable plan after the first paid release. If a later reference contradicts the locked SKU or proof plan, block that color or move unreleased colors to a new run; never weaken quality gates or mutate earlier paid work.

## Same-SKU Batch Compiler And Single Timeline Source

Read `../seedance-ugc-cn-director/references/sku-batch-timeline-contract.md`, `../seedance-ugc-cn-director/references/quality-directing-contract.md`, `../seedance-ugc-cn-director/references/motion-realism-and-commerce-cadence.md`, `../seedance-ugc-cn-director/references/amazon-product-review-evidence-contract.md`, `../seedance-ugc-cn-director/references/streaming-quality-contract.md`, and `../popboom/references/batch-execution-contract.md` before multi-color creative work or any PopBoom submission.

- Compile an eligible same-SKU color partition once: Amazon/product evidence collection, garment analysis, TikTok research, pain/proof mapping, and template selection are shared, while selected-variant and review-scope limits remain explicit.
- Keep one independent canonical timeline per color. Compile that color's script, B-roll, and final prompt from the same timeline and version; never write the three artifacts independently.
- For every new paid submission, require schema `1.4`, `quality_contract_id: zibuyu_ugc_quality_v4`, `market_prompt_contract_id: zibuyu_market_prompt_v1`, `deadline_contract_id: zibuyu_three_layer_deadlines_v1`, `canonical_prompt_v7`, `amazon_product_review_evidence_v1`, the exact allowed model/market/language/profile tuple, exact identity-first interface-tag/reference-role locks, a provenance-locked research bundle, buyer pain-solution map, garment-first fidelity allocation, a claims registry, exact claim/proof/beat/caption evidence bindings, market-specific generation controls with a complete occasion-first `scene_strategy`, a voiceover review, a three-phase creator performance arc around five to seven internal beats, three to four contiguous setup runs in a default 15-second clip, at least four non-CTA product/styling actions, valid scene/speech/motion/hand/posture plans, complete global audiovisual, asset-invariant, and per-shot physical deadlines, `three_layer_deadlines_sha256`, `quality_plan_sha256`, `research_bundle_sha256`, `canonical_beats_sha256`, `market_prompt_profile_sha256`, `generation_controls_sha256`, and `voiceover_review_sha256`. Historical v6/v5 artifacts are read/poll/report-only except for the strict, explicitly approved `legacy_v5_exact_resume` path.
- Apply the seller-winning three-layer hierarchy before prompt compilation: global capture/format/text/audio rules; creator/garment/fabric/outfit/scene/light/sound invariants; then per-beat action physics, anatomy, garment state, handoff continuity, and explicit forbidden outcomes. Reject any prompt that has only shot-level physical bans or only global realism prose.
- Use the director validator's compile mode to build the compact deterministic Shot blocks. Internal intent, motivations, IDs, and beat JSON stay in the machine object; PopBoom receives only the concise reference lock, visible actions/endpoints, camera/light, canonical speech/sound, and compact constraints.
- Keep each video meaningfully different. Every color pair must differ in at least three creative dimensions spanning attention, visual world, and proof/action; changing only color, caption, hashtags, CTA, or random noise is invalid.
- Submit one distinct PopBoom job per color after prompt/reference/model binding passes. In quality-gated streaming mode, use one immutable one-job release ledger per color under the parent run and submit it as soon as its own gates pass; the parent scheduler enforces full-plan balance coverage, user concurrency `12`, round-robin polling, and one shared rate-limit latch. In barrier mode, retain the existing ordered waves. Every ledger snapshots `max_inflight: 12`, `max_inflight_source: user`, and `user_explicit` evidence with allowed value `12`. Persist `resubmit_allowed: false`; any accepted record ID is poll/download/report-only and must never return to `generate_video`.
- Persist `batch-compile.json` and `ledger.json`; require the director validator to return both `valid: true` and `eligible_for_new_submission: true`, and require the ledger validator to return `valid: true`, before any paid PopBoom call.

Plugin-sync rule:

- Whenever a bundled plugin skill is changed, mirror the same optimization into the corresponding standalone skill under `C:\Users\35868\.codex\skills` when that standalone skill exists.
- Whenever a standalone skill is changed, mirror the same optimization into this plugin's editable bundled source. Never hand-edit the installed plugin cache; refresh it through the plugin cachebuster and personal-marketplace reinstall flow, then verify source/cache hashes.
- Keep the main plugin workflow and the component skills consistent. The main workflow should route and coordinate; the component skills should own the detailed execution rules.

## Fixed Model Presets

Treat the exact model names below as complete presets. Do not ask again for country, language, or virtual-human identity when one is provided.

| Model | Country / market | Spoken language | Fixed PopBoom asset ID | Source photo use |
| --- | --- | --- | --- | --- |
| 德1 | DE | de-DE | `asset-20260703091345-n2q24` | Model identity only; never upload to reference material |
| 德2 | DE | de-DE | `asset-20260703091441-5rfz2` | Model identity only; never upload to reference material |
| 德3 | DE | de-DE | `asset-20260819170940-ltnxb` | Model identity from `D:/Codex内容/德3.jpg` only; never upload to reference material |
| 美1 | US | en-US | `asset-20260810141830-tbcnd` | Model identity from `D:/Codex内容/美1.jpg` only; never upload to reference material |
| 美2 | US | en-US | `asset-20260819170810-kbchw` | Model identity from `D:/Codex内容/美2.jpg` only; never upload to reference material |
| 美3 | US | en-US | `asset-20260819170855-8927v` | Model identity from `D:/Codex内容/美3.jpg` only; never upload to reference material |

Validate the selected fixed asset ID in the active personal tenant at batch preflight. Do not resolve these presets by Chinese-name search alone.

For every new fixed-model apparel submission, also match the selected preset's immutable identity version and identity-image SHA-256 from the bundled registry. An accessible asset ID or matching visible thumbnail alone is insufficient. Stop if the source image, remote thumbnail, ledger binding, or registry digest disagrees.

For every fixed-model apparel video, include one persistent upper-left fit-stats overlay in the Seedance/PopBoom prompt from the first frame to the final frame. The overlay must contain only the selected model's data, no `Model` label:

- 美1: `5'6" / 115 lb / Size S`.
- 美2: `5'6" / 200lb / Size 2XL`.
- 美3: `5'6" / 200lb / Size 2XL`.
- 德1: `168 cm / 52 kg / Größe S`.
- 德2: `168 cm / 52 kg / Größe S`.
- 德3: `168 cm / 90 kg / Größe 2XL`.

Place it in the upper-left safe area, make it visible at a glance with clean white text plus subtle dark outline or translucent dark strip, and do not cover the model, garment, hands, face, or product-proof details. This overlay is the only allowed generated on-screen text; still forbid subtitles, captions, watermarks, arrows, stickers, icons, badges, product-link graphics, and UI overlays.

If the user gives a different model name, ask for country/market, spoken language, PopBoom custom model name, and reference image path before generating.

## End-To-End Workflow

1. Intake the user's product images and selected model.
2. Group product images by garment color. If the color grouping is unclear, infer conservatively from filenames and visible garment color; ask only when grouping cannot be determined.
3. Create or resume the persistent run ledger and assign stable SKU-family, variant, batch-compile, timeline, artifact, and job identifiers.
4. Start the shared evidence/research pass and the per-color three-view lane as soon as grouping is stable. Treat this as parallel quality work, not a shortcut: product selling points, buyer pains, evidence scope, pain-solution mapping, recent TikTok learning, and proof-action planning remain mandatory. When an Amazon URL or review request exists, first lock the requested/parent/child ASIN and selected variant; separate catalog, seller, direct-review, buyer-image, summary, and third-party sources; preserve conflicts and exact review scope; build the pain-solution map; then add recent TikTok research or its labeled fallback. When parallel-agent capacity is available, a bounded creative lane may draft the shared plan while the main agent owns reference generation and every quality gate.
5. Before the first paid release, lock and validate immutable `batch-plan.json` under the streaming contract. It covers every intended color and passes the complete pairwise differentiation plan. If the plan cannot be locked confidently, use the barrier workflow.
6. Generate one white-background three-view image per color with `clothing-three-view`. Treat it as the only PopBoom apparel reference. A passing color must have a `zero_human_identity_pixels_v1` full-resolution cue audit bound to the exact image SHA-256; `human_identity_pixels_absent: true` by itself is not a release gate. As soon as one color passes its source-versus-output QC and cue audit, copy it byte-for-byte into that release's run directory, persist SHA-256/bytes/mtime and the audit receipt, and mark it for the trusted local-file upload Hook; never resize or recompress it. Then create that color's independent release without waiting for the remaining references.
7. For every release, build one complete schema `1.4` `single_compile` from the locked shared core, immutable market profile, and that color's exact planned delta/generation controls. Finalize only after the reference path, SHA-256, interface tag, and QC are known. Require a separate canonical timeline, voiceover review, and compiled prompt for every color and block any plan drift.
8. The script must optimize for GMV and conversion:
   - begin with buyer pain, not surface attributes: identify the customer's real wearing anxiety or purchase hesitation first, then decide which garment features solve it;
   - analyze catalog-backed, seller-attributed, user-provided, and visibly provable garment selling points separately;
   - analyze direct-review buyer pains with actual sample size and variation scope; do not turn one comment into consensus;
   - map each selected pain to an independently supported solution claim and visible proof; unresolved complaints stay risk-only;
   - before writing hooks or selecting a template, lock one `core_sellable_wearing_result`: the single buyer-relevant wearing outcome most likely to drive purchase and visibly provable in 15 seconds. Choose it from buyer hesitation plus garment evidence, not from a fixed category habit; secondary details may appear only when they prove that result;
   - before selecting the video form or timeline, lock one occasion-first scene strategy: classify outward/home/mixed wear, name the real destination, state the buyer's styling question, give the complete outfit answer, select the primary scene and video form, and enforce the private-interior gate. The location must help sell the use case, not merely provide a stable background;
   - reject shallow outputs that merely restate what viewers can already see, such as only naming color, neckline, texture, print, sleeve, hem, or decoration without explaining what purchase anxiety that detail resolves;
   - persist the research bundle, evidence registry, review themes, conflicts, excluded sources, pain-solution map, and claims registry; carry exact evidence IDs through every selected claim, proof, beat, caption, and director receipt;
   - combine recent-half-month TikTok learning with a creator-led PopBoom skeleton organized as three viewer-facing macro phases—recognition/result Hook, animated proof, and close/detail conviction—while retaining structural proof, alternate angle or buyer-worry proof, motion/handling endpoint, outfit/use-case proof, and a human-only CTA across five to seven internal beats;
   - for German TikTok/TikTok Shop output, apply the German Account Learning Overlay: make the hook product-visible in the first 1-2 seconds, add an easy either/or comment trigger in the plan or caption, and rewrite high-view-but-low-interaction ideas until they create profile/shop/comment intent;
   - apply the commerce-cadence contract: give the creator one continuous friend-to-friend intention; deliver a newly completed product proof or styling result about every two to three seconds; write every beat as inherited posture/hand/prop state -> one motivated action -> readable endpoint -> natural continuation or intentional cut; use natural acceleration/deceleration, weight transfer, connected arm mechanics, garment lag/settling, and supportive micro-movements instead of slow fashion posing or constant random motion;
   - preserve filming/task hand and object continuity. In mirror selfie POV, one hand retains the phone through a continuous take, another garment/accessory remains owned until an explicit place-down/handoff/cut, the prior endpoint normally becomes the next start, and step-back/turn/walk-closer is motivated by the proof the viewer needs; reject repeated hip resets and attention posture;
   - create a strong 1-2s hook that names the result or the hesitation it resolves, not a product-feature list;
   - when an eligible direct-review/review-theme material-expectation pain maps to an independent exact-variant texture claim, prefer a short negative `evidence_backed_contrast` Hook and immediately follow it with the mapped texture close-up, off-screen proof line, and one `pinch_release`; otherwise keep the normal question/context route;
   - use dense but natural voiceover as one connected live-share recommendation, with a restrained emotional rise from warm recognition to animated certainty to delighted conviction/relief, varied emphasis, and short proof-pivot pauses;
   - whenever writing or delivering Seedance/PopBoom prompts, director packages, or script sections, include a reviewable voiceover block for the user: each market-language spoken line exactly as planned, plus its Chinese translation. Keep this review block outside the PopBoom paste-ready prompt unless the prompt itself needs the target-language spoken line;
   - write every line for natural speech rather than a product page: short everyday sentences, varied proof-line openings, no official transitions or brochure constructions, and no standalone generic praise such as only saying "looks nice", "easy to style", or their German/Chinese equivalents;
   - in a default 15-second script, prove the selected result through front/detail/side-back/action/styling angles. Name the exact target and visible behavior in each line, keep one claim per beat, and remove true-but-disconnected product details instead of averaging attention across all features;
   - include at least four non-CTA product/styling actions in 15 seconds; multiple actions are required across the clip, while each beat performs only one action at a time;
   - include outfit styling ideas, color/scenario matching, accessories, bag, jewelry, and shoes when useful;
   - give every named fabric, comfort, breathability, neckline, sleeve, hem, stitching, texture, pocket, closure, drape, or fit claim its own matching proof frame and action; do not use one generic full-body shot for several different claims;
   - demonstrate stretch or other performance only when linked evidence explicitly authorizes that action; `Low Stretch` forbids a strong pull demonstration, and an unsupported pull action is itself an invalid performance claim;
   - specify relaxed posture and both hands' start/action/end states inherited from adjacent endpoints when possible; preserve persistent phone/task/prop anchors, default to one active demonstration hand, and permit a coordinated two-hand proof only for one evidence-backed action with the mouth out of frame and torso stable;
   - finish with the selected evidence-safe verdict; add a brief lower-left link cue and real human gesture only when `commerce_cta_mode` authorizes it.
   - set one internal commercial directing intent and one functional directorial voice; make garment identity the primary fidelity spend and choose only one secondary spend;
   - bind the exact three-view interface tag to garment identity only, positively replacing its white triptych/repeated-body source layout with one creator in one continuous real-world scene;
   - use three viewer-facing macro phases around five to seven sequential internal beats and three to four contiguous setup runs for a default 15-second clip, normally with five to six short spoken units. Use no more than two visible-lip lines by default; when the selected market delivery mode and lip-sync fidelity spend both allow it, permit only the concise anchors accepted by the director validator. Every claim-proof action gets its own matching concise line;
   - for 15-second American-English creator-led mirror shares, require the full voiceover to pass the validator's safe word-count range and audition roughly 50-60 words at 200-235 WPM for intelligibility; use language-specific pacing elsewhere and require line-matched expression cues with a visible energy progression;
   - reject pre-submission scripts whose non-CTA proof/styling lines use brochure-like language, generic filler, or the same opener three or more times; conversational wording must remain evidence-bound and may not fabricate first-person purchase/testing experience;
- from this skill directory, run `python ..\seedance-ugc-cn-director\scripts\validate_batch_compile.py <batch-compile.json> --compile --output <batch-compile.json>` to materialize `canonical_prompt_v7` plus the deadline, quality, research, timeline, and market hashes, then run that same sibling validator normally and require both `valid: true` and `eligible_for_new_submission: true` before step 9. Resolve the sibling path to an absolute path when the working directory differs.
   - set ledger `workflow_kind: zibuyu_apparel` and exact `batch_compile_sha256`; for each variant, copy the validator's matching `director_receipts` item into that job's fingerprint-bound `director_receipt`, set `job_kind: zibuyu_apparel`, and require the PopBoom ledger validator to accept all receipt/hash/prompt bindings before authorization.
9. Use `popboom` to generate the video through the available execution path:
   - for MCP local references, pass the staged original as a `local-file:///...` URI to `upload_images`; require the trusted Hook to return a usable URL before any paid call, and record `upload_transport: local_file_hook`;
   - if the Hook is unavailable, stop before paid generation and report that exact blocker; do not ask the user to compress the image and do not print Base64 through a terminal;
   - use `mcp_declared` for durations explicitly declared by the normalized live `generate_video` schema;
   - for the default 15-second job omitted by a live description such as `5/10/30/60/120`, use `mcp_observed` only when `../popboom/assets/runtime-capabilities.json` still marks its accepted success evidence valid and `../popboom/scripts/validate_ledger.py` accepts the exact observation binding;
   - on an explicit invalid-parameter rejection without `record_id`, disable the observed route for later new submissions in the run and use the browser/Windows UI workflow; with any `record_id`, poll only and never resubmit;
   - for other undeclared durations, or when the 15-second observed baseline is invalid, use the browser/Windows UI workflow;
   - if neither automated path is available, report the blocker, deliver the Manual Upload Handoff for every validated ready color, and stop before claiming generation.
- use the exact validated `renderings.prompt.compiled_text` (`canonical_prompt_v7`) in the ledger `outbound_request.prompt`; never substitute section 7 voiceover text, Chinese translations, raw reviews, research notes, or a paraphrase. Recompute and match the deadline, prompt, market, and receipt hashes plus the complete ordered outbound-request fingerprint immediately before submission;
   - upload only the clothing three-view image into reference material;
   - open virtual human selection, switch to custom model library, and choose the requested model;
   - set duration to 15s and quality to 720p unless the user explicitly requests another resolution.
   - after choosing the model, especially any US preset, re-check the PopBoom quality field and force it back to 720p if the UI retained 1080p.
10. Immediately after submitting each PopBoom generation request, tell the user that the video generation request has been submitted. Include the `record_id` when available.
11. Poll or monitor the PopBoom task until it reaches a terminal status.
12. When PopBoom reports `succeeded` or the UI shows completion, tell the user that the platform generation task is complete. Include the `record_id`, `video_url`, and status when available. Do not equate this status with creative-quality acceptance.
13. If the rendered video can be inspected, review garment identity/material, claim-to-part close-up accuracy, action-to-voiceover accuracy, action coverage, the three-phase performance arc, relaxed posture, motivated distance changes, exact two-arm/two-hand anatomy, endpoint-to-start hand continuity, persistent phone/prop ownership, lip-sync, expression/emotional progression, voice energy and intelligibility, lighting continuity, visible proof, CTA purity, and pairwise differentiation. Reject a take where the model stands at attention, repeatedly resets to both hips, performs disconnected equal-energy tasks, silently switches the phone hand, loses a held prop, stays emotionally flat, shows the wrong part, or speaks a claim without its planned action. Record exactly one quality verdict (`keep`, `fix_in_post`, `reroll`, `rewrite`, or `stop_or_rescope`), one `primary_failure_variable`, and evidence. If it cannot be inspected, record `unverified`.
14. Never start a paid retake without explicit user authorization. An authorized retake gets a new timeline version, prompt hash, and job fingerprint and changes one variable only. The same defect in two takes requires rewrite/decomposition. Never resubmit an already accepted `record_id`.
15. If PopBoom reports `failed`, stop paid execution for that color and preserve its visible error; still include that failed job in the final Copy Delivery Pack Gate.
16. After the final terminal ledger update, run the Copy Delivery Pack Gate. Return the validated status/video location or failure, creative-quality verdict, caption, and exactly 5 hashtags for every job. A link-only completion response is not a completed plugin workflow.
17. After the entire intended batch has succeeded, every final video has actual-video QA verdict `keep`, and the complete delivery has reached the user, continue through [Post-Production Publishing](../references/post-production-publishing.md), unless the user requested production-only/manual delivery/no publishing. This transition prepares publishing; it does not authorize a TikTok write. Missing PID/date never delays or invalidates production delivery. Do not enter publishing from an individual streaming release while another planned color remains unfinished.

## QA, copy and completion gates

After generation, continue through [Resume and delivery](resume-and-delivery.md) for the complete original QA, pairwise differentiation, caption, completion, deliverable and hard-prohibition gates. These gates are mandatory at delivery; they are not loaded again when already present in context. Caption supplements never alter accepted compile bytes or paid receipts.
