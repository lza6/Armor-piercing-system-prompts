For director/ledger validator commands below, prefer `--summary --result-file <unique-run-local-result.json>`. Read eligibility, selected receipts and errors from the saved full result as needed. Before payment, read the complete fresh authorized outbound object and verify its hash; the summary cannot authorize a call.

> Stage-specific reference. Read for new production/preflight/creative changes, not routine status or delivery. All inline paths and commands in this document are relative to the owning skill directory (one level above this file), unless explicitly absolute. Markdown links resolve from this file.

# Seedance UGC CN Director

## Core Rule

Before producing a full UGC director plan, verify whether the blocking production decisions are known. Do not silently default to the United States or any other market. Publishing platform, video duration, and video sound/visual format are non-blocking: if platform is missing, default to TikTok; if duration is missing, default to 15 seconds; if format is missing, auto-select it after recent TikTok apparel-commerce research and a product-fit check instead of asking the user to decide.

When blocking information is incomplete, output only the Chinese confirmation block in `references/intake-checklist.md` titled `创作前必须确认的事项`. Continue to the complete plan only after the user answers all blocking items, or explicitly says one of:

- `你来定`
- `跳过确认`
- `直接按默认做`
- equivalent wording that clearly authorizes assumptions

If the user authorizes assumptions, state the assumptions briefly in the final plan.

## Mandatory Batch Compiler And Timeline Binding

For every apparel plan, read `references/three-layer-deadline-contract.md`, `references/sku-batch-timeline-contract.md`, `references/quality-directing-contract.md`, `references/motion-realism-and-commerce-cadence.md`, `references/selling-point-action-proof-methodology.md`, `references/occasion-scene-routing.md`, `references/amazon-product-review-evidence-contract.md`, and `references/market-prompt-compiler-contract.md` in full before writing hooks, scripts, B-roll, or the final Seedance/PopBoom prompt. Then read only the selected market profile: `references/us-winning-prompt-profile.md` for United States/American English or `references/de-winning-prompt-profile.md` for Germany/German. For quality-gated multi-color streaming, also read `references/streaming-quality-contract.md`.

- For two or more eligible color-only variants in one batch key, analyze the garment, research TikTok, select the template, and plan differentiation once for the batch. Barrier mode materializes one `sku_batch_compile`; quality-gated streaming materializes one immutable all-color plan plus one complete `single_compile` release per ready color.
- Give every color its own creative delta, canonical timeline, compiled prompt, caption, and five hashtags. Every color pair must pass the contract's differentiation gate; color substitution alone never passes.
- For both single and batch modes, treat the canonical timeline as the only source for timing, dialogue, camera, actions, B-roll, product proof, and CTA. Derive the script, B-roll list, and final prompt from it without independent rewriting.
- Persist the object, run `python scripts/validate_batch_compile.py <batch-compile.json> --compile --output <batch-compile.json>` to rebuild deterministic projections/hashes, then run `python scripts/validate_batch_compile.py <batch-compile.json>` before handing any prompt to PopBoom. Repair the canonical object and recompile when validation fails; never bypass this gate.
- Use `schema_version: "1.4"`, `quality_contract_id: "zibuyu_ugc_quality_v4"`, `market_prompt_contract_id: "zibuyu_market_prompt_v1"`, `three_layer_deadlines.contract_id: "zibuyu_three_layer_deadlines_v1"`, and `research_bundle.contract_id: amazon_product_review_evidence_v1` for every new compile. New schema `1.4` compilation uses `canonical_prompt_v7`. Historical schema `1.4` `zibuyu_ugc_quality_v3` / `canonical_prompt_v6` and schema `1.3` / `canonical_prompt_v5` remain read-, poll-, download-, quality-review-, and report-compatible only. `legacy_v5_exact_resume` is limited to a previously validator-approved, unpaid package after explicit user approval and only when prompt bytes, ordered references, hashes, and request controls remain exactly unchanged. Any content change requires v7 migration; any job with a `record_id` is never migrated or resubmitted.
- Give every variant one directing intent, one directorial voice, garment identity as the primary fidelity spend, one secondary spend, explicit economized elements, continuity anchors, and a reference transfer/non-transfer contract.
- Keep the commercial timeline purposes, but use three to four contiguous camera-setup runs and at least four non-CTA claim-proof actions in a default 15-second video. Multiple actions are required across the clip and may share a setup when completed sequentially; only simultaneous unrelated actions are forbidden. Reframe whenever the spoken claim moves to a different garment part.
- Apply `references/selling-point-action-proof-methodology.md` before writing section 7 scripts, section 8 B-roll, or section 9 prompts: every selected selling point must start from a buyer pain or purchase hesitation, then become one evidence-backed claim, one garment part or wearing effect, one matched frame, one simple action, one `proof_endpoint`, one concise spoken line naming that same target, explicit left/right hand anchors, and sparse realistic micro-cues.
- Apply `references/occasion-scene-routing.md` before choosing a creative format or location. Persist one occasion-first `scene_strategy` that states where the buyer wears the item, which styling uncertainty the scene resolves, the complete outfit answer, the selected video form, and the private-interior policy. Treat scene choice as part of the selling proposition and reject bedroom-by-habit routing for outward-wear garments.
- Keep directing intent, fidelity allocation, market profile, generation controls, research provenance, evidence registry, pain-solution map, claim-proof plan, motivations, and Chinese review translations in the internal object instead of feeding abstract rationale, raw reviews, or review translations to Seedance. Require the complete global, asset, and per-shot `three_layer_deadlines` object plus `three_layer_deadlines_sha256`, `quality_plan_sha256`, `research_bundle_sha256`, `market_prompt_profile_sha256`, `generation_controls_sha256`, `voiceover_review_sha256`, the concise `reference_contract_text`, and deterministic `canonical_prompt_v7` shot text to pass validation.
- Lock the three deadline layers in order: global audiovisual shell before scene or shots; creator, garment, fabric, outfit, scene, light, and sound invariants before the timeline; action physics, anatomy, garment state, continuity handoff, and forbidden outcomes for every canonical beat. Do not reduce this contract to per-shot negative prompts.
- Reserve `@Image1` exclusively for the selected fixed PopBoom creator identity. Every apparel three-view must use `@Image2` or a higher tag, carry `human_identity_pixels_absent: true`, and contain zero real-human skin, anatomy, jewelry, tattoo, or person-specific body cues. A garment reference may never occupy or alias `@Image1`.
- If older per-color, output-section, prompt-pattern, or template wording conflicts, apply the explicit authority order: identity/evidence/safety first; `zibuyu_market_prompt_v1` and the selected market profile next; then the current schema `1.4` timeline/compiler contract; German account overlay; base template; style preference. Historical v5 instructions never override new-work v7 rules.
- Keep `model_preset` as the creative key. Resolve and validate its fixed PopBoom asset only through the sibling PopBoom batch contract; never substitute a model portrait upload or a display-name guess.

## Quality-Gated Multi-Color Streaming

Use `references/streaming-quality-contract.md` when one color should submit before the remaining three-view references finish.

- Draft the shared research, evidence map, template, all-color creative deltas, captions, hashtags, and planned non-CTA timeline signatures while reference generation is in progress when a bounded parallel creative lane is available.
- Lock and validate immutable `batch-plan.json` before the first paid color. Every color pair must already pass the full attention/visual-world/proof-action differentiation plan.
- Do not finalize a color's claims, canonical timeline, prompt reference contract, or paid fingerprint until that color's three-view passed garment QC and the zero-human-identity-pixel audit, and its path, SHA-256, `human_identity_pixels_absent: true`, and exact `@Image2`-or-higher interface tag are known.
- Materialize each ready color as one complete schema `1.4` `single_compile` containing `streaming_release`; the compiler must load the immutable plan and reject any shared-core, market-profile, generation-control, SKU, creative, caption, hashtag, intent, or shot-signature drift.
- A parallel drafting lane may propose the plan but cannot certify evidence, approve references, run the final director gate on behalf of the main agent, or submit paid work.
- Once a release is paid, freeze it. If a later color cannot implement the immutable plan without weakening proof or garment fidelity, block that color and create a new run for unreleased work instead of changing the earlier release.

## Required Inputs

Read or infer from the user-provided product image, product description, or product page:

- product category, visible design, materials, colors, package language, use case, and likely buyer
- market clues from packaging, page language, currency, model styling, claims, and platform context
- available assets, including product image, product page screenshots, model images, fixed creator image, or clothing three-view references

When an Amazon URL is present, collect and separate the selected child-ASIN catalog attributes, seller copy, direct customer reviews, displayed review variants, and buyer images before scriptwriting. Record the number of direct reviews actually read and all access limitations. If Chrome control is unavailable or Amazon blocks a page, use only evidence actually obtained and mark the rest `partial` or `blocked`; never fill gaps with search snippets, AI summaries, reseller pages, or a similar ASIN.

Do not invent regulated claims, exact performance numbers, certifications, discounts, shipping promises, medical/health effects, review counts, cooling technology, elasticity level, anti-wrinkle effect, or certified breathability. Exact fabric composition requires eligible locked evidence from the selected child-ASIN catalog, a clear garment label, verified test, or user-provided source; exact performance numbers still require a verified test.

