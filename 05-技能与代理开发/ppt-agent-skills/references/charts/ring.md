# 环形百分比（推荐用内联 SVG）

> 适用数据类型：pie_data / cost_breakdown。圆弧饱满度=占比。
> 数据需求：value + total（或 percentage），可选 label。
> PPTX 友好实现：内联 SVG circle + stroke-dasharray 计算弧长，圆心大数字叠加。不用 canvas。

`chart_type: ring`

> 适用数据：pie_data / cost_breakdown。圆弧饱满度=占比，适合单指标百分比，数据需有数值+总量。

## 结构骨架

```html
<div style="position:relative; width:80px; height:80px;">
  <svg width="80" height="80" viewBox="0 0 80 80">
    <circle cx="40" cy="40" r="32" fill="none"
            stroke="var(--card-bg-from)" stroke-width="10"/>
    <circle cx="40" cy="40" r="32" fill="none"
            stroke="var(--accent-1)" stroke-width="10"
            stroke-dasharray="180.96 201.06" stroke-linecap="round"
            transform="rotate(-90 40 40)"/>
  </svg>
  <!-- 中心文字用 HTML 叠加，不用 SVG text -->
  <div style="position:absolute; top:50%; left:50%; transform:translate(-50%,-50%);
      font-size:16px; font-weight:700; color:var(--text-primary);">90%</div>
</div>
```

## 计算公式

dasharray 第一个值 = 2 * PI * r * (百分比/100), 第二个值 = 2 * PI * r

> 以上尺寸为**结构参考**。容器大小和圆弧半径应根据所在卡片空间灵活调整。

## 灵动指引

- 圆环的粗细（stroke-width）可以根据卡片大小变化 -- 小卡片用粗环线（8-12），大卡片用细环线（4-6）更优雅
- 未填充弧用 card-bg-from 色（低存在感），已填充弧用 accent 色（高存在感）
- 中心文字不一定只放百分比 -- 可以放核心数值 + 微小标签（如 "15 分钟"）
- 多个环形图并排时，尺寸可以有大小差异（最重要的指标最大），制造视觉层级
