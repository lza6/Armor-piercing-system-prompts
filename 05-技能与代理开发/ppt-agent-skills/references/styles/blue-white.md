# 蓝白商务 (blue_white) -- 最通用的默认风格

> 适用场景：企业介绍、培训课件、教育材料、医疗/金融行业、一般性汇报、未明确指定风格的大多数场景
## style.json 参考

```json
{
  "style_id": "blue_white",
  "style_name": "蓝白商务 (Blue White Business)",
  "mood_keywords": ["蓝图清晰", "信赖锚点", "稳如磐石", "专业呼吸", "机构感"],
  "design_soul": "一份顶级咨询公司的正式报告摊开在黑胡桃木桌面上，蓝色的标题像精密绘制的蓝图标注线，每一处留白都是经过计算的呼吸 -- 克制、可信、零噪音",
  "variation_strategy": "纯内容页用蓝色标题装饰条+细线边框卡片营造报告感（秩序），数据密集页切换为大面积留白+单一accent数字锚点（极简），章节封面用大号蓝色编号+全页留白（呼吸），三种节奏交替形成'翻阅报告'的从容感",
  "decoration_dna": {
    "signature_move": "蓝色渐变横条放在标题下方（4px高, 80px宽）-- McKinsey 报告式的精准标注",
    "forbidden": ["光晕效果", "网格点阵", "角标装饰线"],
    "recommended_combos": [
      "标题下划线 + 双细线分隔 + 蓝色方块编号",
      "渐隐分隔线 + 大号水印 + 卡片悬浮感"
    ]
  },
  "css_variables": {
    "bg_primary": "#FFFFFF",
    "bg_secondary": "#F8FAFC",
    "card_bg_from": "#F1F5F9",
    "card_bg_to": "#E2E8F0",
    "card_border": "rgba(37,99,235,0.12)",
    "card_radius": "12px",
    "text_primary": "#1E293B",
    "text_secondary": "#64748B",
    "accent_1": "#2563EB",
    "accent_2": "#1D4ED8",
    "accent_3": "#059669",
    "accent_4": "#047857"
  },
  "font_family": "PingFang SC, Microsoft YaHei, system-ui, sans-serif"
}
```

## 装饰 DNA -- "McKinsey 报告"感

| 装饰元素 | 实现方式 |
|---------|---------|
| 标题装饰 | **蓝色渐变横条**放在标题下方（4px 高, 80px 宽, `linear-gradient(90deg, #2563EB, #1D4ED8)`） |
| 卡片边框 | **细线边框**（`border:1px solid rgba(37,99,235,0.12)`）+ **微妙阴影**（`box-shadow:0 1px 3px rgba(0,0,0,0.05)`） |
| 分隔线 | **双细线分隔**（两条 1px 线，间距 3px，颜色 `rgba(0,0,0,0.06)`） |
| 编号样式 | 蓝色实心方块编号（`border-radius:4px`，不是圆形） |
| 禁止 | 禁止光晕效果、禁止网格点阵、禁止角标装饰线（太"科技感"） |
| 整体感觉 | 干净、可信、机构级专业感，像一份顶级咨询公司的正式报告 |