## Same-Garment Multi-Color Material Consistency

When the user provides multiple colors of the same garment in one batch, treat them as one SKU family with the same fabric, cut, sewing, thickness, knit/weave texture, opacity, drape, hem behavior, and detail construction. Only the color may change unless the product evidence clearly shows a construction difference.

- Choose the clearest first completed three-view image or the most representative product photos as the material baseline.
- For later colorways, generate or describe them as recolors of the baseline garment: same silhouette, same sleeve/neckline/hem, same seam placement, same fabric texture scale, same knit/weave density, same surface finish, same thickness, same stiffness/softness, and same drape.
- Do not let the script, close-up plan, or prompt describe a later colorway with a different material, larger rib/knit grain, smoother synthetic sheen, heavier fabric, thinner fabric, different stretch, different opacity, or different sleeve/hem behavior.
- When writing any Seedance/PopBoom prompt for a later colorway, include an explicit material lock in English: `same garment as the baseline colorway, color changed only; identical fabric construction, knit/weave texture scale, thickness, opacity, drape, seams, sleeve construction, neckline, and hem`.
- Prefer the positive baseline lock above. Add one compact material-drift exclusion only when a prior take actually changed texture, sheen, thickness, opacity, drape, seams, or sleeve construction; do not paste the full repair list into every first-generation prompt.
- If a close-up mentions fabric or texture, show the same baseline texture on every colorway; vary only the color name and styling angle.
- When producing multiple videos for the same SKU family, keep the product-proof shots structurally consistent across colors so any visual difference reads as color choice, not a different garment.

## Fixed PopBoom Model Presets

The user currently works with these fixed PopBoom custom models. Treat the exact model names below as complete production presets:

| Model name | Reference image | PopBoom custom model | Country / market | Voiceover language | Creator notes |
| --- | --- | --- | --- | --- | --- |
| 德1 | D:/Codex内容/德1.png | 德1 | Germany | German | Blonde German female creator, 28, bright lifestyle/selfie feel, friendly and youthful. |
| 德2 | D:/Codex内容/德2.png | 德2 | Germany | German | Light-brown-haired German female creator, 28, polished calm lifestyle/business feel, trustworthy and clear. |
| 德3 | D:/Codex内容/德3.jpg | 德3 | Germany | German | Plus-size German female creator, 40, shoulder-length blonde hair, calm natural TikTok creator feel, trustworthy and clear. |
| 美1 | D:/Codex内容/美1.jpg | 美1 | United States | American English | American female creator, 28, warm medium complexion, long highlighted brunette waves, brown eyes, soft polished smile, confident TikTok creator feel, direct and energetic. |
| 美2 | D:/Codex内容/美2.jpg | 美2 | United States | American English | Plus-size American female creator, 40, shoulder-length dark brunette waves, brown eyes, friendly smile, approachable TikTok creator feel. |
| 美3 | D:/Codex内容/美3.jpg | 美3 | United States | American English | Plus-size Black American female creator, 40, shoulder-length natural black curls, brown eyes, warm confident smile, friendly TikTok creator feel. |

When the user says 美1, 美2, 美3, 德1, 德2, or 德3:

- Treat country/market, voiceover language, creator identity, and PopBoom virtual-human model as already answered.
- Do not ask the user again for country, language, or which creator/model to use.
- For PopBoom/Seedance video generation, use TikTok/TikTok Shop, 9:16, and 15 seconds unless the user explicitly overrides them.
- Write the script in the model's voiceover language: 德1, 德2, and 德3 use German; 美1, 美2, and 美3 use American English.
- Keep the planning explanation in Chinese unless the user asks for another language, but make the final spoken lines match the model language.
- In the required-decision table, mark these values as filled by the fixed model preset.
- If the user introduces a new model name, ask only for the missing preset fields: country/market, spoken language, PopBoom custom model name, and reference image path.

## Market Prompt Compiler

After the evidence/pain-action map is locked and before selecting a base template, apply `references/market-prompt-compiler-contract.md`.

- Select `us_champion_v1` for 美1, 美2, 美3 / United States / American English and read only `references/us-winning-prompt-profile.md`.
- Select `de_champion_v1` for 德1, 德2, 德3 / Germany / German and read only `references/de-winning-prompt-profile.md`.
- Persist the profile, `generation_controls.prompt_shell_mode`, `delivery_mode`, `camera_mode`, `music_mode`, `verdict_mode`, `commerce_cta_mode`, `screen_text_policy`, and the complete `generation_controls.scene_strategy` before populating the timeline.
- Apply authority in this order: identity/evidence/safety, market compiler, German account-interaction overlay, base template/creative form, then style preferences.
- Never translate one market's completed prompt into the other. Recompile the shared evidence and action-proof plan through the destination profile.
- Treat the seller-winning prompt corpus as descriptive structure only. Do not load the source spreadsheet, raw prompts, corpus statistics, or full analysis reports into production context, and never let frequency override evidence or safety.

## Workflow

Whenever writing a script, prompt, or director package for user review, include a separate reviewable voiceover block that lists every exact market-language spoken line and a line-by-line Chinese translation. Do not replace the market-language line with Chinese inside the PopBoom paste-ready prompt.

1. Inspect the request and product evidence. If an Amazon URL or review-analysis request is present, first apply `references/amazon-product-review-evidence-contract.md`, lock the requested/parent/child ASIN and selected variant, and persist the research bundle before creative analysis.
2. Use reliable product photos to draft the shared garment baseline and all-color differentiation plan while references are pending. For each final release, first confirm that the passed three-view contains zero real-human identity pixels and is tagged `@Image2` or higher; then analyze its garment silhouette, neckline, sleeve shape, length, drape, color, texture, coverage, real wearing occasions, styling uncertainties, visible proof points, close-up opportunities, and full-outfit direction before sealing any hook, scene, video form, voiceover, proof action, timeline, or prompt.
3. If multiple colors of the same garment are present, define the material baseline before writing any script, then enforce color-only variation for all later colorways.
4. Build and persist the evidence registry, review analysis, pain-solution map, exact-claim-ID allowlist, claims registry, and claim-proof map before writing the script. Separate objective attributes, seller claims, direct buyer experience, buyer-image observations, summaries, and excluded sources. For every selected selling point, first name the buyer pain, purchase hesitation, or wearing anxiety it solves; then bind one exact `claim_id`, evidence IDs, one canonical `product_part_id`, one display target, target-language `spoken_claim_terms`, required framing, demonstration action, hand count, and expected visible change. Use `references/selling-point-action-proof-methodology.md` to choose the proof action, complete inherited-start-to-endpoint grammar, left/right hand anchors, `proof_endpoint`, human motive, persistent task/prop anchor, natural handoff, and micro-cues before drafting the spoken line. Carry the exact pain/claim/evidence/part IDs into the proof and beat; bind the hook to one `pain_point_id`; require the spoken line and `core_action` to name the same proof target. Never combine two targets or let a review pain create an unsupported product promise.
5. Select and persist the market profile, all seven scalar `generation_controls` fields, and the required `scene_strategy` through `references/market-prompt-compiler-contract.md` and `references/occasion-scene-routing.md`; read only the matching US or DE profile. Do this before template selection or timeline writing.
6. Check the required blocking decisions with `references/intake-checklist.md`.
7. If any blocking item is incomplete and no user authorization to decide is given, stop after the confirmation block. Do not block on missing publishing platform, video duration, or video sound/visual format; use TikTok, 15 seconds, and an agent-selected format respectively.
8. If blocking decisions are complete or authorized, read:
   - `references/output-blueprint.md` for the final director-plan structure.
   - `references/seedance-prompt-patterns.md` for Seedance prompt formats and UGC creative forms subordinate to the selected market profile.
   - `references/script-template-library.md` for reusable base formats and stability/repair templates subordinate to the market compiler.
   - `references/quality-directing-contract.md` for directing intent, reference-role locks, fidelity allocation, shot/motion density, continuity, and retake rules.
   - `references/motion-realism-and-commerce-cadence.md` for the three-phase creator performance arc, continuous social intention, endpoint-to-start and phone/prop continuity, direct-response action frequency, natural biomechanics, emotional/voice rhythm, and UGC rather than fashion-film pacing.
   - `references/selling-point-action-proof-methodology.md` for turning every selected selling point into a claim-matched frame, proof action, hand trajectory, visible endpoint, spoken line, and realistic micro-cue plan.
   - `references/amazon-product-review-evidence-contract.md` for Amazon identity locking, source grades, review sampling, pain-to-solution eligibility, conflicts, and evidence hashes.
9. For the complete director plan, use the strongest available model in the current Codex session. Always research recent TikTok apparel-commerce videos from the most recent half-month when browsing is available before selecting the video sound/visual format, writing the script, and writing section 12. Produce the complete plan in Chinese, match every spoken beat's `spoken_language` and exact line to the selected market/model profile, and show the user a `voiceover_review` row with a Chinese translation for every beat. Keep every Chinese review translation outside `compiled_text`.
10. Include practical next steps at the end.

