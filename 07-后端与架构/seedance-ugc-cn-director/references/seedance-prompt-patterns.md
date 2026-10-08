# Seedance Prompt Patterns

Use these patterns to build copy-ready Seedance 2.0 prompts for UGC ecommerce ads.

## Canonical Timeline Binding Rule

These patterns supply creative form and local shot arrangement only. They must not independently choose or rewrite market profile, `generation_controls`, `three_layer_deadlines`, `scene_strategy`, beat `scene_id`, timestamps, spoken lines, scenes, core actions, proof endpoints, camera grammar, music mode, verdict/CTA mode, screen-text policy, or reference order. New schema `1.4` prompts use `canonical_prompt_v7`, with fixed-model identity always first and apparel references second or later. Historical v6/v5 work is read/poll/report-only except an explicitly approved `legacy_v5_exact_resume` whose prompt, ordered references, and hashes remain byte-for-byte unchanged.

Populate every pattern from the active variant's canonical timeline using `[B1 start-end]`, `[B2 start-end]`, and so on. Any literal time ranges below are examples for a new timeline grid, not a second source of truth. The compiled prompt must preserve every beat ID, interval, `spoken_language`, spoken line, core action, `proof_endpoint`, and reference binding exactly as defined in the active contract. Never hand-edit the deterministic v6 role lines or move a garment reference into `@Image1`.

Before applying any pattern, load `market-prompt-compiler-contract.md`, `occasion-scene-routing.md`, then only the selected US or DE profile, followed by `quality-directing-contract.md`, `motion-realism-and-commerce-cadence.md`, and `selling-point-action-proof-methodology.md`; when a URL or review is involved, also load `amazon-product-review-evidence-contract.md`. A pattern supplies form only; identity/evidence/safety, the selected market profile and generation controls, the occasion-first destination/outfit answer, the validated research bundle, pain-solution map, directing intent, continuous creator social intention, three-phase performance/emotion arc, garment-first fidelity allocation, reference transfer contract, phone/prop and endpoint continuity, camera-setup budget, selling-point action-proof rows, and per-beat motion budget override decorative pattern habits.

For `hook_semantics: evidence_backed_contrast`, Shot 1 uses the short negative material-expectation contrast while the garment is visible. Shot 2 must immediately cut to the mapped visible texture in a realistic phone detail close-up; the mouth stays out of frame, off-screen voiceover names the texture, one hand performs one small `pinch_release`, and the fabric settles at the recorded `proof_endpoint`. Do not insert a reaction, styling action, or unrelated proof between these shots.

For every pattern, write voiceover as natural speech rather than seller-page prose. Use short everyday sentences, vary proof/styling openers, and reject official transitions, brochure constructions, or standalone generic praise. In a default 15-second clip, select one `core_sellable_wearing_result` and prove it through multiple angles: visible result, structural reason, alternate angle or buyer-worry proof, motion/handling endpoint, and one specific outfit relationship. Keep one target and one claim per beat; remove supported details that do not help prove the selected result.

Every spoken beat carries `spoken_language: en-US` or `de-DE`; every proof beat carries `proof_endpoint`. Project every exact market line plus its Chinese review translation into a separate `voiceover_review`, hash it, and keep the Chinese text outside `compiled_text`, generated audio, screen text, and market caption.

## 通用提示词骨架

