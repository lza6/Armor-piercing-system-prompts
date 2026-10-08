# image_hero（大图+叠加文字块）-- 画面的沉浸

> 适用数据类型：image_candidates。全幅图片+叠加文字，制造情感冲击。
> 结构：需指定 image.usage(hero-background/inline-illustration) + image.placement(full-bleed/left-half) + image.content_description。
> 推荐 card_style：transparent/glass（大图不需边框束缚，或毛玻璃让文字浮在图上）。
> 适用页面类型：cover / section 等氛围页。一页最多1个 image_hero。

## JSON 结构
```json
{
  "card_type": "image_hero",
  "title": "核心标题（28-44px）",
  "subtitle": "补充说明（80字内，可选）",
  "image_prompt": "配图描述（用于生成配图）",
  "data_highlights": [{"value": "87%", "label": "市场渗透率"}]
}
```

## 设计灵魂

### 图像是主角，文字是浮游
- 配图 `object-fit:cover` 铺满区域 -- 图片就是画面本身，不是被"放在"某个位置的装饰
- 遮罩层用真实 `<div>` 半透明渐变（禁止 mask-image）-- 遮罩的目的是让文字在图上可读，而非遮住图片
- 文字层悬浮在图像之上，标题用大号字体保证冲击力，副标题和数据亮点用小号字体保持克制

### 灵动手法
- **渐隐融合**：图片从一侧（如右侧）向另一侧渐隐消失，文字在渐隐区域安身。图文不是分离的两层，而是融为一体的画面
- **底部涌现**：图片铺满上方 70%，文字从底部 30% 的渐变暗区中涌现。像电影海报的标题排版
- **角落低语**：图片铺满全域，文字极小地蜷缩在某个角落，让图像的叙事力量独占舞台

### 实现指引
- 图片可用 `<img>` 标签或 CSS `background-image`，按场景选择最佳方式
- 渐变遮罩方式不限：真实 div、`::before`/`::after`、`mask-image` 均可，选择最佳视觉效果
- 推荐 `transparent` card_style -- 大图不需要卡片边框束缚
