# Same-SKU Batch Compiler And Canonical Timeline Contract

This contract is mandatory for both single-color and same-SKU multi-color apparel work. It is the authority when older instructions repeat analysis per color or independently write the script, B-roll list, and final Seedance/PopBoom prompt.

Read `three-layer-deadline-contract.md`, `quality-directing-contract.md`, `market-prompt-compiler-contract.md`, and `amazon-product-review-evidence-contract.md` with this file. New work uses schema `1.4`, `quality_contract_id: zibuyu_ugc_quality_v4`, `market_prompt_contract_id: zibuyu_market_prompt_v1`, `three_layer_deadlines.contract_id: zibuyu_three_layer_deadlines_v1`, and `canonical_prompt_v7`. Historical v6 and schemas `1.0` through `1.3` remain read-compatible for audit, accepted-job resume/polling, download, and reporting. Recompile or migrate an older object before any repair, rewrite, or new paid submission, except for the narrow `legacy_v5_exact_resume` rule below.

## Compile Modes

- `single_compile`: one eligible color variant.
- `sku_batch_compile`: two or more color-only variants in the same batch key. Invoke the director once for the whole partition.
- `repair`: revise an existing variant by editing its canonical timeline, incrementing `timeline_version`, and recompiling all projections.

PopBoom rendering remains one paid task per color. Batch compilation does not merge colors into one video.

For quality-gated streaming, keep the same creative contract but use `streaming-quality-contract.md`: one immutable all-color v2 `batch-plan.json` and one complete schema `1.4` `single_compile` release per ready color. Streaming is scheduling, not a weaker compile mode, and never authorizes partial timelines or unchecked prompts.

## SKU-Family Eligibility And Partitioning

Assign a stable `sku_family_id` and `variant_id` before creative work. Variants belong to one SKU family only when category, silhouette, neckline, sleeve construction, length, closure, seams/panels, texture scale, thickness, opacity, and drape match and color is the only product difference. Split uncertain or structurally different items before compilation.

Partition an eligible family by this batch key:

```text
sku_family_id + model_preset + market + voiceover_language + market_prompt_profile_id + platform +
duration_seconds + aspect_ratio + resolution + frame_rate_fps
```

Barrier mode requires every three-view reference in the partition to pass garment QC and the zero-human-identity-pixel audit before `sku_batch_compile` begins. Quality-gated streaming may precompile the shared plan before all references finish, but every intended color and product-source group must be known, the all-color differentiation plan must pass, and the current color's own three-view must pass both gates before its final `single_compile` release. A new color added before payment requires a new locked plan; after the first paid release, move new or replanned colors to a new parent run.

## Compile Once Versus Per-Variant Work

Create these exactly once per batch partition:

- garment construction and material baseline;
- product/review evidence collection, source grading, conflict analysis, and buyer-pain to evidenced-solution analysis;
- recent-half-month TikTok research digest or explicitly labeled local-knowledge fallback;
- claims allowlist and safety constraints;
- one selected template and one `timeline_grid_id`;
- market, language, platform, model, duration, ratio, and resolution settings.
- one market prompt contract/profile selection. Select it before the base template; never translate a finished US prompt into German or a finished German prompt into English.

Create these separately for every color:

- color evidence and three-view binding;
- creative delta and variation signature;
- canonical timeline;
- compiled script, B-roll projection, final prompt, caption, and exactly five hashtags.

In streaming mode, plan each color's creative delta, caption, hashtags, non-CTA spoken-intent IDs, and non-CTA visual signatures in the immutable parent plan. Finalize the complete canonical timeline, renderings, exact reference tag/path/hash, and director receipt only after that color's three-view passes QC. The release must match the plan exactly.

Reuse the proof obligations and stability framework, not complete shot content. Never rerun shared analysis merely to change a color name.

For a three-color partition, target an estimated 40-60 percent reduction in creative-reasoning work versus three independent compiles by keeping `shared_analysis_passes: 1`, `research_passes: 1`, and `template_selections: 1`. This is an operating estimate, not a guaranteed wall-clock metric. Never reach it by weakening per-color evidence, canonical timelines, pairwise differentiation, or QA.

## Required Batch Object

Use one in-memory or persisted object with this logical shape. Equivalent JSON is acceptable.

