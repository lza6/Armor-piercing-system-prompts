# Output Blueprint

Use this structure after the user has answered the blocking intake questions or has clearly authorized assumptions. Missing publishing platform, video duration, and video sound/visual format are not blocking; default platform to TikTok, default duration to 15 seconds, and auto-select the format from recent-half-month TikTok apparel-commerce research.

## SKU Batch Output And Timeline Rendering

When `sku_batch_compile` is active, output one shared-core block for locked product identity, Amazon/product evidence, direct-review analysis, conflicts, pain-solution mapping, garment analysis, TikTok digest, claims, selected market profile, selected template, and timeline grid, followed by one variant block per color. Do not repeat the shared analysis for every color.

For each color, section 7 is the canonical timeline and spoken-line projection; section 8 is only the visual projection of that same timeline; section 9 is only the ordered final-prompt compilation of the same beat IDs. All three must carry the same `timeline_id` and `timeline_version`. Section 10 must report timeline-integrity, rendering-consistency, pairwise planned-differentiation, and—when actual videos are available—rendered-differentiation status.

Use one shared hook pool if useful, but assign a different primary hook and creative delta to every color. Follow `sku-batch-timeline-contract.md` for the exact schema and hard gates.

In plugin production, materialize the complete machine-readable object, including the research bundle, evidence sources/items, review themes, pain-solution map, garment signatures, three-view hashes/references, `human_identity_pixels_absent: true`, claims registry, claim-proof plan, selected `market_prompt_profile_id`, all seven `generation_controls`, the complete `zibuyu_three_layer_deadlines_v1` object, selling-point action-proof rows, structured hand/posture plans, per-beat `spoken_language` and `proof_endpoint`, `voiceover_review`, script/B-roll/prompt projections, `canonical_prompt_v7`, reference IDs, `three_layer_deadlines_sha256`, `quality_plan_sha256`, `research_bundle_sha256`, `market_prompt_profile_sha256`, `generation_controls_sha256`, `voiceover_review_sha256`, `canonical_beats_sha256`, and prompt SHA-256. Plan one continuous creator social intention, endpoint/task/prop handoffs, and a recognition-to-proof-to-conviction performance arc, then compile their visible consequences through the existing camera/action/hand/expression fields. The human-readable numbered sections are projections of that object, not separate drafts. Persist and validate the object before PopBoom.

For every new compile, use schema `1.4`, `zibuyu_ugc_quality_v4`, `zibuyu_market_prompt_v1`, `zibuyu_three_layer_deadlines_v1`, and `canonical_prompt_v7`. Materialize the Amazon evidence contract, market profile, generation controls, quality plan, and all three deadline layers: directing intent, directorial voice, garment-first fidelity allocation, exact-ASIN/variant provenance, review sample scope, conflicts, pain-to-solution bindings, claim-to-evidence-to-part-to-camera-to-action-to-endpoint-to-line bindings, economized elements, continuity anchors, exact identity-first interface tags, zero-human-pixel apparel-reference attestations, complete expanded transfer/non-transfer sets, global capture locks, persistent subject/garment/scene/light/sound locks, camera setup IDs, per-beat motion/hand/posture/physics/anatomy/garment-state/continuity locks, speech mode/mouth visibility, `spoken_language`, `proof_endpoint`, and voiceover review. Compile only compact v7 reference locks, deadline projections, market control, and motion-rich shot prose; do not send raw corpus prompts/statistics, raw reviews, Chinese review translations, internal rationale, hashes, IDs, or JSON to PopBoom. Historical v6/v5 remains read/poll/report-only, except an explicitly approved byte-identical `legacy_v5_exact_resume`; any `record_id` is never migrated or resubmitted.

## 完整输出顺序

### 1. 制作参数确认表

Use a compact table:

| 项目 | 确认内容 | 依据/备注 |
|---|---|---|
| 投放国家/地区 |  |  |
| 发布平台 |  |  |
| 口播语言 |  |  |
| 市场提示词配置 | `us_champion_v1` 或 `de_champion_v1` | 必须与市场、语言和固定模特一致 |
| Prompt shell |  | `us_technical_shell` / `us_compact_storyboard` / `de_performance_script` |
| Delivery mode |  | `us_hybrid_share` / `de_live_simple` / `de_hybrid_proof` |
| Camera mode |  | `fixed_phone` / `creator_handheld` / `friend_handheld` |
| Music mode |  | `none` / `low_non_lyrical`，二选一 |
| 结论 / CTA |  | `verdict_mode` + `commerce_cta_mode` |
| 屏幕文字 | `fixed_model_stats_only` | 固定模特尺寸条为唯一例外 |
| 视频形式 |  |  |
| 达人设定 |  |  |
| 视频时长 |  |  |
| 画幅 | 9:16 | 默认短视频竖屏 |
| 素材引用 | `Reference 1 / @Image1 = fixed-model identity only；Reference 2 / @Image2 = garment identity only` | 固定模特在前；服装从 @Image2 开始，原样保留已验证标签 |

If assumptions are used, label them as `工作假设`.

### 2. 产品与受众判断

Include:

- 产品一句话定位
- 市场先选结果：`market_prompt_contract_id`、`market_prompt_profile_id` 与完整 `generation_controls`，并注明美区/德区不是互译关系
- 单一导演/转化意图与导演语气
- 单一连续达人分享意图，以及“共鸣/结果 Hook → 动态证明 → 靠近细节确认”的三段表演与情绪递进
- 主要生成预算：服装身份一致性；次要预算只能选可见证明、口型同步、自然动作或场景可读性之一。该优先级不能删掉所有卖点所需的基础展示动作
- 明确降配项：人群、复杂运镜、复杂舞蹈、多手并行动作、忙乱道具或生成文字等
- 三视图参考的“必须继承 / 禁止继承 / 正向替代”角色契约
- 三视图零人体身份像素审计结果；必须为 `human_identity_pixels_absent: true`
- 可见卖点
- Amazon当前子ASIN/选中变体与抓取状态；若无链接则明确 `not_provided`
- 客观属性表、卖家声明表、直接评价样本说明、买家图是否实际目检
- 实际读过的评价数、评价变体范围、正向/负向/混合主题；不得用页面总评分数冒充样本数
- 冲突项与不可用/排除来源，例如护理冲突、不同ASIN、AI摘要、第三方摘要
- 目标人群
- 使用场景
- 购买动机
- 可能顾虑
- 结构化“痛点 → 直接证据/主题 → 独立解决卖点 → claim/evidence ID → 服装部位 → 特写景别 → 展示动作 → 双手轨迹 → `proof_endpoint` → 同一目标的市场语言口播”表
- 不可确认/不可乱写的卖点

### 3. 虚拟达人拟定

If no fixed creator is provided, design:

- 年龄段
- 性别/气质
- 所在场景
- 穿搭和妆发
- 说话风格
- 为什么这个达人适合该产品和市场

### 4. 达人图提示词

Write a creator reference image prompt in Chinese. Include:

- 画幅：9:16
- 人物外观
- 穿搭
- 场景
- 光线
- 表情/姿态
- 拍摄质感
- 禁止项

Do not request a real public figure or celebrity likeness.

### 5. 产品图提示词

Write a product reference image or product hero-frame prompt in Chinese. Include:

- product appearance based on user evidence
- material/texture/color
- table/hand/body context
- no invented accessories
- no altered logo or fake packaging
- consistency constraints

For an apparel three-view, override any hand/body-context suggestion: use invisible ghost-mannequin support or a featureless neutral non-skin-colored support, and require zero real-human skin, neck/chest/collarbone, arms, hands, fingers, nails, tattoos, jewelry, face, hair, or person-specific body cues. Do not pass it downstream unless `human_identity_pixels_absent: true`.

### 6. 10条Hook

Provide 10 hooks after the market profile is locked. Use the chosen market language. When the planning language is Chinese, provide:

`中文意图 + exact market-language hook`

US hooks follow `us_champion_v1` and natural American friend-to-friend phrasing. German hooks follow `de_champion_v1`, stay plain-spoken German, and do not contain Chinese or high-confidence English template residue. Do not write one market's hooks first and translate them into the other.

Mix hook types:

- 痛点型
- 反常识型
- 结果先行型
- 评论回复型
- 测评型
- 对比型
- 开箱型
- 情绪共鸣型
- 懒人方案型
- 轻 CTA 型

评论回复型、测评型和第一人称体验型只能在直接评价、真实测试或用户明确授权存在时使用。不得让模特伪称买过、穿过或测试过；卖家文案、AI摘要和搜索片段不能生成“买家都说”的 hook。