## Distilled Script Template Library

Before writing section 6 hooks, section 7 script, section 9 Seedance/PopBoom prompt, or section 12 caption/hashtags for apparel, first lock the selected market profile, then read `references/script-template-library.md`.

Use that reference as a base-format and stability library, not as copy text or a market compiler. Select one base template, adapt it to the actual garment, model preset, TikTok research, colorway, visible product proof, and already selected US/DE profile, then audit the output against the stability templates when the video is intended for PopBoom generation.

For same-garment multi-color batches, reuse the same selected base template and proof-shot structure across colors unless product evidence justifies a different structure. Vary only the color-specific hook angle, outfit styling, scene palette, caption, and hashtags while preserving material consistency.

## PopBoom Template Distillation Rules

Use `references/market-prompt-compiler-contract.md` plus the selected US or DE profile as the only source for market-specific seller-winning prompt logic. Use `references/script-template-library.md` for base creative forms, cadence population, stability, and explicit repair tasks. Do not restate or copy those details here.

Non-negotiables: combine the template library with recent-half-month TikTok apparel-commerce learning and the validated research bundle; never invent model stats, prices, discounts, reviews, sales volume, exact body measurements, fabric composition, certifications, medical/health effects, or shipping claims. Never write fake first-person purchase/testing experience or turn a seller bullet, AI summary, buyer image, or sibling-ASIN review into an exact selected-variant fact.

## German Account Conversion Learning

Apply this section whenever the chosen market/language/model is German or Germany, especially for 德1/德2/德3.

The July 2026 `nanettefei5` TikTok Studio exports showed a German apparel account pattern: views can scale while profile visits, comments, and shares stay weak. The active period had 51,859 views, 225 profile visits, 464 likes, 11 comments, and 7 shares; the recent 28-day window had 48,955 views but zero comments and only a 0.255% profile-view rate. Content exports showed traffic concentrated in a few basic/top videos, while some lower-view V-neck/brunch/basic clips had stronger like rates. Use this as a standing creative constraint, not as a one-off report.

For German apparel scripts and prompts, first lock `de_champion_v1`; then apply this account-specific interaction overlay:

- Optimize for profile/shop/comment action, not raw views alone. A high-view idea with weak interaction must be rewritten with a sharper hook, clearer product proof, and easier CTA/comment prompt.
- Put the garment on body in the first frame and make the first 1-2 seconds a visible wearing-result hook. Do not open with slow walking, empty scenery, a generic product label, or a feature list.
- Give German basics, V-neck shirts, blouses, button-down tops, black cardigans, and everyday summer pieces hooks around supported buyer outcomes such as `Basic, aber nicht langweilig`, `locker, aber nicht unförmig`, `schnell angezogen, aber trotzdem ordentlich`, visible neckline/sleeve/hem proof, and brunch/daily/vacation outfit use.
- Include an easy German either/or comment trigger in the plan or caption when it does not distract from purchase intent, such as `Schwarz oder Hellblau?`, `Offen oder geschlossen?`, `Urlaub oder Alltag?`, or `Jeans oder Shorts?`.
- Keep any selected CTA human-only under the German profile. Use an ownership verdict or soft verdict by default; when `commerce_cta_mode: light_link`, use one brief plain `unten links` cue. Let section 12 copy support profile/shop/comment action with German buyer language such as `im Profil`, `welcher Look passt besser`, or a specific styling question.
- For Zibuyu apparel output, section 12 must contain exactly five total hashtags. Choose all five from the garment, market, TikTok Shop context, outfit occasion, style, and buyer search intent; do not force a fixed brand hashtag. Do not use `#Imily Bela`; replace it with a relevant non-brand tag so the total remains exactly five.

## Persistent Fit-Stats Overlay

For all fixed-model apparel videos, include one persistent fit-stats text overlay directly in the Seedance/PopBoom prompt. This is the only allowed generated on-screen text.

- The overlay text must contain only the data, with no `Model` label or extra words.
- For United States / American English videos, use exactly: `5'6" / 115 lb / Size S`.
- For Germany / German videos, use exactly: `168 cm / 52 kg / Größe S`.
- Place it in the upper-left safe area from the first frame to the final frame, clearly visible at a glance but not covering the model's face, hands, body, garment, or any product-proof detail.
- Direct Seedance to render it as clean white text with a subtle dark outline or translucent dark strip so it remains legible in bright scenes.
- Keep every other screen element clean: no subtitles, captions, watermarks, arrows, stickers, icons, badges, product-link graphics, or UI overlays.
- If a user provides different verified model stats later, use the same market formatting rule: US converts height/weight to feet/inches and pounds; Germany uses cm/kg and German size wording.

## Conversion-Oriented Hook And Voiceover Rules

Before writing the voiceover, automatically complete a pain-first selling-point analysis from the product evidence, especially the clothing three-view reference and validated Amazon research bundle when available. Do not begin from obvious visible attributes such as color, neckline, sleeve, hem, print, crochet, knit, ruffle, or pockets. Begin from what the buyer may be worried about: looking wider, boxy, childish, cheap, too exposed, too tight, clingy at the stomach/hips, too plain, hard to style, wrong for the occasion, or different from expectation. Then decide which visible garment evidence can solve that worry.

For apparel scripts:

- For every spoken line, maintain a review translation in Chinese that preserves the sales intent and evidence binding. The Chinese translation is for user approval only; it must not become generated on-screen text or replace the target-market spoken line in the final prompt.
- For every apparel plan, create candidate pains before candidate selling points. Each selected selling point must follow `buyer pain -> desired wearing result -> garment evidence -> proof frame/action -> visible endpoint -> spoken line`. If the line only describes what the viewer can already see, it is too shallow unless it explicitly explains the buyer pain it resolves.
- Before choosing hooks or a template, select one `core_sellable_wearing_result`: the single buyer-relevant wearing outcome most likely to drive purchase and visibly provable in 15 seconds. It must be derived from buyer hesitation plus garment evidence, not from a fixed category habit. Examples of valid outcomes include `oversized but not sloppy`, `loose but still shaped`, `wide-leg but does not swallow height`, `warm but not bulky`, `mini length but secure`, `soft drape without clinging`, or `waist looks defined`; the exact result changes with the garment.
- Score candidate results before scriptwriting: buyer cares about it, the garment visibly supports it, the video can prove it with simple actions from front/side/back/detail/styling angles, and it differentiates the item from ordinary same-category products. Reject a candidate that is only a catalog attribute, generic praise, or a claim that cannot be shown.
- Prefer high-conversion wearing-result language when evidence supports a qualified visual claim, such as `waist looks defined`, `makes the waist read smaller`, `legs look longer`, `not clingy at the hips`, `mini length but secure`, `sweet but not childish`, `polished instead of cheap`, or `covered without looking heavy`. Do not turn these into absolute body-change, medical, performance, or universal fit promises.
- The hook must name the result or the buyer hesitation that result resolves. Do not open with a list of features. Use the pattern `not [feared bad outcome], but [desired wearing result]` when it is natural in the target language, such as `Oversized, but not sloppy.` or `Warm, but not bulky.`
- For German output, audit the hook against the German Account Conversion Learning: it must be product-visible, plain-spoken, and capable of driving a profile/shop/comment action, not only watch time.
- Build the viewer-facing 15-second performance around three macro phases: about 0-4s recognition/result Hook, 4-10s animated proof, and 10-15s close/detail conviction plus a sincere verdict or human-only CTA. Inside those phases, retain five to seven sequential internal beats, at least four non-CTA demonstrations, and at least three distinct part/effect proofs. Other details may appear only when they help prove the core result.
- Start with a buyer hesitation that resolves through evidence-backed product proof. Prefer a valid direct-review theme when available; otherwise label the hook as user-provided, visible-reference, or category inference. One review supports only a single-review attribution, while aggregate language requires at least three unique direct review IDs.
- Classify the opening as `hook_semantics: pain_question`, `evidence_backed_contrast`, or `context`. A pain hook must be a real question about the buyer's existing problem; a context hook cannot assert a garment feature. Use `evidence_backed_contrast` only for a direct-review/review-theme material-expectation pain that maps to an independent exact-variant texture claim: write one short negative declarative contrast, keep review evidence on the hook, and follow it immediately with the mapped texture detail close-up, off-screen proof line, and one `pinch_release`. Otherwise move the product fact to a normal bound proof beat.
- Convert each selected pain point into a concrete allowlisted solution claim and visible benefit: relaxed silhouette, neckline styling, sleeve coverage, drape, length, easy pairing, color versatility, texture, movement, or front/side/back fit. A complaint with no independent product evidence stays `risk_only` and must not become a conversion hook. Do not invent fabric composition, slimming effects, discounts, stock urgency, or review consensus.
- Make the voiceover sales-oriented but natural. Every spoken line should either hook attention, answer a buyer concern, prove a visible benefit, create outfit imagination, or push the viewer toward action.
- Write for the mouth, not for a product page. Use short everyday sentences, contractions and natural particles where idiomatic, as if the creator were showing one detail to a friend in the room. Ban official or brochure-like transitions and constructions such as `this garment features`, `this product offers`, `furthermore`, `moreover`, `in addition`, German `dieses Kleidungsstück verfügt über`, `darüber hinaus`, `des Weiteren`, and Chinese `本产品采用`, `该服装具备`, `此外`, `综上`. Do not repeat the same conversational opener in three or more non-CTA proof/styling lines.
- Avoid repetitive or generic voiceover such as only saying "comfortable", "nice", "looks good", "easy to style", German `sieht gut aus` / `leicht zu kombinieren`, or Chinese `很好看` / `很百搭`. Vary the angle across hook, fit proof, styling, pain-point answer, and CTA.
- Make detail density serve the selected result instead of distributing attention evenly across features. A default 15-second script should prove the core result with several angles: one hero structural proof, one additional part/coverage proof, one fit/drape/movement/side-back proof, and one outfit/use-case proof when supported. Every line names one exact target plus what the viewer can see it do or where it sits; the matching action and endpoint must show that fact. Keep one claim per beat. If a detail does not help prove the selected result, remove it instead of filling the script.
- Prefer lines such as German `Schau mal, wie weit der Ausschnitt sitzt.`, `Der Ärmel endet locker am Oberarm.`, `Beim Drehen öffnet sich der Seitenschlitz.`, and `Mit High-Waist-Jeans bleibt der Saum sichtbar.` Vary sentence openings and adapt naturally to the market; do not translate these examples word for word.
- The first 1-2 seconds must contain a strong hook that earns watch time. Use a specific buyer problem, curiosity gap, outfit transformation, or direct claim tied to visible proof. Do not start with a flat product introduction.
- When an eligible review theme shows that sampled buyers expected an ordinary smooth T-shirt surface but the exact garment visibly has a fine knit texture, prefer a natural negative contrast over a softer question: equivalent to `Not a basic tee—look at this fine knit texture.` Adapt it idiomatically to the target language, do not imply the garment is a different product category, and do not insert an unrelated reaction or styling beat before the texture proof.
- Section 6 must provide 10 distinct hooks, and the final script must clearly choose one primary hook for the opening shot.
- In the final shot, use the selected profile's `verdict_mode` and `commerce_cta_mode`. The verdict must be earned by visible proof. For `light_link`, add only one short market-native cue and pair it with one small real-model down-left gesture or brief glance while the other hand rests. `none` adds no link cue; `evidence_backed_promo` requires eligible offer evidence and explicit user authorization.
- Keep the close casual and buyer-friendly, not stiff: US normally uses a soft verdict with an optional brief link cue; Germany normally uses an ownership/soft verdict with either no CTA or one plain `unten links` cue. Do not copy these structures word for word across markets.
- When a CTA action exists, keep it physically simple and human-only: one hand makes a small real gesture toward the lower-left area while the other hand rests naturally or lightly touches the garment. Do not represent the CTA with a shopping cart, arrow, arrow emoji, sticker, floating symbol, product-link badge, or graphic overlay.

