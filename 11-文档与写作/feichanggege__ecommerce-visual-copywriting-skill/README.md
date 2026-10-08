<div align="center">

# 电商视觉文案设计 Skill

**中文** · [English](README.en.md)

> 把商品资料磨成能落图的主图、详情页分镜脚本、图内文案、设计说明和生图 Prompt。

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-ecommerce--visual--copywriting-blueviolet)](SKILL.md)
[![skills.sh](https://skills.sh/b/feichanggege/ecommerce-visual-copywriting-skill)](https://skills.sh/feichanggege/ecommerce-visual-copywriting-skill)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Verify](https://img.shields.io/badge/verify-python%20tools%2Fverify--skill.py-2ea44f)](tools/verify-skill.py)

**输入产品资料、资质边界、平台要求和视觉参考，先输出视觉策划案与分镜脚本，用户确认后再交付可执行的画面、图内文案、设计说明和生图 Prompt。**

[效果展示](#效果展示) · [快速开始](#快速开始) · [触发方式](#触发方式) · [能交付什么](#能交付什么) · [安全边界](#安全边界) · [验证](#验证)

</div>

---

## 效果展示

这些图展示本 Skill 面向的最终执行形态：主图视觉截流、多角度产品视图、场景详情页、质感卖点表达。

<div align="center">
  <table>
    <tr>
      <td align="center" width="50%">
        <a href="assets/showcase/musang-king-durian-main.jpg">
          <img src="assets/showcase/thumbs/musang-king-durian-main-thumb.jpg" alt="猫山王榴莲主图" width="360" height="360">
        </a><br>
        <strong>高端食品主图</strong>
      </td>
      <td align="center" width="50%">
        <a href="assets/showcase/figure-multi-angle.png">
          <img src="assets/showcase/thumbs/figure-multi-angle-thumb.jpg" alt="手办多角度产品视图" width="360" height="360">
        </a><br>
        <strong>多角度产品视图</strong>
      </td>
    </tr>
    <tr>
      <td align="center" width="50%">
        <a href="assets/showcase/figure-desktop-scene.png">
          <img src="assets/showcase/thumbs/figure-desktop-scene-thumb.jpg" alt="桌面陈列场景图" width="360" height="360">
        </a><br>
        <strong>桌面场景详情</strong>
      </td>
      <td align="center" width="50%">
        <a href="assets/showcase/lumina-pendant-lamp.png">
          <img src="assets/showcase/thumbs/lumina-pendant-lamp-thumb.jpg" alt="玻璃吊灯质感主图" width="360" height="360">
        </a><br>
        <strong>家居灯具质感表达</strong>
      </td>
    </tr>
  </table>
</div>

> 展示图用于说明视觉交付形态，不构成对应商品的法律意见、平台审核承诺或商业数据证明。

---

## 它解决什么问题

你让 AI 写电商图文，最常见的翻车不是“不够华丽”，而是三件事：

- 直接写最终文案，方向错了才返工。
- 卖点听起来猛，但碰到广告法、功效宣称或平台审核红线。
- 输出只有文案，没有画面、层级、材质光影、免责声明和设计执行说明。

这个 Skill 把电商视觉文案拆成一个有门禁的 SOP：先判断转化驱动力，锁定统一视觉风格规范，再输出主图和详情页分镜脚本；用户确认后，才进入最终画面、图内文案、设计说明和生图 Prompt。

## 快速开始

```bash
npx skills add feichanggege/ecommerce-visual-copywriting-skill
```

装完后，对 Agent 说：

```text
帮我把这个产品资料做成一套电商主图和详情页视觉策划方案。先给视觉策划案和分镜脚本，等我确认后再继续出最终文案和生图 Prompt。
```

### 手动安装

如果你的 Agent runtime 还不支持 `npx skills add`，可以手动复制：

```bash
git clone https://github.com/feichanggege/ecommerce-visual-copywriting-skill.git
mkdir -p ~/.codex/skills/ecommerce-visual-copywriting
cp -r ecommerce-visual-copywriting-skill/SKILL.md ecommerce-visual-copywriting-skill/SKILL.en.md ecommerce-visual-copywriting-skill/references ecommerce-visual-copywriting-skill/examples ~/.codex/skills/ecommerce-visual-copywriting/
```

Windows PowerShell：

```powershell
git clone https://github.com/feichanggege/ecommerce-visual-copywriting-skill.git
New-Item -ItemType Directory -Force "$env:USERPROFILE\.codex\skills\ecommerce-visual-copywriting" | Out-Null
Copy-Item -Recurse ecommerce-visual-copywriting-skill\SKILL.md,ecommerce-visual-copywriting-skill\SKILL.en.md,ecommerce-visual-copywriting-skill\references,ecommerce-visual-copywriting-skill\examples "$env:USERPROFILE\.codex\skills\ecommerce-visual-copywriting\"
```

## 语言切换

- 中文：使用 `SKILL.md`。
- 英文版：查看 [README.en.md](README.en.md) 和 [SKILL.en.md](SKILL.en.md)。
- 运行时可直接对 Agent 说：“用英文输出”或 “Output in English”。

## 触发方式

你可以这样说：

- “帮我做一套淘宝主图和详情页文案”
- “先给我主图分镜脚本，不要直接写最终文案”
- “这个产品怎么提炼卖点做主图”
- “按抖音小店风格给我出视觉脚本”
- “这个详情页有没有广告法风险”
- “设计师要开工，给我画面 + 图内文案 + 设计说明”
- “输出英文版电商主图策划”

## 能交付什么

| 场景 | 交付物 | 关键门禁 |
|---|---|---|
| 视觉策划 | 转化驱动力、风格锁、利益翻译表、合规边界 | 先确认方向 |
| 主图文案 | 5 张主图的视觉任务、构图、图内文案、设计说明 | 主图每张不超过 5 行 |
| 详情页 | 首屏、核心利益、材质工艺、场景、证据、FAQ、CTA | 按成交路线组织 |
| 生图 Prompt | 主体、构图、光影、材质、背景、限制项 | 继承统一视觉风格规范 |
| 合规审查 | 高风险词、替换方向、免责声明 | 不编造资质和证据 |
| 自审评分 | 合规性、利益翻译度、视觉第一落点、触感/媒介表达、叙事连贯性 | 任一维度低于 80 重写 |

## 它和普通 AI 写文案有什么不同

| 维度 | 普通 AI 文案 | 本 Skill |
|---|---|---|
| 输出顺序 | 直接生成最终稿 | 先视觉策划，再分镜脚本，确认后执行 |
| 输出形态 | 一段广告文案 | 画面 + 图内文案 + 设计说明 + Prompt |
| 主图逻辑 | 卖点堆砌 | 5 张图各有视觉任务 |
| 详情页逻辑 | 资料堆砌 | 按吸引、欲望、信任、行动推进 |
| 合规边界 | 常混用功效、绝对化、医疗化表达 | 按普通食品、保健食品、运动器材等分层检查 |
| 质量门 | 看起来顺就结束 | 五维独立自审，低于 80 必须重写 |

## 安全边界

这个 Skill 会：

- 基于用户提供的产品资料、资质、检测报告编号和平台要求写作。
- 对不确定资质标注“缺失”，不把未经验证的信息写成事实。
- 对普通食品、运动器材、保健食品使用不同合规边界。
- 在视觉策划和分镜脚本阶段暂停，等用户确认后再进入最终稿。

这个 Skill 不会：

- 代替律师、平台小二或监管机构给出最终法律结论。
- 编造检测报告、专利号、批准文号、销量、用户评价或功效证明。
- 在用户未确认资质时宣传治疗、保健、改善身体功能等高风险卖点。
- 自动发布、上架、发送给设计师或改动店铺后台。

## 文件结构

```text
SKILL.md                         # 中文核心工作流
SKILL.en.md                      # 英文版本
README.md                        # 中文首页
README.en.md                     # English README
references/compliance-rules.md   # 分品类合规规则库和替换表
examples/README.md               # 可复用输入与输出样例
assets/showcase/                 # 效果图展示
assets/showcase-output.svg       # 结构化输出展示卡
tools/verify-skill.py            # 发布前结构和隐私检查
docs/skill-polishing-report.md   # 鲁班打磨记录和对标来源
```

## 验证

本仓库提供一个零依赖检查脚本：

```bash
python tools/verify-skill.py
```

它会检查：

- `SKILL.md` frontmatter、触发词、暂停条件和五维自审。
- `SKILL.en.md` 是否存在并包含英文工作流。
- README 是否包含中文入口、英文切换、效果展示、安全边界和验证说明。
- 展示图、示例、合规规则、marketplace 元数据是否存在。
- 仓库文本是否包含常见 token、cookie、私有路径等泄露风险。

## 适用平台

默认面向支持 Agent Skills 的运行环境，也可手动迁移到 Claude Code、Codex、OpenClaw、Cursor、Windsurf、Continue 等支持自定义指令的工具。

不同平台的加载目录可能不同；真正生效以你的 Agent runtime 文档为准。

## License

[MIT](LICENSE)

---

<div align="center">

*先过视觉策划门，再谈转化率。*

</div>