```text
Reference 1 / @Image1 = fixed-model identity only: preserve the selected fixed creator's exact face, skin tone, ethnicity, hair, age presentation, and body identity; do not use it as garment evidence.
Reference 2 / @Image2 = garment identity only: preserve the same garment's exact color, silhouette, neckline, sleeve construction, hem, seams, texture scale, thickness, opacity, and drape. The apparel reference has passed human_identity_pixels_absent: true. Do not transfer reference skin tone, ethnicity, neck, chest, collarbone, shoulders, arms, hands, fingers, nails, tattoos, jewelry, body shape, face, hair, white background, triptych layout, dividers, repeated bodies, pose, camera, environment, text, or watermark. Positive replacement: only the @Image1 creator wears the same garment in one continuous believable real-world scene.
Market control：[market_prompt_profile_id]；[spoken_language]；[delivery_mode]。德国 profile 必须在这里立即锁定同一达人说自然德语、声音归属和口型规则。
Global：9:16竖屏，[时长]秒；[camera_mode]；[music_mode]；`screen_text_policy: fixed_model_stats_only`；真实UGC手机拍摄。默认15秒使用3-4个连续 camera-setup run；同一 setup 不同时写固定和手持，模式改变必须明确切镜。
Scene and occasion：[真实目的地/使用时刻]；解决买家问题“[去这里该怎么穿]”；完整穿搭答案为[下装+鞋+包+配饰]；采用[video_form]。按 `scene_strategy` 保持单一主场景，或仅允许一次从证明场景到主场景的明确切换；外穿款不得默认卧室/衣柜/浴室/普通家中。

Shot 1（约0-4秒，共鸣/结果 Hook）：产品首帧可见；达人像向朋友分享一样说出购买顾虑或想要的穿着结果，并因需要展示版型而自然后退/调整距离；
Shot 2（约4-10秒，动态证明）：保持同一分享意图、拍摄手、道具和服装状态，在两到三个内部节拍中依次完成经过证据允许的部位/侧面/全身动作。每个内部节拍只有一个目标和动作，上一终点自然承接下一起点；
Shot 3（约10-15秒，靠近细节确认）：如道具需要离开，先明确放下/交接/切镜；再因需要看清决定性细节而走近或重构图，完成细节证明和[verdict_mode]。仅当[commerce_cta_mode]需要时加入短CTA；`light_link`最多一只手做小幅左下动作，`none`不做链接手势。

声音：[口播语言与语气] + [music_mode] + [自然环境音/操作音/包装声/脚步声/衣料摩擦声等]。把所有句子说成一段连贯推荐，以重点词和证明转折短停顿形成“温暖共鸣 → 明亮确信 → 愉悦确信/释然”的递进。如果画面中人物开口说话，口型必须与口播内容逐句一致并绑定语义匹配表情；复杂证明必须使用画外音且嘴不入镜。美区使用自然美式朋友口吻；德区按 `de_live_simple` 或 `de_hybrid_proof`，并确保所有口播槽为自然德语。不要匀速、等重音、全程大声或平淡。除非有真实测试或用户授权，不得让达人伪称购买、试穿、洗涤或亲测。
硬约束：一位达人、同一件服装、恰好两条连续连接肩膀—手腕—手掌的自然手臂、左右手轨迹明确、动作逐个完成、干净无生成文字/水印/UI、真实光影；默认一只展示手活动，自拍拍摄手持续持手机，道具持续占用直到明确放下/交接/切镜；禁止双手反复回胯、立正和无动机远近移动。只有证据支持且声明清楚的协调双手证明才允许两手共同完成同一动作。
```

The bracketed market controls are compiler inputs, not text to leave unresolved in a copy-ready prompt. `canonical_prompt_v7` also carries machine-only `three_layer_deadlines_sha256`, `market_prompt_profile_sha256`, `generation_controls_sha256`, and `voiceover_review_sha256`; never print those hashes or the Chinese review translation inside the prompt prose.

## Pattern A: 真人口播

Best for creator recommendation, fashion try-on, beauty, gadgets, daily-use products.

```text
9:16竖屏，[时长]秒，真实TikTok UGC手机视角，设备不入镜。年轻创作者在[场景]中穿着[interface_tag]锁定的服装，姿态放松，不使用立正站姿。
Opening：0-4秒稳定自拍/腰上镜头，服装已经在身上且清楚可见；创作者用温暖笃定的短hook说出共同需求，并因展示版型而自然调整距离；
Proof sequence：4-10秒保持同一推荐意图和手机/道具状态，按照口播顺序连续完成多个内部证明节拍；每个节拍只完成一个匹配动作并承接上一终点，较复杂动作改为画外音且嘴不入镜；
Final：10-15秒明确处理道具后因细节需要走近/重构图，完成决定性证明，以所选 `verdict_mode` 收口；仅在 `commerce_cta_mode: light_link` 时增加短CTA和一次真人左下小手势。
口播：像一段朋友分享，不像广告；语气从共鸣到证明再到确信递进，重点词重读、证明转折短停顿。出镜开口时口型必须与口播内容一致并有匹配表情。
硬约束：一位达人、同一件服装、自然手脸、干净屏幕；只追加已观察故障的针对性限制。
```

