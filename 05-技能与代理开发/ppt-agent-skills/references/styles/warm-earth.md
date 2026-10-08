# 暖色大地 (warm_earth) -- "精品咖啡馆"感

> 适用场景：消费品/生活方式、创业路演、文化创意、餐饮/零售、人文社科
## style.json 参考

```json
{
  "style_id": "warm_earth",
  "style_name": "暖色大地 (Warm Earth)",
  "mood_keywords": ["精品咖啡馆", "午后暖阳", "手作质感", "奶油温柔", "慢节奏呼吸"],
  "design_soul": "午后的阳光透过咖啡馆木框窗洒在奶油色桌面上，手边一杯拿铁拉花还冒着热气 -- 所有的色彩都带着烘焙过的温暖，所有的线条都像手写字一样柔和不规矩",
  "variation_strategy": "内容页用奶油底+驼色卡片+圆润分隔线制造手作温暖感（舒适），数据页用深咖啡色底+金色数字锚点制造精品咖啡的浓缩感（醇厚），章节封面用大面积暖光渐变+极简一行文字（呼吸），通过'奶油色温柔铺叙'和'深咖啡浓缩聚焦'的交替制造'在咖啡馆翻阅精装杂志'的从容节奏",
  "decoration_dna": {
    "signature_move": "驼色圆角标签式编号（`border-radius:20px; background:rgba(180,140,100,0.12); padding:4px 16px`）-- 手作标签的温暖感",
    "forbidden": ["冷色调装饰(蓝/青/紫)", "直角尖锐元素", "网格点阵", "光晕效果"],
    "recommended_combos": [
      "圆角标签编号 + 暖色渐隐分隔线 + 手写风引用标记",
      "深咖啡底部色带 + 金色关键数字 + 大圆角卡片"
    ]
  },
  "css_variables": {
    "bg_primary": "#FDF8F0",
    "bg_secondary": "#FAF0E4",
    "card_bg_from": "#FFFFFF",
    "card_bg_to": "#FDF8F0",
    "card_border": "rgba(180,140,100,0.12)",
    "card_radius": "14px",
    "text_primary": "#3D2B1F",
    "text_secondary": "#7A6652",
    "accent_1": "#C4956A",
    "accent_2": "#A0785C",
    "accent_3": "#D4A853",
    "accent_4": "#B8922E"
  },
  "font_family": "PingFang SC, Microsoft YaHei, system-ui, sans-serif"
}
```

## 装饰 DNA -- "精品咖啡馆"感

| 装饰元素 | 实现方式 |
|---------|---------|
| 圆角标签编号 | 胶囊形背景块（`border-radius:20px; background:rgba(180,140,100,0.12); padding:4px 16px`）|
| 暖色分隔线 | 渐隐线（`linear-gradient(90deg, transparent, rgba(196,149,106,0.3), transparent)`）|
| 卡片边框 | 极浅驼色边框（`border:1px solid rgba(180,140,100,0.12)`）+ 微阴影（`box-shadow:0 2px 8px rgba(0,0,0,0.04)`）|
| 引用装饰 | 大号左引号（36px, accent 色 20% 透明度） |
| 数字强调 | 金色（`--accent-3: #D4A853`）用于关键数据 |
| 禁止 | 禁止冷色调装饰、禁止直角尖锐元素、禁止网格点阵 |
| 整体感觉 | 奶油色温暖底色 + 驼色/金色点缀，像午后在精品咖啡馆翻阅一本精装杂志 |