当有效直评主题显示“普通光滑T恤预期”与精确变体可见细针织纹理之间存在落差时，10条 Hook 中应包含一个 `evidence_backed_contrast` 否定式选项，例如“这可不是普通的光滑T恤——近看这个细针织纹理。”目标市场文案必须自然改写；紧接的下一拍必须是对应纹理特写、画外音和一次 `pinch_release`，不得用于品类推断或未经支持的触感/性能承诺。

### 7. 口播脚本

Provide one primary script and optionally one alternate. Match the chosen market language.

Immediately after the primary script, include the `voiceover_review` table for the user. For every beat, show `beat_id`, timestamp, `speech_mode`, `spoken_language`, exact market-language `market_line`, Chinese `zh_cn_translation`, `silence_reason`, and `proof_endpoint` when applicable. For purposeful silence, use `spoken_language: null`, explain the silence, and leave the translation null. The exact market line and silence reason must equal the canonical beat. This table is review-only, is bound by `voiceover_review_sha256`, and must stay outside the PopBoom paste-ready prompt, generated audio, screen text, and market caption.

Structure for a default 15-second clip:

- 0-4秒“共鸣/结果 Hook”：产品首帧可见；达人像向朋友分享一样说出需求或穿着结果，并因展示全身版型而自然后退/调整距离
- 4-10秒“动态证明”：保持同一分享意图和拍摄手/道具状态，在两到三个内部节拍中完成结构、垂坠、侧后或动作证明
- 10-15秒“靠近细节确认”：如有道具先明确放下/交接/切镜，再因需要看清决定性细节而走近或重构图；用所选 `verdict_mode` 收口，仅在 `commerce_cta_mode` 要求时加入真人 CTA

The three macro phases are the viewer-perceived performance layer. Inside them, still use five to seven sequential internal beats and three to four contiguous camera-setup runs. Every garment/styling claim requires its own proof binding. Multiple actions are required across the clip, but each internal beat performs only one main action and completes it before the next. Carry the previous endpoint, gaze, weight, filming/task hand, prop ownership, and garment state into the next start whenever possible. A static hold may occupy only the readable tail of an action, never the entire explanatory beat.

Keep the script natural. Avoid corporate ad language.

Write the voiceover for the mouth, not for a listing page. Use short everyday sentences, vary proof/styling openings, and remove official transitions or brochure constructions such as `this garment features`, `furthermore`, German `dieses Kleidungsstück verfügt über`, `darüber hinaus`, or Chinese `本产品采用`, `该服装具备`, `此外`. Remove standalone generic praise such as `looks nice`, `easy to style`, German `sieht gut aus` / `leicht zu kombinieren`, and Chinese `很好看` / `很百搭`. The same opener may not appear in three or more non-CTA proof/styling lines.

For richer product explanation, first select one `core_sellable_wearing_result`: the buyer-relevant wearing outcome most likely to drive purchase and visibly provable in 15 seconds. Start from the customer's real pain, wearing anxiety, or purchase hesitation before naming garment features; visible attributes are evidence, not final selling points. The script must prove that result through multiple angles such as visible result, structural reason, alternate angle or buyer-worry proof, motion/handling endpoint, and concrete outfit/use-case. Each spoken unit must name one exact buyer outcome plus one exact target and one visible behavior, position, construction fact, or pairing relationship. Keep one claim per beat. Remove true-but-disconnected details instead of averaging attention across all features.

For 15 seconds, normally use five to six short spoken units as one connected recommendation. `us_hybrid_share` may use concise visible American-English Hook/close anchors; Germany uses `de_live_simple` only when every low-load gate passes, otherwise default `de_hybrid_proof` with live German Hook/close and the same creator's German off-screen middle proof. Use no more than two visible-lip lines by default. When `secondary_fidelity_spend: lip_sync`, allow up to three visible anchors—one per macro phase—only with locked/subtle camera, simple-or-lower performer load, one active demonstration hand, at most 14 words per line, 36 visible words total, and an explicit expression cue. Every claim-proof action gets one matching concise line; use off-screen voiceover with the mouth out of frame for complex detail and motion proof.

