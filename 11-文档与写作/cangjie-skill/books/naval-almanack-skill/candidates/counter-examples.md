# Counter-Example Candidates — 《纳瓦尔宝典》

> 提取方式: cangjie-skill 阶段 1 降级方案 (主流程串行执行 counter-example-extractor)
> 提取日期: 2026-08-01

- id: ce01
  title: 为别人工作，永远拿最低报酬
  type: counter-example
  source_chapter: 第一章·财富 / 第一节 创造财富 / 认识如何创造财富
  source_quote: |
    实质上，你是在为别人工作，而那个人承担了风险，承担了责任，拥有知识产权和品牌。他们不会给你足够的报酬。他们会支付让你为他们工作的最低限度的报酬。
  failure_mode: |
    只有工资、没有产权：睡觉时没有收入，退休即断粮，永远无法非线性赚钱——即使工资很高。
  mechanism: |
    承担风险/责任/知识产权的人拿走超额回报，雇员只拿市场最低价；
    时间与收入线性挂钩，杠杆为零。
  warning_signs:
    - 收入完全按小时或年薪计，与产出脱钩无份
    - 没有任何股权/知识产权/品牌积累
  bound_to:
    - "拥有产权，别出租时间"
    - "产品化你自己"
  tags: [counter-example, employment, wealth]

- id: ce02
  title: 暗地鄙视财富与比较心态
  type: counter-example
  source_chapter: 第一章·财富 / 第一节 创造财富 / 认识如何创造财富
  source_quote: |
    如果你养成了比较的心态，你就总是会去讨厌做得比你好的人，你就总是会妒忌或羡慕他们。……毫不夸张地说，反财富将阻碍你致富。
  failure_mode: |
    一边想赚钱一边鄙视财富：与能人交易时泄露敌意，被对方察觉，机会流失。
  mechanism: |
    比较心态→妒忌/敌意→人际信号泄漏（人类有共情能力）；心态与目标冲突，精神状态不对。
  warning_signs:
    - 谈起有钱人时下意识贬低
    - 看到别人成功先难受而不是研究
  bound_to:
    - "不要暗地鄙视财富"
    - "乐观者做得更好"
  tags: [counter-example, mindset, envy]

- id: ce03
  title: 沉迷地位游戏
  type: counter-example
  source_chapter: 第一章·财富 / 第一节 创造财富 / 认识如何创造财富
  source_quote: |
    问题是，为了获胜，你必须把别人击倒。这是为什么要在生活中避免玩地位游戏——它们会让你变成愤怒、好斗的人。你总是在窝里斗，损人利己。
  failure_mode: |
    把精力投在排名争夺上：赢家必须击倒别人，人变得愤怒好斗，且奖品本身是零和的。
  mechanism: |
    地位是等级中的相对位置，本质零和；长期参与会重塑人格为攻击型。
  warning_signs:
    - 经常比较「谁更强/谁更有名」
    - 把同事/同行当对手而非伙伴
  bound_to:
    - "地位游戏 vs 财富游戏"
    - "单人游戏与内在记分卡"
  tags: [counter-example, status, zero-sum]

- id: ce04
  title: 与愤世嫉俗者和悲观者合作
  type: counter-example
  source_chapter: 第一章·财富 / 第一节 创造财富 / 认识如何创造财富
  source_quote: |
    不要和愤世嫉俗者、悲观主义者合作。他们会任由坏事发生，以证明他们负面看法是正确的。
  failure_mode: |
    合作者潜意识希望项目失败来印证自己的悲观，在最需要推进时拆台。
  mechanism: |
    自我实现预言：负面信念让人停止努力，然后「果然如此」；
    愤世嫉俗是低成本的心理防御，却会传染整个团队。
  warning_signs:
    - 对方常把新想法先否定一遍
    - 对方把失败当谈资炫耀
  bound_to:
    - "选择高智商、精力旺盛且正直的伙伴"
    - "五只黑猩猩"
  tags: [counter-example, relationship, negativity]