```yaml
schema_version: "1.4"
quality_contract_id: "zibuyu_ugc_quality_v4"
market_prompt_contract_id: zibuyu_market_prompt_v1
normalization_id: "nfkc_casefold_ws_punct_v1"
compile_mode: single_compile | sku_batch_compile | repair
batch_compile_id: string
batch_revision: integer
sku_family_id: string
batch_key:
  model_preset: string
  market: US | DE
  voiceover_language: en-US | de-DE
  market_prompt_profile_id: us_champion_v1 | de_champion_v1
  platform: string
  duration_seconds: number
  aspect_ratio: string
  resolution: string
  frame_rate_fps: positive integer
shared_core:
  research_digest_id: string
  garment_baseline_id: string
  template_id: string
  timeline_grid_id: string
  material_lock: string
  research_bundle:
    contract_id: amazon_product_review_evidence_v1
    requested_url: string | null
    canonical_url: string | null
    marketplace: string | null
    parent_product_id: string | null
    child_product_id: string | null
    selected_variant: {color: string | null, size: string | null}
    captured_at: RFC3339 string | null
    collection_route: chrome | public_web | user_capture | not_applicable
    status: complete | partial | blocked | not_provided
    limitations: [string]
    evidence_sources:
      - source_id: string
        source_kind: amazon_catalog_attribute | amazon_seller_copy | amazon_customer_review | amazon_customer_image | amazon_qa | amazon_review_summary | third_party_summary | visible_reference | garment_label | user_provided | verified_test
        source_role: product_fact | seller_marketing | buyer_experience | visual_observation | discovery_only | verified_measurement
        url: string  # Amazon sources require approved HTTPS Amazon/Amazon-image host and matching path identity
        product_id: string | null
        child_product_id: string | null
        variant_scope: exact_child | parent_family | sibling_child | unknown | not_applicable
        locator: string
        captured_at: RFC3339 string
        content_sha256: string
        review_id: string | null
        parent_review_id: string | null
        rating: number | null
        review_date: string | null
        review_date_status: captured | not_captured | null
        verified_purchase: boolean | unknown | null
    evidence_items:
      - evidence_id: string
        source_id: string
        assertion_kind: visible_feature | exact_composition | fit_attribute | stretch_attribute | weight_attribute | care_attribute | seller_qualitative_claim | buyer_experience | buyer_visual_observation | verified_performance
        statement: string
        exact_product_match: boolean
        exact_variant_match: boolean | unknown
        conflict_status: clear | conflicted | mixed | unverified
        permitted_uses: [claim_direct | claim_qualified | pain_context | visual_only | discovery_only | blocked]
        performance_demo_allowed: boolean
    review_analysis:
      status: collected | partial | blocked | no_reviews | not_provided
      sampled_review_ids: [string]
      buyer_image_count: integer
      themes:
        - theme_id: string
          normalized_theme: string
          sentiment: praise | complaint | mixed
          review_evidence_ids: [evidence_id]
          mention_count: integer
          claim_strength: single_attribution | sample_theme | risk_only
          permitted_script_use: pain_context | claim_qualified | blocked
      conflicts: [object]
      excluded_sources: [object]
  pain_solution_map:
    - pain_point_id: string
      pain_statement: string
      basis: review_theme | direct_review | user_provided | visible_reference | category_inference
      evidence_ids: [evidence_id]
      review_theme_id: string | null
      status: script_eligible | risk_only
      selected_for_script: boolean
      solution_claim_ids: [claim_id]
      target_language_terms: [string]
  claims_allowlist: [claim_id]
  claims_registry:
    - claim_id: string
      feature_id: string
      product_part_id: neckline | sleeve | hem | slit | texture | material_performance | drape | fit | pocket | closure | styling
      spoken_claim_terms: [target-language phrase]
      evidence_basis: visible_reference | user_provided | product_page | verified_test
      evidence_ref: string
      assertion_level: visible | qualitative | numeric
      assertion_kind: visible_feature | exact_composition | fit_attribute | stretch_attribute | weight_attribute | care_attribute | seller_qualitative_claim | buyer_experience | buyer_visual_observation | verified_performance
      claim_mode: direct | qualified | visual_only
      evidence_ids: [evidence_id]
  safety_constraints: [string]
efficiency_accounting:
  independent_baseline_compiles: integer
  shared_analysis_passes: 1
  research_passes: 1
  template_selections: 1
  variant_delta_count: integer
  estimated_reasoning_reduction_percent: number
variants:
  - variant_id: string
    color_name: string
    garment_signature:
      category: string
      silhouette: string
      neckline: string
      sleeve: string
      length: string
      closure: string
      seams_panels: string
      texture_scale: string
      thickness: string
      opacity: string
      drape: string
    source_images: [path]
    three_view_path: path
    three_view_sha256: string
    three_view_qc: passed
    references:
      - reference_id: string
        variant_id: string
        role: apparel_three_view
        interface_tag: string  # new v7 work uses exact ASCII @ImageN with N>=2; historical controlled tags are read-only
        local_path: path
        sha256: string
        human_identity_pixels_absent: true
        must_transfer: [garment_identity, exact_color, silhouette, neckline, sleeve_construction, hem, seams, texture_scale, thickness, opacity, drape]
        must_not_transfer: [white_background, three_panel_layout, panel_dividers, repeated_bodies, multiple_models, reference_skin_tone, reference_ethnicity, reference_neck, reference_chest_collarbone, reference_shoulders, reference_arms, reference_hands_fingers, reference_nails, reference_tattoos, reference_jewelry, reference_body_shape, reference_face_hair, reference_pose, reference_camera, reference_environment, embedded_text, watermark]
        positive_replacement: one_creator_same_garment_single_real_scene
    three_layer_deadlines:
      contract_id: zibuyu_three_layer_deadlines_v1
      global_deadlines: complete exact object from three-layer-deadline-contract.md
      asset_deadlines: complete exact object from three-layer-deadline-contract.md
      shot_deadlines: one complete exact object per canonical beat
    quality_plan:
      directing_intent: string
      directorial_voice: observational_naturalist | intimate_minimalist | graphic_formalist
      primary_fidelity_spend: garment_identity
      secondary_fidelity_spend: visible_proof | lip_sync | natural_motion | scene_readability
      economized_elements: [string]
      hero_proof_id: string
      claim_proof_plan:
        - claim_proof_id: string
          claim_id: string
          spoken_intent_id: string
          beat_id: string
          proof_target: string
          evidence_basis: visible_reference | user_provided | product_page | verified_test
          evidence_ids: [evidence_id]
          framing_class: detail_closeup | chest_to_hem | side_back | full_fit | styling
          action_type: point_trace | touch_release | pinch_release | pull_release | raise_arm | smooth_release | turn_settle | walk_settle | open_close | pocket_use | front_tuck | style_adjust
          hands_required: one | two | body
          expected_visible_change: string
    continuity_anchors:
        creator_identity: string
        garment_identity: string
        outfit: string
        scene: string
        lighting: string
    generation_controls:
      prompt_shell_mode: us_technical_shell | us_compact_storyboard | de_performance_script
      delivery_mode: us_hybrid_share | de_live_simple | de_hybrid_proof
      camera_mode: fixed_phone | creator_handheld | friend_handheld
      music_mode: none | low_non_lyrical
      verdict_mode: soft_verdict | ownership_verdict | none
      commerce_cta_mode: none | light_link | evidence_backed_promo
      screen_text_policy: fixed_model_stats_only
      scene_strategy:
        wear_context: outward_wear | homewear | mixed
        scene_role: occasion_outfit_solution | movement_proof | fit_proof | detail_proof | multi_styling
        primary_scene_id: approved occasion-scene ID
        primary_scene_description: string
        occasion: string
        buyer_styling_question: string
        outfit_answer: string
        video_form: approved occasion-scene video form
        location_plan: single_primary_scene | primary_plus_proof_cut
        bedroom_policy: excluded | proof_only | primary_justified | homewear_primary
        proof_scene_id: approved occasion-scene ID | omitted
        proof_scene_description: string | omitted
        bedroom_justification: string | omitted
      promotion_evidence_ids: [evidence_id] | omitted
    creative_delta:
      hook_angle_id: string
      hook_text_original: string
      color_terms: [string]
      opening_visual_id: string
      scene_id: string
      scene_palette: [string]
      styling_signature: string
      proof_focus_id: string
      pain_focus_id: string
      signature_action_id: string
      spoken_intent_ids: [string]
      caption_angle_id: string
      difference_summary: string
    canonical_timeline:
      timeline_id: string
      timeline_version: integer
      duration_seconds: number
      beats:
        - beat_id: string
          start_seconds: number
          end_seconds: number
          purpose: hook | pain | proof | styling | cta
          framing: string
          camera_motion: string
          camera_setup_id: string
          camera_motivation: string
          scene_id: approved occasion-scene ID
          scene: string
          lighting: string
          light_motivation: string
          actor: string
          core_action: string
          action_target: string
          left_hand_state: string
          right_hand_state: string
          spoken_line: string | null
          spoken_language: en-US | de-DE | null
          silence_reason: string | null
          spoken_intent_id: string
          product_point: string
          styling_point: string | null
          product_visibility: full | detail | partial | none
          reference_binding: string
          micro_expression: string | null
          audio: string | null
          visible_endpoint: string
          proof_endpoint: string | null
          lip_sync_required: boolean
          speech_mode: on_camera_dialogue | offscreen_voiceover | none
          mouth_visibility: visible | not_visible | partial
          motion_budget:
            camera: locked | subtle | active
            performer: still | micro | simple | active
            active_hands: zero | one | two
          beat_role: context_hook | reaction | product_claim | styling | cta
          claim_proof_id: string | null
          proof_target: string | null
          proof_action_type: string | null
          product_part_id: neckline | sleeve | hem | slit | texture | material_performance | drape | fit | pocket | closure | styling | null
          evidence_basis: visible_reference | user_provided | product_page | verified_test | null
          evidence_ids: [evidence_id]
          pain_point_id: string | null
          spoken_claim_ids: [claim_id]
          hook_semantics: pain_question | evidence_backed_contrast | context | null
          posture_id: relaxed_three_quarter | weight_shift_left | weight_shift_right | side_relaxed | seated_relaxed | hands_only_detail | mirror_selfie_relaxed | mirror_step_back | mirror_weight_shift | mirror_close_detail
          hand_plan:
            active_hands: none | left | right | both
            left:
              start_anchor: string
              action: string
              end_anchor: string
            right:
              start_anchor: string
              action: string
              end_anchor: string
    voiceover_review:
      - beat_id: string
        speech_mode: on_camera_dialogue | offscreen_voiceover | none
        spoken_language: en-US | de-DE | null
        market_line: string | null
        zh_cn_translation: string | null
        silence_reason: string | null
    renderings:
      script:
        source_timeline_id: string
        source_timeline_version: integer
        beats:
          - beat_id: string
            start_seconds: number
            end_seconds: number
            spoken_line: string | null
      broll:
        source_timeline_id: string
        source_timeline_version: integer
        shots:
          - shot_id: string
            parent_beat_id: string
            start_seconds: number
            end_seconds: number
            purpose: string
            framing: string
            camera_motion: string
            scene: string
            lighting: string
            actor: string
            core_action: string
            action_target: string
            left_hand_state: string
            right_hand_state: string
            product_point: string
            styling_point: string | null
            product_visibility: string
            reference_binding: string
            micro_expression: string | null
      prompt:
        source_timeline_id: string
        source_timeline_version: integer
        variant_id: string
        serializer_id: canonical_prompt_v7
        reference_ids: [string]
        beats: [canonical beat deep copies]
        reference_contract_text: string
        quality_plan_sha256: string
        research_bundle_sha256: string
        canonical_beats_sha256: string
        market_prompt_profile_sha256: string
        generation_controls_sha256: string
        voiceover_review_sha256: string
        three_layer_deadlines_sha256: string
        compiled_text: string
        compiled_text_sha256: string
    caption: string
    caption_claim_ids: [claim_id]
    caption_pain_point_ids: [pain_point_id]
    caption_claim_bindings: [{claim_id: string, text_anchor: registered target-language phrase}]
    caption_pain_bindings: [{pain_point_id: string, text_anchor: registered target-language phrase}]
    hashtags: [string, string, string, string, string]
validation:
  primary_error: string | null
  planned_differentiation_passed: boolean
  rendered_differentiation_status: verified | unverified | failed | not_rendered
  pairwise_differentiation: []
  timeline_integrity: []
  rendering_consistency: []
  eligible_for_popboom: boolean
  errors:
    - code: string
      scope: string
      variant_ids: [string]
      beat_ids: [string]
```

