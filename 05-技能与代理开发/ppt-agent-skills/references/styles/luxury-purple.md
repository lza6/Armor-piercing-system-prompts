# 紫金奢华 (luxury_purple) -- "高定"感

> 适用场景：时尚/奢侈品、高端服务/地产、设计/创意行业、品牌发布会
## style.json 参考

```json
{
  "style_id": "luxury_purple",
  "style_name": "紫金奢华 (Luxury Purple)",
  "mood_keywords": ["天鹅绒深紫", "金箔碎光", "重力沉降", "高定质感", "每个细节都有重量"],
  "design_soul": "深紫的天鹅绒上散落着碎金箔 -- 不是金碧辉煌的暴发，而是隐匿在暗调中的贵族气息，每一个数据都像被金框裱起来的展品",
  "variation_strategy": "数据展示页用深紫底+金色数字+钻石切割线装饰（奢华阵列），品牌故事页用大面积紫色光晕+中央悬浮的金色标题（氛围），产品对比页用金色描边卡片+紫色渐变背景（精致），通过'紫色弥漫的深沉感'和'金色闪现的明亮点'交替制造'走进高定秀场 -- 灯光暗转亮'的节奏",
  "decoration_dna": {
    "signature_move": "金色描边圆圈编号（`border:2px solid #F59E0B; border-radius:50%`）-- 每个数字都像被金框裱起来",
    "forbidden": ["大面积白色/浅色区域", "直角方块编号", "自然/有机元素"],
    "recommended_combos": [
      "紫色光晕 + 金色点缀编号 + 钻石切割线底部装饰",
      "大号紫色水印 + 金色标题竖线 + 卡片强阴影悬浮"
    ]
  },
  "css_variables": {
    "bg_primary": "#120A2E",
    "bg_secondary": "#1A0B3D",
    "card_bg_from": "#2D1B69",
    "card_bg_to": "#1A0B3D",
    "card_border": "rgba(192,132,252,0.1)",
    "card_radius": "12px",
    "text_primary": "#F5F3FF",
    "text_secondary": "rgba(245,243,255,0.7)",
    "accent_1": "#A855F7",
    "accent_2": "#7C3AED",
    "accent_3": "#F59E0B",
    "accent_4": "#D97706"
  },
  "font_family": "PingFang SC, Microsoft YaHei, system-ui, sans-serif"
}
```

## 装饰 DNA -- "高定"感

| 装饰元素 | 实现方式 |
|---------|---------|
| 紫色光晕 | 大面积 radial-gradient，紫色 5% 透明度，比暗黑科技更弥漫 |
| 金色点缀 | 数据数字用 `--accent-3`（金色 #F59E0B），标签边框用金色 10% |
| 钻石切割线 | 菱形 SVG pattern 作为底部装饰条（45度旋转的小方块连续排列） |
| 卡片效果 | `box-shadow:0 4px 24px rgba(0,0,0,0.3)` 强阴影营造浮起效果 |
| 编号样式 | 金色描边圆圈内数字（`border:2px solid #F59E0B; border-radius:50%`） |
| 禁止 | 禁止大面积白色/浅色区域（破坏奢华暗调） |
| 整体感觉 | 深紫 + 金色的极致奢华，每个细节都有"重量感" |
