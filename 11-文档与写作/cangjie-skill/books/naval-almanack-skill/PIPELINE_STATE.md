# PIPELINE_STATE — 《纳瓦尔宝典：财富与幸福指南》

> 断点续跑用。每完成一个阶段更新此文件。

- **书**: 《纳瓦尔宝典：财富与幸福指南》 (Naval Ravikant / Eric Jorgenson, 2020)
- **源文件**: `/Users/kangarooking/Downloads/《纳瓦尔宝典财富和幸福指南》.md` (240 KB / 2863 行)
- **产物目录**: `books/naval-almanack-skill/`
- **流水线**: RIA-TV++ (cangjie-skill)

## 当前阶段

- [x] **阶段 0: Adler 整书理解** → `BOOK_OVERVIEW.md` (已完成, 用户于 2026-08-01 确认「骨架 OK，按预估方向继续」)
- [x] **阶段 1: 5 个 extractor 并行提取** → `candidates/` (已完成; 因 sub-agent 消息投递故障, 采用降级方案由主流程串行执行, 产出 126 条候选: frameworks 25 / principles 49 / cases 17 / counter-examples 17 / glossary 18)
- [x] **阶段 1.5: 三重验证筛选** → `verified.md` + `rejected/` (19 个通过 + 7 个淘汰, 用户确认「开干吧」)
- [x] **阶段 2: RIA++ 构造 skill** → 19 个 skill 的 `SKILL.md` (已完成, 含六段 R/I/A1/A2/E/B)
- [x] **阶段 3: Zettelkasten 链接** → `INDEX.md` + `GLOSSARY.md` (已完成, related_skills 已回填)
- [x] **阶段 4: 压力测试 (darwin 兼容)** → 19 个 `test-prompts.json` + `test-results.md` (已完成, 全部 100% 通过; 降级自测方式已注明)
- [x] **阶段 5: 交付** → `DIGEST.md` 已生成; 19 个 skill 已安装到 `/Users/kangarooking/Desktop/mygGit/codex-DeepSeek-v4/cangjie-skill-test/.claude/skills/` 并完成完整性校验 (19/19 frontmatter 有效, test-prompts.json 全部合法)

## 下一步

全部完成。可选后续: 在 Claude Code/Codex 宿主中做一次真实触发验证, 或接入 darwin-skill 自动进化 (darwin evolve books/naval-almanack-skill/)。
