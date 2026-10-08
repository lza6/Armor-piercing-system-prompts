> 范围：本文件服务于原仓库的生图/Flow 提示词模式。程序动画模式以 SKILL.md、DESIGN.md 和用户已确认样片为准；不要套用这里的禁标签卡、静止小关键词规则。

# Q 版蜡笔（Q Doodle）风格与运动参考

这是可选的文字风格规范，当前未附示例图或已验证动画工程。只有选用本风格时，才沿用下面的配色、笔触和运动约束；用户指定新参考图时按新图调整相应视觉约束，不混入相互冲突的固定风格词。如何选择和保存风格见 [style-selection-and-updates.md](style-selection-and-updates.md)。

## 视觉签名

每条生图提示词都重复以下核心约束，避免批次漂移：

```text
modern Q-version hand-drawn crayon illustration, vertical 9:16, solid warm-white canvas background, exact base color #F8F6EF, only extremely subtle low-contrast paper grain, thick imperfect black hand-drawn marker/crayon outlines, bold flat wax-crayon blocks, sunflower yellow, saturated cobalt blue and vivid tomato red with only a small muted-green accent, natural restrained Q-version proportions, simple hand-drawn composition, 2–4 large readable visual groups, generous breathing room, no glossy rendering, no realistic lighting, no logos, no watermark, no photorealism, no 3D
```

## 固定背景色

所有静帧和视频尾帧使用同一背景约束：

```text
Use a flat, uniform warm-white canvas background with exact base color #F8F6EF. Preserve the same background color across every shot. Add only extremely subtle low-contrast paper grain. No black background, no gray-brown tint, no yellow-beige color cast, no gradient, no vignette, no colored paper, no border shadow.
```

Flow 的自然语言提示不能保证真正的像素级色值。若必须完全一致，使用一张 `#F8F6EF` 的固定 First Frame，或在生成后把画布背景统一替换为该色号。

## 中文关键词模式

当用户希望关键词直接出现在卡片或视频画面中，使用短词而非长句，并指定具体文字、位置和安全区：

```text
Show the exact Chinese keyword “好奇心” as small, clean, high-contrast hand-drawn lettering directly on the warm-white illustration paper near the upper-left margin, not inside a card, label, sticker, note or text box. Keep it perfectly legible and unchanged throughout the shot, and do not add any other text.
```

文字优先直接写在暖白纸面、留白处或相关物件旁，禁止使用便签纸、标签卡、贴纸、圆角文字框、标题面板、独立白色卡片或底部字幕框。禁止放在底部字幕区，避免放在人物脸部、快速运动物件或复杂背景上。文字块不超过画面高度的 10%–12%，以“小而看得见”为目标。动态图中关键词本身必须保持静止、清晰、不变形。Flow 仍可能生成错别字，因此 2–8 个汉字是默认上限；公司名、金额、日期和长句应提供无文字安全版备用。

人物头部可略大、身体略小，但不能巨头小身、凸眼或儿童卡通。人物用发型、眼镜、服装、姿势和道具做身份锚点，不要求写实肖像。物件允许比人物夸张，笑点来自动作、比例与物件关系。背景保持干净，不加复杂房间、赛博 HUD、复古报刊、泛黄旧纸、棕灰低饱和、细密交叉排线或假文字。

## 构图规则

- 画面为 9:16 竖版，主体组通常占画面宽度 60%–70%，顶部和左右保留小型关键词安全区，底部预留字幕区且不放大字。
- 每镜头使用 2–4 个主要视觉组；手机端一秒可读。避免过高的多层堆叠塔、过多小人物、复杂机械细节和海报式信息堆叠。
- 用明确的左/中/右、前/后、上/下关系写提示词，不只写抽象情绪。
- 真实 Logo、公司名、人物名、日期、金额和长句优先由后期层处理；短中文关键词可直接手写在画面纸面、留白处或相关物件旁。无文字版本保留干净留白，不生成便签纸、标签卡或文字容器。

## 真实实体锚点

生成涉及真实实体的画面时，使用以下规则：

```text
For an exact national flag, preserve the official country name, flag colors, geometry, emblem placement and aspect ratio. Do not substitute a similar flag, random flag or abstract color pattern.
For a real company, use its industry, products, hardware and restrained brand-color cues. Do not invent or deform the exact logo unless an approved reference image is supplied.
For a famous person, use a clearly stylized non-photorealistic chibi caricature with public identity cues such as hairstyle, glasses, clothing and posture. Do not bypass safety restrictions or attempt a photorealistic face clone.
```

如果国旗或 Logo 是镜头的核心信息，提示词必须建议使用参考图或后期叠加原始素材；生成模型只负责周围的 Q 版构图和动作。实体准确性不能靠“更详细的形容词”保证。

## Flow 图生视频运动语法

默认提示词可按以下顺序组织：

1. `Start from the supplied completely blank #F8F6EF warm-white paper frame.`
2. 保持 `locked flat frontal camera`，先让基础物件滑入或以不规则蜡笔线出现。
3. 人物和次要物件作为 `rigid flat paper cutouts` 依次弹入并轻微 settling bounce。
4. 关系元素最后出现：例如放大镜对准电表、砝码压住云、硬币接近嘴、手按下开关。
5. 最后约 0.8 秒 `settle into and hold the exact supplied final illustration`。

统一写明：`tactile 10–12 fps paper stop-motion`、无镜头移动/缩放/视差、无液体变形、无口型、无新增角色、无新增 Logo、无水印、无声音；启用中文关键词时写明关键词直接留在纸面或物件旁、固定、清晰、不变形、不新增其他文字或文字容器。

Flow 使用 First + last 时：First Frame 是完全空的暖白纸，Last Frame 是对应编号完成静帧；提示词必须明确“最终构图与风格参考”。如果只能使用单张完成图，改成：

```text
Animate the supplied completed illustration in place for about 5 seconds while preserving every drawn shape and the exact composition. Use only restrained paper-cutout micro-motion: one or two small object bounces, a short hinge-like hand movement, subtle wheel rotation or one accent appearing. End on the unchanged supplied illustration and hold it still for the final second.
```

## 视觉隐喻速查

| 口播关系 | 优先隐喻 |
|---|---|
| 竞争/跑输 | 赛跑、追逐、拔河 |
| 成本/吞噬 | 漏水、重物下压、吞金兽、计价器 |
| 验证/财报 | 考试、试金石、体检、放大镜 |
| 选择/结论 | 岔路、天平、开关、两扇门 |
| 风险/压力 | 裂缝、警报、摇晃积木、倒计时 |
