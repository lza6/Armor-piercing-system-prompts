# Occasion-First Scene Routing

Use this reference before choosing the video form, location, outfit, or canonical timeline for every apparel prompt. A scene is not background decoration. It is part of the sales argument: it should attract the right occasion interest, show where the garment belongs, and answer the buyer's question about what to wear there.

## Required Scene Strategy

Persist one `scene_strategy` inside `generation_controls` before writing the timeline:

```yaml
scene_strategy:
  wear_context: outward_wear | homewear | mixed
  scene_role: occasion_outfit_solution | movement_proof | fit_proof | detail_proof | multi_styling
  primary_scene_id: cafe_social | beach_vacation | commercial_street | city_sidewalk | office_commute | travel_hotel | entryway_departure | outdoor_leisure | home_living | bedroom_mirror | closet_try_on | bathroom_mirror | other_use_case
  primary_scene_description: natural language description of the real location
  occasion: the concrete destination or use moment
  buyer_styling_question: the buyer's occasion-specific outfit uncertainty
  outfit_answer: complete bottoms, shoes, bag, and accessory formula
  video_form: destination_outfit_share | grwm_departure | friend_filmed_walkthrough | fixed_phone_fit_proof | mirror_to_destination_match_cut | travel_pack_and_wear | multi_styling_switch | home_try_on
  location_plan: single_primary_scene | primary_plus_proof_cut
  bedroom_policy: excluded | proof_only | primary_justified | homewear_primary
  proof_scene_id: optional scene ID; required for primary_plus_proof_cut
  proof_scene_description: optional natural location description; required for primary_plus_proof_cut
  bedroom_justification: optional; required for primary_justified
```

Every schema 1.4 canonical beat also carries a machine-only `scene_id` that equals the selected primary or proof scene. Keep the readable `scene` field as natural visual direction. Do not print enum labels in the generated prose.

## Selection SOP

Apply this order:

1. Classify the garment as `outward_wear`, `homewear`, or `mixed` from product evidence and intended use.
2. Name one concrete occasion the buyer wants permission or help to dress for. Avoid empty labels such as `daily life` when a more useful destination is available.
3. Write the buyer's occasion-specific styling question, for example: `What can I wear to brunch that feels polished but not overdressed?`
4. Select the scene whose native activity answers that question and supports the decisive garment proof.
5. Build the complete outfit answer before writing dialogue: bottoms, shoes, bag, and one or two restrained accessories.
6. Select the video form from the proof and location needs, not from habit.
7. Score the route on five checks: occasion relevance, styling clarity, product-proof readability, native scene interest, and generation stability. Reject a route that fails any two checks.
8. Populate the timeline only after the scene strategy is locked. The Hook, proof actions, framing, movement, sound, and verdict must feel physically native to that location.

## Scene Sales Jobs

| Scene | Buyer problem it can solve | Strong proof/action use |
| --- | --- | --- |
| Café or terrace | `What works for brunch, a casual date, or coffee without looking overdressed?` | seated-to-standing full fit, bag pickup, neckline/sleeve detail, arrival outfit check |
| Beach, boardwalk, or promenade | `What can I wear on vacation that moves well and still feels covered?` | shaded walking, controlled turn, hem/drape movement, sandal and tote pairing |
| Commercial street | `What looks polished enough for shopping, lunch, and city photos?` | friend-filmed walk, storefront-reflection fit check, crossbody bag, full-body proportion proof |
| City sidewalk or street wall | `What is an easy real-world weekend outfit?` | walk-settle, side/back view, sneaker/jeans formula, natural street sound |
| Office lobby or commute edge | `How do I look work-ready without feeling stiff?` | trouser/front-tuck formula, sleeve/closure proof, structured bag, fixed-phone fit check |
| Hotel, suitcase, or travel corridor | `What is easy to pack and can cover more than one trip moment?` | pack-and-wear, one-item styling, crease-safe visible handling only when evidenced |
| Entryway or apartment hallway | `Does this complete the outfit before I leave?` | GRWM finish, shoes/bag pickup, before-going-out full-fit endpoint |
| Outdoor leisure setting | `What works for a market, park, lakeside walk, or casual event?` | movement and layering proof tied to the exact activity |
| Bedroom, closet, or bathroom mirror | `How does it fit, sit, or combine at close range?` | short fit/detail/multi-styling proof only under the private-interior gate below |

Choose the location for the garment and buyer, not because the location appears in this table. A beach is wrong for a structured office blouse when it weakens credibility; an office lobby is wrong for a true cover-up when the vacation use case is the sale.

## Video-Form Router