## Canonical Timeline Rules

The canonical timeline is the only source of truth for dialogue, timing, actions, camera, scene, lighting, product proof, styling, and CTA.

- Use half-open time intervals `[start_seconds, end_seconds)`.
- Use no more than millisecond precision and compare time values as exact decimal numbers, never binary-float tolerances.
- Beats must be ordered, contiguous, non-overlapping, gap-free, start at `0`, and end at the requested duration.
- For the default 15-second template, organize the viewer-facing performance as three macro phases—roughly `0-4` recognition/result Hook, `4-10` animated proof, and `10-15` close/detail conviction—while retaining five to seven sequential internal beats. A practical six-beat grid may still be `0-2.5`, `2.5-5`, `5-7.5`, `7.5-10.5`, `10.5-13`, `13-15`; the v6 serializer groups those internal beats into the three performance phases without reducing action/part coverage.
- Each internal beat has one camera setup, one main action, and explicit states for both hands. Macro phase, setup run, and internal beat are separate layers.
- Schema `1.4` requires a motivated camera, motivated physical light source, visible endpoint, exact `proof_endpoint` for proof/styling beats, `spoken_language`, lip-sync flag, explicit motion budget, `beat_role`, proof binding, posture, complete hand plan, exact evidence IDs, and hook pain binding for every beat.
- Every spoken line must fit its interval at natural conversational speed.
- The CTA remains human-only and visually clean under the existing CTA rules.
- For a default 15-second video, use three viewer-facing macro phases, exactly three or four contiguous `camera_setup_id` runs, and five to seven sequential internal beats. A run is one maximal contiguous sequence of equal setup IDs. Do not reuse a setup non-contiguously (`A -> B -> A`), and do not let one run cross a macro-phase boundary. Change or reframe only when the current frame cannot prove the new target; schema 1.4 may keep two separately bound, readable targets in one setup only when both remain inside the same macro phase.
- Require at least four non-CTA claim-proof actions in 15 seconds and at least three distinct canonical `product_part_id` groups. For another schema 1.4 duration, reserve two seconds for the mandatory human verdict/CTA close and scale the action floor as `max(1, ceil((duration_seconds - 2) / 4))`; this keeps an explicitly requested 5-second clip viable as Hook -> one proof -> close without weakening the 15-second floor. Repeating the same neckline under aliases such as collar/V-neck does not satisfy the distinct-target floor. Prop handling, facial reactions, camera motion, and the CTA gesture do not count as garment proof.
- `shared_core.claims_allowlist` is a non-empty unique array of exact `claim_id` values; every value resolves in `claims_registry`, and every used proof claim is allowlisted. Every schema `1.4` claim resolves its `evidence_ids` in the research bundle, and the exact set is copied unchanged into its proof and beat. Each claim and proof names only one garment part/effect and carries the same canonical `product_part_id`. Every registry entry supplies one or more unique `spoken_claim_terms` in the exact batch locale; the bound line must contain one of them.
- Every claim-proof beat declares exactly one `spoken_claim_ids` value equal to the bound plan's `claim_id`; hook, reaction, and CTA beats use `spoken_claim_ids: []`. This structured binding is authoritative across German, English, Chinese, and other spoken languages.
- A hook declares `hook_semantics: pain_question`, `evidence_backed_contrast`, or `context`, one valid `pain_point_id`, and the pain's exact evidence IDs; other beats use null pain and only proof beats carry claim evidence. A review-derived pain resolves to direct reviews or a valid theme; a risk-only or seller-created pain cannot become a conversion hook. A context hook, reaction, or CTA may not introduce a new garment-part, material, or performance fact. Put every product fact in its own bound proof beat.
- `evidence_backed_contrast` is a narrow schema `1.4` exception for a selected direct-review/review-theme material-expectation pain. It must be a negative declarative statement containing a registered pain term, not a question; the Hook keeps `spoken_claim_ids: []` and only the pain's review evidence. The immediately following beat must be a `purpose: proof`, `beat_role: product_claim` detail close-up whose single claim is one mapped exact-variant `texture` solution, with `proof_action_type: pinch_release`, `speech_mode: offscreen_voiceover`, and `mouth_visibility: not_visible`. No reaction, styling beat, second claim, or unsupported feel/performance language may intervene.
- Keep every beat at least 1.5 seconds. A beat with visible lip-sync must be at least 2 seconds, except that an explicitly requested schema 1.4 clip of 5 seconds or less may use a 1.5-second low-load visible Hook or close when camera, performer, hand, expression, and word-budget gates all pass.
- A product-claim or styling beat must perform one matching action; a passive hold may exist only inside the action's final endpoint, never as the whole explanatory beat.
- Visible lip-sync and CTA beats may not use active camera or active performer motion and may use at most one active demonstration hand. Ordinary detail proof uses one active hand; a verified two-hand proof is allowed only as one coordinated action with a locked/subtle camera, still torso, mouth out of frame, and complete left/right hand trajectories.
- `on_camera_dialogue` requires a visible mouth, exact lip-sync, and a non-empty spoken line; `offscreen_voiceover` requires the mouth out of frame and no lip-sync; `none` carries no spoken line.
- Current schema 1.4/v7 `DE` + `de_live_simple` + `fixed_phone` may opt a product-proof beat into [sequential live proof](sequential-live-proof.md): one explicit silent action window, a silent settling window, then visible German speech with zero moving hands. Only the fully validated timing object activates this narrow exception to the simultaneous-action/lip-sync restrictions above and below. Missing or invalid timing retains all existing rejections. Pinch/pull is limited to at most 2 cm of existing image-evidenced fit allowance and cannot become an elasticity demonstration.
- For whitespace-delimited visible dialogue, default to 10 words or fewer per line, no more than two visible anchors, and 20 visible words total in a 15-second timeline. When `secondary_fidelity_spend: lip_sync`, allow up to three visible anchors—one per macro phase—with 14 words per line and 36 visible words total, only under the locked/subtle camera, simple-or-lower performer, one-active-hand rule.
- Schema 1.4 `en-US` 15-second creator voiceover must contain 40-62 total spoken words. Treat roughly 50-60 words at 200-235 WPM as an aspirational read/TTS target only when clear. Audit `de-DE` locally and never copy the English count. Every on-camera line requires a meaning-matched `micro_expression`, and multiple visible lines require at least two distinct expression cues.
- `voiceover_review` is an exact beat-order review projection. For every spoken beat, `beat_id`, `speech_mode`, `spoken_language`, and `market_line` must equal the canonical beat and `zh_cn_translation` must be non-empty. For a silent beat, `market_line` and translation are null and `silence_reason` equals the beat. Chinese translations never enter `compiled_text`.
- Every spoken `US` beat uses `en-US`; every spoken `DE` beat uses `de-DE`. Reject Chinese characters in either market slot. For German, reject high-confidence complete English/template sentences while allowing registered German-commerce loanwords such as `Top`, `Basic`, `Look`, and `TikTok`.
- `generation_controls` governs delivery, camera, music, verdict, CTA, screen text, and the occasion-first scene strategy for the whole variant. Read `occasion-scene-routing.md`; require every beat `scene_id` to match the single primary scene or the declared one-transition primary-plus-proof plan. Require creative and continuity scene/outfit projections to equal the locked `primary_scene_id` and `outfit_answer`. An outward-wear garment cannot default to a private interior. A setup cannot contradict the chosen camera mode; `none` and `low_non_lyrical` music are mutually exclusive; complex proof uses off-screen voiceover with the mouth out of frame. Require exactly one final CTA/verdict beat. Only `light_link` may use a link instruction, and `evidence_backed_promo` requires explicit eligible promotion evidence. CTA visuals are human-only. `fixed_model_stats_only` permits exactly its one validated fit-stats line; `no_generated_text` permits none.
- Carry gaze, weight, expression, filming/task hand, props, and garment state from each endpoint into the next start state. In mirror selfie POV, the filming hand continuously owns the phone. An alternate garment/accessory cannot disappear without an explicit place-down, handoff, or intentional cut. Three or more symmetric both-hands-to-hips resets are invalid.
- Every variant's `hero_proof_id` must equal its creative delta's `proof_focus_id`; this identifies the highest-priority proof, not the only proof. Directing intent and camera/light/sound choices must serve the full evidence-backed proof sequence rather than decorative style.

