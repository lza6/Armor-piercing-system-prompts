# Three-Layer Non-Negotiable Contract

Use this contract for every new schema `1.4` apparel video compile. It turns the seller-winning prompt pattern into explicit machine gates instead of leaving critical constraints scattered through prose.

The current identifiers are:

- `quality_contract_id: zibuyu_ugc_quality_v4`
- `serializer_id: canonical_prompt_v7`
- `three_layer_deadlines.contract_id: zibuyu_three_layer_deadlines_v1`

`zibuyu_ugc_quality_v3` plus `canonical_prompt_v6` is historical and read-only. Do not use it for a rewrite, repair, new release, or new paid submission. The narrow byte-identical v5 resume rule remains unchanged.

## Core Rule

Every new prompt has three distinct non-negotiable layers, in this order:

1. global audiovisual shell;
2. subject, garment, scene, light, and sound invariants;
3. one physical and continuity deadline for every canonical beat.

Do not treat deadlines as only per-shot negative prompts. A shot can be anatomically valid and still fail because the camera grammar, creator body, garment construction, fabric physics, room, light direction, or live sound changed. Conversely, global realism prose does not replace explicit hand, garment-state, action-physics, and continuity rules for each beat.

The structured `three_layer_deadlines` object is authoritative. `canonical_prompt_v7` is its deterministic human-readable projection. Both the object hash and compiled prompt hash must pass before handoff or payment.

## Layer 1: Global Audiovisual Shell

This layer locks the production envelope for the complete clip:

- platform, aspect ratio, exact duration, resolution, and frame rate;
- one coherent phone-camera ownership and movement grammar;
- lens perspective, depth-of-field behavior, framing limits, and focus behavior;
- native UGC capture character, exposure behavior, sensor noise, motion blur, white-balance behavior, and dynamic-range limits;
- post-processing prohibitions, including beauty filters, skin smoothing, body reshaping, cinematic grading, artificial sharpening, and synthetic background blur;
- direct-phone audio behavior, room reflections, environmental sound, garment/prop sounds, and music policy;
- spoken language and screen-text policy;
- an explicit overlay exclusion set.

Required fields:

```yaml
global_deadlines:
  platform: string
  aspect_ratio: string
  duration_seconds: number
  resolution: string
  frame_rate_fps: integer
  camera_mode: fixed_phone | creator_handheld | friend_handheld
  screen_text_policy: no_generated_text | fixed_model_stats_only
  allowed_screen_text: [] | [exact single fit-stats line]
  music_mode: none | low_non_lyrical
  spoken_language: en-US | de-DE
  capture_quality_lock: concrete natural-language lock
  camera_behavior_lock: concrete natural-language lock
  lens_depth_lock: concrete natural-language lock
  postprocessing_lock: concrete natural-language lock
  audio_capture_lock: concrete natural-language lock
  forbidden_overlays:
    - subtitles
    - translation
    - price
    - product_name
    - link
    - shopping_cart
    - platform_ui
    - sticker
    - logo
    - watermark
```

The numeric and enum fields must exactly project the batch key and `generation_controls`; they are not free prose. `allowed_screen_text` must agree with `screen_text_policy`. A prompt that says both fixed and handheld, both music and no music, or both full depth and portrait blur fails before compilation.

## Layer 2: Asset Invariants

This layer locks every persistent thing that must survive all cuts:

- creator identity, age presentation, natural body proportions, skin texture, hair, makeup, and prohibited body reshaping;
- garment identity, exact color, category, silhouette, neckline, sleeve construction, length, closure, seams/panels, texture scale, thickness, opacity, and drape;
- garment-structure constraints that must remain true from front, side, and back;
- fabric physics under gravity, touch, pull, release, walking, and turning;
- complete outfit, layers, accessories, props, and prop ownership;
- declared scene, environmental objects, use traces, spatial continuity, and excluded substitute locations;
- light source direction, color temperature, shadow direction, allowed exposure response, and forbidden relighting;
- room tone and source-appropriate live sound.

Required fields:

```yaml
asset_deadlines:
  creator_identity: string
  subject_appearance_lock: concrete natural-language lock
  garment_identity: string
  color_name: exact variant color
  garment_signature: complete structured signature
  garment_structure_lock: concrete natural-language lock
  fabric_physics_lock: concrete natural-language lock
  outfit: exact complete outfit
  outfit_prop_lock: concrete natural-language lock
  scene_strategy: exact validated occasion-first scene strategy
  scene_environment_lock: concrete natural-language lock
  lighting: exact continuity anchor
  lighting_lock: concrete natural-language lock
  soundscape_lock: concrete natural-language lock
  forbidden_asset_drift:
    - creator_identity
    - body_shape
    - garment_color
    - garment_structure
    - garment_texture
    - fabric_physics
    - undeclared_scene
    - lighting_direction
    - soundscape
    - outfit_props
```

The color, garment signature, outfit, scene strategy, and lighting anchor must exactly project the selected variant and quality plan. Same-SKU color variants may change only the declared color-dependent values and planned creative delta; creator, body, garment construction, and fabric-physics locks remain identical.

## Layer 3: Per-Shot Physical And Continuity Deadlines

Create exactly one shot deadline for every canonical beat. It inherits layers 1 and 2 and adds what can fail during that action:

- exact beat timing, camera setup, scene, camera motion, local light, and local audio;
- one core action, one physical target, and one readable endpoint;
- complete left/right hand plan and ownership of phones or props;
- action physics, including contact, pull distance, release, gravity response, dressing order, walking response, or closure behavior;
- anatomy constraints, including correct limb count, left/right correspondence, joint range, fingers, and no intersections;
- garment state before, during, and after the action;
- continuity handoff from the prior endpoint into the next beat;
- explicit forbidden failure outcomes.

Required fields:

```yaml
shot_deadlines:
  - beat_id: exact canonical beat ID
    start_seconds: exact beat start
    end_seconds: exact beat end
    camera_setup_id: exact setup run
    scene_id: exact scene
    camera_motion: exact beat camera behavior
    lighting: exact beat light
    audio: exact beat sound
    core_action: exact canonical action
    action_target: exact garment part or styling target
    readable_endpoint: exact proof endpoint
    hand_plan: exact canonical left/right hand plan
    action_physics_lock: concrete natural-language lock
    anatomy_lock: concrete natural-language lock
    garment_state_lock: concrete natural-language lock
    continuity_lock: concrete natural-language lock
    forbidden_outcomes:
      - extra_limbs_or_fingers
      - body_or_garment_intersection
      - garment_structure_change
      - unmotivated_fabric_motion
      - continuity_break
```

Every projected field must exactly equal the canonical beat. A generic repeated sentence is not enough: the four lock strings must describe the specific action, target, garment state, and handoff of that beat. The endpoint must be physically readable before the next action begins.

## Compilation Order

`canonical_prompt_v7` compiles in this order:

1. fixed-model identity and garment-reference role locks;
2. `GLOBAL NON-NEGOTIABLES`;
3. `SUBJECT, GARMENT, SCENE, LIGHT, AND SOUND NON-NEGOTIABLES`;
4. selected market shell, occasion-first scene, and connected creator performance arc;
5. every timestamped beat followed by its own `Shot non-negotiables` and forbidden outcomes.

The serializer may remove duplicate wording, but it may not omit, weaken, contradict, or relocate the authority of any layer.

## Hash And Release Gate

For current work:

- compute `three_layer_deadlines_sha256` from canonical UTF-8 JSON of the full object;
- bind `deadline_contract_id` and `three_layer_deadlines_sha256` into the director receipt;
- bind the complete receipt into the paid request fingerprint;
- for streaming, copy the immutable global and asset layers into each planned color's `deadline_blueprint` and lock the shot field policy before the first release;
- reject a release when its global or asset layer differs from the plan, when a beat lacks a complete shot layer, or when mandatory forbidden outcomes are absent;
- reject PopBoom authorization when the persisted batch compile, receipt, deadline hash, prompt text, or outbound request differs.

Never reconstruct a deadline object from the final prompt at payment time. Validate the persisted structured object, deterministically recompile it, and compare the resulting hashes.

## Authoring Checklist

Before writing shots:

1. Lock global format, camera, UGC image behavior, post-processing exclusions, text policy, language, and audio capture.
2. Lock creator/body, garment/color/construction, fabric physics, outfit/props, scene, light direction, and soundscape.
3. Build the canonical timeline.
4. For each beat, write action physics, anatomy, garment state, continuity handoff, and forbidden outcomes specific to that beat.
5. Compile and validate. Repair the structured layer that failed; do not patch only the outbound prose.

