# Germany Winning Prompt Profile

Use this profile only when `market_prompt_profile_id: de_champion_v1`. Read `market-prompt-compiler-contract.md` first. This profile converts the shared evidence/action plan into a German performance and language script; it is not a translated US prompt and does not override evidence, identity, anatomy, or reference rules.

## Required Controls

```yaml
market_prompt_contract_id: zibuyu_market_prompt_v1
market_prompt_profile_id: de_champion_v1
generation_controls:
  prompt_shell_mode: de_performance_script
  delivery_mode: de_live_simple | de_hybrid_proof
  camera_mode: fixed_phone | creator_handheld | friend_handheld
  music_mode: none | low_non_lyrical
  verdict_mode: ownership_verdict | soft_verdict | none
  commerce_cta_mode: none | light_link | evidence_backed_promo
  screen_text_policy: fixed_model_stats_only
  scene_strategy: required occasion-first object from occasion-scene-routing.md
```

The default is `de_performance_script + de_hybrid_proof`, with an occasion-first plausible daily-use scene for outward wear, the least-loaded camera mode, `none` or truly low non-lyrical music, an ownership-style or soft verdict, and either no CTA or a brief `unten links` light-link close according to the publishing objective. Price, discount, scarcity, shipping, rating, and sales-volume language are never defaults.

## Compiler Identity

Treat the German prompt as a performance/language script with this logic:

`German speaker and lip-sync lock -> concrete buyer concern -> exact part/action proof -> first-person everyday judgment -> ownership verdict or light link`

The immutable v6 identity/reference lines remain first. Immediately after them, state that the one fixed creator speaks natural `de-DE` German, define whether the current beat is live dialogue or the same creator's German off-screen voiceover, and bind visible mouth movement only where required. Do this before scene decoration or shot prose.

## German Performance Skeleton

- Hook: the garment is already on body and readable. Use one plain German buyer concern or wearing result, not a feature list or translated advertising slogan.
- Proof: name the exact visible part, position, movement, construction, or styling relationship; perform one matching action and finish on a readable `proof_endpoint`.
- Judgment: use an everyday first-person evaluation only within the evidence boundary. Do not fabricate purchase, wash, long-term wear, comfort, softness, or testing experience.
- Close: finish with an ownership-style verdict, a soft verdict, or one brief light `unten links` cue. The garment result remains visible and the screen stays clean.

German language localization is more important than forced geographic scenery. For outward wear, choose a calm neighborhood café terrace, pedestrian shopping street, ordinary sidewalk, apartment entryway, office/commute edge, hotel corridor, lakeside promenade, or other credible use location when it answers the buyer better than a private interior. A bedroom, closet, bathroom mirror, or generic home scene is valid only under the private-interior gate. Do not insert German flags, landmarks, signs, stereotyped décor, or artificial local references merely to signal Germany.

## German Occasion And Scene Adapter

Apply `occasion-scene-routing.md` before selecting the performance setup:

- Keep the location useful, natural, and visually restrained. It should make the occasion and outfit solution obvious without becoming tourist scenery.
- Favor practical styling questions such as `Jeans oder Stoffhose?`, `offen oder geschlossen?`, `Alltag oder Urlaub?`, or which shoes/bag complete a café, shopping, commute, or promenade look.
- Use `destination_outfit_share`, `grwm_departure`, a controlled `friend_filmed_walkthrough`, or a stable `fixed_phone_fit_proof` according to the German speech/action load. Scene value does not require an active camera.
- For outward wear, default bedroom/closet/bathroom/home to excluded. Permit a short private proof cut only before one intentional transition to the outward primary scene; a private primary scene requires a specific fit/detail/multi-styling reason.
- Do not confuse natural German daily life with a permanent bedroom setting. A quiet café terrace, ordinary shopping street, entryway, commute edge, or promenade can remain generation-stable while providing stronger use-case meaning.
- Keep landmarks, flags, souvenir styling, artificial German signage, and postcard composition out of the prompt.

## Delivery Modes

### `de_live_simple`

Use only when every visible speaking beat is low-load:

