---
name: lingji-script-edit
description: 当需要直接编辑已有灵机剪影项目文稿——改写、扩写、调整 script.md（口播成稿）或 original.md（原始素材）时使用。file-first 直接编辑这两个 Markdown 文件，编辑器灌回脚本工作台并自动补建版本历史。不用于从稿件创建项目、触发 App 内 AI 写稿/审稿管线，不碰时间线/卡片/配音。
version: 1.0.0
user-invocable: false
---

# 灵机剪影 · 文稿 file-first 编辑

通过直接读写 `script.md` / `original.md`，修改口播成稿或原始素材。编辑器有热重载钩子，保存后内容自动灌回脚本工作台并补建版本历史，无需操作运行中的 App。

若用户要从普通稿件/素材目录一路推进到灵机剪影视频项目，应先使用用户级 `lingji-video-workflow` 总入口；本 skill 只处理已有项目里的文稿文件直改。

详细契约：[`docs/ai-contract/script-editing.md`](../../../docs/ai-contract/script-editing.md)
锁/结果协议全文：[`docs/ai-contract/README.md`](../../../docs/ai-contract/README.md)

## 能改什么 / 不能干什么

| 能改 | 不能动 |
|---|---|
| `<projectDir>/script.md`（口播成稿，改写/扩写/润色） | 触发 App 内 AI 写稿 / 审稿 / 批注管线 |
| `<projectDir>/original.md`（原始素材，改写/补充） | 改 `project.json`（timeline / aiAnalysis，那是视频域） |
| — | 改 `podcast-audio.mp3` / `podcast-subtitles*.srt`（产物，勿碰） |
| — | 改 `covers/` / `ai-cards/`（视频域产物） |

> `script.md` 改完后，若需要重新出口播音频/字幕，请让用户在 App 内执行 TTS；本 skill 只做文稿本身的外部直接编辑。

## file-first 三步流程

**第 1 步：写锁。** 编辑文件前，先写 `<projectDir>/.lingji/edit-lock.json`：

```json
{
  "owner": "codex",
  "scope": "script",
  "startedAt": 1718260000000,
  "heartbeat": 1718260000000,
  "ttlMs": 30000
}
```

锁定期间编辑器暂停自动保存，状态栏显示「AI 正在编辑」，避免内存态覆盖你的修改。若编辑超过约 15s，用当前 epoch 毫秒更新 `heartbeat` 字段重写文件（心跳间隔要小于 `ttlMs`，默认 30s，建议约每 10s 续一次）。

**第 2 步：编辑 Markdown 文件。** 直接读写 `<projectDir>/script.md` 或 `<projectDir>/original.md`。它们是纯 Markdown，无 JSON 格式约束，正常改写即可。

**第 3 步：删锁。** 编辑完成后删除 `.lingji/edit-lock.json`。

> 文稿域**不产生** `.lingji/edit-result.json`（JSON 校验仅用于视频域的 `project.json`），无需查结果文件。

## 保存后的自动行为

- **`script.md` 外部保存后**：编辑器把新内容灌回脚本工作台，并自动补建一条版本历史（`source` 标为 `external`）。不需要手动触发版本存档。
- **`original.md` 外部保存后**：同样被监听并反映到工作台对应标签。

## 典型示例

**润色口播成稿：**

1. 写锁（`scope:"script"`）。
2. 读取 `<projectDir>/script.md`，按用户要求修改内容，写回文件。
3. 删锁。编辑器自动灌回工作台并补版本历史。

**补充原始素材：**

1. 写锁（`scope:"script"`）。
2. 读取 `<projectDir>/original.md`，追加或修改素材内容，写回文件。
3. 删锁。
