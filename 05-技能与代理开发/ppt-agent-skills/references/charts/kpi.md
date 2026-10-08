# KPI 指标卡（数字+趋势箭头+标签）

> 适用数据类型：metrics / number_highlights / milestone_results。
> 数据需求：单个核心数字 + 趋势方向(up/down/flat) + 变化百分比 + 标签。
> 结构：大数字 font-size:40-64px font-weight:800 + SVG三角箭头(16x16) + 变化值 + 标签。
> PPTX 友好实现：纯 HTML+CSS，箭头用内联SVG polygon，tabular-nums 对齐数字。

`chart_type: kpi`

> 适用数据：metrics / number_highlights / milestone_results。大数字+趋势箭头+标签，单个核心指标的极致冲击力呈现。

## 结构骨架

```html
<div style="display:flex; align-items:baseline; gap:8px;">
  <span style="font-size:40px; font-weight:800; color:var(--accent-1);
               font-variant-numeric:tabular-nums;">2.4M</span>
  <!-- 上升箭头 SVG -->
  <svg width="16" height="16" viewBox="0 0 16 16">
    <polygon points="8,2 14,10 2,10" fill="#16A34A"/>
  </svg>
  <span style="font-size:14px; color:#16A34A; font-weight:600;">+12.3%</span>
</div>
<div style="font-size:12px; color:var(--text-secondary); margin-top:4px;">月活跃用户数</div>
```

> 以上数据为**占位示例**。数字、箭头方向、百分比、标签都必须替换为实际数据。

## 关键规则

- 趋势箭头颜色语义：上升=绿色、下降=红色、持平=text-secondary
- 箭头用内联 SVG polygon 或 CSS border 三角形均可
- 数字用 `font-variant-numeric: tabular-nums` 让等宽数字对齐
- 多个 KPI 并排时，数字大小应一致

## 灵动指引

- 数字的字号不必永远 40px -- 在大卡片中可以用 56-64px 制造"数据爆炸"的视觉冲击
- 如果有同比/环比两个对比维度，可以把趋势箭头做成两行（同比/环比各一行）
- KPI 卡特别适合搭配 sparkline 折线图 -- 大数字下方加一条极细的趋势线，信息量翻倍