- id: ce05
  title: 没想清楚就盲目努力
  type: counter-example
  source_chapter: 第一章·财富 / 第一节 创造财富 / 认识如何创造财富
  source_quote: |
    如果你还不知道应该做什么，那么最重要的事就是先去搞清楚。没有弄清楚应该做什么之前，不应该过分盲目地努力。
  failure_mode: |
    每周 80 小时在餐馆打工式努力：方向未定先拼命，努力与财富几乎无关。
  mechanism: |
    把「努力」误当「致富」；努力只是执行层，做什么/和谁做才是决定层。
  warning_signs:
    - 忙到没时间想「我为什么做这个」
    - 用工作时长证明自己的价值
  bound_to:
    - "判断力优于努力"
    - "与谁工作、做什么比努力更重要"
  tags: [counter-example, effort, direction]

- id: ce06
  title: 熟记高级概念却不理解基础
  type: counter-example
  source_chapter: 第一章·财富 / 第二节 培养判断力 / 如何清晰地思考
  source_quote: |
    如果你不能根据需要从基础知识重新推导出概念，你会迷惑不解。你只是熟记而已。
  failure_mode: |
    满口大词、掌握一堆无法重建的高级概念：一旦换个场景就无法判断真伪，被伪科学收割。
  mechanism: |
    知识没有锚点（基础公理），记忆与理解脱节；
    因为害怕数学而无法独立评判，反而高估用数学/伪科学包装的观点。
  warning_signs:
    - 能引用术语但讲不出推导链
    - 复杂词汇堆砌、无法向小孩讲明白
  bound_to:
    - "从基础重建（第一性原理式思考）"
    - "基础比深钻更重要"
  tags: [counter-example, cognition, fake-knowledge]

- id: ce07
  title: 身份认同锁死认知
  type: counter-example
  source_chapter: 第一章·财富 / 第二节 培养判断力 / 如何清晰地思考
  source_quote: |
    我曾经认为自己是自由主义者，但后来我发现我自己为我没有真正思考过的立场辩护，完全因为它们是自由主义经典的一部分。如果你所有的信念都能整齐地捆成一堆，你应该要高度怀疑了。
  failure_mode: |
    给自己贴标签后为标签辩护：立场先行、事实靠后，看不清真相也改不了方向。
  mechanism: |
    身份产生捍卫动机；信念成捆一致=从未独立检验（被部落灌输）。
  warning_signs:
    - 发现自己为「我方立场」找理由而非看证据
    - 信仰清单整齐划一、毫无内部矛盾
  bound_to:
    - "身份清空"
    - "从基础重建"
  tags: [counter-example, identity, bias]

- id: ce08
  title: 欲望遮蔽现实
  type: counter-example
  source_chapter: 第一章·财富 / 第二节 培养判断力 / 如何清晰地思考
  source_quote: |
    第一件蒙蔽我们看清现实的事是我们对现实先入为主的观念。……你生意失败，你痛苦万分，因为你迟迟不认清现实。你一直对自己隐瞒现实。
  failure_mode: |
    因为希望某结果发生，无视相反信号：生意在走下坡却坚信做得很好，直到痛苦逼你认账。
  mechanism: |
    愿望与现实冲突时，大脑优先保护愿望；自我越强，越难面对真相。
  warning_signs:
    - 回避数据/反馈，只收集支持自己的信息
    - 对坏消息第一反应是辩解
  bound_to:
    - "痛苦时刻即真相时刻"
    - "身份清空"
  tags: [counter-example, reality, self-deception]

