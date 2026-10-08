# 朱红宫墙 (royal_red) -- "国风"感

> 适用场景：文化/历史主题、政务汇报、品牌故事、中国风
## style.json 参考

```json
{
  "style_id": "royal_red",
  "style_name": "朱红宫墙 (Royal Red)",
  "mood_keywords": ["宫墙黄昏", "金缕暗纹", "厚重沉淀", "仪式庄严", "千年重量"],
  "design_soul": "故宫红墙上一缕金线划过黄昏，庄重中透出温暖的光 -- 每一处纹理都有千年的重量，金色不是奢华而是时间的包浆",
  "variation_strategy": "内容页用深朱红底+金色角饰+祥云纹理制造宫廷画卷感（庄重），数据页用金色关键数字+印章编号+极简布局（肃穆），章节封面用大面积朱红留白+金色PART编号+一句古诗/引言（仪式），通过'浓墨重彩的金红交织'和'极简朱红空旷+一点金'的交替制造'在故宫长廊中走走停停'的节奏",
  "decoration_dna": {
    "signature_move": "金色中式角纹（L形+回纹细节SVG）-- 故宫窗棂的精致",
    "forbidden": ["科技感元素(网格点阵/光晕/角标线)", "蓝色/紫色系颜色", "现代极简元素"],
    "recommended_combos": [
      "金色角饰 + 祥云纹理底部装饰 + 金色渐隐分隔线",
      "印章编号 + 金色标题竖线 + 大号低透明度汉字水印"
    ]
  },
  "css_variables": {
    "bg_primary": "#8B0000",
    "bg_secondary": "#5C0000",
    "card_bg_from": "#A52A2A",
    "card_bg_to": "#7A0000",
    "card_border": "rgba(255,215,0,0.15)",
    "card_radius": "8px",
    "text_primary": "#FFF8E7",
    "text_secondary": "rgba(255,248,231,0.75)",
    "accent_1": "#FFD700",
    "accent_2": "#FFA500",
    "accent_3": "#FFF8E7",
    "accent_4": "#F5E6C8"
  },
  "font_family": "PingFang SC, STSong, SimSun, Microsoft YaHei, serif"
}
```

## 装饰 DNA -- "国风"感

| 装饰元素 | 实现方式 |
|---------|---------|
| 金色角饰 | 内联 SVG 中式角纹（L 形 + 回纹细节），放在页面四角和卡片角落 |
| 祥云纹理 | 内联 SVG 祥云纹样，6-8% 透明度作为页面底部装饰 |
| 印章编号 | 方框内数字（`border:2px solid #FFD700; border-radius:4px; padding:4px 8px`）模拟印章效果 |
| 分隔线 | 金色渐隐线（`linear-gradient(90deg, transparent, #FFD700, transparent)` 30% opacity） |
| 标题装饰 | 标题左侧竖线加粗到 4px，使用金色 |
| 字体偏好 | 衬线字体优先（`STSong, SimSun` 排在 font-family 前面给中文优雅的书法感） |
| 禁止 | 禁止科技感元素（网格点阵/光晕/角标线）、禁止蓝色/紫色系颜色 |
| 整体感觉 | 故宫红墙 + 金色浮雕的庄重感，传统中有精致 |