Bind every timeline beat to a `camera_setup_id`. A default 15-second video uses three to four contiguous setup runs. Every beat needs one main action, one visible endpoint, `proof_endpoint` for proof, `spoken_language`, motivated camera/light, explicit `speech_mode`, `mouth_visibility`, motion budget, proof binding, relaxed posture, and complete left/right hand start-action-end states. Prefer off-screen voiceover with the mouth out of frame for detail-proof or controlled-motion beats. The same setup cannot be both fixed and handheld; `music_mode` is exactly `none` or `low_non_lyrical`.

If the creator/model appears on camera and speaks, the script must require mouth/lip movements to match the spoken words and bind one meaning-specific micro-expression to the line. Specify connected conversational delivery with varied emphasis and short proof-pivot pauses. For a 15-second American-English creator-led mirror share, hard-audit the validator's 40-62-word safe range and audition roughly 50-60 words at 200-235 WPM only when clear; use language-specific pacing elsewhere. Direct the emotional rise from warm recognition to animated certainty to delighted conviction/relief, not one flat energy level.

For German, show the speaker/voice/lip-sync lock before scene and shot prose. Audit `hier`, `so`, and `genau da` against the exact visible target. German language localization is stronger than forced geographic décor: do not add landmarks, flags, or stereotyped signs merely to signal Germany. Use an ownership/soft verdict or a brief `unten links` light-link close according to `generation_controls`; price and discount are not defaults.

If the user did not specify video sound/visual format, state that the format was selected by the agent based on recent-half-month TikTok apparel-commerce research and product fit. Explain why the selected format fits the exact garment: category, silhouette, material/texture, color, visible details, styling scene, and buyer motivation. If the user did not specify platform or duration, use TikTok and 15 seconds without asking follow-up questions.

### 8. B-roll镜头清单

Use a numbered list with:

- shot name
- duration
- camera direction
- product/creator action
- proof target, human motive, active hand, persistent task/prop anchor, prior endpoint/start handoff, motion path, visible endpoint, matching `proof_endpoint`, and natural continuation or intentional cut
- one or two realistic micro-cues when face/body is visible
- purpose

Include tactile close-ups, product movement, hand interaction, packaging or page evidence if available, and final hero shot.

### 9. Seedance即可直接粘贴的视频生成提示词

Output a copy-ready prompt. It must include:

- exact interface-tag reference mapping
- exact deterministic first lines `Reference 1 / @Image1 = fixed-model identity only` and `Reference 2 / @Image2 = garment identity only`
- selected `market_prompt_profile_id` consequences and market language/performance lock immediately after the reference lines; Germany locks the German speaker, voice ownership, and lip-sync mode here
- exact `prompt_shell_mode`, `delivery_mode`, `camera_mode`, `music_mode`, `verdict_mode`, `commerce_cta_mode`, and `screen_text_policy` consequences without exposing internal IDs
- `human_identity_pixels_absent: true` plus the complete prohibition on transferring reference skin tone, ethnicity, neck/chest/collarbone, shoulders, arms/hands/fingers/nails, tattoos, jewelry, body shape, face, or hair
- 9:16 ratio
- exact duration
- UGC phone-camera style
- timestamped shot plan
- creator action
- exact single proof target, canonical `product_part_id`, exact allowlisted `spoken_claim_ids` binding, claim-matched framing, positive demonstration action, matching active-hand side, visible result, `proof_endpoint`, `spoken_language`, and same-target canonical line for every product claim
- left/right hand start anchors inherited from the prior endpoint when possible, one motivated action path, left/right hand end anchors, phone/prop ownership, readable endpoint, natural handoff/cut, and sparse line-matched micro-cues for every selling-point proof beat
- product consistency constraints
- voice/sound direction
- lip-sync requirement when a visible speaker is present
- normal conversational speech speed
- casual everyday buyer-friendly voiceover wording
- varied conversational sentence openings with no brochure/official constructions or standalone generic praise
- one selected core sellable wearing result, proven through front/detail/side-back/action/styling angles, each limited to one exact target and visible behavior/position/styling relationship
- one compact hard-constraint slot plus at most one observed-failure-specific repair constraint
- the concise garment-only `reference_contract_text`, including the positive replacement of one creator in one continuous real-world scene rather than the source three-panel/white-background layout
- exactly three viewer-facing macro-phase blocks from `canonical_prompt_v7`—recognition/result Hook, animated proof, and close/detail conviction—while retaining every internal beat, market-language canonical line, action, hand plan, endpoint, and beat-specific shot deadline; keep internal IDs/JSON, raw corpus material, Chinese review translations, and hashes out of the generation text
- machine-only `market_prompt_profile_sha256`, `generation_controls_sha256`, and `voiceover_review_sha256` bindings outside the copy-ready prose

