# 架构

```text
用户内容 / SRT / 参考图
        ↓
单一 Skill 路由与脚本设计
   ┌────┴────────────┐
逐笔故事             程序动画
vendor MIT 后端       HTML/SVG/HyperFrames
   └────┬────────────┘
旁白 + 时间轴 + 确定性字幕/关键词
        ↓
最终 MP4 + 可编辑资产 + 验证报告
```

`vendor/srt-whiteboard-animation/` 是运行时库，不是第二个 Skill：没有 `SKILL.md`、agent 元数据或独立触发规则。这样既保留连续笔迹能力，也避免上游交互流程与主 Skill 冲突。来源和修改边界见其 `UPSTREAM.md` 与 MIT `LICENSE`。

样式层、音频层和渲染层彼此独立。换画风不应自动换声音；重做一幕也不应重新生成已经通过的旁白。