## CTA Visual Purity And Prompt-Trap Rules

The final CTA often causes video models to invent arrows, emoji stickers, product-link graphics, or shopping-cart icons. Prevent this before generation:

- Treat any arrow, arrow emoji, arrow sticker, pointer sticker, shopping-cart icon, floating link marker, product-link badge, animated pointer, UI overlay, or emoji graphic as a hard generation error.
- The final shot may only show the real model's body, face, hands, garment, outfit, and real scene. No generated visual aid may appear.
- When writing the final Seedance/PopBoom prompt, avoid wording that asks the model to "show an arrow", "add a pointer", "add a link icon", "display a sticker", "show a shopping cart", or "put an icon on screen".
- Prefer wording such as: "The model makes one small natural down-left hand gesture; only the persistent upper-left fit-stats text remains on screen, with no graphics, no stickers, no arrows, no icons, and no UI overlays."
- A `light_link` spoken CTA may mention the lower-left link, but the visual instruction must remain human-only and explicitly forbid graphics. A `commerce_cta_mode: none` close contains only the profile-selected verdict and no link gesture.

## Creator-Led Natural Share And Performance Arc Rules

Apply these performance rules after occasion-first scene routing to mirror selfies, private-interior proof cuts, phone-shot recommendations, destination outfit shares, friend-filmed walks, and any creator-led format where the model's personality is part of conversion. These formats are options, not default scene choices:

- Direct one continuous friend-to-friend social intention, such as `I know the result you want, and I am excited to show why this solves it`. Do not direct the performer as six isolated product-inspection tasks.
- Shape the viewer-perceived performance into three macro phases: warm recognition/result Hook around 0-4s, brighter animated proof around 4-10s, and close/detail conviction around 10-15s. The macro phases sit above—not instead of—the five to seven internal beats and required proof-action floor.
- Carry gaze, weight, expression, hand occupancy, props, and garment state forward. The previous endpoint normally becomes the next start state. A mirror-selfie filming hand continuously holds the phone; a hand holding another color, bag, or garment stays occupied until an explicit place-down, handoff, or intentional cut.
- Motivate body and camera distance by proof need: step back because the viewer needs the full fit, move/turn because fabric or silhouette behavior is the proof, and walk closer because the viewer needs to inspect the decisive detail. Do not choreograph ordinary UGC with unnecessary centimeters, degrees, symmetric hip resets, attention posture, or equal timed freezes.
- Use one active demonstration hand at a time while the filming/task hand remains anchored. Preserve natural acceleration/deceleration, weight transfer, shoulder-elbow-wrist follow-through, fabric lag, and a readable endpoint without freezing the creator between proofs.
- Direct an emotional rise: `warm recognition -> animated certainty -> delighted conviction or relief`. Pair each visible speaking line with a meaning-specific expression and use at least two distinct visible expression states across a multi-line clip. Passion comes from eye changes, growing smile, emphasis, and short proof-pivot pauses—not shouting, frantic gestures, a fixed grin, or one flat energy level.
- Write the voice as one connected live-share thought with idiomatic target-language connectors equivalent to `look`, `when I move`, `see how`, and `honestly` only when truthful. For a 15-second American-English creator-led mirror share, aim for roughly 50-60 total words and about 200-235 WPM only when a read/TTS audition remains clear; use a broader validator safety range and do not copy English word counts into German or other languages.
- Default visible lip-sync remains conservative. When `secondary_fidelity_spend: lip_sync`, allow up to three concise visible-dialogue anchors—one per macro phase—only with locked/subtle camera, performer load `simple` or lower, one active demonstration hand, at most 14 whitespace-delimited words per line, 36 visible words total, and an explicit expression cue. Keep complex movement/detail proof off-screen.
- Evidence rules do not relax for natural speech. `incredibly soft`, `doesn't cling`, `my new favorite`, or other tactile/experience claims require eligible support and identity-safe wording; otherwise speak only to visible drape, space, construction, fit, or styling evidence.

## Voiceover Density And Shot Coverage Rules

For 15-second apparel UGC scripts, avoid long silent stretches. The default should be dense but natural voiceover coverage.

When writing section 7 and the final Seedance prompt:

- Give each commercial unit either a short spoken line or an explicit purposeful silence. Prefer off-screen voiceover for detail/motion proof and reserve visible lip-sync for the hook and/or CTA.
- For a 15-second video, normally use five to six short spoken units across hook, buyer concern, several visible proofs, styling/use-case, and CTA. Every claim-proof action needs its own matching concise voiceover line. By default use no more than two visible-lip lines; when lip-sync is the declared secondary fidelity spend, follow the conditional three-anchor budget above. Carry complex detail/motion proof with off-screen voiceover and the mouth out of frame.
- Do not leave more than 2 consecutive seconds without either voiceover, a natural reaction sound, or a purposeful product-detail close-up.
- Each timestamped beat should pair scene/framing, one action/endpoint, speech mode and canonical line or silence, and the product point being shown.
- Keep spoken lines short enough for normal speech speed. By default, keep each whitespace-delimited visible line at 10 words or fewer and the 15-second visible-lip total at 20 words or fewer; use the conditional 14/36 limits only when lip-sync is the selected secondary fidelity spend. For American-English 15-second creator-led mirror shares, hard-audit the full spoken total against the validator's safe range and audition the aspirational 50-60-word, 200-235-WPM delivery for clarity.
- Use voiceover even when the model's mouth is not visible, but if the model's mouth is visible, require exact lip-sync to the spoken words.
- Make each spoken line add a new proof angle for the same selected wearing result, not a disconnected feature list: front result, structural detail, side/back behavior, movement/handling, outfit use, then purchase CTA.
- Join those lines into one recommendation with natural connective phrasing, varied emphasis, and brief proof-pivot pauses; do not make every sentence the same length, stress, or falling intonation.
- Give the non-CTA information units a deliberate result-proof sequence: visible result -> structural reason -> alternate angle or buyer-worry angle -> action endpoint -> concrete outfit pairing or real-life use. Do not create an unsupported fourth product claim, and do not include a supported detail if it distracts from the core result.
- Phrase each proof as target + visible behavior/position, not as a catalog label or generic praise. `The garment features a side slit` and `It looks nice` are invalid; `See how the side slit opens when I turn?` is the intended level of specificity.
- Match the model action to the spoken product point. Example: mention the V-neck while the model points once to the neckline; mention sleeve coverage while she gently touches one cuff; mention drape while she slowly turns; mention easy styling while she does one front tuck.
- Require a matching demonstration action for every garment or styling claim. The camera must show the exact named part close enough to read the proof; a smile, nod, prop movement, camera push-in, or motionless hold does not count.
- Keep actions realistic and stable: one action per beat, both hands accounted for, no busy gestures, no dancing, no exaggerated pointing, and no simultaneous unrelated pulling/turning/pointing. Use off-screen voiceover with the mouth out of frame for complex garment demonstrations, then continue with the next sequential proof action.
- The final 2-3 seconds must include a spoken evidence-safe verdict and, when the selected `commerce_cta_mode` is not `none`, its matching human-only CTA action; do not make the ending an unexplained silent sales gesture.

