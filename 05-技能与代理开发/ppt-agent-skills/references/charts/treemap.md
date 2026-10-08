# 矩形树图 Treemap（层级结构占比）

> 适用数据类型：distribution_data / cost_breakdown。面积大小=重要性大小。
> 数据需求：4-12个项目，每项需有 label + value。
> PPTX 友好实现：CSS Grid 计算面积占比，每格用不同 accent 色+白色标签。不用 D3。

`chart_type: treemap`

> 适用数据：distribution_data / cost_breakdown。面积大小=重要性大小，适合层级结构占比可视化，需有标签+数值。

## 结构原理

用 CSS Grid 模拟矩形树图，适合展示有层级关系的面积对比。

- 面积比例通过 `grid-template-columns` 和 `grid-template-rows` 的 fr 值控制
- 每个块内部放数据标签（百分比/数值 + 类别名）
- 块的颜色用不同 accent 色区分

## 关键规则

- fr 值必须反映真实数据比例（面积 = 占比，禁止为美观而扭曲比例）
- 最大块跨列或跨行（grid-row: 1 / -1），让它在视觉上明显最大
- 每个块的文字颜色要确保在对应 accent 色块上可读
- 块间 gap 极小（2-3px），让树图有"拼合"的视觉效果

## 灵动指引

- 3-4 个块是最佳数量 -- 太多块就变成"马赛克"失去了面积对比的冲击力
- 块的圆角和容器一致（统一 border-radius），整体更精致
- 数值标注的字号可以跟随块的大小变化 -- 最大块用最大字号，最小块用最小字号
- 容器高度根据卡片空间灵活调整