Compile projections without invention:

- Script: project `beat_id`, interval, and `spoken_line` from the timeline verbatim.
- B-roll: project the listed visual fields from the same beats. A parent beat may be subdivided only when each child has `parent_beat_id`, inherits every non-time source field, and the child intervals exactly cover the parent's visual interval.
- Final prompt: deep-copy the canonical beats in order and preserve timings, spoken lines/languages, proof endpoints/bindings, evidence IDs, actions, hand plans, reference bindings, and one exact shot deadline per beat. Historical v6 and schemas `1.0` through `1.3` retain their recorded serializers byte-for-byte for read/poll/report compatibility. Every newly compiled schema `1.4` object serializes evidence-bound, identity-isolated, market-compiled creator-performance instructions with `canonical_prompt_v7`.

For new schema `1.4` compilation, use `canonical_prompt_v7` and `scripts/market_prompt_contract.py` as the machine source for market profiles, allowed tuples, profile hashes, generation-control hashes, deadline hashes, voiceover-review hashes, and ordering. Keep directing intent, allocation, research bundle, raw source metadata, pain mapping, evidence registry, Chinese translations, and claim-proof planning inside the machine object. Include the concise `reference_contract_text` verbatim in `compiled_text`, then serialize global audiovisual non-negotiables, subject/garment/scene/light/sound non-negotiables, and exactly three viewer-facing macro-phase blocks containing every internal timestamp, canonical market-language line, proof target/endpoint, matching frame, exact demonstration action, left/right hand behavior, state/prop handoff, and shot non-negotiables. The contract begins with `Reference 1 / @Image1 = fixed-model identity only`, followed by `Reference 2 / @Image2 = garment identity only`; every additional garment tag is `@Image3+`.

