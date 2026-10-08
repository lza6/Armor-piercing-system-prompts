# 进度条（百分比/完成度）

> 适用数据类型：progress_tracker。填充长度=完成度。
> 数据需求：percentage(0-100) + label + 可选 milestone_markers[]。
> PPTX 友好实现：外层 div(background:轨道色) + 内层 div(width:N%, background:渐变)，高度 8-12px 圆角。

`chart_type: progress_bar`

> 适用数据：progress_tracker。填充长度=完成度，需有百分比数值+标签，适合单指标进度展示。

## 结构骨架

```css
.progress-bar {
  height: 8px; border-radius: 4px;
  background: var(--card-bg-from);
  overflow: hidden;
}
.progress-bar .fill {
  height: 100%; border-radius: 4px;
  background: linear-gradient(90deg, var(--accent-1), var(--accent-2));
  /* width 用内联 style 设置百分比 */
}
```

## 灵动指引

- 进度条的高度不必永远 8px -- 在大面积卡片中可以用 12-16px 制造更强的存在感
- 填充色可以根据数据语义变化：高值用 accent 色（积极），低值用红色（警示）
- 进度条左上方叠加百分比数字可以增强信息量
- 多个进度条竖排列时，可以给每条设置不同的 accent 色 + 递增延迟动画（HTML 预览增强）
