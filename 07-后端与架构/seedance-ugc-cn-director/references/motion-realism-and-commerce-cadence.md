# Motion Realism And Commerce Cadence Contract

Use this contract with `market-prompt-compiler-contract.md`, the selected US or DE profile, and `quality-directing-contract.md` for every apparel timeline and Seedance/PopBoom prompt. It turns "move naturally" into observable choreography and keeps the clip paced like direct-response commerce rather than a slow fashion film. Market must be selected first; this motion contract may not override the profile's delivery, camera, music, verdict, commerce CTA, or screen-text controls.

## 1. Optimize For Completed Proof, Not Constant Motion

For a default 15-second clip:

- organize the viewer-facing performance as three macro phases—recognition/result Hook, animated proof, and close/detail conviction—while keeping five to seven internal sequential proof beats grouped into three to four contiguous camera-setup runs;
- complete at least four non-CTA product or styling actions, including at least three garment-part/effect proofs;
- deliver a newly completed product proof or styling result about every two to three seconds;
- let each action reach a readable endpoint and decelerate long enough to read it; for proof beats, record the same concrete state in `proof_endpoint`; carry that endpoint into the next start state, and reset or cut only when clarity requires it;
- count an action only when it targets the garment or styling result and produces a visible endpoint.

Do not count blinking, smiling, nodding, breathing, a camera push-in, a prop move, or a CTA gesture as product proof. Micro-movements make the creator feel alive; they never replace merchandise demonstration.

Treat the action limit as a simultaneity limit, not a clip-wide scarcity rule. Keep several useful actions across the clip, but never make the creator walk, turn, pull fabric, point, and visibly speak at the same time. The three macro phases are a performance and narrative layer, not permission to collapse evidence into only three generic actions.

## 2. Give The Creator One Continuous Social Intention

Direct the person as someone showing a useful find to a friend, not as a model completing isolated product-inspection tasks.

- Give the clip one continuous social intention, such as `I know what you have been looking for, and I want to prove why this solves it`.
- Give each macro phase one immediate human objective: call out the shared need, prove the result in motion, then move closer to confirm the decisive detail and close with conviction.
- Carry gaze, weight, facial energy, hand occupancy, props, and garment state forward from one internal beat to the next. The previous endpoint should normally become the next start state.
- Use motivated distance changes. Step back because the viewer needs the full fit; move or turn because the fabric behavior is the proof; come closer because the viewer needs to inspect a detail.
- Keep a persistent task anchor when the format creates one. In a mirror selfie, the filming hand keeps holding the phone through the entire continuous take. A hand holding an alternate color, bag, or garment remains occupied until an explicit place-down, handoff, or intentional cut.
- Do not snap both hands back to symmetric hip anchors, return to attention posture, or freeze between every proof. Use a relaxed side, phone, prop, pocket edge, side seam, or the previous action endpoint as the next natural anchor.
- Prefer outcome-led direction over coordinate-led puppeteering. Exact centimeters, degrees, and timed freezes are allowed only when product proof truly requires them; ordinary creator motion should be described by motive, path, and readable result.

Keep one proof target and one main action inside each internal beat. Continuous intention means the beats flow like one share; it does not authorize simultaneous unrelated actions.

## 3. Write Every Beat With A Complete Action Grammar

Specify every beat in this order:

1. `start_state`: framing, body orientation, relaxed posture, planted foot or weight-bearing leg, gaze, and garment position;
2. `left_hand_start` and `right_hand_start`: exact stable anchors before movement;
3. `main_action`: one active hand or one body action aimed at one proof target;
4. `motion_path`: a short concrete trajectory from the declared start anchor to the target;
5. `visible_endpoint`: the completed product state the viewer must be able to read;
6. `left_hand_end` and `right_hand_end`: stable end anchors after the action;
7. `handoff`: preferably a motivated continuation whose endpoint becomes the next start state; otherwise one explicit place-down, reset, or intentional cut.

Map this grammar into the canonical fields: `posture_id`, `core_action`, `action_target`, `hand_plan.left`, `hand_plan.right`, `visible_endpoint`, `proof_endpoint`, and `camera_setup_id`. Do not hide unrelated or independently countable actions inside one `core_action` with `while`, `and`, or a chain of verbs. One coordinated action cycle such as touch-return, pinch-release, pull-release, or turn-settle remains one action.

Prefer instructions such as:

> Mirror-selfie stance, weight relaxed on the left leg. The right hand keeps the phone at face height. The left hand rises from the side seam, touches the sleeve cuff once, then relaxes beside the torso. The cuff remains unobstructed as she glances from it back to the mirror.

Reject vague instructions such as `naturally shows the sleeve`, `moves dynamically`, or `keeps gesturing while talking`.

## 4. Direct Biomechanically Believable Motion