## Close-Up Proof Shot Rules

Whenever the voiceover mentions fabric feel, material quality, comfort, breathability, drape, softness, stretch, stitching, neckline, sleeve, hem, pockets, buttons, print, texture, or another design detail, give that specific claim its own matching close-up/enlarged framing and physical demonstration action. Do not satisfy several different part claims with one generic full-body shot or one unrelated gesture.

Use close-ups as proof:

- For fabric/material claims, show visible texture, knit/weave/slub surface, natural folds, or drape with a controlled touch, pinch-release, rub-release, or hem-release action. Demonstrate stretch only when explicit evidence allows it; then use one controlled pull-release cycle with fully specified hands and visible recovery.
- When comfort or breathability is not independently evidenced, do not state it as a product claim. You may show only visible relaxed fit, sleeve room, drape, fabric movement, or a controlled pinch-release, and must describe only what is visible; the action itself never proves comfort or breathability.
- For neckline claims, use a chest-to-neckline close-up and one simple gesture pointing once to the V-neck, collar seam, button line, or trim.
- For sleeve/arm-coverage claims, use a sleeve-cuff close-up with one hand gently touching the cuff while the other hand rests naturally.
- For hem/coverage claims, use a waist-to-hem close-up or side shot showing the hem length and drape over bottoms.
- For stitching/design details, use a steady macro-like close-up of the seam, cuff, neckline, print, texture, or hardware, with no busy hand movement.
- Place one matched close-up for every specific part/material claim. A 15-second apparel script normally needs at least three part/effect proofs and four non-CTA product/styling actions.
- Keep close-ups realistic: phone camera moves slightly closer or uses a gentle push-in, not an unnatural microscope view.
- Keep the garment dominant in the frame. Hands, jewelry, bag straps, hair, or props must not cover the exact detail being explained.
- Claim exact fabric composition only from eligible locked selected-child catalog, clear garment-label, verified-test, or user-provided evidence. Cooling technology, elasticity level, anti-wrinkle effect, certified breathability, and other performance claims require their own eligible evidence; use strictly visible wording such as "you can see the texture" or "the hem settles into folds" when only visual evidence exists.

## Outfit Styling And Occasion-First Scene Rules

Treat apparel videos as product proof, outfit inspiration, and a destination answer. The viewer should understand where to wear the item, how to style it there, and which occasion-specific uncertainty the outfit solves. Read and apply `references/occasion-scene-routing.md` in full.

Before writing the script or Seedance prompt:

- Classify the garment as `outward_wear`, `homewear`, or `mixed`, then name one concrete destination or use moment. Do not use a bedroom, closet, bathroom mirror, or generic home interior as the automatic answer for outward-wear clothing.
- Write the buyer's styling question before selecting the location, for example what to wear to a café, beach, commercial street, office, trip, weekend walk, or casual date without looking overdressed, shapeless, exposed, or hard to coordinate.
- Select the scene for its sales job: it must add relevant occasion interest, make the garment proof readable, and demonstrate a complete styling answer. Scene novelty without occasion or proof value fails.
- Persist the complete machine `scene_strategy` before the timeline. Give every v6 beat a matching `scene_id`, and keep natural scene prose, outfit, actions, sound, lighting, and continuity consistent with that ID.
- Route the video form from the scene and proof need: destination outfit share, GRWM departure, friend-filmed walkthrough, fixed-phone fit proof, mirror-to-destination match cut, travel pack-and-wear, multi-styling switch, or home try-on. Do not repeat mirror try-on merely because it is easy to generate.
- For United States work, use café, beach/boardwalk, commercial street, sidewalk, travel, market, or event-arrival interest more boldly when it fits the product. For Germany, use equally real outward-use scenes with calmer, plausible daily-life treatment and no forced landmarks or tourist decoration.
- For outward wear, set `bedroom_policy: excluded` by default. A private scene may be `proof_only` for at most the first two beats before one intentional move to the outward primary scene. A private primary scene requires a specific fit/detail/multi-styling reason; `convenient`, `natural light`, `UGC feel`, or `bedroom is common` is not enough.
- Match the garment color with the scene palette. Use neutral or complementary backgrounds that make the garment stand out without color clash; avoid backgrounds that make the product disappear.
- Build a simple complete outfit answer around the garment: bottoms, shoes, bag, and restrained jewelry/bracelet/watch/belt/sunglasses or hair styling when useful.
- Use accessories to support the garment, not steal attention. Keep them realistic and compatible with the garment's price/style; avoid luxury-brand claims or visible brand logos unless provided by the user.
- Explain the styling answer through the voiceover: why this pairing works, what occasion it fits, and what buyer problem it solves. Pair it with one visible action such as adding a shoulder bag, completing a front tuck, stepping back to show shoes, or picking up the departure accessory.
- Keep styling actions stable: introduce at most one accessory action per beat, account for both hands, avoid busy rummaging, and do not change several outfit pieces at once.
- Keep the main garment visible and dominant in every outfit shot. Accessories and scenery must not cover the neckline, sleeve, hem, print, color, or key selling point.
- If multiple colors of the same garment are being scripted, vary the occasion, outfit answer, or scene treatment where appropriate. Do not send every color back to the same bedroom or mirror setup.

## Model Micro-Expression And Lighting Realism Rules

When a visible model or creator appears, direct the face like a real short-video creator, not a stiff render.

- Give each beat a small natural facial cue that matches the spoken line and product moment: relaxed smile, tiny eyebrow lift, quick friendly glance to camera, slight nod, subtle "this is nice" reaction, or a brief thoughtful look before showing a detail.
- Keep expressions dynamic but restrained. Avoid frozen smiles, blank eyes, over-wide grins, exaggerated acting, doll-like skin, plastic face texture, or the same expression through the whole video.
- Across a creator-led clip, require a readable emotional progression from warm recognition to brighter proof energy to pleased conviction or relief. When there are multiple visible speaking beats, use at least two meaningfully distinct expression cues rather than renaming the same smile.
- Make eye direction intentional: look into the camera for hook and CTA, glance down at the fabric or detail during close-ups, glance toward the mirror or outfit when showing styling, then return naturally to camera.
- Add tiny real-life imperfections when useful: a natural blink, micro head tilt, soft breath before speaking, small mouth corner movement, or casual smile after touching the garment. Do not make the model look sleepy, awkward, or distracted.
- If the mouth is visible during voiceover, facial expression and mouth movement must match the spoken meaning and exact words, with normal conversational pacing.

Lighting must look physically real and match the scene.

- Specify the light source and direction in every apparel prompt: soft window light from one side, warm indoor lamp light, shaded outdoor daylight, bathroom mirror light, or overcast natural light.
- Describe believable shadows and highlights: soft shadow under the chin, gentle fabric folds, sleeve shadow on the arm, hem shadow over shorts, and natural catchlight in the eyes when the face is visible.
- Keep light and shadow consistent across cuts. Avoid floating highlights, overly glossy skin, harsh beauty-filter glow, flat shadowless lighting, unrealistic rim light, and scene-object shadows that point in conflicting directions.
- Use lighting to support the garment: side light can reveal fabric texture and drape; soft front light can keep color accurate; shaded daylight can make casual outfits look natural.
- Do not let shadows, overexposure, hair, hands, jewelry, or bag straps hide the product detail being explained.

## Realistic Scene Detail And Generation Stability Rules

When writing every Seedance/PopBoom video prompt, describe the scene like a real creator shot it, not like a vague ad concept.

Include concrete scene details:

- location type, such as a café terrace, beach/boardwalk, commercial street, city sidewalk, office/commute edge, travel hotel, entryway, porch, outdoor leisure setting, or a specifically justified private proof area;
- time and lighting, such as soft morning window light from camera left, warm indoor lamp light, shaded outdoor daylight, or natural overcast light;
- camera setup, such as handheld phone camera, mirror selfie angle, waist-up medium shot, chest-to-hem close-up, slow push-in, or steady tripod-like phone shot;
- model position and body orientation, such as standing three-quarter to camera, shoulders relaxed, feet planted, one slow half-turn, or side profile for hem and drape;
- product placement in frame, including neckline, sleeve, hem, fabric movement, side/back view, color, and how much of the garment remains visible;
- realistic environment details, such as a bed, clothing rack, mirror edge, wooden floor, neutral wall, simple closet, or real street background, only when they fit the product.