The deterministic v7 generation text applies the three deadline layers and selected market profile before the creative template. US uses either the technical-shell or compact-storyboard order with American friend-share delivery. DE locks the German speaker/language/lip-sync after immutable reference/global/asset controls, then compiles the German concern-to-part-to-action-to-verdict performance. Stability means correct global capture, persistent asset identity, beat-specific anatomy and physics, and non-overlapping actions, not a motionless model.

Keep the structured prompt beats outside the generation prose. Store the SHA-256 of their canonical UTF-8 JSON serialization in `canonical_beats_sha256`; never paste internal beat IDs or the full JSON block into the PopBoom generation text.

Reference and serializer rules:

- Every `reference_binding` must resolve to exactly one entry in that variant's `references`; every reference entry's `variant_id`, path, and SHA-256 must match the variant's three-view evidence.
- Reference position 1 and `@Image1` are reserved for fixed-model identity. Every apparel reference uses a unique, contiguous `@Image2+` tag and declares `human_identity_pixels_absent: true`; a garment `@Image1`, missing zero-pixel attestation, duplicate index, mixed `@图片`/`@Image` family, or index gap blocks compilation.
- `prompt.reference_ids` must equal the set of references used by its beats and may not resolve to another variant.
- Historical schema `1.0` keeps `canonical_prompt_v1`: serialize `prompt.beats` as UTF-8 JSON with non-ASCII characters preserved, object keys sorted, and compact separators, and include the exact JSON block in `compiled_text`.
- Historical schema `1.4` v3/v6 and schemas `1.0` through `1.3` retain their recorded serializers for compatibility. New schema `1.4` compilation requires `canonical_prompt_v7`: hash canonical beats, three-layer deadlines, the research bundle, selected market profile, generation controls, and voiceover review separately, but compile only deterministic generation instructions into `compiled_text`. Historical artifacts return `eligible_for_new_submission: false` unless the execution validator separately proves the strict `legacy_v5_exact_resume` exception.
- `compiled_text_sha256` is the lowercase hexadecimal SHA-256 of the exact UTF-8 `compiled_text` bytes.
- All variants in one color-only SKU family must have equal `garment_signature` values. Color is intentionally excluded from that signature.
- Every schema `1.1` or later apparel reference must carry the complete version-appropriate `must_transfer` and `must_not_transfer` sets from the required batch object. New schema `1.4` requires the expanded human-cue exclusion set and zero-human-pixel attestation. The fixed PopBoom model remains a separate creator-identity control in position 1 / `@Image1` and never becomes apparel reference material.

