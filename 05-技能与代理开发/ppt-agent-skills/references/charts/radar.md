# 雷达图 / 蜘蛛网图（多维度对比）

> 适用数据类型：score_card。多边形面积=综合实力。
> 数据需求：3-6个维度，每维度需有 dimension_name + value(0-100)。
> PPTX 友好实现：内联 SVG polygon，网格线用 stroke-dasharray，数据多边形用 fill-opacity:0.3。不用 canvas。

`chart_type: radar`

> 适用数据：score_card。多边形面积=综合实力，3-6维度为宜，需有维度名+数值。

## 结构骨架

用 SVG polygon 绘制。适合展示 3-6 个维度的能力/指标对比。

```html
<div style="position:relative; width:200px; height:200px;">
  <svg width="200" height="200" viewBox="0 0 200 200">
    <!-- 网格线（3层同心多边形） -->
    <polygon points="..." fill="none" stroke="var(--card-border)" stroke-width="1"/>
    <!-- 轴线（从中心到每个顶点） -->
    <line x1="100" y1="..." x2="100" y2="..." stroke="var(--card-border)" stroke-width="1"/>
    <!-- 数据区域（半透明填充的多边形） -->
    <polygon points="..."
             fill="var(--accent-1)" opacity="0.15"
             stroke="var(--accent-1)" stroke-width="2"/>
    <!-- 数据点（每个顶点上的圆点） -->
    <circle cx="..." cy="..." r="4" fill="var(--accent-1)"/>
  </svg>
  <!-- 维度标签用 HTML position:absolute 叠加在 SVG 外围（禁止 SVG text） -->
  <span style="position:absolute; top:2px; left:50%; transform:translateX(-50%);
      font-size:11px; color:var(--text-secondary);">维度名</span>
</div>
```

## 关键规则

- **维度数限制**：3-6 维最佳，6 维以上标签太挤不推荐
- **文字标签**可用 HTML 叠加（position:absolute）或 SVG text，按场景灵活选择
- 网格线用极细极淡的 card-border 色（视觉存在但不抢戏）
- 数据多边形用 accent 色 + 15% opacity 填充 + 实色描边

## 灵动指引

- 雷达图的多边形顶点坐标需要根据维度数和数据值计算（正 N 边形的几何坐标 + 数据比例缩放）
- 容器尺寸根据卡片空间灵活调整（不必永远 200x200），只需保持正方形比例
- 如果需要两组数据对比，可以画两个不同 accent 色的数据多边形叠加
- 标签位置需根据多边形顶点位置微调，确保不被 SVG 图形遮挡
