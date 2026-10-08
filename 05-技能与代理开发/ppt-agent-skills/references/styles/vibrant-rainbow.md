# 活力彩虹 (vibrant_rainbow) -- "创意"感

> 适用场景：社交/娱乐平台、营销/推广材料、年轻品牌、创意方案
## style.json 参考

```json
{
  "style_id": "vibrant_rainbow",
  "style_name": "活力彩虹 (Vibrant Rainbow)",
  "mood_keywords": ["弹跳色块", "棱镜折射", "派对蜜液", "不设限的笑", "每张卡片都是糖果"],
  "design_soul": "阳光穿过棱镜在白墙上折射出一道彩虹 -- 每张卡片都像一颗不同颜色的糖果，整体看起来是一场视觉的派对，但白色背景永远保持清爽透气",
  "variation_strategy": "数据页用多色圆点标记+每张卡片不同accent色标题竖线（彩虹糖盒），概念讲解页用大面积白底+单一超大彩色渐变标题（极简聚焦），活动/案例页用多色标签云+圆润大圆角卡片阵列（活力满溢），通过'多色密集展示'和'纯白底+单一彩色焦点'交替制造'开箱彩蛋 -- 每页翻开都有惊喜色彩'的节奏",
  "decoration_dna": {
    "signature_move": "多色渐变标题下划线（`linear-gradient(90deg, #F97316, #EC4899, #8B5CF6, #06B6D4)`）-- 彩虹的浓缩",
    "forbidden": ["阴影", "暗色调元素", "严肃装饰线", "网格点阵"],
    "recommended_combos": [
      "多色渐变标题线 + 4色交替圆点列表 + 彩色标签边框",
      "品牌色块(多种accent色10%透明度) + 大圆角卡片 + 渐隐分隔线"
    ]
  },
  "css_variables": {
    "bg_primary": "#FFFFFF",
    "bg_secondary": "#FFF7ED",
    "card_bg_from": "#FFFFFF",
    "card_bg_to": "#FFF1E6",
    "card_border": "rgba(251,146,60,0.15)",
    "card_radius": "20px",
    "text_primary": "#1C1917",
    "text_secondary": "#57534E",
    "accent_1": "#F97316",
    "accent_2": "#EC4899",
    "accent_3": "#8B5CF6",
    "accent_4": "#06B6D4"
  },
  "font_family": "'PingFang SC', 'Microsoft YaHei', system-ui, sans-serif"
}
```

## 装饰 DNA -- "创意"感

| 装饰元素 | 实现方式 |
|---------|---------|
| 多彩渐变色块 | 每张卡片可使用不同 accent 色作为标题竖线/徽标颜色 |
| 圆润大圆角 | 卡片 `border-radius:20px`（比其他风格大一圈） |
| 彩色圆点标记 | 列表项用 4 种 accent 色交替的圆点（而非统一颜色） |
| 渐变标题 | 标题下划线用多色渐变（`linear-gradient(90deg, #F97316, #EC4899, #8B5CF6, #06B6D4)`） |
| 标签颜色 | tag_cloud 中每个标签随机使用不同 accent 色的边框 |
| 禁止 | 禁止阴影、禁止暗色调、禁止严肃的装饰线 |
| 整体感觉 | 明快、活泼、年轻，像 Instagram/TikTok 的品牌风格 |