No orphan shot, reordered beat, paraphrased spoken line, or prompt-only action is allowed. When any source field changes, edit the canonical timeline first, increment `timeline_version`, invalidate all old projections, and recompile all three.

### Strict Legacy v5 Exact Resume

`legacy_v5_exact_resume` is an execution exception, not a new compile mode and not a migration shortcut. It may authorize only one package that was already schema `1.3` / `canonical_prompt_v5`, previously returned `valid: true` and `eligible_for_new_submission: true`, has never received a `record_id`, and is explicitly approved by the user for unchanged continuation. The PopBoom ledger validator must bind the original exact batch-file digest, compiled-text digest, all receipt hashes, reference bytes/order/URLs, fixed-model identity, first-frame binding, generation settings, and outbound request. Every equality flag must be derived, not asserted by prose. Any changed byte, missing prior validation receipt, repaired timeline, new reference, new setting, ambiguous acceptance, or existing `record_id` disables the exception and requires a fresh v1.4/v7 compile or poll/report-only handling. Never rewrite a locked streaming plan in place; create a new current parent for migrated unreleased colors.

## Pairwise Creative Differentiation Gate

Audit every color pair, not only each color against a baseline. Any pair must differ in at least three of these six dimensions:

1. `hook_angle_id`
2. `opening_visual_id`
3. `scene_id` or scene palette
4. `styling_signature`
5. `proof_focus_id`
6. `signature_action_id`

