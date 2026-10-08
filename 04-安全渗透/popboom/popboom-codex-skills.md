# PopBoom 视频生成平台 - MCP Tools

## 基本信息

- **MCP Server 端点**: `https://tkvideo.zbycorp.com:5002/mcp`
- **传输协议**: Streamable HTTP
- **认证方式**: Bearer Token
- **API Key**: 仅由 Codex 现有安全 MCP 配置提供，不在文档、日志、脚本或台账中写入明文

---

## 获取 API Key

1. 登录 PopBoom 平台
2. 进入「开放平台」→「我的 API Keys」
3. 点击「创建 Key」，输入名称后创建
4. **立即复制并保存 Key 值**（仅显示一次，关闭后无法再次查看）

---

## 可用工具

### 1. generate_video — 视频生成

提交 AI 视频生成任务。生成过程约 1-3 分钟，请使用 `check_task` 轮询结果。

**扣积分**（从租户积分余额中扣除）

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `prompt` | string | 是 | 视频描述文本，支持中文 |
| `model` | string | 否 | 当前仅支持 `sd2`（默认），Seedance 2.0 模型 |
| `resolution` | string | 否 | 分辨率：`480p` / `720p`(默认) / `1080p` |
| `duration` | integer | 否 | 当前工具描述声明 `5`(默认) / `10` / `30` / `60` / `120` 秒，但运行时已于 2026-07-30 接受并完成 `duration: 15`（`record_id: 200704`）。15 秒须按 `references/batch-execution-contract.md` 使用 `mcp_declared` 或证据绑定的 `mcp_observed`；禁止静默降为 10/30 秒 |
| `ratio` | string | 否 | 比例：`9:16`(默认) / `16:9` / `1:1` |
| `first_frame_url` | string | 否 | 首帧图片 URL（图生视频模式） |
| `ref_image_urls` | array | 否 | 参考图片 URL 列表，支持两种格式：<br>1. **公网 URL**: `https://...`<br>2. **模特资产 ID**: `asset://<asset_id>`（从 `list_builtin_portraits` 或 `list_custom_portraits` 获取） |

**图生视频示例**（传入参考图 URL）:
```json
{
  "name": "generate_video",
  "arguments": {
    "prompt": "产品展示视频，光线柔和，运镜流畅",
    "first_frame_url": "https://your-oss.com/images/product.jpg",
    "ref_image_urls": [
      "https://your-oss.com/images/ref1.jpg",
      "https://your-oss.com/images/ref2.jpg"
    ],
    "ratio": "9:16",
    "duration": 5
  }
}
```

**图生视频示例**（使用模特资产 ID）:
```json
{
  "name": "generate_video",
  "arguments": {
    "prompt": "模特穿着冬季大衣在雪地里行走",
    "ref_image_urls": [
      "asset://portrait_abc123",
      "https://your-oss.com/images/clothing_ref.jpg"
    ],
    "ratio": "9:16",
    "duration": 5
  }
}
```

> **提示**：图片需先上传到公网可访问的 OSS/图床（可使用 `upload_images` 工具），传入完整 URL。仅传入 `ref_image_urls` 不传 `prompt` 也是可以的（纯图生视频）。`asset://` 格式可直接引用模特库中的资产 ID。

**返回值**:
```json
{
  "record_id": 134080,
  "model": "doubao-seedance-2-0-260128",
  "status": "queued"
}
```
> 返回 `record_id`（整数）而非 `task_id`。任务排队后由系统统一出队，`task_id` 在状态变为 `running` 后自动分配。后续用 `check_task(record_id)` 轮询进度。

### 2. check_task — 查询任务

查询视频生成任务的状态和结果。

**不扣积分**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `record_id` | integer | 是 | generate_video 返回的记录 ID |

**返回值（排队中）**:
```json
{
  "record_id": 134080,
  "status": "queued"
}
```

