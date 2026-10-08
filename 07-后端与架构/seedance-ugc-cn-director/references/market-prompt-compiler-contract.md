# Market Prompt Compiler Contract

Use this contract for every new United States or Germany apparel script, canonical timeline, and Seedance/PopBoom prompt. It is the shared compiler layer between evidence-backed selling-point selection and the existing creative-form/template layer.

This contract distills recurring structures from a user-provided corpus of seller-winning prompts. The corpus is descriptive evidence about repeated prompt construction, not causal proof that any isolated phrase, camera mode, or CTA guarantees sales. Do not load the source workbook, raw prompts, corpus statistics, or complete analysis reports into normal production context. Evidence, product truth, safety, identity fidelity, and current user instructions always override corpus frequency.

## Authority And Load Order

Apply rules in this order:

1. identity, reference-role, product-evidence, claim-safety, and paid-execution rules;
2. this market compiler contract, `occasion-scene-routing.md`, and the selected market profile;
3. the German account-interaction overlay when the market is Germany;
4. the selected base template and creative form;
5. optional styling, pacing, and tone preferences.

A lower layer may vary creative expression but may not weaken or contradict a higher layer. If a profile example conflicts with evidence, remove the claim or action rather than forcing the example.

## Version Contract

Every new market-compiled work item uses all five identifiers together:

- `schema_version: "1.4"`;
- `quality_contract_id: "zibuyu_ugc_quality_v4"`;
- `serializer_id: "canonical_prompt_v7"`;
- `three_layer_deadlines.contract_id: "zibuyu_three_layer_deadlines_v1"`;
- `market_prompt_contract_id: "zibuyu_market_prompt_v1"`.

Schema `1.4` / `zibuyu_ugc_quality_v3` / `canonical_prompt_v6` and schema `1.3` / `zibuyu_ugc_quality_v2` / `canonical_prompt_v5` remain read-, poll-, download-, quality-review-, and report-compatible for historical work. They are not valid for a new prompt, rewrite, repair, release, or paid submission.

The only unpaid exception is `legacy_v5_exact_resume`: a previously validator-approved but not-yet-submitted v5 package may resume only after explicit user approval and only when its prompt bytes, ordered references, reference hashes, model, duration, ratio, resolution, and every request hash remain exactly unchanged. Any content change requires migration and a fresh v7 compile. A job that already has a `record_id` is accepted work: never migrate, regenerate, or resubmit it; only poll, download, inspect, or report it.

## Market Selection Gate

Select the market before choosing a base template, writing hooks, or building a timeline.

- `美1`, `美2`, `美3`, United States, or American English selects `us_champion_v1` and requires `references/us-winning-prompt-profile.md`.
- `德1`, `德2`, `德3`, Germany, or German selects `de_champion_v1` and requires `references/de-winning-prompt-profile.md`.
- Never translate a completed US prompt into German or a completed German prompt into English. Recompile from the shared evidence and action-proof plan through the destination profile.
- If market, voiceover language, and fixed-model preset disagree, stop before prompt writing and resolve the identity/market conflict.

## Required Structured Controls

Persist one selected market profile and one `generation_controls` object before populating the canonical timeline:

```yaml
market_prompt_contract_id: zibuyu_market_prompt_v1
market_prompt_profile_id: us_champion_v1 | de_champion_v1
generation_controls:
  prompt_shell_mode: us_technical_shell | us_compact_storyboard | de_performance_script
  delivery_mode: us_hybrid_share | de_live_simple | de_hybrid_proof
  camera_mode: fixed_phone | creator_handheld | friend_handheld
  music_mode: none | low_non_lyrical
  verdict_mode: soft_verdict | ownership_verdict | none
  commerce_cta_mode: none | light_link | evidence_backed_promo
  screen_text_policy: no_generated_text | fixed_model_stats_only
  scene_strategy:
    wear_context: outward_wear | homewear | mixed
    scene_role: occasion_outfit_solution | movement_proof | fit_proof | detail_proof | multi_styling
    primary_scene_id: one approved occasion-scene ID
    primary_scene_description: concrete natural-language location
    occasion: concrete destination or use moment
    buyer_styling_question: occasion-specific outfit uncertainty
    outfit_answer: complete bottoms, shoes, bag, and accessory formula
    video_form: one approved occasion-scene video form
    location_plan: single_primary_scene | primary_plus_proof_cut
    bedroom_policy: excluded | proof_only | primary_justified | homewear_primary
    proof_scene_id: optional; required for primary_plus_proof_cut
    proof_scene_description: optional; required for primary_plus_proof_cut
    bedroom_justification: optional; required for primary_justified
  promotion_evidence_ids: [evidence_id]  # optional; required for evidence_backed_promo
```