- id: ce09
  title: 大脑本能逃避短期痛苦
  type: counter-example
  source_chapter: 第一章·财富 / 第二节 培养判断力 / 往高处走
  source_quote: |
    你的大脑想要避免冲突而试图去避开短期的痛苦。……你必须在感到痛苦时摆脱这种倾向（它是一种潜意识的倾向）。
  failure_mode: |
    两个均等选择之间永远选「短期不痛」的那个，长期收益全部流失。
  mechanism: |
    大脑过度看重短期幸福（进化残留），回避短期痛苦；
    而按复利规律，短期痛苦恰好对应长期收益。
  warning_signs:
    - 选工作/项目时下意识挑「轻松」的
    - 每次读书/健身都在「等一下」中度过
  bound_to:
    - "短期痛苦原则"
    - "复利游戏"
  tags: [counter-example, bias, discipline]

- id: ce10
  title: 赚钱后升级生活方式
  type: counter-example
  source_chapter: 第一章·财富 / 第一节 创造财富 / 找到感觉像玩的工作
  source_quote: |
    对钱的贪婪对我们来说是不好的，因为它是一个无底洞。它将永远占据你的心灵。如果爱钱，并能赚钱，将永远无法得到满足了。
  failure_mode: |
    收入上升消费同涨：永远缺钱、永远焦虑、财务自由无限推迟。
  mechanism: |
    欲望一旦开启不会在某个数字停止；享乐适应让人造物品的满足迅速衰减。
  warning_signs:
    - 收入涨了就换车换房换生活方式
    - 储蓄率常年为零
  bound_to:
    - "别升级生活方式"
    - "退休定义"
  tags: [counter-example, lifestyle, desire]

- id: ce11
  title: 多巴胺零食（社交媒体/新闻/游戏）
  type: counter-example
  source_chapter: 第一章·财富 / 第二节 培养判断力 / 学会爱上阅读
  source_quote: |
    我只是整日吃着小多巴胺的零食。我得到了我小140字的多巴胺。我发推特，然后留意谁转推了我的推文。这是有趣和令人高兴的事情，但这是我在玩的一个游戏。
  failure_mode: |
    用碎片刺激代替深度输入：注意力变短、学不到东西、幸福感下降。
  mechanism: |
    平台按即时奖赏设计，把长期后果换成短期快感；
    屏幕活动与更少幸福相关（作者断言「没有例外」）。
  warning_signs:
    - 一有空就刷手机，坐下静不下来
    - 看书超过十分钟就难受
  bound_to:
    - "少用屏幕，多做非屏幕活动"
    - "每天阅读一至两小时"
  tags: [counter-example, digital, dopamine]

- id: ce12
  title: 干坐着希望事情改变
  type: counter-example
  source_chapter: 第二章·幸福 / 第一节 学习幸福 / 在接受中寻找幸福
  source_quote: |
    一个不好的选择是，干坐着，希望你可以改变它，但你却没有改变它；希望你可以离开它，但你却没有离开它，也没接受它。这种挣扎或厌恶是我们大多数痛苦的原因。
  failure_mode: |
    在「改变/接受/离开」三选项之外的选择：既不改变、也不离开、更不接受，只是干等。
  mechanism: |
    逃避选择的代价被延迟支付，变成长期慢性痛苦；挣扎消耗的精力远超任何单一选项。
  warning_signs:
    - 反复抱怨同一件事却不动手
    - 心里清楚该离开/该接受却一直耗着
  bound_to:
    - "情境三选项（改变/接受/离开）"
    - "接受"
  tags: [counter-example, avoidance, suffering]

- id: ce13
  title: 嘴上说要改，实际在拖延
  type: counter-example
  source_chapter: 第二章·幸福 / 第二节 自救 / 选择去塑造你自己
  source_quote: |
    当你说“我要做这个”和“我要做那个”的时候，你其实是在拖延。你在给你自己找借口。……至少如果你有自知之明，你可以想，“我说我想去做这个，但其实我不想去做，因为如果我真的想做这个，我就会去做了。”
  failure_mode: |
    用「我打算/我要」代替行动：宣称要戒烟、要锻炼，却永远停留在准备阶段。
  mechanism: |
    语言承诺制造改变的假象，释放行动压力；
    真正的检验是行为本身——想改变就立刻做，不想就诚实承认并设小目标。
  warning_signs:
    - 反复说「我明天开始」「等这个忙完」
    - 对外只承诺「试试」而非具体行动
  bound_to:
    - "更新自我形象以替代自律"
    - "行动迅速，对结果耐心"
  tags: [counter-example, procrastination, self-deception]

