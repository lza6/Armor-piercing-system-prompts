# matrix_chart（象限矩阵块）-- 象限的定位

> 适用数据类型：matrix_data / swot / competitive_matrix。2x2象限坐标定位。
> 结构：axes(x_label, y_label) + quadrants[4]({label, items[], color})。
> 设计要点：每象限独立色块，items用定位圆点标记，适合战略分析和多维评估。
> 推荐布局：single-focus / primary-secondary。

## JSON 结构
```json
{
  "card_type": "matrix_chart",
  "title": "市场定位分析",
  "x_label": "X轴含义（如：执行难度）",
  "y_label": "Y轴含义（如：业务价值）",
  "quadrants": [
    {"position": "top-right", "title": "优先执行", "items": ["项目A","项目B"], "highlight": true},
    {"position": "top-left", "title": "长期投资", "items": ["项目C"], "highlight": false},
    {"position": "bottom-right", "title": "快速见效", "items": ["项目D"], "highlight": false},
    {"position": "bottom-left", "title": "低优先级", "items": ["项目E"], "highlight": false}
  ]
}
```

## 设计灵魂

### 象限的视觉张力
- **十字轴**是整个组件的脊柱 -- 不要只是两条灰线。可以用 accent 色极低透明度的粗线（2-3px），让轴线有"存在感"但不抢内容
- **Highlight 象限**应当从视觉上"跳出来" -- accent 色 15% 透明度背景 + 标题加粗加色 + 项目标签用 accent 胶囊
- **其他象限**保持克制 -- 5% 透明度或完全透明，项目标签用 text-secondary

### 灵动手法
- 不要让四个象限看起来完全对称 -- highlight 象限可以面积微大/色调微浓，制造"视觉重力偏移"
- 轴标签在四端用 12px + letter-spacing:1px，像坐标系的刻度铭牌
- 项目标签用胶囊形圆角药丸，可以在象限内自由散落（非严格排列），暗示"定位"的概念

### 推荐 `transparent` card_style + 跨列跨行占据大区域
- 象限图需要足够的空间才能呈现清晰的四象限结构
- 推荐  占据全部可用空间
