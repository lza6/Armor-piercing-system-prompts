# 迷你折线图 Sparkline（趋势方向）

> 适用数据类型：trend_series。一条折线=趋势方向。
> 数据需求：5-12个数据点序列，不需Y轴标签。适合嵌入卡片内的迷你趋势图。
> PPTX 友好实现：内联 SVG polyline，stroke-width:2，无网格线，可选渐变 fill 下方区域。

`chart_type: sparkline`

> 适用数据：trend_series。一条折线=趋势方向，不需Y轴刻度，适合嵌入卡片内的迷你趋势图。

## 结构骨架

```html
<svg width="120" height="40" viewBox="0 0 120 40">
  <!-- 面积填充（趋势线下方的柔和阴影） -->
  <path d="M0,35 L20,28 L40,30 L60,20 L80,15 L100,10 L120,5 L120,40 L0,40 Z"
        fill="var(--accent-1)" opacity="0.1"/>
  <!-- 趋势线 -->
  <polyline points="0,35 20,28 40,30 60,20 80,15 100,10 120,5"
            fill="none" stroke="var(--accent-1)" stroke-width="2" stroke-linecap="round"/>
  <!-- 终点圆点 -->
  <circle cx="120" cy="5" r="3" fill="var(--accent-1)"/>
</svg>
```

> 数据点坐标根据实际趋势调整 y 值（高=好 -> y 值小，低=差 -> y 值大）。

## 灵动指引

- sparkline 最佳用途是紧贴在大数字旁边 -- 数字是"现在"，折线是"怎么到这的"
- 面积填充（path）的 opacity 控制在 0.08-0.15，太浓会抢数字的视觉权重
- 终点圆点让观众知道"当前值在哪"
- 多个 sparkline 并排时（如多指标对比），可以统一 viewBox 尺寸但用不同 accent 色
