# Director Command 示例库（10 种页面类型 * 2 个示例）

> 非默认资料。human-only calibration library。
>
> 本文件不进入 planning runtime preload，不应整包注入子代理上下文。它只用于人工校准、调试和对照分析，不作为默认主链参考。

> 策划阶段撰写 `director_command` 时参考本文件。每个示例采用结构化格式，包含 mood / spatial_strategy / anchor_treatment / techniques / prose 五个子字段。
> **注意**：这些是起点不是模板，必须根据本页实际内容、大纲的 `page_emotion`、以及上下页的视觉差异要求进行变异。

---

## 1. 封面页

### 示例 A：气势磅礴型

```json
{
  "mood": "开场宣言 / 不容置疑的统治力",
  "spatial_strategy": "上 60% 极大标题 + 下 40% 深色留白仅一行副标题",
  "anchor_treatment": "主标题 120px+ 贯穿画面，渐变填充从 accent-1 到 accent-2",
  "techniques": ["T1", "T7"],
  "prose": "主标题像纪念碑一样矗立在画面上方，字号大到几乎要冲破天花板。底部 40% 是深邃的黑暗，仅有一行 14px 的副标题和品牌信息漂浮其中。GROWTH 水印以 300px 幽灵般躺在右下角 opacity 0.03"
}
```

### 示例 B：沉稳精密型

```json
{
  "mood": "精密仪器启动 / 蓄势待发",
  "spatial_strategy": "左 65% 标题偏心 + 右 35% 留白仅 HUD 角线",
  "anchor_treatment": "主标题偏心于左侧三分线，字重 900 但字号克制在 56px",
  "techniques": ["T9", "T5"],
  "prose": "画面像刚启动的仪表盘，标题安静地停在左侧三分线上，右侧大面积留白中只有 HUD 角线在呼吸。底部一条 accent 斜切色带从左下划向右下，像扫描光掠过"
}
```

---

## 2. 目录页

### 示例 A：路线图型

```json
{
  "mood": "全景鸟瞰 / 清晰的航线图",
  "spatial_strategy": "纵向 3-4 等分区域，每区域一个 Part 标题",
  "anchor_treatment": "Part 编号用 64px accent 色数字，标题紧随其后 20px",
  "techniques": ["T2", "T9"],
  "prose": "像在高空俯瞰航线，3 个 Part 从上到下依次排列，每个 Part 编号像航点标记一样用超大数字锚定位置。Part 之间用极细渐隐分隔线轻柔划分，不用硬线"
}
```

### 示例 B：极简呼吸型

```json
{
  "mood": "一目了然 / 呼吸空间充足",
  "spatial_strategy": "居中偏左竖排 Part 标题，右侧 50% 完全留白",
  "anchor_treatment": "Part 标题用 overline 字母间距标识层级，无数字编号",
  "techniques": ["T7", "T5"],
  "prose": "画面极度克制，Part 标题竖向排列在左侧 50%，右侧完全空旷。一条极细的 accent 斜切线从画面左上贯穿到右下角，像一条隐形路径串联所有章节"
}
```

---

## 3. 章节封面

### 示例 A：巨型编号压迫型

```json
{
  "mood": "新篇章开启 / 肃穆宣告",
  "spatial_strategy": "80% 留白 + 20% 右下角标题",
  "anchor_treatment": "PART 编号 160px+ opacity 0.04 铺满左半画面背景",
  "techniques": ["T1", "T7"],
  "prose": "画面近乎空白，只有 PART 01 四个字像水印一样以 160px 躺在画面左侧 opacity 0.04。章节标题极度偏心地蜷缩在右下角，用 28px 字重 700 安静地等待"
}
```

### 示例 B：色块切割型

```json
{
  "mood": "场景切换 / 空间重置",
  "spatial_strategy": "左 30% accent 色块 + 右 70% 留白",
  "anchor_treatment": "章节标题置于色块内竖排 writing-mode，白色文字",
  "techniques": ["T8", "T5"],
  "prose": "画面被一块竖直的 accent 色带从左侧 30% 处斜切，像舞台幕布被掀起一半。标题藏在色带内以竖排方式书写，右侧 70% 只有一行极小的 Part 编号"
}
```

---

## 4. 数据仪表盘页

### 示例 A：数据压境型

