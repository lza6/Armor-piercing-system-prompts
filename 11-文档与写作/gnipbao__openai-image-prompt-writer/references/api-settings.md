# GPT Image 2.5 参数与执行边界

核验于 2026-09-10，依据 [Image prompting](https://developers.openai.com/api/docs/guides/image-prompting) 与 [Image generation](https://developers.openai.com/api/docs/guides/image-generation)。运行 API 前核验当前文档。纯提示词写作不需要密钥或联网调用。

## 先选择使用表面

| 表面 | 如何放置模型与设置 |
| --- | --- |
| 只写提示词 / ChatGPT 界面 | 自然语言描述画面；不假设界面支持任意 API 参数 |
| Image API | `client.images.generate` 新建，`client.images.edit` 编辑；图像模型 ID 放在请求的 `model` |
| Responses API | 顶层 `model` 是支持图像工具的主模型；图像模型放在 `tools` 中 `type: image_generation` 对象的 `model` 字段 |
| Codex 图像工具 | 只使用实时 schema 暴露的字段；不能由 API 文档推断它能选某个模型或 quality |

下列 JSON 都是 **Image API 设置片段**，不是完整请求；还需 `prompt`，编辑时还需真实输入图片。不要把整段设置直接复制成 Responses API 的顶层请求。

## 模型和质量

速度优先可从 `gpt-image-2.5-flare` 开始；复杂质量与精确编辑需求可从 `gpt-image-2.5-sunburst` 开始。用户指定模型时保留指定，不擅自替换。

`quality` 支持 `auto`（默认）、`low`、`medium`、`high`、`xhigh`、`max`。草案可试较低档；小字、密集标签与图表可比较 `medium` / `high`。只有未满足的质量要求和延迟预算值得时才测试 `xhigh` / `max`。

比较模型先固定提示词、参考图、尺寸、输出格式及显式 quality；质量达标后，再分别测试速度或降低 quality。官方示例的某个档位不是所有图像的最佳参数。

## 比例与像素

`size` 可为 `auto`，或 `WIDTHxHEIGHT`。自定义尺寸必须同时满足：

- 两边都是正整数、16 的倍数，且都不超过 3840。
- 长边不超过短边的 3 倍。
- 总像素数在 655360 到 8294400 之间，含两端。
- 总像素超过 3686400 时属于实验性分辨率；提示这一限制，不把它误判为不合法。

以下为本技能计算并检查的示例，非平台强制尺寸：

| 目标比例 | 合规 `size` | 场景示例 |
| --- | --- | --- |
| 1:1 | `1024x1024` | 图标、方形素材 |
| 3:4 | `1152x1536` | 竖版海报 |
| 4:3 | `1536x1152` | 横向讲解图 |
| 9:16 | `864x1536` | 竖屏画面 |
| 16:9 | `1536x864` | 横向幻灯片图 |
| 2:3 | `1024x1536` | 纵向照片 |
| 3:2 | `1536x1024` | 横向照片 |
| 16:9 4K | `3840x2160` | 实验性高分辨率输出 |

`1024x1536` 是 2:3，不能标作 3:4。`1080x1920` 虽为 9:16，但 1080 不是 16 的倍数；可先用同为 9:16 的 `864x1536`，如必须交付前一尺寸则另外说明必要的导出流程。不要把尺寸建议偷偷写成已经满足的交付尺寸。

## 透明与压缩

- `background` 支持 `auto`、`opaque`、`transparent`。
- 透明素材显式使用 `background: transparent` 与 `output_format: png` 或 `webp`；PNG 是默认输出格式。
- `output_compression` 仅用于 `jpeg` / `webp`，取值 0–100。PNG 不带这个字段。
- 检查解码后的 alpha：背景应透明，主体边缘应干净，玻璃、毛发或软阴影不能因误处理变成实色。

普通海报设置示例：

```json
{"model":"gpt-image-2.5-flare","size":"1152x1536","quality":"medium","output_format":"png"}
```

透明产品编辑设置示例：

```json
{"model":"gpt-image-2.5-sunburst","size":"1024x1024","quality":"high","background":"transparent","output_format":"png"}
```

## 参考图与局部编辑

实际输入顺序要与提示词里的图 1、图 2 对齐。没有上传或传递图片，仅写“参考图 1”无法提供图像条件。

需要 mask 时另行核验当前接口：官方文档要求 mask 有 alpha，图与 mask 的格式、尺寸匹配，文件大小符合接口限制；多图编辑的 mask 作用于第一张图。mask 是模型的编辑引导，并不保证区域边界逐像素不变。严格保持区域需由专门编辑流程完成合成。

不要照搬旧模型的 `input_fidelity`；本技能不为 2.5 输出该未在此处核验的参数。也不默认添加 `seed`、`steps`、`cfg_scale`、Midjourney `--ar` 或 `--no`。这些不属于已核验的设置片段。

## 离线检查设置

将 Image API 设置片段存为 JSON 后运行：

```sh
python3 scripts/validate_settings.py /path/to/settings.json
```

命令需从技能目录运行，或使用脚本绝对路径。仅依赖 Python 标准库，不访问网络。支持 `model`、`size`、`quality`、`background`、`output_format`、`output_compression`。遇到其他键会提示“未覆盖”；这是检查器范围限制，不能据此宣称官方 API 不支持该字段。

检查通过表示符合本快照内的参数规则；不证明账号能调用、不验证图片文件或提示词语义，也不检验真实透明度与画质。完整请求及新增参数应交给当前 SDK/API 文档验证。