- Use natural ease-in and ease-out instead of constant-speed limb motion or instant starts and stops.
- Lead arm movement from the shoulder, then let the elbow and wrist follow in one connected chain.
- Lead turns and steps with a small weight transfer through the hips and supporting leg; keep the feet planted for hand-only detail proofs.
- Let fabric react a fraction after the hand or body moves, then settle into believable folds before the cut.
- Keep wrists neutral, fingers softly grouped, elbows slightly relaxed, shoulders uneven by a small natural amount, and the torso responsive without swaying.
- Add one or two low-load human cues when the face is visible: a natural blink, soft breath, tiny head tilt, small mouth-corner change, or subtle weight shift.
- Keep eye direction causal: camera/mirror for Hook and CTA, garment target during proof, mirror or outfit during styling, then naturally back to the viewer.
- Preserve state carryover: shoulders, weight-bearing leg, phone height, free-hand location, prop ownership, and expression should evolve from the previous endpoint rather than restarting from a neutral pose.

Do not prescribe continuous random movement. A real creator often pauses for a fraction of a second after completing a demonstration so the viewer can read the result.

## 5. Preserve Native Commerce UGC Energy

Direct the clip as an authentic phone-shot recommendation to a friend:

- keep the garment visible in the first frame and make the first action reveal the selected wearing result or buyer concern;
- use short, decisive demonstrations and immediate transitions to the next proof;
- allow slight handheld breathing only when it does not compete with lip-sync or detail proof;
- keep close-ups, visible dialogue, and CTA shots locked or subtly moving;
- use ordinary room tone, fabric rustle, one footstep, or bag-strap sound where motivated;
- preserve realistic skin, garment folds, room light, and small imperfections.

Honor the selected generation controls. `camera_mode` is one global grammar—`fixed_phone`, `creator_handheld`, or `friend_handheld`. A proof setup may lock or subtly reframe, but the same `camera_setup_id` cannot be both fixed and handheld; a grammar change requires an intentional cut and a new setup run. `music_mode` is exactly `none` or `low_non_lyrical`; natural room/garment sound is allowed under either, but the two music modes never coexist and music never competes with speech.

Shape energy rather than holding one expression. Use a restrained three-step emotional arc: warm recognition in the Hook, brighter animated certainty during proof, then delighted conviction or relief at the decisive result. Create energy through emphasis, short conversational pauses, eye changes, and a smile that grows with the proof—not shouting, a fixed grin, or frantic movement.

Reject `cinematic fashion film`, `luxury commercial`, `graceful slow motion`, catwalk pacing, continuous posing, dramatic orbit shots, or a long beauty hold unless the user explicitly asks for a non-commerce fashion treatment. Do not solve low energy by adding dancing, frantic cuts, repeated pointing, or unmotivated camera movement.

## 6. Partition Motion Load By Beat Type

- Hook with visible dialogue: locked/subtle phone camera, relaxed stance, one reveal action or one small hand action, at most one active demonstration hand; a filming hand may remain continuously task-anchored on the phone.
- Detail proof: mouth out of frame, off-screen voiceover, locked/subtle close-up, one active hand, other hand anchored.
- Turn, step, sit, or drape proof: off-screen voiceover, stable camera, hands anchored unless one hand is the proof, one completed body action.
- Styling proof: one front tuck, bag placement, necklace touch, shoe reveal, or step-back; never change several accessories in one beat.
- profile close: visible verdict dialogue or profile-valid voiceover uses still/micro performer load. Only `commerce_cta_mode: light_link` or an authorized evidence-backed promo adds one small down-left human gesture with the other hand anchored; `none` adds no link gesture. Generated CTA graphics are always prohibited.

If a beat exceeds these loads, split it or move its speech to off-screen voiceover. Preserve the product action instead of deleting all movement.

When `secondary_fidelity_spend: lip_sync` makes creator delivery the retention engine, allow up to three visible-dialogue anchor beats—one per macro phase—only when each uses locked/subtle camera, simple or lower performer load, no more than one active demonstration hand, concise speech, and an explicit expression cue. Keep complex detail and movement proof off-screen.

## 7. Default 15-Second Cadence Skeleton

- `0-4s — recognition/result Hook`: garment visible immediately; use one motivated reveal or step-back and friend-to-friend language that names the buyer need or desired wearing result.
- `4-10s — animated proof`: keep the same creator intention alive while completing two or three internal proof beats. Use movement, drape, side/back, handling, or fit evidence; every proof still has one target and endpoint.
- `10-15s — close/detail conviction`: move closer or intentionally reframe for one decisive detail, resolve the main worry, then close with the selected market verdict and any profile-authorized human-only CTA.

Adjust boundaries to the garment and spoken language, but preserve five to seven internal beats, the completed-proof rhythm, and the required action floor. Do not let an explanatory beat remain motionless from start to finish. When a prop disappears between phases, declare the place-down or intentional cut.