**返回值（出队后）**:
```json
{
  "record_id": 134080,
  "task_id": "cgt-20260612112053-zznhv",
  "status": "running"
}
```

**返回值（完成后）**:
```json
{
  "record_id": 134080,
  "task_id": "cgt-20260612112053-zznhv",
  "status": "succeeded",
  "video_url": "https://oss.example.com/videos/xxx.mp4"
}
```

状态说明: `queued` 排队中 → `running` 处理中 → `succeeded` 完成 / `failed` 失败。`task_id` 仅在 `running` 及之后的状态才有值。

### 3. download_video — 获取下载链接

获取已生成视频的下载地址。

**不扣积分**（积分已在生成时扣除）

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `record_id` | integer | 是 | generate_video 返回的记录 ID |

**返回值**:
```json
{
  "record_id": 134080,
  "task_id": "cgt-20260612...",
  "video_url": "https://oss.example.com/videos/xxx.mp4",
  "oss_url": "https://oss-mirror.example.com/videos/xxx.mp4"
}
```

### 4. query_balance — 积分查询

查询当前账户的积分余额。

**不扣积分**

无需参数。

**返回值**:
```json
{
  "remaining": 940,
  "total_points": 1000,
  "used_points": 60,
  "package_name": "专业版"
}
```

### 5. upload_images — 批量文件上传

将本地图片或视频批量上传到云端，返回公网可访问的 URL 列表。支持一次上传多个文件（并发上传，最多 10 个）。图片 URL 可传入 `generate_video` 的 `first_frame_url` 或 `ref_image_urls`，视频 URL 可用于 `publish_video` 的 `video_url`。

**不扣积分**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `images` | array | 是 | 文件列表，每项包含 `image_data`（base64 编码）和 `filename`（用于推断文件类型，如 `product.jpg`、`demo.mp4`） |

**支持的格式**:
- 图片：jpg、jpeg、png、webp、gif、bmp（最大 20MB）
- 视频：mp4、mov、avi、webm（最大 100MB）

**示例**:
```json
{
  "name": "upload_images",
  "arguments": {
    "images": [
      {"image_data": "/9j/4AAQSkZJRg...", "filename": "product.jpg"},
      {"image_data": "iVBORw0KGgoAAA...", "filename": "model.png"},
      {"image_data": "AAAAIGZ0eXBpc29t...", "filename": "demo.mp4"}
    ]
  }
}
```

**返回值**:
```
📤 上传结果：成功 3 个，失败 0 个
  (图片 2 张)  (视频 1 个)

✅ product.jpg: https://oss.example.com/temp_uploads/mcp_abc123.jpg
✅ model.png: https://oss.example.com/temp_uploads/mcp_def456.png
✅ demo.mp4: https://oss.example.com/temp_uploads/mcp_ghi789.mp4
```

> **插件本地图片上传**：在本插件内，先把原始图片逐字节暂存到当前工作区或 `$CODEX_HOME/zibuyu-runs/<run_id>`，然后将 `image_data` 写成绝对 `local-file:///C:/.../three-view.png` URI。已信任的 `PreToolUse` Hook 会在 MCP 调用发出前读取原文件并重写为 base64。模型和 shell 不得手工生成、回显、复制或保存 base64；Codex CLI 的诊断调用视图仍可能显示宿主重写后的参数，这是宿主显示限制，不是上传失败或人工回退，正常生产应在新建的 Codex Desktop 任务中运行。不要把普通 `C:\...` 路径直接当成 `image_data`。插件外部没有该 Hook 时，远程工具仍只接受真实 base64。

### 6. list_builtin_portraits — 查询内置模特库

查询系统内置的虚拟人像模特库（约 6000 张），支持关键词搜索和分页。返回的 `asset_id` 可用于 `generate_video` 的 `ref_image_urls`。

**不扣积分**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `keyword` | string | 否 | 搜索关键词，模糊匹配国籍、年龄、性别、职业、描述 |
| `page` | integer | 否 | 页码，默认 1 |
| `page_size` | integer | 否 | 每页数量，默认 20，最大 100 |