```json
{
  "mood": "数据压境 / 窒息般的统治力",
  "spatial_strategy": "70% 深色留白 + 30% 信息爆炸集中在左上",
  "anchor_treatment": "180px 核心数字脱框裸露于左上角，无背景无边框",
  "techniques": ["T7", "T2", "T1"],
  "prose": "那个百分比数字像陨石般砸在左上角的深渊中，180px 字号没有任何容器包裹它，辅助文字统统 0.2 透明度埋入底部暗区，背景底部贯穿半页 GROWTH 水印 opacity 0.04"
}
```

### 示例 B：控制台仪表型

```json
{
  "mood": "精密监控 / 数据驾驶舱",
  "spatial_strategy": "上 30% 主 KPI 横向排列 + 下 70% 网格辅助指标",
  "anchor_treatment": "核心 KPI 用 96px 数字 + 紧贴 12px 注解的极致字号共生",
  "techniques": ["T2", "T10", "T4"],
  "prose": "画面上方像仪表盘的主屏幕，2-3 个核心 KPI 以极致字号共生排列。下方密集的辅助指标卡片使用浮岛面板效果悬浮，每张卡片底部都有 sparkline 作为数据铺底纹理"
}
```

---

## 5. 对比分析页

### 示例 A：楚河汉界撕裂型

```json
{
  "mood": "对峙撕裂 / 不平衡的降维打击",
  "spatial_strategy": "左 35% 旧方案压缩灰暗 + 右 65% 新方案开阔光亮",
  "anchor_treatment": "新方案卡片 accent 悬浮 + 旧方案卡片灰色内凹",
  "techniques": ["T8", "T3", "T4"],
  "prose": "构造一场严重不平衡的楚河汉界！左边的旧方案蜷缩着、灰暗着、卡片互相叠压挤在 35% 的空间里。右侧新方案占据 65% 的呼吸空间，accent 色的悬浮卡片带着发光阴影，形成肉眼可见的降维打击"
}
```

### 示例 B：天平秤型

```json
{
  "mood": "理性评估 / 客观权衡",
  "spatial_strategy": "50/50 对称但视觉重量故意不等",
  "anchor_treatment": "推荐方案侧用 accent 标记 + elevated 悬浮",
  "techniques": ["T3", "T9"],
  "prose": "画面看似 50/50 对称，但推荐方案侧的卡片透过 elevated 阴影微微浮起，顶部有 accent 脉冲锚点标记。对比方的卡片用 outline 样式安静地平铺，形成微妙的倾斜感"
}
```

---

## 6. 流程/时间线页

### 示例 A：编年史卷轴型

```json
{
  "mood": "时间之河 / 编年史徐徐展开",
  "spatial_strategy": "横向流动布局，左起右收，时间线贯穿中轴",
  "anchor_treatment": "时间节点用 accent 脉冲锚点标记，关键节点放大",
  "techniques": ["T9", "T5"],
  "prose": "时间线像一条 accent 色的河流从左侧涌入画面，沿水平中轴贯穿到右侧。关键里程碑用脉冲锚点突出标记，节点上方下方交错悬挂描述卡片，一条斜切色带在底部收束走势"
}
```

### 示例 B：阶梯递进型

```json
{
  "mood": "步步高升 / 不可逆的进化",
  "spatial_strategy": "从左下到右上的对角阶梯布局",
  "anchor_treatment": "最终阶段用 accent 强调 + 其余阶段递减视觉重量",
  "techniques": ["T3", "T2"],
  "prose": "3-4 个阶段从左下角像阶梯一样攀升到右上角，每级比前一级略微偏右偏上。最后一级用 accent 色和更大字号爆发，前面的阶梯用递减的透明度暗示已完成的历程，卡片之间用负 margin 叠压制造连续感"
}
```

---

## 7. 金句/引言页

### 示例 A：死寂威严型

```json
{
  "mood": "极简金字塔 / 死寂般的威严",
  "spatial_strategy": "85% 留白 + 15% 偏心金句",
  "anchor_treatment": "金句 36px 字重 800，偏心于画面左下三分点",
  "techniques": ["T7", "T1"],
  "prose": "呈现苹果级的留白与克制！全场 85% 画布面积不放任何元素，只在左下三分点悬浮一行 36px 的中心论断，字重极高承载巨大的空间压迫感。右上角一行 280px 的关键词水印 opacity 0.03 与金句遥相呼应"
}
```