The three differences must include at least one from each group:

- attention: 1 or 2;
- visual world: 3 or 4;
- proof/action: 5 or 6.

Additional hard rules:

- `hook_text_original` must equal the timeline's non-empty hook-purpose spoken lines joined in beat order. A silent visual hook uses an empty string.
- Under `nfkc_casefold_ws_punct_v1`, apply Unicode NFKC, case-fold, remove every NFKC/case-folded term listed in all compared variants' `color_terms`, replace punctuation/symbol characters with spaces, then collapse whitespace. Equal normalized hooks are a color-only collision.
- At least two non-CTA beats must use different `spoken_intent_id` values.
- At least two non-CTA beats must differ in the combined `framing + core_action + product_point` signature.
- Changing only garment color, caption, hashtags, CTA gesture, or random generation noise does not count as video differentiation.
- Model, market, language, material lock, template, timeline grid, safety constraints, and CTA policy may remain shared.
- Never invent a new material, performance claim, or garment structure to force differentiation.
- Captions must remain semantically distinct after the same color-term normalization. Every color has exactly five hashtags; each starts with `#`, contains no whitespace, and is unique within that variant after NFKC/case-fold. For every pair, the normalized hashtag intersection may contain at most three tags, so each side contributes at least two unique tags. Caption and hashtag changes do not count toward the video gate.

If any pair fails, revise only the colliding creative delta, rerun the complete pairwise matrix, and generate timelines only after all pairs pass.

In quality-gated streaming mode, run this pairwise gate on the immutable planned creative deltas, captions, hashtags, intent IDs, and visual signatures before the first paid release. Every per-color release then proves that its final canonical timeline implements those exact signatures. Do not postpone differentiation until later colors are rendered.

## Executable Preflight Gate

Persist the compiled object beside the run ledger as `$CODEX_HOME/zibuyu-runs/<run_id>/batch-compile.json`, using the current user's `.codex` directory when `CODEX_HOME` is unset. Before any PopBoom upload or paid submission, run:

```text
python scripts/validate_batch_compile.py <absolute-path-to-batch-compile.json> --compile --output <absolute-path-to-batch-compile.json>
python scripts/validate_batch_compile.py <absolute-path-to-batch-compile.json>
```

For a quality-gated streaming parent, first run:

```text
python scripts/validate_streaming_plan.py <absolute-path-to-batch-plan.json>
```

Then run the two normal compiler commands against the current color's release file. That release must contain an absolute `streaming_release.plan_path`, the exact returned `streaming_plan_sha256`, and the matching `variant_id`. The normal compiler loads the plan and blocks shared-core, SKU, source, color, creative-delta, caption, hashtag, non-CTA intent, visual-signature, or reference-QC drift.

Use an available Python 3 runtime; the script uses only the standard library. Treat its computed output as authoritative over any prefilled `validation` claims in the input. For any new or repaired submission, continue only when the process exits `0`, returns both `valid: true` and `eligible_for_new_submission: true`, and has no errors. A historical `1.0`/`1.1`/`1.2` object may still return `valid: true` for read/poll compatibility while returning `eligible_for_new_submission: false`; that is never paid-call authorization. On failure, persist the returned `primary_error` and full `errors` array, repair the canonical source object, regenerate all three projections, and rerun the gate. If the bundled script or Python runtime is unavailable, stop before paid submission rather than silently skip validation.

For a persisted file, the CLI result also returns its exact-byte `batch_compile_sha256` and per-variant `director_receipts`. Each new receipt binds `variant_id`, timeline ID/version, that file hash, schema/quality/market-contract/serializer IDs, market, locale, profile ID, the quality-plan/research-bundle/canonical-beat/compiled-text hashes, all three market/profile/control/review hashes, and the two boolean verdicts. It also proves that the fixed-model/garment role map and zero-human-pixel reference gate passed. In streaming mode, the exact release file contains the immutable plan path/hash binding, so the receipt transitively freezes that global quality plan. Copy the matching receipt unchanged into the same variant's PopBoom `zibuyu_apparel` job and let `validate_ledger.py` bind it into the paid request fingerprint; never reuse it across colors or markets.

For schema `1.4`, the validator additionally enforces Amazon/source identity, review IDs and sampling language, buyer-image limits, conflicts, pain-to-solution eligibility, exact claim/proof/beat/caption evidence propagation, research hash, directing/fidelity plan and hash, the exact market/model/language/profile tuple, market-specific delivery/camera/music/verdict/CTA/screen-text controls, exact voiceover-review projection, language contamination gates, three or four contiguous non-reused setup runs, the three-phase performance arc, voiceover density, distinct visible expression cues, repeated-hip-reset rejection, camera-to-part-to-proof-endpoint alignment, sequential demonstrations, hand/posture plans, performance authorization, CTA purity, identity-first reference ordering, zero-human-pixel apparel references, meta-leak rejection, and deterministic v6 serialization. Never downgrade an object or label a changed v5 package as an exact resume to bypass these errors.