### 10. 成片检查清单

Include script and prompt pre-submission checks:

- 是否只有一个明确导演/转化意图，所有镜头、光线、表演和声音都服务于该意图与核心证明
- 是否先锁定 `zibuyu_market_prompt_v1` 与正确的 `us_champion_v1` / `de_champion_v1`，再选择创意形式；市场、模特、语言和七个 `generation_controls` 是否一致
- 是否只有一个连续的达人分享意图，并把五到七个内部节拍组织成“共鸣/结果 Hook → 动态证明 → 靠近细节确认”，而不是相互断开的等能量产品动作
- 是否逐字保留 `Reference 1 / @Image1 = fixed-model identity only` 与 `Reference 2 / @Image2 = garment identity only`，没有把服装放入 @Image1
- 三视图是否通过 `human_identity_pixels_absent: true`，只控制服装身份，且明确不继承肤色、种族、颈胸/锁骨、肩臂、手/手指/指甲、纹身、首饰、体型、脸发、白底、三栏布局、分隔线、重复人物、姿势、场景、镜头或文字
- 15秒是否有五到七个顺序节拍、三到四个连续合理机位 run、至少四个非CTA商品/穿搭动作和至少三个不同的部位/效果证明，而不是重复同一部位
- 每条卖点口播是否用唯一 `spoken_claim_ids` 和原样 `evidence_ids` 精确绑定一个被介绍部位、对应特写/侧面/全身景别、正向完成的证明动作、双手轨迹、可见结果和相同 `proof_endpoint`
- Amazon URL、父/子ASIN、选中变体、抓取状态、直接评价样本与买家图目检状态是否如实记录；同父款评论是否明确限定范围
- 卖家文案、直接评价、买家图片、Amazon AI摘要和第三方摘要是否保持分层；是否禁止摘要/图片证明柔软、弹力、成分、透气、舒适或护理
- 评价痛点是否绑定有效 `pain_point_id`，并由独立产品证据支持的解决 claim 回应；风险项是否没有被硬写成促单承诺
- 护理、尺码、材质或弹力冲突是否保持 conflicted/mixed 并从绝对话术中删除
- 是否不存在整段 `stands still`、`holds still`、`continues pose` 或仅靠相机推近却没有商品动作的讲解拍
- 每个节拍是否只有一个主动作且动作按顺序完成；可见口型和 CTA 是否最多一只主动手；细节双手动作是否有明确证据、同一目标和完整左右手计划
- 上一拍终点是否自然成为下一拍起点；自拍手机手是否持续占用，道具是否直到明确放下/交接/切镜才离开；是否避免双手反复回胯、立正和无动机远近移动
- `quality_plan_sha256`、`research_bundle_sha256`、`market_prompt_profile_sha256`、`generation_controls_sha256`、`voiceover_review_sha256` 与 `canonical_beats_sha256` 是否匹配内部对象；`reference_contract_text` 是否逐字进入最终 `compiled_text`
- 产品是否前2秒出现
- 卖点是否只有一个清晰主线
- 每个 spoken beat 的 `spoken_language` 是否符合投放市场；`voiceover_review` 是否逐beat包含完全相同的市场原文和中文审稿翻译，且中文没有进入 prompt/音频/屏幕/市场 Caption
- 美区是否使用 `us_hybrid_share`；德区是否只使用合格的 `de_live_simple` 或默认 `de_hybrid_proof`，德语槽是否没有中文/高置信英语模板残留，`hier/so/genau da` 是否指向当前可见部位
- 有口播且人物出镜说话时，口型是否与口播内容逐句一致
- 美式英语15秒达人分享是否通过40-62词硬范围，并在清晰时试听约50-60词、200-235 WPM；其他语言是否按本地口语节奏试听
- 口播是否日常口语化，像真实买家/达人自然分享，避免生硬广告腔
- 口播是否已删除书面/官方转折、产品手册句式和单独的“很好看/很百搭”式空泛赞美；非CTA证明/穿搭句是否没有连续三次使用同一开头
- 在证据允许时，是否用四个具体信息角度覆盖主视觉细节、第二部位/遮挡细节、版型/垂坠/侧后动态和具体穿搭关系；证据不足时是否用穿搭/场景补位而没有编造第四个卖点
- 产品外观、颜色、版型、材质观感、细节是否在所有镜头中保持一致
- 达人是否像真实 UGC 创作者，有自然微表情、真实光影和合理动作
- 表情与声音是否从温暖共鸣、明亮确信递进到愉悦确信/释然，多条可见口播是否至少有两种语义不同的表情状态
- 达人是否保持放松三分之四站姿、自然重心或与证明相符的姿态，而不是正面立正站桩
- 是否只有证据支持时才通过拉伸、回弹或类似动作暗示弹力/性能
- 若属性为 `Low Stretch`，是否明确禁止强拉伸展示，并改用轻捏、轻放、垂坠或结构特写
- 是否避免乱写认证、疗效、数据、折扣、销量、物流或评价
- 是否适合平台节奏
- 是否有可剪成前3秒预览的 hook
- 结尾是否先给所选可信 verdict；`commerce_cta_mode: none` 是否没有链接动作，`light_link` 是否只允许真实人物做一次小幅左下动作；是否不出现购物车、箭头、箭头 emoji、贴纸、图标、表情包、浮层或商品链接徽章，且价格/折扣不是默认
- 同一 camera setup 是否没有固定/手持冲突，`music_mode` 是否只选 `none` 或 `low_non_lyrical`，屏幕是否只有 `fixed_model_stats_only` 例外
- 是否没有三只手、额外手臂、缺手、畸形手指、漂浮手指、肢体合并、人物与场景错位、产品与身体错位