## Pattern B: 旁白 + 手部演示

Best for products that need close-up proof or where no creator face is needed.

```text
9:16竖屏，[时长]秒，真实UGC手部演示视频，手机近距离拍摄，使用 `scene_strategy` 已锁定的真实用途或证明场景。外穿服装优先在其目的地场景完成证明；卧室/衣柜/浴室/普通家中仅在私密室内门禁允许时使用。
Opening：产品首帧可见，直接呈现[痛点/结果]并做一次自然产品揭示；
Proof sequence：嘴不入镜，画外音逐条对应材质/领口/袖口/下摆/结构；每条使用匹配特写和一个动作，完成后再切换下一目标；
Final：完成全身穿着/使用结果动作后给出所选可信结论，并仅在 profile 要求时说短CTA。
声音：[语言]旁白 + 自然操作声，旁白要日常口语化、像真实买家自然分享，不要生硬广告腔，语速自然不快不慢。
硬约束：同一件服装、每个卖点一个顺序证明动作、左右手完整交代、干净屏幕；只追加已观察故障的针对性限制。
```

## Pattern C: 测评反应

Best for “I tried it” style, first impression, try-on, unboxing, comparison.

Use this pattern only when a real try-on/test exists or the user explicitly authorizes that role-play. Direct Amazon reviews may inspire qualified buyer-pain wording, but they do not authorize the generated creator to claim that she personally bought or tested the garment.

```text
9:16竖屏，[时长]秒，真实TikTok测评反应风格，创作者在[场景]中第一次试用/试穿产品。
Opening：创作者穿着产品提出一个真实疑问或顾虑，服装首帧可见，并自然进入放松的试穿姿态；
Proof sequence：嘴不入镜，画外音每次说明一个可见事实；镜头对准该部位，完成对应动作与终点后再进入下一证明；
Final：用一个全身/侧面动作完成试穿结果，再说一条证据范围准确的观察/结论；仅在 profile 要求时加CTA短句，不得冒充真实购买者或复制买家评论身份。
声音：[语言]口播，语气自然，有轻微停顿和真实反应；如果创作者出镜开口，口型逐句匹配口播内容，语速为正常自然说话速度，口播像真实买家试穿后的自然表达。
硬约束：一位达人、同一件服装、自然反应、干净屏幕；只追加已观察故障的针对性限制。
```

## Pattern D: 剧情对话

Best for pain-point products, fashion, home goods, gifts, products with a relatable problem.

```text
9:16竖屏，[时长]秒，真实朋友反应式UGC；默认只有一位出镜达人，朋友始终画外，不生成第二个人物。
Opening：画外朋友提出[具体痛点/穿搭问题]，达人穿着[interface_tag]锁定的服装自然回应，并做一个与hook一致的放松版型动作；
Proof sequence：嘴不入镜，达人依次用多个简单、非重叠动作证明被提到的具体部位，朋友/达人使用画外短句，不做双人互动；
Final：达人独自完成最后一个穿搭/侧面展示动作和 profile 规定的结论；只有 `light_link` 才增加一次真人左下小手势。
台词：[语言]，短句、口语化；只有可见嘴部说话时才要求逐句口型同步。
正向约束：始终一位出镜达人、一个连续真实场景、同一件服装；问题朋友仅作为画外声音存在。
```

## Pattern E: 混合形式

Best default for ecommerce UGC ads.