Rules:

- US accepts only the two US shell modes and `us_hybrid_share`.
- Germany accepts only `de_performance_script` and either `de_live_simple` or `de_hybrid_proof`.
- `camera_mode` is one global capture grammar. A detail setup may become locked or subtly reframed for proof, but the prompt may not call the same setup both fixed and handheld. Any mode change requires an intentional cut and a new contiguous `camera_setup_id` run.
- `music_mode` is exclusive. `none` and `low_non_lyrical` may never coexist. Natural location ambience, fabric rustle, footsteps, café sounds, breeze, or bag sounds are environmental sound, not music.
- `evidence_backed_promo` requires a unique non-empty `promotion_evidence_ids` array whose IDs resolve to independently eligible exact-variant user-provided offer evidence, plus explicit user authorization. Omit this optional field for all other commerce modes. Promotional language is never a market-profile default.
- `screen_text_policy` is `fixed_model_stats_only` only when the task explicitly requires the persistent fit-stats overlay. Use `no_generated_text` when the prompt bans all visible text. Both modes prohibit subtitles, captions, translations, product names, prices, logos, watermarks, arrows, stickers, badges, link/cart graphics, and platform UI; `no_generated_text` also requires an empty `allowed_screen_text` list.
- `scene_strategy` is required and is covered by `generation_controls_sha256`. Read `occasion-scene-routing.md` for the complete field enums, scene/use-case router, video-form router, US/DE adapters, and private-interior gate.
- Every v7 beat carries a machine `scene_id`. Under `single_primary_scene`, all beats use the primary ID. Under `primary_plus_proof_cut`, allow one transition, keep the primary scene in the majority and final beat, and use a private proof scene only in the first two beats.
- `creative_delta.scene_id` and `quality_plan.continuity_anchors.scene` must equal `scene_strategy.primary_scene_id`; `creative_delta.styling_signature` and `quality_plan.continuity_anchors.outfit` must equal `scene_strategy.outfit_answer`.
- An outward-wear garment may not default to `bedroom_mirror`, `closet_try_on`, `bathroom_mirror`, or `home_living`. A private primary scene requires a specific fit/detail/multi-styling justification; convenience, natural light, generic UGC feel, or generation stability is insufficient.

## Shared Winning-Prompt Skeleton

Every market profile must preserve this common skeleton:

1. Show the worn garment and the selected wearing result in the first 1-2 seconds.
2. Lock exactly one `core_sellable_wearing_result` derived from buyer hesitation plus eligible garment evidence.
3. Lock one occasion-first scene strategy that answers `where would the buyer wear this?` and `how should she style it there?` before selecting the base creative form.
4. Bind every commercial claim through one exact chain:

   `claim -> product part/effect -> matching frame -> one proof action -> proof_endpoint -> the spoken line naming that same target`

5. Organize the viewer experience into three macro phases: recognition/result Hook, animated proof, and close/detail conviction.
6. For a default 15-second clip, use three to four contiguous camera-setup runs and five to seven sequential internal beats. Macro phase, setup run, and internal beat are different layers; several beats may share one setup when their actions are sequential and readable.
7. Carry posture, gaze, weight, filming/task hand, props, garment state, scene identity, and the prior endpoint forward. Reset, place down, hand off, change location, or cut only when declared.
8. Put complex detail, turn, pull, closure, or styling proof under off-screen voiceover with the mouth out of frame. Visible dialogue is allowed only under the profile's load budget.
9. Keep CTA visual behavior human-only. A real hand gesture or glance is permitted; generated arrows, carts, icons, stickers, badges, or link graphics are not.
10. Keep evidence gates intact for softness, comfort, breathability, stretch, opacity, anti-wrinkle behavior, composition, performance, price, discount, stock, shipping, and personal-experience claims.

## Beat And Voiceover Review Fields

Every new v7 canonical beat adds:

- `spoken_language`: exact locale for a spoken beat, or `null` for silence;
- `proof_endpoint`: the concrete product state the viewer can read after the action, or `null` for a non-proof beat.

`proof_endpoint` must agree with the visible endpoint and the bound claim-proof plan. It cannot be a feeling, adjective, camera move, smile, or abstract sales result.

Keep one separate `voiceover_review` projection with one row per beat:

```yaml
- beat_id: string
  speech_mode: on_camera_dialogue | offscreen_voiceover | none
  spoken_language: en-US | de-DE | null
  market_line: string | null
  zh_cn_translation: string | null
  silence_reason: string | null
```

The market line and `silence_reason` must equal the canonical beat fields. A spoken row requires a meaningful Chinese translation in `zh_cn_translation`; labels, sequence numbers, `待翻译`, `TBD`, or any other placeholder fail validation. A silent row keeps it null. The Chinese text is review-only: never insert it into `compiled_text`, generated subtitles, screen text, audio, or the market caption.

## Prompt Hash Bindings

Every v7 prompt projection carries four additional lowercase SHA-256 values:

- `market_prompt_profile_sha256`: canonical UTF-8 JSON hash of the exact selected profile object from `market_prompt_contract.py`;
- `generation_controls_sha256`: canonical UTF-8 JSON hash of the exact `generation_controls` object;
- `voiceover_review_sha256`: canonical UTF-8 JSON hash of the complete review projection, including the Chinese review translations.
- `three_layer_deadlines_sha256`: canonical UTF-8 JSON hash of the complete global, asset, and per-shot deadline object.

These hashes are machine fields only. Do not print them, internal IDs, review translations, or source analysis in the generation prose.

## Canonical Prompt V7 Order

Compile `canonical_prompt_v7` in this order:

1. deterministic fixed-model identity and garment-reference role locks;
2. the complete `GLOBAL NON-NEGOTIABLES` projection for format, camera, capture quality, lens/depth, post-processing, audio, language, music, and screen text;
3. the complete `SUBJECT, GARMENT, SCENE, LIGHT, AND SOUND NON-NEGOTIABLES` projection for creator/body, garment/color/construction, fabric physics, outfit/props, scene, lighting, and soundscape;
4. selected market language/performance control plus occasion, buyer styling question, complete outfit answer, primary scene, video form, and location-continuity control;
5. exactly three viewer-facing macro-phase blocks containing all five to seven timestamped internal beats, with one beat-specific `Shot non-negotiables` projection after every beat;
6. market-native voice/delivery direction and the selected verdict/commerce close.

Read `three-layer-deadline-contract.md` for the authoritative object fields, exact projection rules, mandatory failure exclusions, and hash/release gate. Deadline prose is not an optional negative-prompt appendix.

The US profile uses this order as a generation control specification. The German profile places its German speaker, voice, and lip-sync lock immediately after the immutable identity/reference lines, before scene and shot prose.

## Conflict And Rejection Gate

Reject or repair before handoff when any condition is true:

- market/profile/model/language mismatch;
- a completed prompt was translated instead of recompiled;
- product is absent in the first 1-2 seconds;
- more than one core wearing result competes for the Hook;
- `scene_strategy` is missing, does not answer a concrete occasion styling question, or lacks a complete outfit answer;
- an outward-wear garment defaults to a private interior without the declared proof-only or specific primary justification route;
- beat `scene_id` values drift from the selected primary/proof plan, location hopping exceeds one transition, or the primary use-case scene is absent from the final result;
- a claim, target, frame, action, endpoint, and spoken line do not describe the same proof;
- a complex proof is combined with visible dialogue, active camera, and multiple active actions;
- the same camera setup is both fixed and handheld;
- music modes conflict or music competes with speech;
- a German spoken slot contains Chinese or high-confidence English template residue outside approved names/tokens;
- Chinese review translation appears in generation text;
- the CTA relies on a generated graphic rather than the person;
- generated text exceeds the one fit-stats overlay;
- price, discount, stock, urgency, performance, or personal experience is unsupported;
- raw corpus prompts, corpus statistics, internal analysis, or meta-instructions leak into `compiled_text`.
- any global, asset, or shot deadline layer is missing, drifts from its batch/variant/beat source, lacks mandatory exclusions, or hashes differently from the persisted object.