Add this check before submission: every selected selling point must pass `selling-point-action-proof-methodology.md`: one claim, one proof target, one matched frame, one action path, one visible endpoint plus matching `proof_endpoint`, one concise same-target market-language spoken line, explicit hand anchors, and micro-cues that support rather than replace proof.

When an actual rendered video is available, append a separate rendered-take result: `keep`, `fix_in_post`, `reroll`, `rewrite`, or `stop_or_rescope`, plus one `primary_failure_variable` and concise evidence. If the video cannot be inspected, use `unverified`. A PopBoom `succeeded` status is task completion, not a creative-quality verdict.

### 11. 下一步操作

Suggest concrete next steps:

- 让用户选 1 条 hook
- 生成首帧图
- 生成达人参考图
- 先记录固定模特为 `Reference 1 / @Image1`，再准备通过零人体像素审计的服装三视图并原样记录为 `Reference 2 / @Image2+`
- 进入 Seedance 生成
- 根据首版结果做二次提示词微调

### 12. 促单文案与热门标签

Output one single copy-ready paragraph only. The paragraph must combine:

- the concise conversion caption;
- exactly 5 highly relevant popular hashtags.

Do not split the caption and hashtags into separate bullets, tables, or explanation blocks. The user should be able to copy the whole paragraph directly into TikTok.

Language rule:

- The caption and all 5 hashtags must match the confirmed target country/region and voiceover language.
- For the US market, write the paragraph in natural American English.
- For China, write in Chinese; for Germany, write in German; for France, write in French; for other markets, use the local common ad language or the confirmed voiceover language.
- Do not mix Chinese planning text into the final paragraph unless the confirmed market/language is Chinese.

Content rules:

- Always use recent-half-month TikTok apparel-commerce learning when this skill is invoked for apparel/ecommerce planning. Apply it to video sound/visual format choice, the script, and this caption/hashtag section.
- The format, caption, and tags must fit the exact garment rather than only copying a popular generic clothing format.
- Prefer conversion-oriented, TikTok-native captions: short, specific, product-visible, low-pressure CTA, no unsupported discounts, ratings, sales volume, shipping promises, or claims.
- Persist `caption_claim_ids`, `caption_pain_point_ids`, `caption_claim_bindings`, and `caption_pain_bindings`. Every binding must point to a registered target-language anchor that occurs in the final caption; IDs alone are insufficient. The caption may use only allowlisted claims and selected script-eligible pain points; it may not introduce a new Amazon/review assertion.
- Hashtags must balance high relevance and discoverability. Do not use unrelated viral tags just because they are popular.

## Tone

Default to Chinese planning. Be direct and production-oriented. Avoid fluffy marketing analysis.