**示例**:
```json
{
  "name": "list_builtin_portraits",
  "arguments": {
    "keyword": "年轻女性 亚洲",
    "page": 1,
    "page_size": 10
  }
}
```

**返回值**:
```
🎭 内置模特库搜索结果
关键词: 年轻女性 亚洲 | 第 1 页 | 共 120 条

- portrait_abc123 | 中国, 女, 25岁, 模特
  描述: 亚洲年轻女性，长发，适合服装展示
  缩略图: https://oss.example.com/virtual_people_models/images/portrait_abc123.png

- portrait_def456 | 日本, 女, 30岁, 演员
  描述: 成熟女性，短发，适合商务场景
  缩略图: https://oss.example.com/virtual_people_models/images/portrait_def456.png

---
💡 使用方法：在 generate_video 的 ref_image_urls 中传入 asset://<asset_id>
   例如: "ref_image_urls": ["asset://portrait_abc123"]
```

### 7. list_custom_portraits — 查询自制模特库

查询当前租户的自定义模特库（租户隔离，只能看到自己租户的模特）。支持关键词搜索、素材组分过滤和分页。

**不扣积分**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `keyword` | string | 否 | 搜索关键词，模糊匹配国籍、年龄、性别、职业、描述、作者 |
| `page` | integer | 否 | 页码，默认 1 |
| `page_size` | integer | 否 | 每页数量，默认 20，最大 100 |
| `group_id` | string | 否 | 素材组 ID，用于过滤特定素材组下的模特 |

**示例**:
```json
{
  "name": "list_custom_portraits",
  "arguments": {
    "keyword": "冬季大衣",
    "group_id": "group_xyz",
    "page": 1,
    "page_size": 10
  }
}
```

**返回值**:
```
🎭 自制模特库搜索结果
关键词: 冬季大衣 | 素材组: group_xyz | 第 1 页 | 共 5 条

- custom_abc123 | 中国, 女, 28岁, 模特, 张三
  描述: 冬季服装展示模特，适合外套拍摄
  缩略图: https://oss.example.com/thumbnails/custom_abc123.png

---
💡 使用方法：在 generate_video 的 ref_image_urls 中传入 asset://<asset_id>
   例如: "ref_image_urls": ["asset://custom_abc123"]
```

---

## 开放平台工具

以下工具用于 TikTok 渠道管理和视频发布。

### 8. list_channels — 查询可用渠道

查询当前租户下可用的 TikTok 渠道列表。租户隔离，不返回 token 等敏感数据。

**不扣积分**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `keyword` | string | 否 | 搜索关键词，模糊匹配达人名称、店铺名称、标签 |
| `status` | string | 否 | 渠道状态：`active`（可用）/ `inactive`（已停用），默认不过滤 |
| `auth_type` | integer | 否 | 授权类型：`1`=渠道绑定, `0`=店铺授权，默认不过滤 |
| `page` | integer | 否 | 页码，默认 1 |
| `page_size` | integer | 否 | 每页数量，默认 20，最大 100 |

**示例**:
```json
{
  "name": "list_channels",
  "arguments": {
    "keyword": "服饰",
    "status": "active",
    "page": 1,
    "page_size": 10
  }
}
```

**返回值**:
```
📺 可用渠道列表
共 5 条 | 第 1 页

🟢 **张三的服饰店**
  channel_id: ch_abc123
  达人ID: creator_xyz | 类型: 店铺创作者
  店铺: 张三服饰旗舰店 | 授权: 渠道绑定
  地区: CN | 标签: 服装,女装
  达人API: 已授权 | 绑定时间: 2026-01-15T10:30:00
```

> **注意**：返回数据不包含 access_token、refresh_token 等敏感信息。

### 9. list_published_videos — 查询已发布视频列表

查询当前租户下已发布的视频记录。支持按渠道、状态、关键词过滤。

