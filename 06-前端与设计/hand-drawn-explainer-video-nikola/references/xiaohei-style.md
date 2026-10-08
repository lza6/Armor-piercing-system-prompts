# 小黑风格

这个视觉预设借鉴 Ian 开源的 [ian-xiaohei-illustrations](https://github.com/helloianneo/ian-xiaohei-illustrations)，感谢原作者公开小黑视觉 IP、风格规范与配图工作流。本仓库只是把这种视觉语言适配到逐笔故事动画，不主张原创，统一称为“小黑风格”。

它适合解释抽象方法、系统关系、工作状态和反常识观点；不适合要求准确肖像的真实人物传记。

调用时直接说“使用小黑风格”，并另行指定画面结构（单场景、多幕故事或左右双语义岛）。这是逐笔故事路线中的视觉风格，不是第三条制作路线；它也不强制使用双语义岛。

## 视觉 DNA

- 纯白或极浅暖白背景，至少约 35% 自然留白；
- 黑色略抖的手绘线，密度低，不使用铅笔噪点、密集排线或厚重阴影；
- 主体是黑色小生物，白色点眼、简洁四肢、表情克制；
- 用一个荒诞但能解释观点的核心动作，不做素材拼贴；
- 红、橙、蓝只作极少量箭头、圈画、贴纸或强调；
- 图片模型不写长中文。准确关键词、数字和标题在后期用确定性文字层生成。

## 生图提示词骨架

```text
16:9 minimalist hand-drawn editorial illustration on a pure white background.
One small black creature with tiny white dot eyes is [核心动作].
Use sparse slightly wobbly black ink lines, large clean negative space,
and only a few red, orange, and blue annotation accents.
The action must clearly express [概念/冲突]. Low line density, flat shapes,
no long text, no pseudo-writing, no photorealism, no glossy 3D, no dense texture.
Keep the lower 18% clear for subtitles and preserve a 6%-10% vertical gutter
when the scene contains left and right semantic islands.
```

## 逐笔动画适配

- 一个区域只承担一个动作或结论；双岛画面先左后右，岛间不得有跨区背景线。
- 黑色实心面积过大时，轮廓后补色会显得突兀，应缩小主体或拆分区域。
- 关键词在区域主要轮廓完成后出现，不能提前泄露结论。
- 小黑本体不能用于替代真实人物肖像；真实人物采用自然肤色 Q 版规范。

## 避免

- 可爱儿童绘本、商业扁平插画、PPT 图标阵列；
- 黑脸或全黑人体去表现真实人物；
- 满屏便利贴、分栏卡片和装饰线；
- 依赖文字才能理解的构图。
