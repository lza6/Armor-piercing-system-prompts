# 原仓库提示词模式

仅在用户要求提示词、或明确选择 Flow/Nano Banana 外部生成时使用。本模式不调用 API、不购买额度、不渲染视频。

未指定其他画风时，读 [style-guide.md](style-guide.md) 获取原 Q 版蜡笔画风与 Flow 首尾帧语法；用户提供参考图时读 [style-selection-and-updates.md](style-selection-and-updates.md)，替换不一致的配色、笔触、比例与背景描述。下面的 Q 版蜡笔示例不是所有风格的固定要求。[output-example.md](output-example.md) 是题材示例，其事实不是新任务的事实来源。
按语义拆分，常见每镜头 4–6 秒但以稿件与用户要求为准。每镜头一个核心视觉隐喻，每条提示词自包含，不使用“同上”。

```text
镜头 01
对应口播：
建议时长：
视觉隐喻：
画面中文关键词：

【生图提示词】
完整英文描述：9:16、#F8F6EF、自然克制 Q 版人物、粗黑手绘线、黄/蓝/红色板、具体位置关系、主体与字幕安全区。

【图生视频提示词】
完成插画作 Last Frame、空白暖白纸作 First Frame；locked camera、rigid paper cutouts、tactile paper stop-motion；明确哪个物件先出现、人物何时动作、怎样落到尾帧。no lip sync、no added characters/logos、no audio。

【实体准确性提醒】
哪些文字、旗帜、Logo、日期、金额应使用原始资产或确定性后期图层。
```

仅单张参考图时改为原位微动作，不要求不存在的首帧控制。不要保证生成模型像素级保持背景、中文或国旗准确。短关键词可提供直接生成版，同时给无字安全版；长文字与精确数字默认后期。
只写提示词时不引入配音配置、工程打包或样片批准流程。
