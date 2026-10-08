# 暗黑科技 (dark_tech) -- "发布会"感

> 适用场景：技术产品发布、开发者工具演示、AI/ML 模型展示。注意：仅当主题**明确**是技术产品展示、开发者工具、深度技术架构讲解时才选用。
## style.json 参考

```json
{
  "style_id": "dark_tech",
  "style_name": "暗黑科技 (Dark Tech)",
  "mood_keywords": ["深空冷寂", "精密仪器", "微光脉搏", "数据洪流", "未来感"],
  "design_soul": "天文台穹顶内部，深蓝黑幕中冷青的仪器扫描光有节奏地划过 -- 精密、冷寂、但每一次扫描都暗含脉搏。屏幕上的数据像星图一样精确排列",
  "variation_strategy": "数据页用网格点阵+角标装饰线制造精密仪器控制台感（紧张高密度），章节封面用大面积深空留白+单一光晕呼吸点（释放），产品展示页用全屏暗底+中央悬浮的发光数据面板（聚焦），三种极端交替形成'在深空中切换仪器面板'的节奏",
  "decoration_dna": {
    "signature_move": "网格点阵底纹 + L形角标装饰线 -- 精密仪器控制台的骨架感",
    "forbidden": ["渐变色块", "叶片装饰", "波浪分隔线"],
    "recommended_combos": [
      "网格点阵 + 角标装饰 + 大号水印数字",
      "光晕效果 + 脉冲圆点 + 半透明数字水印"
    ]
  },
  "css_variables": {
    "bg_primary": "#0B1120",
    "bg_secondary": "#0F172A",
    "card_bg_from": "#1E293B",
    "card_bg_to": "#0F172A",
    "card_border": "rgba(255,255,255,0.05)",
    "card_radius": "12px",
    "text_primary": "#FFFFFF",
    "text_secondary": "rgba(255,255,255,0.7)",
    "accent_1": "#22D3EE",
    "accent_2": "#3B82F6",
    "accent_3": "#FDE047",
    "accent_4": "#F59E0B"
  },
  "font_family": "PingFang SC, Microsoft YaHei, system-ui, sans-serif"
}
```

## 装饰 DNA -- "发布会"感

| 装饰元素 | 实现方式 |
|---------|---------|
| 网格点阵 | `radial-gradient(circle, rgba(255,255,255,0.03) 1px, transparent 1px), background-size:40px 40px` |
| 角标装饰线 | L 形 SVG path（`border-top + border-left`），accent 色 20% 透明度 |
| 光晕效果 | radial-gradient 超大半透明圆(400-600px)，accent 色 5-8% 透明度 |
| 半透明数字水印 | 120-160px 超大数字，accent 色 opacity 0.03-0.05 |
| 卡片分隔线 | 1px solid rgba(255,255,255,0.05) |
| 脉冲圆点 | 6px accent 色圆点 + 外圈 14px 10% 透明度圆环（双圈效果） |
| 整体感觉 | 深空科技、精密仪器感，像 Apple/NVIDIA 的产品发布会 |