**不扣积分**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `channel_id` | string | 否 | 渠道 ID，用于过滤特定渠道的视频 |
| `status` | string | 否 | 状态过滤：`uploading`/`uploaded`/`prechecking`/`publishing`/`published`/`failed` |
| `keyword` | string | 否 | 搜索关键词，模糊匹配视频标题、商品标题 |
| `page` | integer | 否 | 页码，默认 1 |
| `page_size` | integer | 否 | 每页数量，默认 20，最大 100 |

**示例**:
```json
{
  "name": "list_published_videos",
  "arguments": {
    "status": "published",
    "keyword": "冬季新品",
    "page": 1,
    "page_size": 10
  }
}
```

**返回值**:
```
📋 已发布视频列表
共 12 条 | 第 1 页

📢 **冬季新品展示视频**
  log_id: log_abc123
  channel_id: ch_xyz789
  tiktok_video_id: v_123456789
  商品: 冬季羽绒服 (prod_abc)
  播放: 1520 | 点赞: 89 | 评论: 12 | 分享: 5
  发布时间: 2026-06-14T15:30:00 | 创建时间: 2026-06-14T15:00:00
```

### 10. get_video_detail — 查询视频详情

通过 TikTok 视频 ID 调用 Analytics API 获取实时表现数据（GMV、销量、CTR），同时返回本地发布记录和统计数据。

**不扣积分**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `tiktok_video_id` | string | 是 | TikTok 视频 ID（可从 `list_published_videos` 获取） |

**示例**:
```json
{
  "name": "get_video_detail",
  "arguments": {
    "tiktok_video_id": "v_123456789"
  }
}
```

**返回值**:
```
🎬 视频详情

TikTok 视频 ID: v_123456789
渠道: 张三的服饰店 (ch_abc123)
视频标题: 冬季新品展示视频
商品: 冬季羽绒服 (prod_abc)
发布状态: published
发布时间: 2026-06-14T15:30:00

📊 本地统计:
  播放: 1520 | 点赞: 89
  评论: 12 | 分享: 5
  转化: 3 | GMV: 299.00

📡 TK Analytics 实时表现:
  销量: 3 件 | 订单: 2 单 | GMV: 299.00
  CTR: 0.0523 | 统计天数: 30 天

📦 OSS 归档: https://oss.example.com/tiktok_videos/tk_log_abc.mp4
```

### 11. publish_video — 发布视频

将视频发布到指定的 TikTok 渠道。系统异步执行上传、可选预检、发布流程，立即返回 `log_id` 用于后续查询进度。

**不扣积分**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `channel_id` | string | 是 | 目标 TikTok 渠道 ID（从 `list_channels` 获取） |
| `video_url` | string | 是 | 视频文件的公网可访问 URL（系统自动下载，最大 500MB） |
| `video_title` | string | 是 | 视频标题 |
| `product_id` | string | 是 | 关联商品 ID（通过 `list_channel_products` 查询获取，用于挂车） |
| `product_title` | string | 是 | 商品标题（通过 `list_channel_products` 查询获取，或使用 SKU 数据中的商品名称） |
| `link_title` | string | 否 | 商品链接展示标题，不填则默认使用 product_title 截断到 30 字符 |
| `cover_timestamp_ms` | integer | 否 | 封面帧时间戳（毫秒），默认 0 |
| `run_precheck` | boolean | 否 | 是否执行侵权预检，默认 `false` |

**示例（基础发布）**:
```json
{
  "name": "publish_video",
  "arguments": {
    "channel_id": "ch_abc123",
    "video_url": "https://oss.example.com/videos/product_demo.mp4",
    "video_title": "冬季新品展示",
    "product_id": "prod_abc",
    "product_title": "冬季羽绒服"
  }
}
```

