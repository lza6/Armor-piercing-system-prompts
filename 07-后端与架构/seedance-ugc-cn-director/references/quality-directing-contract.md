# Seedance UGC Quality Directing Contract

Use this contract with `sku-batch-timeline-contract.md`, `market-prompt-compiler-contract.md`, the selected US or DE market profile, `motion-realism-and-commerce-cadence.md`, `selling-point-action-proof-methodology.md`, and `amazon-product-review-evidence-contract.md` for every new apparel plan before writing hooks, timelines, or a Seedance/PopBoom prompt. It converts product/review provenance, market execution, and directing judgment into compact, testable constraints that protect garment fidelity, human stability, commerce-paced motion, lip-sync, and visible product proof.

For a multi-color release submitted before other references finish, also apply `streaming-quality-contract.md`. Streaming changes scheduling only; every quality requirement below still applies to each complete release.

This contract conceptually adapts the intent-first directing, fidelity-allocation, reference-role, event-density, continuity, and one-variable-retake ideas from the MIT-licensed [Emily2040/seedance-2.0](https://github.com/Emily2040/seedance-2.0) project for Zibuyu's 15-second apparel UGC workflow. The wording and machine contract here are specific to this plugin.

## Contents

1. Set one commercial directing intent
2. Bind every reference by role
3. Allocate fidelity before adding shots
4. Sequence useful actions without anatomy overload
5. Bind every spoken claim to camera and proof action
6. Make every beat directed and causal
7. Protect continuity across cuts
8. Compile in priority order
9. Run the schema and pre-submission gate
10. Review takes with one primary variable

## 1. Set One Commercial Directing Intent

Give every color variant one short `directing_intent`: the single buyer response the video should create. Write it as a visible commercial turn, not an aesthetic adjective.

Also define one continuous creator social intention: what the creator wants the viewer/friend to recognize, believe after proof, and feel certain enough to do. Direct a default 15-second creator-led clip as three viewer-perceived macro phases—recognition/result Hook, animated proof, and close/detail conviction—while retaining the internal proof-beat count and evidence obligations below. This is a performance layer, not permission to reduce the video to three generic actions.

Good examples:

- make the relaxed drape feel trustworthy and easy to style;
- turn sleeve-coverage concern into confidence through one clear proof shot;
- make this color feel like the easiest weekend outfit choice.

Reject `cinematic`, `premium`, `beautiful`, `viral`, or `high-end` as standalone intentions. They do not decide a shot.

Choose one functional `directorial_voice` for the variant: normally `observational_naturalist`, with `intimate_minimalist` or `graphic_formalist` only when product evidence and the selected format justify it. Mirror try-on, friend reaction, and detail proof remain template/format choices rather than directorial voices. Camera, lighting, performance, sound, and cut rhythm must all serve the same intent and voice. Remove any decorative technique that cannot answer “why does this help prove the garment or move the buyer?”

Select the market before the directorial voice or creative form. Persist `market_prompt_contract_id: zibuyu_market_prompt_v1`, `market_prompt_profile_id`, and all seven `generation_controls` fields from `market-prompt-compiler-contract.md`. Authority is identity/evidence/safety first, market compiler second, German account-interaction overlay third, base template fourth, and style preference last. A template may not override the selected delivery, camera, music, verdict, commerce CTA, or screen-text mode.

## 2. Bind Every Reference By Role

For every new schema `1.4` compile, reserve the first submitted reference position and `@Image1` for the selected fixed PopBoom model. The prompt must begin with the deterministic mapping `Reference 1 / @Image1 = fixed-model identity only`. Preserve that creator's exact face, skin tone, ethnicity, hair, age presentation, and body identity; never use it as garment evidence.

Store every apparel-reference interface tag in `interface_tag` and preserve it exactly. New `canonical_prompt_v7` garment tokens use ASCII `@ImageN` with numeric index 2 or higher; the first garment mapping is `Reference 2 / @Image2 = garment identity only`. Historical serializers may retain already-recorded controlled tags for read-only audit. Do not renumber, translate, normalize, silently swap, create gaps, or compile new-work garment `@Image1`, non-ASCII tag families, or mojibake tags. `reference_contract_text` and the compiled prompt must contain each exact validated tag.

For the apparel three-view reference, require this role contract:

- primary role: `apparel_three_view` / garment identity only;
- `human_identity_pixels_absent: true`: the accepted bitmap contains no real-human identity pixels or cues;
- `must_transfer`: garment identity, exact color, silhouette, neckline, sleeve construction, hem, seams, texture scale, thickness, opacity, and drape;
- `must_not_transfer`: white background, three-panel/triptych layout, panel dividers, repeated bodies, multiple models, reference skin tone, ethnicity, neck, chest, collarbone, shoulders, arms, hands, fingers, nails, tattoos, jewelry, body shape, face or hair, reference pose, camera, environment, embedded text, and watermarks;
- `positive_replacement: one_creator_same_garment_single_real_scene`: only the `@Image1` fixed creator, wearing the same garment in one continuous believable real-world scene.

The fixed PopBoom custom model controls creator identity only. It must not replace the garment, color, material, scene, camera, or product reference. The three-view image must never control creator identity. Any visible real skin, anatomy, jewelry, tattoo, manicure, or person-specific body cue in a three-view fails the upstream zero-human-identity-pixel audit and blocks compilation; text exclusions or prompt wording may not be used to waive that failure.

Text controls what references cannot carry reliably: action over time, camera behavior, timing, motivated light, spoken lines, sound, scene transition, and constraints. Do not re-describe dense visual details already locked by the reference unless naming the exact detail being proved.

Every compiled prompt must include one concise `reference_contract_text` that states the transfer, non-transfer, and positive replacement. Use positive replacement before the compact negative constraint slot so exclusions do not become the main visual instruction.

## 3. Allocate Fidelity Before Adding Shots

For apparel ecommerce, set `primary_fidelity_spend: garment_identity`. Choose exactly one secondary spend:

- `visible_proof` for neckline, sleeve, hem, texture, construction, or drape;
- `lip_sync` when the visible creator's spoken delivery is the retention engine;
- `natural_motion` for one controlled turn, step, fabric release, or styling action;
- `scene_readability` when occasion and outfit context drive conversion.

List what is deliberately economized in `economized_elements`. Usually economize crowds, extra people, busy props, rapid camera moves, complex choreography, multiple simultaneous hand actions, small on-screen text, and decorative scene changes.

The single secondary fidelity spend sets rendering priority; it does not remove the baseline claim-proof actions required to sell the garment. Even when the secondary spend is lip-sync or scene readability, every spoken product claim still needs its matched frame and correctly performed demonstration.

If a beat asks for perfect garment detail, bold body motion, active camera motion, and visible lip-sync at once, split or simplify it. Product identity and the selected secondary spend win; everything else yields.

## 4. Sequence Useful Actions Without Anatomy Overload

For a default 15-second video:

- use three to four contiguous camera-setup runs, with the exact count driven by the garment claims that need distinct framing;
- require at least four sequential non-CTA product/styling demonstration actions in a default 15-second clip, including at least three garment-part/effect proofs; a prop move, facial reaction, camera move, or CTA gesture does not count toward this action floor;
- allow several action-bearing beats in one setup when the actions occur sequentially, each action finishes at a visible endpoint, and the hands hand that endpoint naturally into the next start state;
- group the viewer-facing performance into three macro phases—roughly 0-4s recognition/result Hook, 4-10s animated proof, and 10-15s close/detail conviction—while preserving five to seven internal beats and the full action floor;
- change or reframe the setup whenever the spoken claim moves to another garment part that the current frame cannot show clearly;
- allow one main physical action and one visible endpoint per beat, then hold that endpoint only long enough to read the proof; do not spend an explanatory beat standing still;
- treat the action cap as a simultaneity rule, not a clip-wide scarcity rule: never combine unrelated gestures at once, but do not remove useful demonstrations;
- keep every beat at least 1.5 seconds; give visible lip-sync at least 2 seconds;
- keep the garment large enough in frame for the detail being discussed;
- keep generated text limited to the required persistent upper-left fit-stats overlay when fixed-model stats are provided; move all badges, price cards, subtitles, and any other text to post-production.
- preserve task and object continuity across internal beats. A mirror-selfie filming hand remains on the phone; a held alternate color, garment, bag, or accessory remains owned until an explicit place-down, handoff, or intentional cut. The previous endpoint normally becomes the next start state; never force both hands back to symmetric hip anchors or the body back to attention posture between proofs.
- motivate distance changes by viewer need: step back for full fit, move or turn for fabric/silhouette behavior, and walk closer or reframe for a decisive detail. Use exact centimeters/degrees only when the product evidence truly requires them.

Use the `motion_budget` object:

```yaml
motion_budget:
  camera: locked | subtle | active
  performer: still | micro | simple | active
  active_hands: zero | one | two
```

Hard stability rules:

- visible lip-sync: camera is `locked` or `subtle`, performer is not `active`, and at most one hand is active;
- ordinary detail proof: camera is `locked` or `subtle`, performer is not `active`, and one hand performs the demonstration while the other hand has an explicit stable anchor;
- verified two-hand proof: allow exactly two active hands only for one symmetric, evidence-backed action such as a controlled stretch pull-release or width spread-release; keep the mouth out of frame, torso still, camera locked/subtle, both wrists attached and visible, and complete one action cycle before the cut;
- profile close: camera is `locked` or `subtle`, performer is `still` or `micro`; a verdict uses no sales graphic, and only `commerce_cta_mode: light_link` or an authorized evidence-backed promo permits at most one hand to make the down-left gesture;
- never combine active camera + active performer in an apparel proof beat.

Apply the complete action grammar in `motion-realism-and-commerce-cadence.md` and the target-specific action selector in `selling-point-action-proof-methodology.md`. Every beat starts from an explicit posture and left/right hand anchors, performs one action through a motivated trajectory, and reaches one visible endpoint. Carry that endpoint naturally into the next start state; reset or cut only when clarity requires it and declare any prop transfer/disappearance. In a default 15-second clip, deliver a newly completed product proof or styling result about every two to three seconds. Use natural acceleration/deceleration, shoulder-elbow-wrist follow-through, plausible weight transfer, and garment lag/settling. Blinks, breathing, smiles, nods, and camera motion improve realism but never satisfy the product-action floor.

## 5. Bind Every Spoken Claim To Camera And Proof Action

For schema `1.4`, create the research bundle, pain-solution map, selected market profile, `generation_controls`, and `quality_plan.claim_proof_plan` before the canonical timeline. Give every garment claim one action-proof entry, selected and audited through `selling-point-action-proof-methodology.md`, containing:

- `claim_proof_id` and intended `spoken_intent_id`;
- `proof_target`: the exact garment part or behavior being discussed;
- `evidence_basis`: `visible_reference`, `user_provided`, or `product_page`;
- `evidence_ids`: the exact source-backed item IDs copied unchanged from claim to proof to beat;
- `framing_class`: `detail_closeup`, `chest_to_hem`, `side_back`, `full_fit`, or `styling`;
- `action_type`: one physical demonstration such as `point_trace`, `touch_release`, `pinch_release`, `pull_release`, `raise_arm`, `smooth_release`, `turn_settle`, `open_close`, `pocket_use`, `front_tuck`, or `style_adjust`;
- `hands_required`: `one`, `two`, or `body`;
- `expected_visible_change`, matching `proof_endpoint`, and the target `beat_id`.
- `spoken_line_shape`, explicit left/right hand anchors, and sparse `micro_cues` when the face/body is visible.
- planning-only `human_motivation`, `persistent_task_anchor`, and `handoff_from_prior` annotations that compile into visible action/hand/camera text without being pasted as internal rationale.

Bind every `pain`, `proof`, and `styling` beat to exactly one plan entry through `claim_proof_id`, set `spoken_claim_ids` to the one exact allowlisted `claim_id`, and carry the same canonical `product_part_id` and exact `evidence_ids` from registry to plan to beat. Non-proof beats use an empty claim array and null part ID; a hook instead binds one validated `pain_point_id` and its pain evidence. The narrow exception is `evidence_backed_contrast`: an eligible review-derived material-expectation hook may preview one negative texture contrast while still carrying no claim ID, but the immediately following beat must bind the mapped exact-variant `texture` claim, use a detail close-up, off-screen voiceover, and one `pinch_release`. No reaction, styling beat, or second claim may intervene. Each claim/proof names one part/effect only; aliases such as neckline/collar/V-neck remain one `neckline` group and cannot fake the three-group floor. The beat's framing must show the full named target, its positive `core_action` must perform the planned action with the same declared hand, and its `proof_endpoint` must equal the concrete product state recorded in `visible_endpoint`. When a line names the neckline, frame the neckline and touch/trace it; when it names a sleeve, frame the sleeve and raise/touch it; when it names drape, frame the hem or side and release/turn it; when it names a pocket or closure, operate that exact feature.

Do not accept generic standing, smiling, nodding, holding the final outfit, or resting both hands as product proof. Natural micro-expressions support performance but never substitute for the garment action.

Treat performance claims as evidence-gated. Demonstrate stretch/elasticity, breathability, cooling, water resistance, opacity, anti-wrinkle behavior, or similar performance only when the exact linked evidence item is non-conflicted, matches the product/variant scope, and declares `performance_demo_allowed: true`. Seller copy or a direct review may inform a qualified statement, but an AI summary, third-party summary, buyer image, mismatched child, or free-text `product_page` note never unlocks performance. Numeric performance requires a verified test. A catalog value of `Low Stretch` explicitly blocks a strong pull demonstration. One allowlisted performance category never authorizes another. Hooks and CTAs cannot mention or imply unbound performance. A two-hand pull on unsupported fabric is not proof; replace it with an allowed visual action such as texture close-up, pinch-release for drape, sleeve raise, hem release, or side turn.

### Buyer Evidence Is Not Product Fact

- Amazon catalog attributes and readable garment labels may support objective composition, fit, weight, length, stretch classification, or care for the locked variant.
- Seller bullets remain seller claims. They cannot create a buyer pain or the phrase “buyers say”.
- Direct reviews support buyer experience and pain, not automatic objective truth. One review permits a single attribution; a sample theme requires at least three unique review IDs and sample-scoped wording.
- Customer images support only what is visibly inspected: color, fit, length, and construction. They cannot prove softness, comfort, composition, stretch, breathability, care, opacity performance, or durability.
- Amazon AI/review summaries, Q&A without authority, search snippets, and third-party summaries are discovery-only.
- A review-derived complaint becomes a conversion hook only when `pain_solution_map` marks it `script_eligible` and maps it to an independently evidenced allowlisted solution claim. Otherwise keep it `risk_only`.
- Prefer an `evidence_backed_contrast` negative Hook over a material-expectation question only when the exact next beat proves the mapped visible texture. The Hook may contrast an ordinary smooth-T-shirt expectation with the visible fine-knit surface, but it may not claim softness, comfort, breathability, stretch, or composition.
- Conflicting care fields, mixed size feedback, and mixed material/stretch feedback remain conflicts. Remove absolute copy instead of selecting the favorable side.

## 6. Make Every Beat Directed And Causal

Every canonical beat in schema `1.4` retains the existing quality fields and additionally requires:

- `camera_setup_id`: identifies the actual setup so several business beats can share one shot;
- `camera_motivation`: why this framing/move helps the buyer see the intended proof;
- `light_motivation`: the believable source and why it reveals color, texture, drape, or face naturally;
- `visible_endpoint`: the completed state the beat must reach before the next beat;
- `lip_sync_required`: true only when a visible mouth speaks the canonical line;
- `speech_mode`: `on_camera_dialogue`, `offscreen_voiceover`, or `none`;
- `spoken_language`: the exact selected locale for speech or `null` for silence;
- `proof_endpoint`: the concrete readable product state for a proof beat or `null` for a non-proof beat;
- `mouth_visibility`: `visible`, `not_visible`, or `partial`;
- `motion_budget`: the three-axis load described above.

Write actions as cause → visible consequence → endpoint. Prefer “she releases the hem; the fabric settles into one clean fold” over a list of unrelated gestures. Camera movement must finish on the proof, not continue after it. Lighting must come from a real window, lamp, mirror light, shaded sky, or other visible/credible source.

Name one specific sound cue per setup when useful—fabric rustle, footstep, room tone, bag strap movement, or a short spoken line. Apply exactly one global `music_mode`: `none` or `low_non_lyrical`. Natural environmental sound is allowed under either mode, but the two music modes may not coexist and music may not compete with product proof or visible lip-sync.

Treat each directed action as inherited start state -> human motive -> cause -> visible consequence -> endpoint -> motivated continuation/cut. Keep the existing causal proof wording, but make the declared posture and hand anchors the executable beginning and the prior endpoint, task anchor, or explicit cut the executable handoff.

Speech-mode rules:

- `on_camera_dialogue` requires a non-empty canonical line, `mouth_visibility: visible`, `lip_sync_required: true`, a locked/subtle camera, at most one active demonstration hand, and an explicit line-matched `micro_expression`. By default, keep each whitespace-delimited English/German visible line at 10 words or fewer and the 15-second visible-lip total at 20 words or fewer. When `secondary_fidelity_spend: lip_sync`, allow up to three visible-dialogue anchors—one per macro phase—with at most 14 words per line and 36 visible words total; the camera remains locked/subtle, performer load remains `simple` or lower, and complex movement/detail proof remains off-screen;
- `offscreen_voiceover` requires `mouth_visibility: not_visible` and `lip_sync_required: false`; use it for close-up proof or controlled movement so the model does not solve face phonemes and product motion together;
- `none` requires no spoken line and no lip-sync.
- `us_hybrid_share` permits concise visible American-English Hook/close anchors and requires complex middle proof to use off-screen voiceover;
- `de_live_simple` is valid only when every visible German line passes the locked/subtle camera, simple-or-lower performer, at-most-one-active-hand, lip-sync, expression, word, and timing gates;
- `de_hybrid_proof` is the German default: Hook/close may be live German while the same creator's middle German proof is off-screen with the mouth out of frame.

Every spoken beat must carry `spoken_language: en-US` or `de-DE` matching model, market, and profile. Keep one beat-by-beat `voiceover_review` with the exact market line plus Chinese review translation; Chinese is review-only and never enters the generation prompt, audio, screen text, or market caption. German spoken slots reject Chinese and high-confidence English template residue outside an explicit approved-name/token allowlist. German `hier`, `so`, and `genau da` must resolve to the exact framed proof target.

### Spoken-Language And Detail-Density Gate

Write voiceover for natural speech, not as seller copy read aloud. Use short, single-purpose sentences and idiomatic everyday wording. Reject official transitions and brochure constructions such as `this garment features`, `this product offers`, `furthermore`, `moreover`, `in addition`, German `dieses Kleidungsstück verfügt über`, `darüber hinaus`, `des Weiteren`, or Chinese `本产品采用`, `该服装具备`, `此外`, `综上`. Also reject standalone generic praise such as `looks nice`, `looks good`, `easy to style`, German `sieht gut aus` / `leicht zu kombinieren`, or Chinese `很好看` / `很百搭`. Do not reuse the same opener in three or more non-CTA proof/styling lines.

Write all units as one connected friend-to-friend recommendation rather than isolated catalog sentences. For a 15-second American-English creator-led mirror share, use roughly 50-60 total words and about 200-235 WPM as an aspirational read/TTS target only when every word remains intelligible; the hard validator uses a broader safe total. Mark emphasis and short proof-pivot pauses. Direct the voice and visible face through warm recognition, brighter animated certainty, then delighted conviction or relief. Require at least two meaningfully distinct visible expression cues across a clip with multiple on-camera lines. Energy comes from changing stakes and emphasis, not uniform loudness, a fixed grin, or frantic speed.

Richer detail does not mean denser sentences or equal coverage of every feature. For a default 15-second clip, first select one `core_sellable_wearing_result`: the buyer-relevant wearing outcome that is most likely to drive purchase and can be visibly proven. Then prove that result through several angles: visible result, structural reason, alternate angle or buyer-worry proof, motion/handling endpoint, and one concrete outfit relationship or wearing occasion. Each proof line must name one exact target and one visible behavior, position, construction fact, or pairing relationship. Keep one claim per beat and let the action/endpoint carry the proof. When a supported product detail does not help prove the selected result, remove it instead of adding feature coverage.

Good German progression: `Schau mal, wie weit der Ausschnitt sitzt.` -> `Der Ärmel endet locker am Oberarm.` -> `Beim Drehen öffnet sich der Seitenschlitz.` -> `Mit High-Waist-Jeans bleibt der Saum sichtbar.` Treat these as specificity examples, not reusable copy.

## 7. Protect Continuity Across Cuts

Store non-negotiable `continuity_anchors` for creator identity, garment identity, outfit, scene, and lighting, then project them exactly into the asset deadline layer. A deliberate new setup may change framing or pose, but it must not silently change the garment, material, color, model, accessories, location logic, light direction, soundscape, or product ownership.

Treat each `visible_endpoint` as the handoff into the next beat. If the next setup starts from a different pose or side, declare the cut intentionally in the camera/action language rather than implying impossible continuous motion.

Carry gaze, weight, expression, filming/task hand, props, and garment state through every adjacent internal beat, including Hook-to-proof and proof-to-close transitions. In mirror selfie POV, the phone hand cannot silently switch, lower, disappear, or become the proof hand. If an alternate-color garment or accessory leaves the frame, show its place-down/handoff or declare the intentional cut.

For multi-color batches, preserve construction and material anchors across all colors while keeping the existing creative-differentiation gate. Never invent a material or product feature merely to make variants different.

## 8. Lock Three Deadline Layers

Before final prompt serialization, materialize the complete `zibuyu_three_layer_deadlines_v1` object described in `three-layer-deadline-contract.md`:

1. lock the global audiovisual shell, including format, frame rate, camera ownership/movement, UGC capture behavior, lens/depth, post-processing bans, screen text, language, and direct-phone sound;
2. lock subject/body, garment/color/construction, fabric physics, outfit/props, scene, light direction, and soundscape across every cut;
3. lock action physics, anatomy, garment state, continuity handoff, and forbidden failure outcomes for every canonical beat.

The three layers are cumulative. Never move a persistent body, garment, scene, light, or sound invariant into only one shot, and never assume a global realism paragraph supplies beat-specific hand or fabric physics.

## 9. Compile In Priority Order

Build the final prompt in this order:

1. deterministic identity-first role lock: `Reference 1 / @Image1` fixed-model identity, then `Reference 2 / @Image2+` garment identity with zero-human-pixel attestation;
2. the complete global deadline projection;
3. the complete subject, garment, scene, light, and sound asset projection;
4. selected market language/performance control; the German speaker/voice/lip-sync lock appears before the scene and shot prose;
5. exactly three viewer-facing macro phases containing all timestamped internal beats, with one action, `proof_endpoint`, explicit state/prop handoff, and one shot-deadline projection per beat;
6. market-native canonical speech/delivery plus the selected verdict and commerce close.

Put subject, garment, and action before look adjectives. Delete duplicated synonyms, generic quality boosters, contradictory camera moves, and re-descriptions that fight the reference image.

Use `canonical_prompt_v7` for every new schema `1.4` compile. Its first two deterministic role lines are `Reference 1 / @Image1 = fixed-model identity only` and `Reference 2 / @Image2 = garment identity only`; additional garment references continue from `@Image3`. Keep structured canonical beats, selected market profile, generation controls, three-layer deadlines, voiceover review, research bundle, pain map, and claim-proof plan in the machine object and hash them; do not paste raw prompts/corpus data, raw reviews, source URLs, Chinese review translations, internal IDs, or JSON into the generation prompt. For a default creator-led performance, organize the final generation text as three continuous natural-language macro-phase blocks while retaining every validated internal timestamp, `spoken_language`, canonical line, demonstration action, `proof_endpoint`, hand plan, and shot deadline. Each internal claim proof follows proof target → subject/action → proof endpoint → camera → light → speech/sound. Carry task/prop/garment state forward inside and across phases; several actions may share a setup only when sequential and completed, never simultaneous.

Keep `directing_intent`, `directorial_voice`, fidelity allocation, market profile, generation controls, camera/light motivations, continuity reasoning, and `voiceover_review` inside the structured plan. Hash the existing plan/evidence/beat projections and also require `market_prompt_profile_sha256`, `generation_controls_sha256`, and `voiceover_review_sha256`. Do not feed abstract intent, internal IDs, budget labels, hashes, rationale prose, or Chinese review translations to Seedance. The generation text should contain only their visible consequences: subject, garment role lock, action, endpoint, camera, physical light, canonical market-language speech, sound, verdict/CTA behavior, and compact constraints.

The prompt rendering must carry `reference_contract_text`, and that string must appear verbatim in `compiled_text`. Keep it concise. Use only compact hard constraints plus constraints tied to an observed failure; do not dump every repair negative into first-generation prompts.

## 9. Schema And Pre-Submission Gate

Use `schema_version: "1.4"`, `quality_contract_id: "zibuyu_ugc_quality_v4"`, `market_prompt_contract_id: "zibuyu_market_prompt_v1"`, `three_layer_deadlines.contract_id: "zibuyu_three_layer_deadlines_v1"`, `research_bundle.contract_id: amazon_product_review_evidence_v1`, and `canonical_prompt_v7` for every new compile. Historical schema `1.4` v3/v6, schema `1.3` v2/v5, and older objects remain read/poll/download/review/report-compatible only. `legacy_v5_exact_resume` is allowed solely for a previously validator-approved unpaid package, after explicit user approval, when prompt bytes, ordered references, hashes, and request controls remain exactly unchanged. Any edit requires v7 migration. A job with a `record_id` is accepted work and is never migrated or resubmitted.

Before PopBoom submission, require:

- one valid quality plan per variant;
- one matching `us_champion_v1` or `de_champion_v1` profile plus complete, mutually consistent `generation_controls` selected before the base template;
- `camera_mode` is not contradicted inside a setup, `music_mode` is exactly `none` or `low_non_lyrical`, and `screen_text_policy` is exactly `fixed_model_stats_only`;
- the selected verdict/commerce close is market-native and evidence-safe; any CTA is performed only by the creator, while price/discount/urgency is absent unless independently eligible and explicitly authorized;
- a fixed-model identity reference in position 1 / `@Image1`, with every garment reference in position 2 or later / `@Image2+`;
- `human_identity_pixels_absent: true` for every apparel three-view, supported by the upstream visual audit rather than prompt wording alone;
- a claim-proof plan with the required product-action coverage and one-to-one beat bindings;
- the full apparel reference transfer/non-transfer contract;
- three to four contiguous camera-setup runs for a default 15-second clip, at least four sequential non-CTA product/styling demonstrations including three garment-part/effect proofs, and no actionless product explanation;
- native direct-response cadence with a completed product/styling result about every two to three seconds, full start-action-end hand grammar, natural ease-in/ease-out and weight transfer, readable endpoint holds, and no slow-motion fashion posing or idle explanatory beat;
- a coherent recognition -> animated proof -> close/detail conviction macro arc around the five to seven internal beats, with one continuous creator intention instead of disconnected equal-energy product poses;
- natural, mouth-written voiceover with no brochure/official constructions, standalone generic praise, or one opener repeated across three or more non-CTA proof/styling lines;
- American-English creator-led 15-second voiceover within the validator's safe density range, with connected phrasing, varied emphasis/pauses, line-matched expressions, and visible emotional progression; other languages use a language-appropriate audition rather than copied English word counts;
- Germany uses valid `de_live_simple` or default `de_hybrid_proof`; German spoken slots contain no Chinese or high-confidence English template residue, and `hier`, `so`, or `genau da` resolves to the visible target;
- one selected core sellable wearing result, proven through multiple front/detail/side-back/action/styling angles, while preserving the hard floor of three part/effect proofs and removing disconnected feature coverage;
- `spoken_language` for every spoken beat, and a visible endpoint plus matching `proof_endpoint` and motivated camera/light for every proof beat;
- exact left/right hand states, with a stable unused-hand anchor or a validated two-hand exception;
- persistent phone/task/prop ownership and endpoint-to-start continuity, with every place-down, handoff, reset, and cut explicitly motivated;
- selling-point action-proof rows pass the audit in `selling-point-action-proof-methodology.md`: spoken target, frame, action, hand path, endpoint, micro-cues, and evidence scope all match;
- evidence-backed handling of any performance demonstration;
- source/ASIN/variant-locked Amazon evidence, direct-review sampling, buyer-image limits, conflicts, pain-to-solution eligibility, and exact evidence-ID propagation;
- valid `three_layer_deadlines_sha256`, `quality_plan_sha256`, `research_bundle_sha256`, `market_prompt_profile_sha256`, `generation_controls_sha256`, and `voiceover_review_sha256` values plus verbatim inclusion of the concise `reference_contract_text` in generation text;
- `canonical_prompt_v7` compact generation text, deterministic identity/garment role mappings, explicit global/asset/shot non-negotiable projections, matching `canonical_beats_sha256`, profile-valid language/delivery/camera/music/verdict/CTA modes, and no raw corpus material, reviews, Chinese review translation, or full internal JSON in the text sent to PopBoom;
- `python scripts/validate_batch_compile.py <batch-compile.json>` exits `0` with both `valid: true` and `eligible_for_new_submission: true`.
- for quality-gated streaming, immutable `batch-plan.json` has already passed `validate_streaming_plan.py`, and the current release's `streaming_release` binding, three-view QC, planned intent/visual signatures, and exact plan hash pass inside the normal batch compiler.
- The validator result reports exact-file `batch_compile_sha256` and one per-variant `director_receipts` item carrying `variant_id`, timeline ID/version, schema/contract/serializer, and quality-plan/research-bundle/canonical-beat/compiled-text hashes. Bind that unchanged receipt into the matching `zibuyu_apparel` PopBoom ledger job; do not hand-copy, retype, or move a receipt across colors.

Never bypass a quality error to reach a paid generation call.

## 10. Review Takes With One Primary Variable

When a rendered take can be inspected, record one verdict:

- `keep`: the primary fidelity spend and full required claim-proof sequence, including the designated highest-priority proof, are delivered with no fatal error;
- `fix_in_post`: only trim, color, captions, sound mix, or a few edge frames need work;
- `reroll`: prompt is sound and the failure looks like sampling variance;
- `rewrite`: the same failure recurs or the prompt overloaded the model;
- `stop_or_rescope`: the requested proof cannot be made reliable inside the current shot budget.

Record `primary_failure_variable` and change only one of: one prompt clause, seed/sample, mode, or one reference binding. Do not change several variables in one retake. Two takes with the same failure require a rewrite or decomposition, not another blind reroll.

Never start a paid retake without explicit user authorization. If the rendered video cannot be inspected, report quality as `unverified` rather than inferring success from the prompt.
