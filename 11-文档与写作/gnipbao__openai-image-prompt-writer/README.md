# GPT Image 2.5 提示词写作 Skill

把图像需求、旧提示词和参考图说明，整理成可以直接复制的图像提示词。依据 OpenAI 官方指南，支持新建、优化、局部编辑、多图融合、精确文字、角色一致性与透明素材。

**默认只写提示词。** 只有明确要求生成或编辑图片时，才交给当前环境实际可用的图像工具。

A Codex skill for writing and refining GPT Image 2.5 prompts, with explicit reference roles, editing constraints, exact text, and an offline settings validator. The skill is primarily written in Chinese and follows the user's requested language.

## 快速使用

安装后在 Codex 中输入：

```text
$openai-image-prompt-writer
为咖啡店写一个 3:4 竖版海报提示词，奶油白与桂花黄，杯子在右下。
唯一文案是“给自己半日闲”，出现一次。只给最终提示词。
```

也可以优化编辑要求：

```text
$openai-image-prompt-writer
我会上传一张产品图。写一个只将背景换成浅灰的编辑提示词，
保留产品外形、材质、位置和标签文字，不要生成图片。
```

查看 [6 个完整原创示例](examples/worked-examples.md) 和 [15 个行为回归用例](examples/retest-prompts.json)。

## 安装到 Codex

需要 Git。macOS / Linux / WSL：

```sh
git clone https://github.com/gnipbao/openai-image-prompt-writer.git \
  "${CODEX_HOME:-$HOME/.codex}/skills/openai-image-prompt-writer"
```

Windows PowerShell：

```powershell
$codexSkillRoot = if ($env:CODEX_HOME) { $env:CODEX_HOME } else { Join-Path $HOME '.codex' }
git clone https://github.com/gnipbao/openai-image-prompt-writer.git (Join-Path $codexSkillRoot 'skills/openai-image-prompt-writer')
```

如果目标目录已存在，先检查其中的本地修改。通过 Git 安装的副本可在该目录运行 `git pull --ff-only`；其他方式安装的副本应先备份到 `skills` 目录之外，再更换。不要直接覆盖未知内容。

安装后下一轮可用 `$openai-image-prompt-writer` 调用。只写提示词无需 Python、API key 或额外插件；离线参数检查器需要 Python 3.10 或更高版本。

如使用 Codex 内置 Skill Installer，可指定本仓库的默认分支 `codex/main`：

```sh
python3 "${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo gnipbao/openai-image-prompt-writer --ref codex/main \
  --path . --name openai-image-prompt-writer
```

该方式需要环境已包含内置安装脚本，且目标技能目录尚不存在。

## 工作方式

1. 区分新建、优化、仅诊断、可填模板和实际生图。
2. 提取用途、主体、构图、文字、参考图角色及必须保留的内容。
3. 只选择必要的摄影、文字、编辑、多图、图表等模块。
4. 交付一个可复制提示词；API 设置按需单独列出。
5. 收到失败结果后，保留正确部分并修正具体偏差。

不要求固定长度或特殊语法。用户只要最终提示词时，不强制附分析报告。未提供参考图时可以先写模板，但不会声称已查看或编辑图像。

## 离线验证 API 设置

检查器只处理 Image API 的设置子集，不是完整请求校验器，也不会发送请求。

在仓库根目录运行：

```sh
python3 -B scripts/validate_settings.py examples/settings.json
```

输出 JSON 中 `valid: true` 表示符合本地资料快照内的规则。检查器覆盖模型 ID、quality、像素尺寸、透明格式与压缩参数；未知字段会明确提示未覆盖，不能据此认定官方接口不支持。

检查器不检查账号权限、提示词质量或真实图像 alpha。更多说明见 [API 设置与边界](references/api-settings.md)。

运行完整离线测试：

```sh
python3 -B -m unittest discover -s tests -v
```

测试通过 CLI 输入输出覆盖尺寸边界、3:4 比例、实验性分辨率、透明 PNG/WebP、非法压缩组合、重复字段、错误类型和缺失文件。GitHub Actions 自动执行相同命令，依赖仅为 Python 标准库。

## 文件结构

```text
SKILL.md                         触发条件、流程、输出与执行边界
agents/openai.yaml               Codex 显示信息
references/official-basis.md     来源、模型名称与设计选择
references/prompt-patterns.md    按场景选择的提示词结构
references/api-settings.md       参数规则与使用表面
examples/worked-examples.md      6 个原创示例
examples/retest-prompts.json     15 个待执行行为用例
examples/settings.json          可运行的参数示例
scripts/validate_settings.py    离线检查器
tests/test_settings.py           可复现的 CLI 回归测试
```

## 验证范围

当前提供文件结构检查、离线参数测试和文本写作示例。创建过程完成了同上下文文本演练；没有独立评测、图像效果基准、文字正确率统计或延迟/成本承诺。行为用例清单不等于所有用例已在图像模型上通过。

本技能不会绕过图像工具的实际能力或内容限制。它不保证精确文字、角色一致性或逐像素不变的编辑；严格图像合成需使用相应编辑流程。只有实际使用图像 API 或生成工具时，才可能产生该服务的费用。

## 官方依据与维护

资料快照核验于 **2026-09-10**：

- [OpenAI Image prompting](https://developers.openai.com/api/docs/guides/image-prompting)
- [OpenAI Image generation](https://developers.openai.com/api/docs/guides/image-generation)

本项目是独立社区 Skill，非 OpenAI 官方发布或认证。官方资料原文、品牌与第三方素材保留各自的权利；本仓库不包含官方示例图片、账户凭据或用户图片。

修改模型和参数时，请在 [官方依据](references/official-basis.md) 记录来源与实际核验日期，并同步更新参数说明、检查器和相关测试。提交问题时可提供精简、去除敏感信息的复现输入与完整报错，不要提交密钥或私密素材。

## License

[MIT](LICENSE) © 2026 gnipbao，适用于本仓库原创代码与文档。
