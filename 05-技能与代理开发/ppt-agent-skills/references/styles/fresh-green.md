# 清新自然 (fresh_green) -- "有机"感

> 适用场景：环保/可持续发展、健康/医疗/养生、食品/农业、美妆/护肤
## style.json 参考

```json
{
  "style_id": "fresh_green",
  "style_name": "清新自然 (Fresh Green)",
  "mood_keywords": ["晨光透叶", "清泉细流", "有机呼吸", "温柔生长", "圆润柔和"],
  "design_soul": "晨光透过嫩叶的筛孔，在白色桌面上投射出薄荷色的光斑 -- 空气里弥漫着泥土和新芽的味道，所有的线条都是圆弧的，所有的色彩都是温暖的低饱和度",
  "variation_strategy": "数据页用白底+绿色波浪分隔线+叶片小装饰制造有机田园感（温柔信息），概念讲解页用大面积极浅绿背景+圆润大卡片+大号留白（呼吸），案例页用卡片内嵌配图+自然光晕渐变（生活场景），通过'密集圆润卡片'和'极简留白+单一叶片'交替形成'在花园中漫步'的从容节奏",
  "decoration_dna": {
    "signature_move": "波浪分隔线（SVG path 波浪替代直线）-- 自然界没有直线",
    "forbidden": ["直角元素", "深色阴影", "冷色调装饰", "网格点阵", "角标装饰线"],
    "recommended_combos": [
      "叶片装饰 + 波浪分隔线 + 圆润大圆角卡片",
      "极浅绿背景渐变 + 叶片编号 + 渐隐分隔线"
    ]
  },
  "css_variables": {
    "bg_primary": "#F0FDF4",
    "bg_secondary": "#ECFDF5",
    "card_bg_from": "#FFFFFF",
    "card_bg_to": "#F0FDF4",
    "card_border": "rgba(22,163,74,0.12)",
    "card_radius": "16px",
    "text_primary": "#14532D",
    "text_secondary": "#4B5563",
    "accent_1": "#16A34A",
    "accent_2": "#059669",
    "accent_3": "#F59E0B",
    "accent_4": "#D97706"
  },
  "font_family": "PingFang SC, Microsoft YaHei, system-ui, sans-serif"
}
```

## 装饰 DNA -- "有机"感

| 装饰元素 | 实现方式 |
|---------|---------|
| 叶片装饰 | 内联 SVG 叶片（`path d="M12 22c4-4 8-8 8-12A8 8 0 0 0 4 10c0 4 4 8 8 12z"`），放在页面角落或卡片装饰 |
| 波浪分隔线 | SVG path 波浪线替代直线分隔（`path d="M0,8 Q30,0 60,8 Q90,16 120,8" stroke="#16A34A" stroke-width="1.5" fill="none"`） |
| 卡片边框 | 圆角加大到 16px，边框用绿色低透明度 |
| 编号样式 | 叶片形状内的数字（用 SVG 叶片 + HTML 叠加数字） |
| 背景渐变 | 极浅的绿到白渐变（`linear-gradient(180deg, #F0FDF4, #FFFFFF)`） |
| 禁止 | 禁止直角、禁止深色阴影、禁止冷色调元素 |
| 整体感觉 | 阳光透过树叶的温暖感，圆润柔和，适合健康/环保主题 |