**示例（含预检）**:
```json
{
  "name": "publish_video",
  "arguments": {
    "channel_id": "ch_abc123",
    "video_url": "https://oss.example.com/videos/product_demo.mp4",
    "video_title": "冬季新品展示",
    "product_id": "prod_abc",
    "product_title": "冬季羽绒服",
    "link_title": "限时优惠",
    "cover_timestamp_ms": 1000,
    "run_precheck": true
  }
}
```

**返回值**:
```
✅ 视频发布任务已提交

📋 发布记录 ID: log_abc123
📺 渠道: 张三的服饰店
🎬 标题: 冬季新品展示
🔍 预检: 是，将先执行侵权预检
📦 视频大小: 15.2 MB

任务在后台异步执行（上传 → 预检 → 发布），使用 `check_publish` 或 `list_published_videos` 查询进度。
```

> **流程说明**：视频首先上传至 TikTok，如开启预检则会检查侵权和画质，通过后自动发布。整个过程约 1-5 分钟。可通过 `check_publish` 或 `list_published_videos` 轮询状态。

### 12. check_publish — 查询发布状态

查询发布任务的当前状态。支持传入 `log_id`（立即发布）或 `schedule_id`（定时发布）。

**不扣积分**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `log_id` | string | 否 | publish_video 返回的发布记录 ID（立即发布时使用） |
| `schedule_id` | string | 否 | publish_video 返回的排期 ID（定时发布时使用） |

> **注意**：`log_id` 和 `schedule_id` 至少提供一个。

**返回值 — 立即发布（上传中）**:
```
📤 发布任务状态: 上传中

📋 发布记录 ID: log_abc123
📺 渠道 ID: ch_abc123
🎬 标题: 冬季新品展示
📌 状态: uploading
🕐 创建时间: 2026-06-14 15:00:00
```

**返回值 — 立即发布（预检中）**:
```
🔍 发布任务状态: 预检中

📋 发布记录 ID: log_abc123
📺 渠道 ID: ch_abc123
🎬 标题: 冬季新品展示
📌 状态: prechecking
🕐 创建时间: 2026-06-14 15:00:00
```

**返回值 — 立即发布（发布成功）**:
```
🎉 发布任务状态: 已发布

📋 发布记录 ID: log_abc123
📺 渠道 ID: ch_abc123
🎬 标题: 冬季新品展示
📌 状态: published
🆔 TikTok 视频 ID: v_123456789
🛍️ 商品 ID: prod_abc
📦 商品标题: 冬季羽绒服
🕐 发布时间: 2026-06-14 15:05:00
🕐 创建时间: 2026-06-14 15:00:00

💡 使用 get_video_detail(tiktok_video_id="v_123456789") 查看实时数据
```

**返回值 — 立即发布（失败）**:
```
❌ 发布任务状态: 失败

📋 发布记录 ID: log_abc123
📺 渠道 ID: ch_abc123
🎬 标题: 冬季新品展示
📌 状态: failed
⚠️ 错误信息: Precheck failed: copyright music detected
🕐 创建时间: 2026-06-14 15:00:00

💡 修复问题后可使用 publish_video 重新提交
```

**返回值 — 定时发布（等待中）**:
```
⏳ 定时发布状态: 等待发布

📋 排期 ID: e159592a-24e3-431d-b2c8-4c93f7e6adde
📺 渠道 ID: db123245-5059-49ae-b891-aad85ec7c4ad
🎬 标题: Coffee run romper
🕐 计划发布时间: 2026-06-17 09:00:00 -0400
🛍️ 商品 ID: 1729487798986905390
📦 商品标题: N7Z134TL
📎 TikTok 文件 ID: v12d00gd0024d8p3l7fog65lulaosilg

💡 视频已上传至 TikTok，系统将在指定时间自动发布
```

