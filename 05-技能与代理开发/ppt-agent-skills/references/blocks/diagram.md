# diagram（图解块）-- 结构的星图

> 适用数据类型：hierarchies / architecture_diagram / cycle_flow / decision_tree / pyramid_layers / stakeholder_map。
> 结构：nodes[]节点 + edges[]连线，支持 layered/radial/tree/flowchart 四种布局模式。
> 实现方式不限：CSS Grid嵌套盒子、内联SVG节点连线、Flexbox+伪元素连接线。
> 推荐 card_style：transparent（自带视觉骨架，方块包裹会干扰）。推荐布局：single-focus / t-shape。

## JSON 结构
```json
{
  "card_type": "diagram",
  "diagram_type": "pyramid | flowchart | hub-spoke | layers | cycle",
  "nodes": [
    {"id": "1", "label": "节点名", "description": "描述（20字内）", "level": 1, "connects_to": ["2","3"]}
  ]
}
```

## 子类型的灵动设计思维

| diagram_type | 视觉灵魂 | 灵动手法 |
|-------------|---------|---------|
| pyramid | 权力/重要性的阶层感 | 让底层宽厚扎实、顶层锐利紧凑，层与层之间可以有微妙的光影渐变暗示"越往上越珍贵" |
| flowchart | 因果链的推动力 | 节点之间的连线不必死板横平竖直，可以用曲线暗示"流动"；箭头的大小可以随重要性变化 |
| hub-spoke | 中心辐射的控制力 | 中心大圆是不可动摇的核心，周围小圆是它的触角。可以让某些辐射臂更粗更显眼（代表更重要的分支） |
| layers | 技术栈/抽象层级的纵深 | 从上到下颜色渐深或渐浅，制造"地层沉积"的纵深感。层与层之间允许微妙的交错线暗示"层间通信" |
| cycle | 循环往复的韵律 | 节点沿轨迹排列，弧线连接暗示"永不停歇的循环"。可以让某个节点特别突出表示"当前阶段" |

## 实现指引
- 节点间关系用内联 SVG 连线、CSS border、伪元素等均可，选择最佳视觉效果
- 文字标注可用 HTML 元素或 SVG `<text>`，按场景灵活选择
- 推荐 `transparent` card_style -- 图解自带节点连线的视觉骨架