Stability rules:

- Use one model only unless the user explicitly requests otherwise.
- Use one clear camera angle and one main body action per beat. Avoid fast choreography, dancing, crossed arms, spinning, crowded scenes, or multiple simultaneous hand actions.
- Prefer slow natural movement: one gentle turn, one hem touch, one sleeve touch, one small step, or one lower-left-link gesture.
- Keep both hands accounted for in every shot with start, action, and end anchors. If one hand demonstrates, preserve the other hand's persistent phone/task/prop anchor. If a verified proof genuinely needs both hands, make both hands cooperate on the same single action while the torso and camera remain stable.
- Apply `references/motion-realism-and-commerce-cadence.md`: in a default 15-second clip, deliver a newly completed product proof or styling result about every two to three seconds; inherit the prior endpoint, ease through one motivated action, let its result read, then continue naturally or declare the necessary cut. Micro-movements support realism but never count as proof.
- Carry gaze, weight, expression, phone/prop ownership, and garment state across adjacent beats. Do not repeatedly reset both hands to the hips, snap to attention, or let a held object disappear without a place-down, handoff, or cut.
- Avoid prompts that require the model to point, pull fabric, walk, turn, and speak at the same time.
- Use one compact positive anatomy replacement in first-generation prompts: one creator with exactly two natural arms connected shoulder-to-wrist-to-hand, two visible hands when the frame includes them, normal shoulders/wrists, and sequential actions. Default to one active hand; allow two only for one declared evidence-backed coordinated proof. Add one observed-failure-specific anatomy exclusion only during repair.
- For apparel, keep the garment visible in the first 1-2 seconds and in the CTA shot. Do not let props, arms, hair, or camera crop hide the main garment.

## Prompt Writing And Audit

Use the strongest available model in the current Codex session after the required blocking production decisions are complete or the user has explicitly authorized assumptions. Draft the full director plan and prompt language directly inside Codex.

Before final output, always audit and revise:

- preserve all user-provided facts and visible product details;
- preserve Amazon source identity, review sample scope, child-variant labels, conflicts, and blocked/partial status; reject any claim whose evidence IDs do not survive unchanged through claim, proof, beat, caption, and receipt;
- select the market before choosing a base template; require `zibuyu_market_prompt_v1`, the matching `us_champion_v1` or `de_champion_v1` profile, and all seven `generation_controls` fields; reject translation of a completed prompt from one market to the other;
- state one commercial directing intent and remove any camera, lighting, performance, sound, or styling choice that does not serve that intent or visible product proof;
- state one continuous creator social intention and organize the viewer-facing performance as recognition/result Hook -> animated proof -> close/detail conviction around the five to seven internal beats;
- require one complete occasion-first `scene_strategy`: the scene must state where the item is worn, answer one buyer styling question with a complete outfit formula, select a fitting video form, and pass the private-interior gate. Reject an outward-wear prompt that defaults to bedroom/closet/bathroom/home without the declared exception;
- reserve `@Image1` for the selected fixed creator identity and preserve that creator's exact face, skin tone, ethnicity, hair, age presentation, and body identity; never use `@Image1` for garment identity;
- bind each clothing three-view at `@Image2` or higher to garment identity only: transfer construction, exact color, silhouette, neckline, sleeve, hem, seams, texture scale, thickness, opacity, and drape; require `human_identity_pixels_absent: true`, and do not transfer skin tone, ethnicity, neck, chest, collarbone, shoulders, arms, hands, fingers, nails, tattoos, jewelry, body shape, face/hair, white background, three-panel layout, panel dividers, repeated bodies, pose, camera, environment, text, or watermark;
- make garment identity the primary fidelity spend and choose only one secondary spend from visible proof, lip-sync, natural motion, or scene readability; explicitly economize crowds, busy props, rapid camera, complex choreography, and simultaneous hand actions;
- keep a default 15-second video to three to four contiguous camera-setup runs and five to seven sequential beats; require at least four non-CTA claim-proof actions, one main action and `proof_endpoint` per proof beat, and no overlapping unrelated motion systems;
- enforce native commerce cadence and believable kinetics: no idle explanatory beat or slow-motion fashion posing; use natural acceleration/deceleration, shoulder-elbow-wrist follow-through, plausible weight transfer, garment lag/settling, and endpoint-to-next-start continuity; reset or cut only when motivated;
- enforce persistent phone/task/prop continuity and motivated distance changes; reject silent hand switches, disappearing alternate-color garments/accessories, repeated symmetric hip resets, attention posture, and disconnected equal-energy product poses;
- when visible lip-sync, detail proof, or CTA is active, keep camera/performer/hand load within `references/quality-directing-contract.md` and preserve one stable creator, garment, outfit, scene logic, and light direction;
- for same-garment multi-color batches, enforce the baseline-material rule: later colorways are color-only variations with identical knit/weave texture scale, fabric thickness, opacity, drape, seam placement, sleeve construction, neckline, and hem;
- remove unsupported claims, invented certifications, exact metrics, reviews, discounts, medical/health effects, or shipping promises; keep Amazon catalog attributes, seller copy, direct reviews, buyer images, AI summaries, and third-party summaries in their separate source classes;
- after locking the market profile, apply one selected base format from `references/script-template-library.md`, then apply the relevant stability template before finalizing the Seedance/PopBoom prompt;
- apply the selected market skeleton: short product-visible hook, several claim-matched close-up/action proofs, full-fit or styling result, then an evidence-safe verdict and any profile-authorized human-only CTA; never reserve a long explanatory section for standing still;
- use the off-screen-friend dialogue format only when it improves retention and product fit; keep only one on-screen model by default;
- filter template-derived risks: no invented model stats, prices, brand dupes, discounts, reviews, exact body measurements, or unsupported fabric/performance claims;
- enforce Seedance prompt rules: timestamps, product in first 1-2 seconds, one commercial throughline supported by several claim-matched garment proofs, 9:16 unless changed, concrete camera/action language, positive replacements, and one compact hard-constraint slot; one throughline never means one feature, one proof, or one model action;
- enforce realistic scene detail and scene identity: every v6 beat must carry the strategy-approved `scene_id` plus readable location, lighting, light direction, believable shadow behavior, camera setup, model posture, product placement, location-native sound, and one physically simple action. In a two-location plan, allow one intentional transition and keep the outward primary scene in the majority and in the close;
- enforce outfit styling logic: match garment color, style type, scene palette, wearing occasion, bottoms, shoes, bag, and simple jewelry/accessories so the video gives buyers a usable outfit idea;
- keep accessories supportive and realistic: no brand/logo invention, no luxury claims, no accessory blocking the garment, and no multiple accessory changes in one beat;
- enforce generation stability before output: one model only unless requested, exact left/right hand trajectories, sequential non-overlapping actions, a precise relaxed posture, and one compact positive anatomy replacement; default to one active hand and allow a declared coordinated two-hand proof only when evidence supports it;
- enforce conversion structure: strong product-visible Hook in the first 1-2 seconds, pain-point-to-selling-point logic in the voiceover, a profile-selected verdict, and only the CTA mode/real-model gesture authorized by `generation_controls`; no shopping-cart icons, arrows, arrow emoji/stickers, pointer or emoji stickers, floating pointer graphics, product-link badges, pop-up badges, or extra UI overlays;
- enforce dense but solvable voiceover coverage: normally use five to six short spoken units in 15 seconds, use the default two-line visible-lip budget unless lip-sync is the declared secondary spend, and keep complex close-up/motion proof off-screen; give every claim-proof action one matching concise line and pair each beat with scene/framing, action, speech mode, and product point;
- enforce mouth-written, everyday voiceover: reject brochure/official constructions, standalone generic praise, and the same opener across three or more non-CTA proof/styling units; vary sentence shape without forcing slang, fake purchase history, or exaggerated reactions;
- enforce result-led detail richness without overload: select one core sellable wearing result, then prove it through front/detail/side-back/action/styling angles in 15 seconds; remove disconnected feature coverage even when those features are true;
- enforce natural facial micro-expressions: visible models should have small realistic expression changes, intentional eye direction, natural blinks or tiny head movements, and expressions that match the spoken line;
- enforce a three-stage emotional and vocal rise—warm recognition, animated certainty, delighted conviction/relief—with connected phrasing, varied emphasis, short proof-pivot pauses, and at least two distinct visible expression cues when multiple lines are on camera;
- enforce realistic light and shadow: specify the light source/direction, keep highlights and shadows physically consistent, and avoid flat, plastic, overexposed, or AI-glossy lighting;
- enforce one-to-one claim proof: whenever the script names fabric/material quality, comfort, breathability, drape, softness, neckline, sleeve, hem, stitching, texture, or another design detail, use a frame centered on that target plus the mapped action and visible result;
- in multi-color same-garment prompts, include the positive English material lock before submitting; add a targeted material-drift negative only when evidence or a previous take requires it;
- avoid unsupported material claims: exact composition may come only from a locked catalog attribute, readable garment label, verified test, or explicit user-provided label; numeric performance still requires verified test. Treat `Low Stretch` as a prohibition on a strong pull demonstration, not permission to advertise stretch;
- ensure no more than 2 consecutive seconds are silent unless the beat is a purposeful close-up or transition;
- ensure the voiceover is not monotonous or single-angle; it must move through Hook, buyer pain point, visible benefit/proof, styling or use-case imagination, and the selected verdict/optional commerce close;
- keep the required output blueprint order and concise Chinese planning voice;
- in section 12, merge the conversion caption and exactly 5 hashtags into one copy-ready paragraph, with no bullets, table, or separate explanation;
- for German Zibuyu apparel output, require exactly 5 hashtags and choose all five from the garment, Germany/TikTok Shop context, outfit occasion, style, and buyer search intent; do not force a fixed brand hashtag; do not use `#Imily Bela`; replace it with a relevant non-brand tag so the total remains exactly five;
- when voiceover is spoken by an on-screen model or creator, enforce accurate lip-sync/mouth shapes with the script, the active default-or-lip-sync-priority word budget, conversational articulation, and casual everyday wording that feels natural to buyers;
- for 15-second American-English creator-led mirror shares, validate the full spoken total against the safe machine range and audition roughly 50-60 words at 200-235 WPM; for other languages, audition locally rather than copying the English count;
- avoid rushed, unnaturally slow, flat, equal-stress, stiff, robotic, translated, uniformly shouted, or hard-sell ad delivery;
- do not mention internal model/tool usage in the final plan unless the user asks about process;
- for apparel or ecommerce UGC planning, research recent TikTok apparel-commerce videos from the most recent half-month when browsing is available. Apply the research when choosing the video sound/visual format, writing the script, and writing section 12. If live/current evidence is unavailable, state that the choice is based on local platform knowledge rather than a live ranking;
- if the user does not proactively specify the publishing platform, default to TikTok and mark it as a non-blocking default;
- if the user does not proactively specify the video duration, default to 15 seconds and mark it as a non-blocking default;
- if the user does not proactively specify the video sound/visual format, do not ask them to choose it. Select the most suitable format based on current platform knowledge or available research, the product evidence, and the target market.