**返回值 — 定时发布（已发布）**:
```
🎉 定时发布状态: 已发布

📋 排期 ID: e159592a-24e3-431d-b2c8-4c93f7e6adde
📺 渠道 ID: db123245-5059-49ae-b891-aad85ec7c4ad
🎬 标题: Coffee run romper
🕐 计划发布时间: 2026-06-17 09:00:00 -0400
🛍️ 商品 ID: 1729487798986905390
📦 商品标题: N7Z134TL
📎 TikTok 文件 ID: v12d00gd0024d8p3l7fog65lulaosilg
📋 发布记录 ID: log_abc123

💡 使用 check_publish(log_id="log_abc123") 查看发布详情
```

状态说明:
- 立即发布: `uploading` 上传中 → `uploaded` 已上传 → `prechecking` 预检中（仅开启预检时）→ `publishing` 发布中 → `published` 已发布 / `failed` 失败。
- 定时发布: `pending` 等待发布 → `published` 已发布 / `failed` 失败 / `cancelled` 已取消。

### 13. list_channel_products — 查询渠道商品

查询 TikTok 渠道的橱窗或店铺商品列表，用于发布视频时选择挂车商品。

**不扣积分**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `channel_id` | string | 是 | TikTok 渠道 ID |
| `source` | string | 否 | 商品来源：`showcase`（橱窗商品，默认）/ `shop`（店铺商品） |
| `keyword` | string | 否 | 搜索关键词，模糊匹配商品标题 |
| `page_token` | string | 否 | 翻页 token，首次请求不传，后续从返回值获取 |

**示例（橱窗商品）**:
```json
{
  "name": "list_channel_products",
  "arguments": {
    "channel_id": "ch_abc123",
    "source": "showcase",
    "keyword": "羽绒服"
  }
}
```

**返回值**:
```
🛍️ 橱窗商品列表
渠道: 张三的服饰店
共 25 件 | 本页 3 件
下一页 token: eyJvZmZzZXQiOjUw...

📦 **冬季羽绒服加厚款**
  product_id: prod_17290045
  价格: CNY 299.00 | 品牌: 张三服饰旗舰店
  分类: 女装/羽绒服 | 状态: 已添加
  图片: https://cdn.tiktok.com/..., https://cdn.tiktok.com/...

📦 **轻薄羽绒背心**
  product_id: prod_17290046
  价格: CNY 159.00 | 品牌: 张三服饰旗舰店
  分类: 女装/马甲 | 状态: 已添加
  图片: https://cdn.tiktok.com/...

---
💡 发布视频时使用 product_id 参数挂车，如 publish_video(product_id="prod_17290045")
```

---

## 推荐工作流

> **工具懒加载预检**：如果 `/mcp` 或 MCP inventory 已列出 PopBoom，但当前回合仍不能直接调用某个工具，先用平台工具搜索精确加载 `query_balance`、`list_custom_portraits`、`upload_images`、`generate_video` 或 `check_task`。只有精确搜索仍失败时才能判定工具不可用；工具搜索本身不扣 PopBoom 积分。

### 流程 1：文生视频（纯文本）

```
generate_video(prompt="一只猫在草地上奔跑", ratio="16:9")
  → record_id
  → check_task(record_id) 轮询直到 succeeded
  → 优先使用 check_task 返回的 video_url；缺失或需要本地镜像时才调用一次 download_video(record_id)
```

### 流程 2：图生视频（本地图片 + 模特）

```
1. 将原始图片逐字节暂存到当前 run 目录，计算 SHA-256/字节数；调用 upload_images(images=[{"image_data":"local-file:///C:/.../product.png","filename":"product.png"}]) → Hook 直读原图并返回图片 URL
2. list_builtin_portraits(keyword="年轻女性") → 获取模特 asset_id
3. generate_video(
     prompt="产品展示视频",
     ref_image_urls=["asset://<asset_id>", "https://oss.../product.jpg"],
     ratio="9:16"
   )
   → record_id
4. check_task(record_id) 轮询
5. 优先使用 check_task 返回的 video_url；缺失或需要本地镜像时才调用一次 download_video(record_id)
```

### 流程 3：TikTok 视频发布

