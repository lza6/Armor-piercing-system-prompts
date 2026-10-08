# comparison（对比块）-- 碰撞的擂台

> 适用数据类型：before_after / pros_cons / scenario_comparison / competitive_matrix。
> 结构：双面板正面对比，left+right各含label、points[]、accent色设定，底部可选verdict总结句。
> 设计要点：右侧推荐方案用更强accent色+更大字号+更丰满内容，左侧中性色+克制排版 -- 视觉上引导结论。
> 推荐 card_style：outline（轻轻分隔两面板）。推荐布局：symmetric。

## JSON 结构
```json
{
  "card_type": "comparison",
  "title": "传统方式 vs 新方案",
  "left": {"label": "方案A / 现状", "points": ["维度1", "维度2", "维度3"], "accent": "neutral"},
  "right": {"label": "方案B / 目标", "points": ["维度1", "维度2", "维度3"], "accent": "primary"},
  "verdict": "底部总结句（可选）"
}
```

## 设计灵魂

### 对比的戏剧性
- 左右两面板不应该看起来一模一样只是内容不同 -- 那是最死板的网页前端思维
- 有立场的对比：如果右侧是"推荐方案"，让右面板用更强的 accent 色、更大的字号、更丰满的内容，左面板用中性色 + 克制的排版。视觉上就已经在"引导结论"
- 无立场的对比：两面板用不同的 accent 色（accent-1 vs accent-2），但结构完全对称

### 灵动手法
- VS 分隔符可以是一个跨越中轴线的圆形 accent 色块 + "VS" 文字，打破左右的物理分割
- 对比维度上下对齐，让观众的视线可以水平扫视对比，制造"逐条PK"的紧张感
- verdict（总结句）跨两面板居中，是"裁判宣布结果"的画龙点睛

### 每面板 3-5 个对比维度
- 维度太少（< 3）对比不充分，维度太多（> 5）画面拥挤
- 推荐 `outline` card_style -- 描边轻轻分隔两个面板，不占视觉权重