## Validation And Status

Block PopBoom submission when any of these conditions exists:

- `BATCH_NOT_SAME_SKU`, `MISSING_VARIANT_REFERENCE`, or `SUBMISSION_FIELD_MISSING`;
- `DIFF_COLOR_ONLY`, `DIFF_THRESHOLD_NOT_MET`, or `DIFF_PAIR_COLLISION`;
- `TIMELINE_GAP`, `TIMELINE_OVERLAP`, or `DURATION_MISMATCH`;
- `TIMELINE_VO_DRIFT`, `TIMELINE_ACTION_DRIFT`, `TIMELINE_EXTRA_BEAT`, `TIMELINE_MISSING_BEAT`, or `TIMELINE_ORDER_DRIFT`;
- `STALE_TIMELINE_VERSION`, `VARIANT_REFERENCE_MISMATCH`, or `PROMPT_REUSED_ACROSS_VARIANTS`;
- `SCHEMA_VERSION_INVALID`, `QUALITY_CONTRACT_INVALID`, `REFERENCE_TAG_INVALID`, or `REFERENCE_TRANSFER_CONTRACT_INVALID`;
- `QUALITY_PLAN_MISSING`, `DIRECTING_INTENT_INVALID`, `DIRECTORIAL_VOICE_INVALID`, `FIDELITY_ALLOCATION_INVALID`, `DIRECTING_INTENT_DRIFT`, or `QUALITY_PLAN_HASH_MISMATCH`;
- `SHOT_DENSITY_EXCEEDED`, `SHOT_ACTION_DENSITY_EXCEEDED`, `BEAT_QUALITY_FIELD_MISSING`, `BEAT_ENUM_INVALID`, `SPEECH_MODE_INVALID`, `MOTION_BUDGET_INVALID`, `PERFORMANCE_ARC_INVALID`, `VOICEOVER_PACING_INVALID`, `FLAT_PERFORMANCE_RISK`, `LIPSYNC_COMPLEXITY_OVERLOAD`, `DETAIL_SHOT_COMPLEXITY_OVERLOAD`, or `CTA_COMPLEXITY_OVERLOAD`;
- `CLAIMS_REGISTRY_INVALID`, `PROOF_PLAN_MISSING`, `CLAIM_EVIDENCE_INVALID`, `UNSUPPORTED_PERFORMANCE_DEMO`, `CLAIM_PROOF_FRAMING_MISMATCH`, `CLAIM_PROOF_ACTION_MISMATCH`, `CLAIM_PROOF_ENDPOINT_MISMATCH`, `STATIC_SELLING_BEAT`, `ACTION_COVERAGE_INSUFFICIENT`, `HAND_PLAN_INVALID`, `HAND_ACTIVITY_MISMATCH`, `ANATOMY_RISK_OVERLOAD`, `STIFF_POSTURE_RISK`, `ACTION_SEQUENCE_OVERLOAD`, or `ACTION_TRANSITION_INVALID`;
- `RESEARCH_BUNDLE_INVALID`, `REVIEW_EVIDENCE_INVALID`, `BUYER_PAIN_MAP_INVALID`, `PAIN_PROOF_BINDING_INVALID`, or `CONTRAST_HOOK_PROOF_INVALID`;
- `PROMPT_QUALITY_BLOCK_MISSING`, `CANONICAL_BEATS_HASH_MISMATCH`, `PROMPT_V2_FULL_JSON_LEAK`, or `PROMPT_SERIALIZER_MISMATCH`.
- `MARKET_PROFILE_INVALID`, `MARKET_LANGUAGE_MISMATCH`, `MODEL_PRESET_MARKET_MISMATCH`, `LANGUAGE_GATE_FAILED`, or `VOICEOVER_REVIEW_DRIFT`;
- `DELIVERY_MODE_CONFLICT`, `CAMERA_MODE_CONFLICT`, `MUSIC_MODE_CONFLICT`, `CUT_BLOCK_DENSITY_INVALID`, or `CUT_BLOCK_SEQUENCE_INVALID`;
- `CTA_POLICY_INVALID`, `CTA_POSITION_INVALID`, `SCREEN_TEXT_POLICY_INVALID`, `PROMPT_META_LEAK`, or any market/profile/control/review hash mismatch.

The `errors` array may retain every applicable code, but automation must choose one deterministic `primary_error`. For differentiation failures use this precedence: `DIFF_COLOR_ONLY` first, then `DIFF_THRESHOLD_NOT_MET`, then `DIFF_PAIR_COLLISION`. `DIFF_PAIR_COLLISION` remains an aggregate/pair-location code and must not replace the more specific cause. Thus a color-only clone reports primary `DIFF_COLOR_ONLY` while retaining the threshold and colliding-pair details as secondary errors.

Track two different states:

- `planned_differentiation_passed`: pairwise plans and compiled artifacts passed before submission.
- `rendered_differentiation_verified`: the actual videos were inspected and each pair shows at least one non-color audible difference and one non-color visible difference in scene, styling, proof, or core action.

If finished videos cannot be inspected or transcribed, use `rendered_differentiation_unverified`; never claim the rendered videos are differentiated based only on their prompts.