- `destination_outfit_share`: use when one real location and one complete look are the main answer. The creator is already dressed at the destination and explains why the outfit works there.
- `grwm_departure`: use an entryway or hallway when finishing the outfit is the story. Each action adds one useful styling decision before leaving.
- `friend_filmed_walkthrough`: use for street, café arrival, commercial street, promenade, or outdoor leisure when full-body motion and proportion matter. The friend stays off camera.
- `fixed_phone_fit_proof`: use when precise fit, live dialogue, or garment fidelity needs a settled camera. It may still be located in a café terrace, office lobby, sidewalk corner, or entryway.
- `mirror_to_destination_match_cut`: use only when an initial private proof genuinely adds value, followed by one intentional cut to the outward-use scene where the look remains for the rest of the clip.
- `travel_pack_and_wear`: use for hotel/travel logic when the garment and evidence support the packing/use story.
- `multi_styling_switch`: use when one item solving several outfit decisions is the core result; keep changes simple and evidence-safe.
- `home_try_on`: reserve for homewear or a justified private fit-proof concept.

Do not default every garment to mirror selfie, try-on, or bedroom UGC. Likewise, do not force every garment into an outdoor walk. The selected form must make the core wearing result easier to believe.

## Private-Interior Gate

Treat `bedroom_mirror`, `closet_try_on`, `bathroom_mirror`, and `home_living` as private-interior scenes.

- For `outward_wear`, default `bedroom_policy` to `excluded`.
- Use `proof_only` only with `primary_plus_proof_cut`. The private scene may occupy no more than the first two beats, then the video makes one intentional transition to the outward primary scene and stays there through the close.
- Use `primary_justified` for outward wear only when the dominant sales job is `fit_proof`, `detail_proof`, or `multi_styling`, and write a specific justification. `Convenient to film`, `natural light`, `UGC feel`, and `bedroom is common` are not valid reasons.
- Use `homewear_primary` only when the garment is genuinely homewear and the home use case is the product truth.
- A private scene may never be chosen merely because it is stable for generation. Stability affects execution inside the right scene; it does not choose the buyer's destination.
- If an outward-wear garment remains primarily in a private interior and the strategy cannot explain what buyer problem that choice solves better than café, street, travel, office, entryway, or another real-use location, reject and reroute it.

## United States Adapter

For `us_champion_v1`, use location interest more boldly when it fits the garment:

- favor recognizable but non-branded lifestyle occasions such as brunch café, beach/boardwalk, commercial street, weekend market, sidewalk, travel hotel, or casual event arrival;
- let the location strengthen the Hook, such as a destination outfit check or friend-filmed arrival, while keeping garment proof dominant;
- permit warmer energy, more visible movement, and friend-handheld or creator-handheld grammar when the proof load allows it;
- avoid staged influencer spectacle, crowded backgrounds, trademarked storefront focus, or scenery that hides the garment.

US scene traffic is useful only when the viewer immediately understands both the destination and the outfit solution.

## Germany Adapter

For `de_champion_v1`, keep the same occasion-first logic with a more restrained, plausible visual treatment:

- favor a calm neighborhood café terrace, pedestrian shopping street, ordinary sidewalk, apartment entryway, office/commute edge, hotel corridor, lakeside promenade, or practical vacation setting;
- use clean natural backgrounds, controlled movement, and the least-loaded camera grammar that preserves German speech and product proof;
- make the styling answer practical and specific, such as jeans versus trousers, open versus closed, everyday versus vacation, or one clear shoe/bag choice;
- avoid flags, landmarks, souvenir scenery, artificial German signs, or postcard tourism. Geographic decoration never substitutes for natural German language and believable daily use.

## Timeline And Continuity Rules

- Prefer `single_primary_scene` for a stable 15-second generation. Use three to four camera setups inside the same physically coherent location.
- Use `primary_plus_proof_cut` only when the second location performs a distinct proof job. Allow one scene transition, not location hopping.
- Keep the primary use-case scene in the majority of beats and in the final verdict/CTA beat.
- Give every beat the correct `scene_id`; never let prose name a bedroom while the machine field claims café or street.
- Project the locked strategy exactly: `creative_delta.scene_id` and `quality_plan.continuity_anchors.scene` equal `primary_scene_id`; `creative_delta.styling_signature` and `quality_plan.continuity_anchors.outfit` equal `outfit_answer`.
- Match location sound and movement to the scene: quiet cups/chairs at a café, restrained footsteps on a shopping street, breeze/fabric movement at a promenade, or entryway shoe/bag sounds. Do not add generic room tone to every location.
- Keep lighting physically plausible for the chosen place and preserve garment color/readability.
- Keep the full outfit consistent across scene cuts unless a declared styling change is the point.

## Rejection Gate

Reject or rewrite before handoff when:

- the strategy does not state where the buyer would wear the garment;
- the scene does not answer a concrete styling question;
- the outfit answer is missing bottoms, shoes, or a bag/accessory decision;
- creative or continuity scene/outfit fields drift from the locked strategy;
- an outward-wear item defaults to a bedroom/closet/bathroom/home scene without the private-interior exception;
- `scene_id`, readable scene prose, outfit, action, audio, or light contradict one another;
- the primary use-case scene is absent from most beats or from the final result;
- the location is chosen only for visual novelty and does not improve product proof or occasion imagination;
- the same SKU/color batch repeats the same private scene without a garment- or strategy-specific reason;
- US output ignores a useful lifestyle occasion, or German output forces tourist/landmark localization.
