# 极简灰白 (minimal_gray) -- "学术论文"感

> 适用场景：学术/研究报告、法务/合规、咨询/顾问报告、数据分析
## style.json 参考

```json
{
  "style_id": "minimal_gray",
  "style_name": "极简灰白 (Minimal Gray)",
  "mood_keywords": ["Helvetica纯净", "手术室般精确", "零噪音", "信息至上", "冷调秩序"],
  "design_soul": "一本 Dieter Rams 的工业设计图册翻到了空白页 -- 除了一行用 Helvetica Bold 排印的数据和一条红色下划线，整个页面只有纯净的白。每一像素的留白都在说：'信息本身就是装饰'",
  "variation_strategy": "数据页用大面积留白+唯一的红色数字锚点（极简聚焦），文字密集页用极细灰色分隔线+严格的网格对齐（排字工坊），章节封面仅放编号和标题（空旷冷寂），变化不靠装饰而靠密度的剧烈反差 -- 上一页满满当当的信息阵列，下一页只有一句话浮在白色沙漠中",
  "decoration_dna": {
    "signature_move": "零装饰本身就是装饰 -- 靠字重/字号的极端反差和精准对齐制造高级感",
    "forbidden": ["渐变色块", "光晕效果", "装饰SVG", "彩色标签", "角标装饰"],
    "recommended_combos": [
      "单线分隔 + 小圆点编号 + 大面积留白",
      "红色关键数字 + 极细灰线 + 零装饰"
    ]
  },
  "css_variables": {
    "bg_primary": "#FAFAFA",
    "bg_secondary": "#F5F5F5",
    "card_bg_from": "#FFFFFF",
    "card_bg_to": "#FAFAFA",
    "card_border": "rgba(0,0,0,0.08)",
    "card_radius": "8px",
    "text_primary": "#171717",
    "text_secondary": "#737373",
    "accent_1": "#171717",
    "accent_2": "#404040",
    "accent_3": "#DC2626",
    "accent_4": "#B91C1C"
  },
  "font_family": "'Inter', 'PingFang SC', 'Microsoft YaHei', system-ui, sans-serif"
}
```

## 装饰 DNA -- "学术论文"感

| 装饰元素 | 实现方式 |
|---------|---------|
| 标题装饰 | **无装饰线**，标题用 font-weight:800 + 大号字自身的视觉重量即可 |
| 卡片边框 | **纯细线**（`border:1px solid rgba(0,0,0,0.08)`），无阴影，无圆角（`border-radius:4px`） |
| 分隔线 | **单线**（1px, `rgba(0,0,0,0.06)`） |
| 强调方式 | 只用红色（`--accent-3: #DC2626`）强调关键数据，其余全灰/黑 |
| 编号样式 | 小号数字 + 圆点（`8px 圆点, #171717`） |
| 禁止 | 禁止渐变、禁止光晕、禁止装饰 SVG、禁止彩色标签 |
| 整体感觉 | Helvetica 式的纯净，信息优先，零视觉噪音 |