```text
9:16竖屏，[时长]秒，混合UGC广告：自拍口播 + 产品近景 + B-roll演示。整体像真实TikTok创作者内容，自然光，手持手机镜头。
Opening（0-4秒）：腰上自拍hook，[interface_tag]锁定的服装首帧可见；用温暖笃定的短句说出需求/结果，并做一个有动机的版型展示动作；
Proof sequence（4-10秒）：胸口到下摆、部位特写、侧面或全身镜头按卖点连续承接；画外音配合多个顺序简单动作，每个动作到达可见终点，手机/道具/重心状态不无故重置；
Final（10-15秒）：明确放下/交接道具后因看细节而走近或重构图，完成决定性证明 + profile 规定的可信结论；仅在 `light_link` 时增加短CTA和一只手的真人左下小动作。
声音：[语言]口播 + 自然环境音，节奏快但不吵；所有句子连成一段真实分享，表情和声音从共鸣到证明再到确信递进，出镜说话时口型与逐字内容一致。
硬约束：一位达人、同一件服装、自然双手/口型/光影、干净屏幕；只追加已观察故障的针对性限制。
```

## Fashion / Apparel Notes

For each clothing video, sequence rather than stack simultaneously:

- one commercial throughline, supported by at least three distinct garment-part/effect proofs in a default 15-second clip
- one continuous creator social intention and three viewer-facing macro phases around five to seven internal beats
- one matching close-up or fit frame for each spoken part claim
- one action-proof row per selected selling point: proof target, frame, action selector choice, left/right hand anchors, visible endpoint, concise spoken line, and sparse micro-cues
- four or more sequential non-CTA product/styling actions, such as a neckline trace, cuff touch, arm raise, pinch-release, hem release, slow quarter-turn, pocket use, closure operation, or front tuck
- one outfit pairing; do not perform several accessory changes inside the generation
- one occasion-first scene strategy that names where the buyer wears the garment, her styling question, the complete outfit answer, the video form, and the private-interior policy; every beat carries the matching `scene_id`
- three to four contiguous camera-setup runs in a default 15-second prompt; change framing when the named garment part changes
- one physical action and one visible endpoint per beat; multiple actions are allowed across beats but never simultaneously
- a locked/subtle camera and normally one active hand for visible lip-sync/detail proof/CTA; a declared evidence-backed coordinated two-hand proof is the only detail exception
- endpoint-to-start continuity for gaze, weight, expression, phone/task hand, props, and garment state; no repeated both-hands-to-hips reset or disappearing prop

Avoid unsupported claims like exact slimming or temperature control. Exact composition requires an eligible locked catalog/label source; numeric performance requires verified test evidence. A `Low Stretch` attribute means no strong two-hand pull demonstration.

## Product Page To Video Notes

If using a product page:

- lock the requested URL, parent/child product ID, selected variant, capture route/time, and research status before extraction
- separate structured catalog attributes, seller copy, direct customer reviews, customer images, Q&A, Amazon review summaries, and third-party summaries under `amazon-product-review-evidence-contract.md`
- preserve package language and visible product identity; do not substitute a related ASIN or sibling variant as an exact selected-variant fact
- convert only eligible evidence items into one matching visual demonstration per prompt; carry the same evidence IDs through claim, proof, and beat
- use direct reviews only for qualified buyer experience/pain; one review supports one-buyer attribution and sample-theme language requires at least three unique review IDs
- use an evidence-backed negative material-expectation contrast only when it is review-derived, script-eligible, and immediately followed by its independently evidenced texture close-up and `pinch_release`
- use customer images only for actually inspected visible fit, length, color, or construction; never infer feel, composition, performance, comfort, care, or durability from an image
- treat Amazon AI summaries, search snippets, and third-party summaries as discovery-only
- record `partial`/`blocked` and remove unsupported statements when Amazon denies access; do not bypass CAPTCHA or fabricate missing reviews
- preserve conflicts such as incompatible care instructions or mixed sizing feedback; do not choose the more marketable side
- do not include prices, discounts, ratings, review totals, or shipping promises unless explicitly eligible and requested

## Prompt Quality Checklist

Before finalizing the Seedance prompt, check:

- New work uses schema `1.4`, `zibuyu_ugc_quality_v4`, `canonical_prompt_v7`, `zibuyu_three_layer_deadlines_v1`, and `zibuyu_market_prompt_v1`; historical v6/v5 is not silently rewritten or resubmitted.
- Market is selected before the creative pattern; `us_champion_v1` or `de_champion_v1` matches the model, market, and language, all seven scalar `generation_controls` fields are present, and the complete occasion-first `scene_strategy` is present and mutually consistent.
- The first two lines exactly map `Reference 1 / @Image1 = fixed-model identity only` and `Reference 2 / @Image2 = garment identity only`; no garment reference occupies `@Image1`.
- Every apparel reference is tagged `@Image2+`, has `human_identity_pixels_absent: true`, and excludes all skin/anatomy/person-specific identity cues rather than only face and hair.
- Product appears in first 2 seconds.
- One directing intent governs camera, light, performance, sound, and proof.
- The scene answers one concrete destination-specific styling question with a complete outfit formula; every beat `scene_id` matches the locked location plan, and outward wear has not defaulted to a private interior.
- The apparel three-view is role-locked to garment identity and explicitly cannot transfer skin tone, ethnicity, neck/chest/collarbone, shoulders, arms/hands/fingers/nails, tattoos, jewelry, body shape, face/hair, white triptych layout, repeated bodies, pose, camera, environment, or text.
- Garment identity is the primary fidelity spend and only one secondary spend remains active.
- A default 15-second video uses three to four contiguous camera-setup runs and five to seven sequential beats.
- Those internal beats read to the viewer as recognition/result Hook, animated proof, then close/detail conviction under one continuous friend-to-friend intention.
- Every beat has one action, one visible endpoint, motivated camera/light, and a valid motion budget.
- Every spoken beat has the exact `spoken_language`; every proof beat has a concrete `proof_endpoint` matching the claim, target, frame, action, and spoken line.
- Every selected selling point follows `selling-point-action-proof-methodology.md`: one claim, one target, one frame, one action path, one endpoint, and one matching concise spoken line.
- Prompt has one commercial throughline with multiple individually proven garment points.
- Every garment/styling claim has a matching frame, action target, hand plan, action, and visible result; no explanatory beat is a full-length static hold.
- Every selected pain and claim resolves to eligible source IDs; seller copy, summaries, mismatched variants, conflicts, and buyer images have not been promoted beyond their allowed use.
- Any `evidence_backed_contrast` Hook is negative and declarative, carries the review pain rather than a product claim ID, and is followed immediately by the mapped exact-variant texture proof.
- `Low Stretch` or another limited-performance attribute has not been converted into an exaggerated pull/recovery demonstration.
- Camera behavior is concrete.
- Actions are physical and visible.
- Phone/task/prop ownership and the previous endpoint carry forward; every distance change, place-down, handoff, reset, or cut is motivated and explicit.
- Voice language matches user choice.
- US uses `us_hybrid_share`; Germany uses valid `de_live_simple` or default `de_hybrid_proof`. Complex proof is off-screen voiceover with the mouth out of frame.
- German spoken slots contain no Chinese or high-confidence English template residue, and `hier`, `so`, or `genau da` points to the exact visible target.
- If a visible person speaks, mouth/lip movement matches the voiceover content.
- Each visible line has a meaning-matched expression, and multiple visible lines use at least two distinct expression states across the three-stage energy rise.
- American-English 15-second creator voiceover stays within 40-62 total words; roughly 50-60 words at 200-235 WPM is used only when a read/TTS audition stays clear. Other languages use local pacing.
- Voiceover wording is casual, everyday, and buyer-friendly, not stiff brand-ad copy.
- No official/brochure construction or standalone generic praise remains, and the same opener is not reused across three or more non-CTA proof/styling lines.
- The non-CTA sequence proves one selected core sellable wearing result through concrete angles when evidence permits; every line names one exact target plus a visible behavior, position, construction fact, or outfit relationship without stacking claims.
- Negative constraints protect product consistency.
- `camera_mode` and `music_mode` do not conflict; the same setup is not both fixed and handheld, and `none` is not combined with `low_non_lyrical`.
- The close follows `verdict_mode` and `commerce_cta_mode`; any CTA is human-only, price/discount language is not a default, and `fixed_model_stats_only` is the sole screen-text policy.
- `voiceover_review` contains every exact market line plus Chinese review translation, while the Chinese text stays outside the prompt. `market_prompt_profile_sha256`, `generation_controls_sha256`, and `voiceover_review_sha256` are present as machine-only bindings.
- The prompt can be copied directly into Seedance without extra explanation.
