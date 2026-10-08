# United States Winning Prompt Profile

Use this profile only when `market_prompt_profile_id: us_champion_v1`. Read `market-prompt-compiler-contract.md` first. This profile converts the shared evidence/action plan into a native American creator-style generation control specification; it is not a translation template and does not override evidence, identity, anatomy, or reference rules.

## Required Controls

```yaml
market_prompt_contract_id: zibuyu_market_prompt_v1
market_prompt_profile_id: us_champion_v1
generation_controls:
  prompt_shell_mode: us_technical_shell | us_compact_storyboard
  delivery_mode: us_hybrid_share
  camera_mode: fixed_phone | creator_handheld | friend_handheld
  music_mode: none | low_non_lyrical
  verdict_mode: soft_verdict | ownership_verdict | none
  commerce_cta_mode: none | light_link | evidence_backed_promo
  screen_text_policy: fixed_model_stats_only
  scene_strategy: required occasion-first object from occasion-scene-routing.md
```

Defaults for TikTok Shop apparel are `us_technical_shell`, `us_hybrid_share`, an occasion-first outward-use scene when the garment is outward wear, the least-loaded camera mode that fits the concept, `none` or truly low non-lyrical music, a soft evidence-backed verdict, and an optional brief `light_link` close. Never default to price, discount, scarcity, shipping, rating, or sales-volume language.

## Compiler Identity

Treat the US prompt as a generation control specification with this logic:

`native phone UGC shell -> creator/garment identity -> proof environment -> timestamped proof sequence -> American friend delivery -> compact continuity and negative constraints`

The immutable v6 reference-role lines still come first. Product truth and garment fidelity remain more important than scene novelty, energy, or slang.

## Prompt Shell Modes

### `us_technical_shell`

Use by default when generation reliability and product fidelity matter most.

- After the reference locks, declare vertical native phone UGC, selected camera grammar, real physical light, natural room treatment, one creator, and one continuous garment identity.
- Define the life scene as both a use-case answer and a proof environment. For outward wear, consider a café, beach/boardwalk, commercial street, sidewalk, travel setting, market/event arrival, office/commute edge, or entryway before any private interior. The location must make the occasion, outfit formula, fit, part, movement, or styling result readable.
- Serialize the three macro phases and every timestamped internal proof beat before adding voice and compact constraints.
- Keep constraint wording short and non-repetitive. One positive replacement plus the smallest useful prohibition set is stronger than multiple overlapping lists.

### `us_compact_storyboard`

Use when the concept is simple and the garment/reference lock is already strong.

- Keep the same identity-first v6 header and generation controls.
- Move quickly into three concise macro-phase blocks while preserving all required internal timestamps, proof actions, hand states, endpoints, speech modes, and continuity.
- Do not omit the evidence gate, product-first opening, audio mode, fit-stats-only screen policy, or compact stability constraints merely because the shell is shorter.

## US Performance Skeleton

- Hook: the garment is already worn and readable. The creator names one buyer hesitation or desired wearing result in natural American English, as if showing a useful find to a friend.
- Proof: each line names one exact target and the action shows that same target reaching a readable endpoint. Move closer, turn, step back, or reframe only because the viewer needs that proof.
- Close: give a sincere soft or ownership-style verdict when enabled, then optionally add one brief light-link CTA. Do not let the CTA replace the final garment result.

Use contractions and ordinary spoken phrasing when idiomatic. Avoid catalog language, exaggerated slang, generic hype, fake purchase/testing history, and repeated openers. The creator may sound warm, direct, amused, relieved, or pleasantly surprised, but never like a scripted announcer.

## US Occasion And Scene Adapter

Apply `occasion-scene-routing.md` before the performance skeleton:

- Use recognizable, non-branded lifestyle occasions as legitimate attention sources when they fit the garment: brunch café, beach or boardwalk, commercial street, weekend market, city sidewalk, travel hotel, casual event arrival, or an outdoor leisure stop.
- Make the Hook communicate both destination and outfit solution quickly. The scene may create curiosity, but the garment and wearing result remain the subject.
- Favor `destination_outfit_share`, `friend_filmed_walkthrough`, `grwm_departure`, or `travel_pack_and_wear` when those forms prove the use case better than mirror try-on. Keep the selected camera mode within the action/lip-sync budget.
- For outward wear, set the private-interior policy to `excluded` by default. Use an initial bedroom/closet/bathroom proof only in a declared mirror-to-destination route, or keep a private primary scene only for a specific fit/detail/multi-styling reason.
- Do not use a bedroom because it feels generically authentic. American UGC authenticity can come from plausible phone capture at the actual destination.
- Avoid crowded spectacle, trademarked storefront emphasis, staged influencer luxury, or scenery that steals garment readability.

## Speech And Lip-Sync

`delivery_mode: us_hybrid_share` means:

- Hook and close may use short `on_camera_dialogue` under the visible-lip budget;
- middle detail, drape, turn, handling, closure, and styling proofs use `offscreen_voiceover` with the mouth out of frame;
- every spoken beat uses `spoken_language: en-US`;
- visible words, lip-sync, expression, camera, performer, and active-hand load must pass the quality contract;
- the review projection repeats the exact English line and adds a Chinese review translation outside the prompt.

Do not make the creator visibly speak through a complex two-hand proof, active turn, active camera move, or fine detail close-up.

## Camera Selection

- `fixed_phone`: best for precise fit, live Hook/close, and high lip-sync or garment-fidelity demand.
- `creator_handheld`: use for mirror-selfie or phone-owned creator sharing; the filming hand remains on the phone through a continuous take.
- `friend_handheld`: use when a second person films but remains off camera; do not generate a second visible model.

Choose one global grammar. Within it, detail setups may lock or subtly reframe. Changing grammar requires an explicit cut and new setup run. Never request “fixed tripod” and “constant handheld movement” for the same setup.

## Audio

- `none`: dialogue, location-native ambience, fabric movement, footsteps, and other motivated natural sounds only.
- `low_non_lyrical`: quiet non-lyrical music under clear speech, never competing with proof sounds or lip-sync.

Do not write both modes into one prompt. Do not add lyrics, dramatic trailer music, or a fashion-film soundtrack by default.

## Verdict And Commerce Close

- `soft_verdict`: a casual conclusion based only on what the video visibly proved.
- `ownership_verdict`: a credible keep-or-wear conclusion that does not invent purchase or long-term use history.
- `none`: omit a spoken verdict only when the selected concept closes with an authorized commerce CTA.
- `commerce_cta_mode: none`: allowed for non-Shop content or explicit user direction.
- `light_link`: one short spoken link cue plus at most one small human hand gesture or glance; no generated graphic.
- `evidence_backed_promo`: allowed only with eligible offer evidence and explicit user authorization.

Default structure for Shop work: `soft verdict -> optional light link`. Price, discount, urgency, stock, ratings, sales volume, and shipping are not default US winner-profile ingredients.

## Compact Constraint Pack

Keep only the constraints that materially protect the result:

- one fixed creator wearing the same reference-bound garment;
- exact color, silhouette, neckline, sleeve, hem, seams, texture scale, thickness, opacity, and drape;
- sequential actions, explicit hand/prop continuity, and believable physical light;
- the one persistent US fit-stats overlay;
- no subtitles, other text, logos, watermarks, CTA graphics, identity drift, garment drift, or unsupported claims.

Do not repeat equivalent “no text/no logo/no watermark” or anatomy negatives in multiple sections of the same first-generation prompt.

## US Profile Audit

Approve only when:

- the prompt reads as a control specification, not a translated sales paragraph;
- the garment/result is visible in the first 1-2 seconds;
- the occasion-first scene strategy answers one concrete styling question, the complete outfit fits that location, and an outward-wear item has not defaulted to a private interior;
- each claim, part/effect, frame/action, `proof_endpoint`, and English line agree;
- all spoken beats are `en-US` and every Chinese translation remains review-only;
- complex proof is off-screen voiceover;
- camera and music modes are internally consistent;
- the verdict is earned by visible proof;
- CTA behavior is human-only and promotional language is evidence-authorized;
- constraints are compact and do not compete with the positive scene/action description.