- id: ce14
  title: 妒忌
  type: counter-example
  source_chapter: 第二章·幸福 / 第一节 学习幸福 / 妒忌是幸福的敌人
  source_quote: |
    妒忌是我很难克服的一种情绪。……它是一种如此令人厌恶的情绪，因为在一天结束时，并不会因为妒忌而过得更好。你是不快乐，但你妒忌的人仍然会成功。
  failure_mode: |
    为别人的成功付费：自己不快乐，被妒忌的人毫无损失，关系还受损。
  mechanism: |
    试图只复制他人某几个片段（身材/钱/气质）而拒绝整个身份——「要成为那个人」才是完整交换，
    不愿全换则妒忌无意义。
  warning_signs:
    - 看到别人成功先感到刺痛
    - 希望别人失败来证明自己
  bound_to:
    - "单人游戏与内在记分卡"
    - "幸福是默认状态"
  tags: [counter-example, envy, happiness]

- id: ce15
  title: 孤注一掷与破产风险
  type: counter-example
  source_chapter: 第一章·财富 / 第一节 创造财富 / 认识如何创造财富
  source_quote: |
    远离那些事会导致你失去所有资本和所有积蓄的事情。不要做孤注一掷的事情。相反，要理性乐观地押注潜力大的东西。
  failure_mode: |
    把全部资本押在单次机会上：一次失败出局，失去复利资格；违法则进监狱。
  mechanism: |
    破产=失去游戏资格（没有本金、没有时间、甚至失去自由）；
    孤注一掷把概率压向尾部灾难。
  warning_signs:
    - 决策时没有「输光了怎么办」的预案
    - 用「这次一定成」给自己壮胆
  bound_to:
    - "避免破产与永久性损失"
    - "复利游戏"
  tags: [counter-example, risk, ruin]

- id: ce16
  title: 读错次序的博学
  type: counter-example
  source_chapter: 第一章·财富 / 第二节 培养判断力 / 学会爱上阅读
  source_quote: |
    我认为在我生活中遇到的那些非常博学的人，并不是很聪明。主要是因为虽然他们非常博学，但他们以错误的次序阅读了错误的东西。他们一开始读的是一系列虚假的或部分真实的东西。
  failure_mode: |
    大量阅读但地基是二手/错误内容：世界观公理本身就歪，新想法全部被错误框架过滤。
  mechanism: |
    知识结构像建筑：基础决定承重；从解读本/伪科学起步，会积累无法修正的偏见。
  warning_signs:
    - 读了上百本「关于 X」的书却没读 X 的原著
    - 知识面广但一问底层逻辑就糊
  bound_to:
    - "读原著与经典，忽略同代人"
    - "从基础重建"
  tags: [counter-example, reading, foundation]

- id: ce17
  title: 为获得社会认可而读书
  type: counter-example
  source_chapter: 第二章·幸福 / 第二节 自救 / 选择去塑造你自己
  source_quote: |
    我觉得现在人们读的所有东西几乎都是为了得到社会认可而设计的。……社会认可是群体内部的。如果你想得到社会认可，一定要去读群体正在读的东西。
  failure_mode: |
    读「大家都在读」的书、追随大众品味：学到的是共识而非真知，且失去独立思考。
  mechanism: |
    社会认可是内部游戏，羊群逻辑保证你不会落后但也不会领先；
    生活回报在脱离群体的一侧。
  warning_signs:
    - 选书先问「这书别人听过吗」
    - 读书目标之一是「能聊到一块」
  bound_to:
    - "读原著与经典，忽略同代人"
    - "通过真实性避开竞争"
  tags: [counter-example, reading, conformity]
