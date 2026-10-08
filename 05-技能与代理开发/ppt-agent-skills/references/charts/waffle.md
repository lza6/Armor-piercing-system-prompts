# 点阵图 Waffle Chart（百分比直觉化）

> 适用数据类型：distribution_data / progress_tracker。100格点阵亮灭=百分比。
> 数据需求：percentage(0-100)。适合单指标直觉呈现。
> PPTX 友好实现：10x10 CSS Grid，每格 8x8px gap:2px，亮格用 accent 色，暗格用 opacity:0.15。

`chart_type: waffle`

> 适用数据：distribution_data / progress_tracker。100格点阵亮灭=百分比，最直觉的比例呈现，适合单指标。

## 结构骨架

```html
<div style="display:grid; grid-template-columns:repeat(10,1fr); gap:3px; width:100px;">
  <!-- 填充点（accent 色） -->
  <div style="width:8px; height:8px; border-radius:2px; background:var(--accent-1);"></div>
  <!-- 重复填充点... -->
  <!-- 空点（card-bg-from 色） -->
  <div style="width:8px; height:8px; border-radius:2px; background:var(--card-bg-from);"></div>
  <!-- 重复空点... -->
</div>
```

10x10 = 100 格，填充数量 = 百分比值。

> 以上为**结构参考**。格点的尺寸和容器宽度应根据卡片空间灵活调整。

## 灵动指引

- 点阵图比进度条更直觉但占用空间更大 -- 适合在大面积卡片或单一焦点版式中使用
- 格点不一定要正方形 -- 圆形（border-radius:50%）更柔和，方形（border-radius:2px）更科技
- 如果百分比数据是多分类的（如 A=40% B=30% C=30%），可以用 3 种 accent 色的格点，制造"彩色拼图"效果
- 注意管线性能：100 个小元素接近 svg2pptx 的性能边界，考虑用 5x10=50 格（每格代表 2%）来减少元素数