- fixed or subtly moving phone camera;
- still, micro, or simple performer motion;
- no complex turn, pull, closure, multi-step styling, or fine detail proof;
- at most one active demonstration hand;
- concise German line within the active visible-word and timing budget;
- visible mouth, exact lip-sync, and a meaning-matched expression.

If any one condition fails, use `de_hybrid_proof` instead of simplifying away useful product proof.

### `de_hybrid_proof`

This is the default:

- Hook and close may be live `on_camera_dialogue` in German;
- middle product/detail/motion proofs use the same creator's `offscreen_voiceover` in German with the mouth out of frame;
- every proof remains bound to one visible target, action, and endpoint;
- the voice still sounds like one connected recommendation rather than unrelated narration fragments.

Do not describe the middle voice as a different narrator or let the speaking identity change across the clip.

## Deictic Alignment

German demonstratives and pointing expressions must match the visible proof:

- `hier` points to the exact part currently framed;
- `so` describes the exact position, motion, or styling result currently visible;
- `genau da` requires one clear physical target and at most one small natural indication gesture.

Reject a line when `hier`, `so`, or `genau da` refers to a neckline, sleeve, hem, slit, texture, pocket, closure, drape, fit, or styling result that the frame/action does not show. Do not use a generic full-body shot to cover several deictic claims.

## German Language Gate

Every spoken beat uses `spoken_language: de-DE`. The exact German line appears in the prompt; its Chinese translation appears only in `voiceover_review`.

Reject:

- Chinese characters in a German spoken slot;
- high-confidence English template residue, including English sentence frames or transition phrases copied from the US profile;
- mixed-language Hook or CTA caused by translating only part of a completed US script;
- stiff catalog German, official transitions, and literal seller-bullet translation.

Allow verified brand names, product names, platform names, model codes, and common borrowed tokens only when they are intentional and do not form an English sentence frame. Language lint must use an allowlist rather than rejecting every Latin token that also exists in English.

## Camera And Audio

- `fixed_phone`: preferred for precise live German delivery and stable lip-sync.
- `creator_handheld`: allowed for a mirror/selfie concept when the filming hand remains continuously owned.
- `friend_handheld`: allowed when the off-camera friend films; the friend remains invisible unless explicitly authorized.
- Detail setups may lock or subtly reframe under any global grammar. A grammar change requires an explicit cut and new setup run.
- `music_mode` is exactly `none` or `low_non_lyrical`; speech stays foreground and the two modes never coexist.

## Verdict And Commerce Close

- `ownership_verdict`: plain German wording that expresses a credible keep/wear judgment without inventing purchase history.
- `soft_verdict`: a low-pressure conclusion tied to the visible result.
- `none`: omit a spoken verdict only when the selected concept closes with an authorized commerce CTA.
- `commerce_cta_mode: none`: acceptable when the publishing objective does not require a link cue.
- `light_link`: one brief natural `unten links` line with at most one small human gesture or glance.
- `evidence_backed_promo`: only with eligible offer evidence and explicit user authorization.

Do not make price or discount the default German close. Do not generate arrows, shopping carts, badges, stickers, link icons, or UI.

## German Account Overlay Boundary

After this profile is locked, the German account-interaction overlay may sharpen the Hook, comment prompt, caption, profile/shop intent, or styling choice. It may not change the language, identity, evidence, speech-load, camera-mode, music-mode, CTA-purity, or screen-text contract. Account data is an optimization overlay, not proof that a product claim is true.

## DE Profile Audit

Approve only when:

- German speaker/voice/lip-sync control appears immediately after the identity/reference lock;
- the garment/result is visible in the first 1-2 seconds;
- the occasion-first scene strategy gives one practical outfit answer, matches the restrained German treatment, and does not default outward wear to a private interior;
- every spoken beat is `de-DE` and contains no Chinese or high-confidence English template residue;
- `de_live_simple` satisfies every load condition, otherwise the prompt uses `de_hybrid_proof`;
- each claim, exact target, frame/action, `proof_endpoint`, and German line agree;
- `hier`, `so`, and `genau da` point to what is visibly shown;
- the setting fits the garment without forced German landmarks;
- the verdict is evidence-safe and the optional CTA is plain, brief, and human-only;
- Chinese review translations, corpus material, and internal analysis remain outside the generated prompt.