## Optional User-Requested Video Repair And Script Stabilization

Use this section only when the user explicitly provides a failed PopBoom/Seedance video, visible error, or generation problem and asks for a separate repair task. Do not run this section as part of the default PopBoom completion workflow.

When revising a failed video script or prompt:

- Treat this skill as the script repair owner. Computer-use or browser tools may operate UI workflows, but they must not own creative script correction.
- Triage the take as `keep`, `fix_in_post`, `reroll`, `rewrite`, or `stop_or_rescope`; record one `primary_failure_variable` and change only one prompt clause, sample/seed, mode, or reference binding per authorized retake.
- Treat the same failure in two takes as systematic: rewrite or split the overloaded beat instead of adding more adjectives or blind rerolls. Never initiate a paid retake without explicit user authorization.
- Use the Repair Report Contract and the relevant repair template in `references/script-template-library.md`.
- Classify the failure before rewriting and name the exact observed error in the repaired prompt.
- Keep each action physically simple and correct: one model, one claim target, one clear camera angle per beat, one main action, explicit left/right hand start-action-end states, realistic light/shadows, and direct constraints for the observed failure. Do not repair anatomy by removing all useful garment actions.
- If the same failure appears across multiple colors, repair the shared prompt pattern instead of writing unrelated fixes for each color. Reuse the corrected stable structure for the remaining colors and vary only color, outfit styling, scene palette, and visible detail proof.
- For apparel videos, keep the garment visible in the first 1-2 seconds and describe front, side, back, fabric movement, neckline, sleeve, hem, and color in concrete visual language.
- If repairing a CTA shot, keep the CTA human-only and explicitly remove shopping-cart icons, arrows, arrow emoji, stickers, pointer graphics, product-link badges, pop-up badges, and UI overlays.
- Preserve the confirmed market, language, model preset, duration, platform, and product facts from the original script.
- If a visible model or creator speaks, keep mouth/lip movements synced to the exact spoken words and keep speech at a normal conversational pace with casual everyday wording.
- Return the corrected copy-ready Seedance/PopBoom prompt/script plus the repair output fields required by the Repair Report Contract.

## Default Behavior After User Authorizes Assumptions

Use these defaults only after the user says `你来定`, `跳过确认`, `直接按默认做`, or equivalent:

- 投放国家/地区：根据产品包装、页面语言、产品风格和用户上下文推测；如果没有线索，选择“美国 TikTok Shop”并标注为工作假设。
- 发布平台：TikTok。
- 成片口播语言：美国市场用美式英语；中国市场用中文；德国市场用德语；其他市场使用当地常见投放语言或英语并标注。
- 视频声音/画面形式：不要使用固定默认形式。必须先自动学习最近半个月 TikTok 相关服装类带货视频中销量好、转化率较高的视频形式，再结合该服装的品类、版型、材质/纹理、颜色、可见细节、穿搭场景和购买动机，选择最适合这件服装的形式。
- 达人：如无固定达人，设计虚拟达人形象。
- 视频时长：15 秒。

## Output Requirements

The complete UGC director plan must include all sections below, in this order:

1. 制作参数确认表
2. 产品与受众判断
3. 虚拟达人拟定
4. 达人图提示词
5. 产品图提示词
6. 10条Hook
7. 口播脚本
8. B-roll镜头清单
9. Seedance即可直接粘贴的视频生成提示词
10. 成片检查清单
11. 下一步操作
12. 促单文案与热门标签

Section 2 must include the pre-script evidence analysis: locked product/variant identity, objective attributes, seller claims, exact direct-review sample size and variant scope, positive/negative/mixed themes, buyer-image limits, conflicts, excluded sources, and the pain -> evidence -> solution claim -> close-up -> action -> visible-endpoint map. Use concise Chinese planning language. Avoid generic brand-ad language; prefer UGC-native detail: phone camera, real room lighting, controlled product handling, tactile close-ups, small imperfections, and visible proof. Use first-person purchase/test language only when an actual test or user authorization exists. In section 7, write the voiceover script as a shot-by-shot timeline where each beat includes one action/endpoint, speech mode plus canonical line or silence, product selling point, and when relevant the outfit/styling idea being shown.

## Seedance Prompt Rules

