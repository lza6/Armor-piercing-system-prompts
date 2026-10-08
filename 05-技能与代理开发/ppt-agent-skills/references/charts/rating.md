# 评分指示器（5分制）

> 适用数据类型：score_card / ranked_list。实心vs空心=已达到vs未达到。
> 数据需求：score(1-5) + label。
> PPTX 友好实现：5个内联SVG圆点(r=6)，实心用 accent 色 fill，空心用 stroke-only + fill:none。

`chart_type: rating`

> 适用数据：score_card / ranked_list。实心vs空心=已达到vs未达到，5分制评分直觉呈现。

## 结构原理

用一行圆点表示评分：实心圆 = 已得分，空心圆 = 未得分。

```html
<div style="display:flex; gap:6px;">
  <!-- 实心圆（已得分） -->
  <div style="width:12px; height:12px; border-radius:50%; background:var(--accent-1);"></div>
  <!-- 空心圆（未得分） -->
  <div style="width:12px; height:12px; border-radius:50%; border:2px solid var(--accent-1); background:transparent;"></div>
</div>
```

> 以上为 **结构参考**。实心/空心数量根据实际评分调整。

## 灵动指引

- 圆点不一定要用圆形 -- 条状矩形（4px x 12px）也可以表达评分，且视觉更现代
- 圆点大小可以根据卡片空间调整（8-14px 范围内）
- 如果是半分制（如 4.5/5），可以用半实心圆（左半实心右半空心，用 clip-path 或 overlay div 实现）
- 评分指示器特别适合在 list 卡片中紧跟每条项目后面
