---
name: naval-almanack
description: |
  当用户询问《纳瓦尔宝典》这本书本身(书名/作者/章节/整书概览/术语)，或涉及本书未独立成 Skill 的主题时调用:
  幸福训练/欲望管理、接受与内耗、冥想与心猴、地位游戏与攀比、选朋友与圈子、打工vs创业(委托代理)、
  诚实沟通、身份与自我形象、期望与愤怒、生命意义、该不该信玄学(理性验证)、长期复利与声誉、判断力训练。
  以下意图优先使用同包独立 Skill: wealth-structure(致富/股权期权)、productize-yourself(职业定位/副业方向)、
  decision-heuristics(重大选择纠结)、hourly-rate-time(时薪/外包/时间分配)、reading-metaskill(怎么读书学习)、screen-detox(刷手机上瘾)。
  Triggers: 纳瓦尔/Naval/纳瓦尔宝典/幸福/冥想/内耗/攀比/圈子/复利/人生意义
metadata:
  generated_by: cangjie compile_single v0.1 (phase0 prototype)
  variant: router
  source_pack: books/naval-almanack-skill
  capability_count: 19
  entrypoint_count: 7
---
# 《纳瓦尔宝典：财富与幸福指南》 — 来源路由入口（compact pack）

## 触发与不触发

**适用**：与本书能力域相关的咨询与任务（见下方路由表的意图列）。
**不适用**：
- 具体投资标的分析、选股与理财产品建议
- 临床抑郁、精神疾病急性发作等需要专业治疗的场景（先就医）
- 急性自杀风险（先紧急求助）
- 需要外交辞令的谈判策略与具体薪酬数字

## 核心原则（常驻速览，概览类问题读到这里即可回答）

1. 财富=睡觉时仍在赚钱的资产；靠「责任→产权→杠杆」构建，无法靠出租时间致富。
2. 用特殊知识（对你像玩、对别人像工作）产品化自己，成为不可替代。
3. 判断力比努力重要；无法决定就答否；两条均等的路选短期更痛苦的那条。
4. 财富、知识、声誉、关系都遵循复利；只玩长期正和游戏，与能共事一辈子的人同行。
5. 幸福是可训练的技能：减少欲望、活在当下；任何处境只有三选项——改变/接受/离开。
6. 阅读是终极元技能：读你所爱、原著优先，每天 1-2 小时即进入极少数人行列。

## 能力路由（先读本表，按意图加载 1 张能力卡）

| 用户意图 | 先读 | 补读/备注 |
|---|---|---|
| 怎么赚钱/如何致富；该不该要期权/股权；评估一个生意或副业的结构 | references/capabilities/wealth-structure.md | 已晋级为独立 Skill `wealth-structure`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 职业/副业/自由职业方向纠结；我该做什么才能赚钱且不被替代；寻找独特优势 | references/capabilities/productize-yourself.md | 已晋级为独立 Skill `productize-yourself`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 换工作/买房/搬城/合伙/结婚等重大选择纠结；列了利弊表还是拿不定主意 | references/capabilities/decision-heuristics.md | 已晋级为独立 Skill `decision-heuristics`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 琐事/外包值不值得；时间分配与安排；离财务自由还差什么 | references/capabilities/hourly-rate-time.md | 已晋级为独立 Skill `hourly-rate-time`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 想养成阅读习惯；读什么书/怎么读；如何学习新领域/入门某学科 | references/capabilities/reading-metaskill.md | 已晋级为独立 Skill `reading-metaskill`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 刷手机/短视频/社交媒体上瘾；想戒断多巴胺零食 | references/capabilities/screen-detox.md | 已晋级为独立 Skill `screen-detox`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 需要方向性判断/这事靠谱吗；想提升决策能力/学心智模型 | references/capabilities/judgment-training.md | references/capabilities/reading-metaskill.md、references/capabilities/decision-heuristics.md |
| 要不要长期投入某段关系/某个项目；如何积累声誉与信任 | references/capabilities/long-term-compounding.md | references/capabilities/peer-selection.md、references/capabilities/wealth-structure.md |
| 为什么大公司磨洋工/小公司拼命；打工还是创业/该不该自己干 | references/capabilities/principal-agent.md | references/capabilities/judgment-training.md |
| 怎么才能更幸福；为什么得到了还不满足；怎么减少焦虑 | references/capabilities/happiness-skill.md | references/capabilities/acceptance.md、references/capabilities/monkey-mind-meditation.md |
| 为什么有人攻击我/大家卷来卷去；怎么不在乎别人眼光；为什么妒忌 | references/capabilities/game-selection.md | references/capabilities/happiness-skill.md、references/capabilities/long-term-compounding.md |
| 困在无法改变的处境里反复内耗；该忍还是该走 | references/capabilities/acceptance.md | references/capabilities/life-meaning.md、references/capabilities/happiness-skill.md |
| 被别人的期望压垮；容易愤怒/怎么不生气；感觉被工作困住/怎么说不 | references/capabilities/self-liberation.md | references/capabilities/happiness-skill.md、references/capabilities/honesty-communication.md |
| 脑子停不下来/焦虑反刍；想学冥想/提升专注力 | references/capabilities/monkey-mind-meditation.md | references/capabilities/happiness-skill.md、references/capabilities/rational-buddhism.md |
| 交朋友/选伴侣/换圈子纠结；感觉被周围人拖累/该不该疏远某人 | references/capabilities/peer-selection.md | references/capabilities/long-term-compounding.md、references/capabilities/honesty-communication.md |
| 发现自己在为立场辩护/看不清现实；反复立志却改变不了自己 | references/capabilities/identity-work.md | references/capabilities/acceptance.md、references/capabilities/honesty-communication.md |
| 怎么真诚沟通/拒绝别人/给反馈；不想说场面话/发现自己在隐瞒 | references/capabilities/honesty-communication.md | references/capabilities/identity-work.md |
| 某主张/玄学/灵修该不该信；想建立自己的验证标准 | references/capabilities/rational-buddhism.md | references/capabilities/life-meaning.md、references/capabilities/monkey-mind-meditation.md |
| 活着的意义是什么/虚无感；找不到方向/该为什么努力 | references/capabilities/life-meaning.md | references/capabilities/rational-buddhism.md、references/capabilities/acceptance.md |

**非能力类查询**：
- 书名/作者/章节/整书概览 → references/overview.md
- 术语解释 → references/glossary.md
- 决策规则速查（不需要原文依据时） → references/cheatsheet.md
- 完整意图与关键词索引（本表未覆盖的意图先查这里） → references/capability-index.md

## 加载规则

- 每次任务先读本文件，再按路由表加载 **1** 张能力卡；任务明确跨域时最多加载 2 张。
- 概览/书名类问题不加载能力卡，用「核心原则」与 overview.md 回答。
- 路由表与 capability-index.md 都无法命中的意图，明确告知超出本书范围，不要硬套。

## 边界与判停

- 意图在路由表与能力索引中均无法命中 → 明确告知超出本书范围，不硬套书中框架。
- 涉及医疗/法律/人身安全 → 停止套用方法论，建议专业求助。
- 纯事实查询（书名/作者/出版年/章节）→ 用核心原则与 overview 回答后即停，不展开人生建议。
