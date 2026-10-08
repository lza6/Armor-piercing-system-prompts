# 指标行（数字+标签+进度条 组合）

> 适用数据类型：metrics / kv_pairs。横向一行=一个指标故事。
> 数据需求：3-5个指标，每个需有 value + label + 可选 progress(0-100)。
> 结构：flex 横排，每项内部 value(大字号) + label(小字号) + progress-bar(可选)。

辅助组件，可与其他图表搭配使用。

> 适用数据：metrics / kv_pairs。横向一行=一个指标故事（数字+标签+进度条），适合3-5个并列指标。

## 结构骨架

```html
<div style="display:flex; align-items:center; gap:12px; margin-bottom:10px;">
  <span style="font-size:24px; font-weight:800; color:var(--accent-1);
               font-variant-numeric:tabular-nums; min-width:60px;">87%</span>
  <div style="flex:1;">
    <div style="font-size:12px; color:var(--text-secondary); margin-bottom:4px;">用户满意度</div>
    <div class="progress-bar"><div class="fill" style="width:87%"></div></div>
  </div>
</div>
```

> 以上数据为**占位示例**。数字、标签、进度条宽度都必须替换为实际数据。

## 灵动指引

- 多行指标竖排时，可以给每行用不同的 accent 色 -- 或者统一 accent 色但用透明度梯度
- 最重要的指标行可以数字更大、进度条更粗，制造视觉锚点
- 指标行特别适合在 list 类或 data 类卡片中使用，3-5 行刚好