### 示例 B：引号仪式型

```json
{
  "mood": "权威引用 / 庄重的仪式感",
  "spatial_strategy": "居中偏上引文 + 下方极小署名",
  "anchor_treatment": "超大引号装饰 + 引文用 accent 渐变文字填充",
  "techniques": ["T7", "T6"],
  "prose": "一个 120px 的装饰性引号像印章一样压在画面左上角 opacity 0.15。引文居中偏上用 accent 渐变文字填充，下方用极小的 11px 标注作者和来源，四周是壮阔的留白"
}
```

---

## 8. 团队/人物介绍页

### 示例 A：星系散布型

```json
{
  "mood": "群星闪耀 / 每个人是独立的星球",
  "spatial_strategy": "非等距散布，核心人物居中偏大，其余围绕",
  "anchor_treatment": "核心人物用 accent 圆形头像框 + 其余用淡色圆形",
  "techniques": ["T8", "T6"],
  "prose": "每个人物是一颗独立的星球散布在画面中，核心人物偏心于画面左侧三分点，头像用 accent 色圆形边框强调。其余人物大小递减地散布在右侧，底层有一个 400px 的超大圆形装饰 opacity 0.05 统领全场"
}
```

### 示例 B：名片矩阵型

```json
{
  "mood": "整齐划一 / 专业团队列阵",
  "spatial_strategy": "2 行网格排列，pero每张卡片有微妙的高度差异",
  "anchor_treatment": "关键人物卡片用 elevated 悬浮 + accent 顶部色带",
  "techniques": ["T4", "T3"],
  "prose": "人物卡片像名片一样排成 2 行网格，但绝不等高等宽 -- 关键人物的卡片更高更宽并悬浮起来，其余卡片用负 margin 微微侵入相邻区域制造层次感"
}
```

---

## 9. 架构/系统图页

### 示例 A：洋葱解构型

```json
{
  "mood": "层层剥开 / 由外而内的解构之旅",
  "spatial_strategy": "居中同心结构 + 四角标注文字",
  "anchor_treatment": "核心层用 accent 强调 + 外层递减透明度",
  "techniques": ["T6", "T9"],
  "prose": "系统架构像洋葱一样同心展开，核心层用 accent 色实心圆居中，外部每层透明度递减。四角散布着指向各层的标注文字，用 accent 脉冲锚点连接标注与对应层级"
}
```

### 示例 B：蓝图鸟瞰型

```json
{
  "mood": "技术蓝图 / 工程师的鸟瞰视角",
  "spatial_strategy": "从上到下的层级流动，每层横向展开",
  "anchor_treatment": "核心模块用 accent + elevated 突出，边缘模块用 outline",
  "techniques": ["T4", "T5"],
  "prose": "架构图像蓝图一样从上到下展开层级关系，每层用一条极细的 accent 斜切色带分隔。核心模块像浮岛一样用多层阴影凸起，边缘模块用 outline 样式安静地附着在主体周围"
}
```

---

## 10. 结束页

### 示例 A：收束镜像型

```json
{
  "mood": "圆满闭环 / 封面的收束镜像",
  "spatial_strategy": "与封面构图镜像：封面主体在左则结尾在右",
  "anchor_treatment": "核心 CTA 用 accent 色药丸按钮，回顾要点用极小字",
  "techniques": ["T7", "T1"],
  "prose": "封面的镜像倒影 -- 如果封面标题在左上，结尾的 CTA 就放在右下。画面保持封面同样的留白比例但构图翻转，底部角落用封面水印的变体（内容不同但尺寸位置呼应）收束全场"
}
```

### 示例 B：沉思余韵型

```json
{
  "mood": "沉思余韵 / 余音绕梁",
  "spatial_strategy": "60% 上方分散要点回顾 + 40% 下方聚焦 CTA",
  "anchor_treatment": "CTA 区域用 accent 渐变底色 + 白色大字行动号召",
  "techniques": ["T8", "T9"],
  "prose": "上方 60% 用散布的小卡片轻柔回顾 3-5 个核心要点，视觉重量很轻很克制。下方 40% 像深海一样用 accent 渐变色块沉淀，中间浮着白色的行动号召文字，用脉冲锚点标记联系方式"
}
```