- Keep the directing intent internal. Put the deterministic identity-first role lock first in generation text—`Reference 1 / @Image1` fixed-model identity, then `Reference 2 / @Image2+` garment identity—then apply the selected market profile's language/performance lock, generation controls, visible actions/endpoints, concrete camera/light, spoken line/sound, continuity anchors, and compact constraints in the v6 order.
- For a default 15-second prompt, organize the viewer-facing performance as three continuous macro phases—recognition/result Hook, animated proof, and close/detail conviction—containing five to seven internal beats. Use three to four contiguous camera-setup runs; macro phase, setup run, and internal beat are separate layers. Allow sequential completed actions, but never overlap them.
- Include one main action, one visible endpoint, one motivated camera instruction, one motivated physical light source, and one specific sound intent per beat.
- Treat `@Image1` as the fixed creator-identity source only. Treat each exact clothing-three-view `interface_tag` (`@Image2` or higher) as garment identity only and explicitly replace any source support/layout with the one `@Image1` creator wearing the garment in one continuous believable real-world scene.
- Use concrete timestamps and actions.
- For each timestamped beat, include shot/framing, one action and visible endpoint, `proof_endpoint` when it is a proof, speech mode/mouth visibility/lip-sync state, `spoken_language`, canonical spoken line or silence, and product detail being shown.
- When a spoken line mentions fabric, comfort, breathability, drape, texture, stitching, neckline, sleeve, hem, or other design detail, use a close-up, chest-to-hem shot, cuff close-up, neckline close-up, or gentle push-in that clearly shows the detail.
- For same-garment multi-color batches, explicitly state that every colorway uses the same baseline fabric construction and texture; only the color changes.
- Include a matching product-detail close-up and action for every specific part/material claim; do not merge unrelated claims into one generic proof beat.
- Keep voiceover present through most of the video: for a 15-second script, normally use five to six short spoken units and give every claim-proof action a matching concise line. Use no more than two visible-lip lines by default; when `secondary_fidelity_spend: lip_sync`, allow at most three anchors under the 14-words-per-line/36-visible-words budget and strict load limits. Avoid more than 2 seconds of silence without a product-detail reason.
- Make model actions and camera distance match the spoken product point, such as pointing once to the V-neck in a neckline close-up, touching one sleeve cuff in a cuff close-up, smoothing the hem in a waist-to-hem shot, or doing a slow half-turn for drape/back coverage. Add one lower-left human gesture only when `commerce_cta_mode: light_link` or an authorized evidence-backed promo requires it.
- Describe natural facial micro-expressions when the face is visible: relaxed smile, quick glance, subtle eyebrow lift, tiny nod, natural blink, or small reaction that matches the spoken line. Across a creator-led clip, progress from warm recognition to animated certainty to pleased conviction/relief and use at least two distinct expression cues when multiple lines are visible; avoid stiff frozen faces, blank eyes, fixed grins, doll-like skin, and exaggerated acting.
- Describe realistic light and shadow in every prompt: exact light source and direction, soft shadows on face/body/fabric, natural highlights, eye catchlight when visible, and consistent shadows across the scene.
- Describe realistic scene details in every prompt: location, lighting, camera setup, model posture, background, product placement, and simple movement.
- Compile the occasion-first scene sales job before the Shot blocks: name the destination/use moment, buyer styling question, complete outfit answer, selected video form, and location-continuity plan. Keep enum labels machine-only.
- Describe the complete outfit styling: matching bottoms, shoes, bag, jewelry/bracelet/watch, and why they fit the garment color and style.
- Match the scene to the clothing style and color palette; use backgrounds that make the garment readable and suitable for its real wearing occasion.
- Add styling actions that help buyers imagine wearing it, such as one front tuck, adding a small bag, touching a necklace once, showing a bracelet near the sleeve cuff, or stepping back to show shoes.
- Keep each beat anatomically stable: one model, one camera angle, one main action, explicit posture, and complete left/right hand trajectories. Stability means the action is correctly performed and ends clearly, not that the model remains motionless.
- Write each beat as inherited start state -> one motivated proof action -> readable endpoint -> natural continuation or intentional cut. Maintain direct-response cadence with a newly completed product/styling action about every two to three seconds, while keeping micro-expressions and breathing supportive rather than substituting them for proof.
- For every selling-point proof beat, render the method from `references/selling-point-action-proof-methodology.md`: proof target, human motive, matching frame, exact left/right hand starts inherited from the prior endpoint when possible, one action path, readable endpoint, phone/prop ownership, natural handoff/cut, causal micro-cues, and the concise line that names the same target.
- In mirror selfie POV, keep the filming hand on the phone throughout a continuous take, keep alternate-color garments/accessories owned until an explicit place-down/handoff/cut, and motivate step-back, movement, and walk-closer by the proof the viewer needs. Never reset both hands to the hips between internal beats.
- Sequence several apparel-safe actions across the clip: neckline trace, cuff touch, arm raise, pinch-release, hem smooth/release, side turn, pocket use, closure operation, front tuck, and fit reveal, followed by the selected verdict and any profile-authorized human-only CTA gesture.
- Place the product in the first 1-2 seconds.
- Put a strong, buyer-relevant hook in the first 1-2 seconds; never open with a weak generic product intro.
- Base spoken selling points on the persisted pain-solution map and allowlisted evidence IDs. Review-derived pains must remain qualified to the sampled variation-family scope; seller or summary text cannot masquerade as buyer feedback.
- End with the selected market profile's evidence-safe verdict. If `commerce_cta_mode: light_link`, add one brief target-market link cue and describe only the model's small natural lower-left hand gesture or brief down-left glance; if `none`, add neither cue nor link gesture. The final shot must stay visually clean. Do not generate a shopping cart, arrow, arrow emoji/sticker, pointer or emoji sticker, floating icon, animated pointer graphic, product-link badge, pop-up badge, or extra UI overlay.
- Keep one coherent commercial throughline per prompt, supported by several distinct claim-matched part/effect proofs and actions; never interpret this as permission to show only one feature or leave the model static.
- Specify 9:16 vertical format unless the user chooses otherwise.
- Preserve the exact interface-provided garment reference tag in `interface_tag`; never invent, translate, renumber, or normalize it. New `canonical_prompt_v7` compilation accepts only ASCII `@ImageN` garment tags whose numeric index is 2 or greater, because `@Image1` is reserved for the fixed creator identity; historical serializers may retain their already-recorded controlled tags for read-only audit. Stop on new-work garment `@Image1`, aliases, duplicates, gaps, non-ASCII tag families, or mojibake instead of compiling it.
- Mark every spoken beat as `on_camera_dialogue`, `offscreen_voiceover`, or `none`, with matching mouth visibility and lip-sync state. Use off-screen voiceover with the mouth out of frame for detail or motion proof.
- Give every spoken beat an exact `spoken_language` and every proof beat a concrete `proof_endpoint`. Apply `us_hybrid_share`, `de_live_simple`, or `de_hybrid_proof` exactly as selected; `de_live_simple` is allowed only under the German profile's low-load gate.
- Keep a separate beat-by-beat `voiceover_review` containing the exact market line and Chinese review translation. Hash it as `voiceover_review_sha256`; never place the Chinese translation in the prompt, audio, screen text, or market caption.
- Keep generated in-video text limited to the required persistent upper-left fit-stats overlay; provide all captions separately.
- If a visible model or creator speaks, require lip-sync: mouth movements must match the voiceover/script content.
- Voiceover pacing must be natural conversational speed, neither too fast nor too slow, with proof-pivot pauses and varied emphasis. For 15-second American-English creator-led mirror shares, audition roughly 50-60 words at 200-235 WPM only when clear and enforce the validator's broader safe total; use language-specific judgment elsewhere.
- Voiceover wording must be casual, everyday, and buyer-friendly; avoid stiff brand-ad language, robotic phrasing, and translation-like wording.
- Voiceover must sound spoken rather than read: use short single-purpose sentences, vary proof-line openers, and reject official transitions, brochure constructions, and standalone generic praise. Do not use the same opener in three or more non-CTA proof/styling lines.
- For detail richness, select one core sellable wearing result and prove it through concrete information angles when evidence permits. Name the exact part plus the visible position, movement, construction, or styling relationship; keep one claim per beat, never fill an evidence gap with a fabricated feature, and remove details that do not support the selected result.
- Voiceover should sound like one connected recommendation and progress through recognition/result Hook, animated product proof, and conviction/styling/CTA, instead of isolated equal-energy sentences or a mostly silent middle.
- Voiceover should include at least one practical outfit idea, such as what bottoms, shoes, bag, or jewelry to pair with the garment and what real-life scene it fits.
- Keep the first-generation constraint slot compact. Lead with the deterministic identity/reference lock: `Reference 1 / @Image1 = fixed-model identity only`; `Reference 2 / @Image2 = garment identity only` (and any additional garment reference follows the same `@Image2+` rule) and has passed `human_identity_pixels_absent: true`. Then state positive replacements: one `@Image1` creator, the same reference-bound garment, exactly two connected arms/hands, explicit left/right trajectories, sequential actions with one target at a time, one believable scene, consistent physical light, and the required persistent upper-left fit-stats overlay. Default to one active hand; allow one verified coordinated two-hand proof. Block transfer of every apparel-reference human cue plus all other generated text/watermarks/UI/CTA graphics and product identity/color/material changes in one concise line; add only one observed-failure-specific negative during repair.
- Require `market_prompt_profile_sha256`, `generation_controls_sha256`, and `voiceover_review_sha256` as machine-only prompt bindings alongside the existing quality, research, canonical-beat, and compiled-text hashes. Never print hashes, raw corpus prompts, corpus statistics, internal analysis, or review translations inside `compiled_text`.

## References

- Always read `references/three-layer-deadline-contract.md`, `references/quality-directing-contract.md`, `references/sku-batch-timeline-contract.md`, `references/amazon-product-review-evidence-contract.md`, and `references/market-prompt-compiler-contract.md` before any full apparel compile; together they own schema `1.4`, the global/asset/shot deadline hierarchy, Amazon/review provenance, pain-to-solution binding, claim-proof/action binding, market routing, quality allocation, canonical timing, and submission gates.
- After market selection, read only `references/us-winning-prompt-profile.md` for `us_champion_v1` or `references/de-winning-prompt-profile.md` for `de_champion_v1`; never load both profile documents for ordinary single-market work.
- Read `references/motion-realism-and-commerce-cadence.md` before composing any apparel timeline or generation prompt; it owns commerce action frequency, complete action grammar, human kinetics, and the compact cadence clause compiled into `canonical_prompt_v7`.
- Read `references/selling-point-action-proof-methodology.md` before writing apparel scripts, B-roll, or final prompts; it owns the selling-point-to-action-proof mapping, action selector, hand-anchor grammar, micro-cue placement, and action-proof audit.
- Read `references/occasion-scene-routing.md` before choosing a creative form, scene, outfit, or timeline; it owns destination/use-case selection, scene traffic, video-form routing, the private-interior gate, US/DE scene treatment, and `scene_strategy` validation.
- Read `references/streaming-quality-contract.md` before any multi-color release is allowed to submit ahead of unfinished colors.
- Use `references/intake-checklist.md` whenever blocking decisions are missing, but do not block on missing publishing platform, video duration, or video sound/visual format; default platform to TikTok, duration to 15 seconds, and auto-select the format as part of the plan.
- Use `references/output-blueprint.md` for the required complete output structure.
- Use `references/seedance-prompt-patterns.md` for Seedance-ready visual forms after market controls are locked.
- Use `references/script-template-library.md` for base creative formats, stability audits, or explicit user-requested repairs after market controls are locked.
