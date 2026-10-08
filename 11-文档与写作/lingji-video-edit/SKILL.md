---
name: lingji-video-edit
description: 当需要直接编辑已有灵机剪影项目的视频内容——调整 overlay 时间位置（startMs/durationMs）、进出场动画（motion.enter/exit）、画面坐标、文字/字幕样式、或修改 Motion Card 动画源码（motionCard.tsx）时使用。通过 file-first 直接编辑 project.json 与 ai-cards/<overlayId>/motionCard.tsx，编辑器会实时热重载预览。不用于从稿件创建项目、重新生成封面/卡片配图/配音或导出 MP4。
version: 1.0.0
user-invocable: false
---

# 灵机剪影 · 视频 file-first 编辑

通过直接读写项目文件，修改时间线 overlay 动画/时间/位置/样式，或编辑 Motion Card 动画源码。编辑器有热重载钩子，改完自动刷新预览，无需操作运行中的 App。

若用户要从普通稿件/素材目录一路推进到灵机剪影视频项目，应先使用用户级 `lingji-video-workflow` 总入口；本 skill 只处理已有项目里的视频时间线与 Motion Card 直改。

详细字段契约：[`docs/ai-contract/video-editing.md`](../../../docs/ai-contract/video-editing.md)
锁/结果协议全文：[`docs/ai-contract/README.md`](../../../docs/ai-contract/README.md)

## 能改什么 / 不能干什么

| 能改 | 不能动 |
|---|---|
| `timeline.overlays[i].startMs` / `durationMs` | 触发 TTS / 重新配音 |
| `timeline.overlays[i].position`（x / y / width / height） | 重新生成封面 / 卡片配图 / AI 画图 |
| `timeline.overlays[i].motion`（enter / exit / loop 及时长） | 导出 MP4（让用户在 App 内执行） |
| `timeline.overlays[i].textData`（文字内容、字体、颜色、动画） | 改 `podcast-audio.mp3` / `*.srt`（产物，勿碰） |
| `timeline.overlays[i].audioData`（音量、淡入淡出） | 改 `covers/` / `ai-cards/<id>/image.png`（产物） |
| `timeline.subtitle`（字幕全局样式） | 碰 `aiAnalysis` / `script` 段（那是其它域） |
| `ai-cards/<overlayId>/motionCard.tsx`（Motion Card 源码） | 在 `project.json` 里写 tsx 源码（那里只存 tsxPath 指针） |

## file-first 三步流程

**第 1 步：写锁。** 编辑任何文件前，先写 `<projectDir>/.lingji/edit-lock.json`：

```json
{
  "owner": "codex",
  "scope": "video",
  "startedAt": 1718260000000,
  "heartbeat": 1718260000000,
  "ttlMs": 30000
}
```

锁定期间编辑器暂停自动保存，状态栏显示「AI 正在编辑」，避免内存态覆盖你的修改。若编辑超过约 15s，用当前 epoch 毫秒更新 `heartbeat` 字段重写文件（心跳间隔要小于 `ttlMs`）。

**第 2 步：编辑文件。** 根据任务类型：

- **改时间线**：编辑 `<projectDir>/project.json` 的 `timeline.overlays[i]` 或 `timeline.subtitle` 段。保持 JSON 合法，不要动 `aiAnalysis` 和 `script` 段，也不要改 overlay 的 `id` 字段。
- **改 Motion Card 源码**：编辑 `<projectDir>/ai-cards/<overlayId>/motionCard.tsx`，不要在 `project.json` 里写 tsx 源码（那里只有 `tsxPath` 指针）。

**第 3 步：删锁，查结果。** 编辑完成后删除 `.lingji/edit-lock.json`。若改了 `project.json`，读 `.lingji/edit-result.json` 确认 `ok:true`；若 `ok:false`，按 `errors[].field` / `errors[].message` 定位并重写修复，直到通过。（改 `motionCard.tsx` 不产生 `edit-result.json`，保存后编辑器直接重编译刷新预览。）

## 动画枚举速查

改 `motion.enter` / `motion.exit` 必须用下列值，其它值会被校验拒绝：

- **enter（进场）**：`none` · `fadeIn` · `slideInLeft` · `slideInRight` · `slideInUp` · `slideInDown` · `scaleIn` · `bounceIn`
- **exit（出场）**：`none` · `fadeOut` · `slideOutLeft` · `slideOutRight` · `slideOutUp` · `slideOutDown` · `scaleOut` · `bounceOut`
- **loop**（overlay）：`none` · `pulse` · `float` · `flicker`
- **loop**（textData.animation，多一个）：`none` · `pulse` · `float` · `flicker` · `typewriter`

## motionCard.tsx 硬约束

1. 文件是纯 TSX 源码，首尾不要写 ` ```tsx ` code fence。
2. 必须有 `export default` 一个 React 函数组件且渲染真实 JSX（不能 `return null`）。
3. 优先用 `useCurrentFrame()` 驱动动画（帧驱动），可用 `interpolate` / `spring` / `<AbsoluteFill>` 等 Remotion API。
4. 组件须纯函数，无副作用，无外部网络请求（预览与导出同一份编译产物）。

## 典型示例

**把某 overlay 进场改为 fadeIn、时长 800ms：**

在 `project.json` 找到目标 `timeline.overlays[i]`（通过 `id` 定位），修改或新增 `motion` 段：

```jsonc
"motion": {
  "enter": "fadeIn",
  "enterDurationMs": 800,
  "exit": "fadeOut",
  "exitDurationMs": 600,
  "loop": "none"
}
```

改完读 `.lingji/edit-result.json` 确认 `ok:true`。

**调整 Motion Card 动画：**

直接编辑 `<projectDir>/ai-cards/<overlayId>/motionCard.tsx`，改 `interpolate` 的输入帧区间等动画逻辑，保存即可。不要改 `project.json`。
