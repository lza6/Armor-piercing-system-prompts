# timeline（时间线块）-- 时间的河流

> 适用数据类型：timelines / journey_map / gantt_data。横向/纵向轴线+节点。
> 结构：orientation(horizontal/vertical) + nodes[]({time, title, description, highlight})。
> 设计要点：highlight 节点用 accent 实心+更大尺寸，普通节点描边+小尺寸。4-8节点为宜，超过8个拆页。
> 推荐 card_style：transparent（自带轴线骨架）。推荐布局：l-shape / waterfall。

## JSON 结构不变（策划稿中的数据格式）
```json
{
  "card_type": "timeline",
  "orientation": "horizontal | vertical",
  "nodes": [
    {"time": "2020", "title": "事件标题", "description": "简述（30字内）", "highlight": false}
  ]
}
```

## 设计灵魂（不是代码模板）

### 横向时间线的灵动表达
- 轴线不必是死板的直线 -- 可以是微微弯曲的弧线、可以在 highlight 节点处膨胀加粗、可以在末端渐隐消失暗示"未来仍在延伸"
- 节点交替上下排列，制造视觉的呼吸起伏 -- 打破所有节点都在同一水平线上的单调
- Highlight 节点用 accent 实心 + 更大的尺寸，普通节点用描边 + 更小的尺寸，形成明确的主次

### 纵向时间线的灵动表达
- 左侧时间标签可以用不同透明度/字号，越近的越清晰 -- 制造"时间的景深感"
- 右侧描述区域的内容密度可以不均匀 -- 重要事件给更多空间，次要事件精炼浓缩

### 实现指引
- 轴线和连线实现方式不限（真实 div、伪元素 `::before`/`::after`、内联 SVG 均可）
- 箭头可用内联 SVG 或 CSS border 三角形
- 4-8 个节点为宜，超过 8 个拆页
- 推荐 `transparent` card_style -- 时间线自带轴线骨架，不需要方块包裹