## 8. Voice Delivery And Emotional Rhythm

- Write the voiceover as one connected recommendation, not six isolated catalog sentences. Use live-share connectors such as target-language equivalents of `look`, `this`, `when I move`, `see how`, and `honestly` when truthful and idiomatic.
- For a 15-second American-English creator-led mirror share, aim for roughly 50-60 total words and about 200-235 WPM only when a TTS/read-aloud pass remains clear. Use a broader validator safety range rather than forcing every format to hit the upper target.
- Do not copy the English word count into German or other languages. Audition for local conversational clarity and preserve the same three-stage energy arc.
- Mark emphasis words and phrase breaks. Use brief pauses at proof pivots and sincerity markers; avoid equal stress, equal-length sentences, and identical falling intonation on every beat.
- Passion means credible certainty plus changing emotional stakes. Do not solve flat delivery by making the voice uniformly louder or faster.
- `us_hybrid_share` may show concise American-English Hook/close dialogue; middle complex proof is off-screen voiceover with the mouth out of frame.
- `de_live_simple` is allowed only for locked/subtle camera, simple-or-lower performer motion, at most one active hand, exact German lip-sync, and concise timing-safe lines. Otherwise use default `de_hybrid_proof`: German Hook/close may be live, and the same creator's German middle proof is off-screen with the mouth out of frame.
- Every spoken beat carries `spoken_language`; every exact market line has a separate Chinese review translation in `voiceover_review`. Chinese review text never enters the generated prompt or audio. German spoken slots reject Chinese/high-confidence English template residue, and `hier`, `so`, or `genau da` must resolve to the visible proof target.

## 9. Canonical Prompt Clause

The deterministic `canonical_prompt_v7` serializer must carry the visible consequence of this contract after its fixed first two role lines, global deadline layer, and asset deadline layer. `Reference 1 / @Image1 = fixed-model identity only` and `Reference 2 / @Image2 = garment identity only` remain immutable. Motion text and per-shot deadlines may never reorder or blur those roles, weaken the global/asset locks, or contradict `generation_controls`:

> Use native direct-response commerce cadence inside one continuous friend-to-friend recommendation: organize the performance as recognition, animated proof, then close-detail conviction while completing a new product-proof or styling action about every 2-3 seconds. Carry gaze, weight, expression, filming/task hand, props, and garment state forward; let each endpoint hand off naturally into the next start state, and reset or cut only when clarity requires it. Motivate distance changes by what the viewer needs to see. Do not snap to attention, symmetrically reset both hands to the hips, or perform isolated equal-energy product poses. Use natural acceleration/deceleration, shoulder-elbow-wrist follow-through, small breathing/blink/weight shifts, realistic garment lag/settling, and an emotional rise from warm recognition to animated proof to delighted conviction; micro-movements never replace proof.

Keep the detailed rationale, market profile, generation controls, `voiceover_review`, and `market_prompt_profile_sha256` / `generation_controls_sha256` / `voiceover_review_sha256` bindings in the internal plan. Feed Seedance only the compact cadence clause, concrete beat actions, endpoints, hand plans, camera/light, market-language speech/sound, selected close behavior, and continuity constraints.

## 10. Audit And Repair

Before submission, reject a timeline when any answer is no:

- Does a new completed product/styling result arrive roughly every two to three seconds?
- Does every proof beat name one target, one action, one visible endpoint, and the same concrete `proof_endpoint`?
- Do both hands have start, action, and end anchors?
- Does the body have a plausible weight-bearing state for every turn or step?
- Do the hand, fabric, and body accelerate, decelerate, follow through, and settle naturally?
- Are micro-expressions supportive rather than substituted for product action?
- Does the clip feel like a phone-shot recommendation rather than a slow fashion commercial?
- Can the viewer feel three connected performance phases even though the proof plan retains five to seven internal beats?
- Does one social intention motivate every distance change and product action?
- Does each endpoint carry naturally into the next start state without a repeated hip reset or attention pose?
- Are phone, alternate-color garment, bag, and other props continuously owned until an explicit handoff or cut?
- Does voice and expression rise from recognition to proof to conviction instead of staying flat?
- Do model, market, `spoken_language`, delivery mode, camera mode, and music mode agree without mixed-market or fixed/handheld conflicts?
- Is complex proof off-screen with the mouth out of frame, and is any final CTA exactly the profile-authorized human-only behavior?

For a failed take, record one primary motion variable: `cadence`, `action_overload`, `hand_path`, `weight_transfer`, `constant_speed`, `endpoint_missing`, or `fashion_film_pacing`. Change only that variable for an authorized retake. If the same failure repeats twice, split or rewrite the beat instead of adding more adjectives.