```
1. list_channels(status="active") → 获取可用渠道
2. list_channel_products(channel_id="<渠道ID>", source="showcase")
   → 获取橱窗/店铺商品，选好要挂车的 product_id
3. upload_images(images=[{..., "filename": "demo.mp4"}] )
   → 上传视频文件获取 URL
   （也可用已生成的视频 URL：优先读取 check_task.video_url，必要时再调用 download_video）
4. publish_video(
     channel_id="<从步骤1获取>",
     video_url="<从步骤3获取>",
     video_title="产品展示",
     product_id="<从步骤2获取>",
     product_title="商品名称",
     run_precheck=true
   )
   → log_id
5. check_publish(log_id) 轮询到 status=published
6. get_video_detail(tiktok_video_id="...") 查看实时数据
```

### 流程 4：查看已发布视频数据

```
1. list_published_videos(channel_id="ch_xxx", status="published")
   → 获取已发布视频列表和 tiktok_video_id
2. get_video_detail(tiktok_video_id="v_xxx")
   → 获取 TK 官方实时数据（播放量、点赞、评论等）
```

---

## 积分消耗参考

| 分辨率 | 5秒视频 | 10秒视频 | 15秒预算基线 |
|--------|---------|----------|---------------|
| 480p | 28 积分 | 55 积分 | 84 积分（2026-07-30 实扣） |
| 720p | 60 积分 | 119 积分 | 至少 180 积分（按 5 秒成本的 3 倍预留） |
| 1080p | 149 积分 | 297 积分 | 至少 447 积分（按 5 秒成本的 3 倍预留） |

- 图生视频（带 `first_frame_url`）：积分翻倍
- 10 积分 = 1 元人民币

---

## 在 Codex 中配置

1. MCP 名称使用 `PopBoom`。
2. 传输类型使用 `streamable_http`。
3. URL 统一使用 `https://tkvideo.zbycorp.com:5002/mcp`。
4. 认证只使用 Codex 已有的安全 MCP 配置；不要在文档、日志、脚本或台账中复制 Bearer 值。
5. Codex 自动完成 `initialize` 和 `tools/list`。每个生产批次只做一次非付费能力预检，并把服务版本与相关 schema 摘要写入台账。

HTTP `/sse`、占位域名和旧 SSE 会话脚本均为历史路径，不得用于生产。不要为了测试连接手写开发者客户端，也不要为每个批次重复发送付费 `generate_video` canary；使用已登记的成功证据并在真实业务提交前走余额与账本门。生产路由、15 秒能力、固定模特资产、余额检查、上传去重和轮询规则以 `references/batch-execution-contract.md` 为准。

---

## 安全测试方法

- 使用 Codex 当前 MCP 连接执行 `initialize` / `tools/list` 或等价的只读检查。
- 验证 `query_balance`、`list_custom_portraits`、`upload_images`、`generate_video`、`check_task` 可用；`download_video` 仅在成功任务缺少 URL 或需要本地镜像时使用。
- 只记录端点、服务版本、工具名和 schema 摘要；绝不输出认证 Header。
- 固定模特按已登记的 asset ID 做当前租户唯一性验证，不依赖中文显示名搜索。
- 连接检查不得触发上传或付费生成。

Codex 会自动处理 MCP 协议，用户无需手动编写客户端代码。

---

## 注意事项

- API Key 创建后仅显示一次，请妥善保管
- 如 Key 泄露，请在平台中立即撤销并创建新 Key
- 视频生成需消耗积分，请确保账户有足够余额
- 每个租户有并发限制，超出后任务将进入排队
- 所有调用均有日志记录，可在平台「调用日志」中查看
- `upload_images` 单次最多上传 10 个文件；当前文档列出的图片上限为 20MB。插件不得为了规避终端输出限制而自动压缩或降分辨率；只有 PopBoom 返回明确大小错误后，才报告并等待用户授权衍生图方案
- `list_custom_portraits` 自动按 API Key 所属租户隔离，不会看到其他租户的模特
