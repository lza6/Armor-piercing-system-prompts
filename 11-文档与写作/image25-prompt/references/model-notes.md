# Image 2.5 适配说明

核对日期：2026-09-09。执行遇到版本冲突时核对官方链接。技能不提供模型权限或API客户端。

## 官方参数快照

- `gpt-image-2.5-flare`：速度优先。
- `gpt-image-2.5-sunburst`：高质量和精细编辑优先。

| 字段 | 值 |
|---|---|
| quality | auto、low、medium、high、xhigh、max |
| size | auto或WIDTHxHEIGHT |
| background | auto、opaque、transparent |
| output_format | png、jpeg、webp |

自定义尺寸两边为16的倍数；最长边≤3840；长短边比≤3；总像素655360至8294400。超过3686400总像素属于实验范围。透明背景配PNG/WebP。
[提示词文档](https://developers.openai.com/api/docs/guides/image-prompting) · [输出文档](https://developers.openai.com/api/docs/guides/image-generation)

## 本项目执行建议

图片说明与API设置分开，只有工具支持的字段才提交。不擅自切换用户点名的模型。以下只是建议参数，不是执行命令：
```json
{
  "model": "gpt-image-2.5-flare",
  "size": "1024x1536",
  "quality": "medium",
  "background": "opaque",
  "output_format": "png"
}
```

按需求测试更高质量档，不把max设为默认。额度、权限、SDK兼容性和费用由实际环境决定，不硬编码单张价格。

旧Image 2 CLI可能限制枚举，不能仅改模型名便宣称兼容。用户未请求接入时，不安装客户端、不修改密钥或环境。

## 产品与宿主工具

Sketch、图片批注、模板、分享属于ChatGPT产品界面，不是安装技能新增的按钮。应核实当前入口；不能使用Sketch时可接受上传草图。本文未实测所有界面入口。

宿主工具可能只接受prompt和参考图；此时将尺寸和透明要求写入视觉请求，再检查实际文件。不得捏造未提交的API字段。后端不明标为unknown，不能根据发布日期推断。

本包不发送请求、不读取密钥；提示词可复制到ChatGPT、现有生图工具或已验证的API流程。不承诺身份绝对一致、事实绝对准确、无限轮不漂移或遮罩外像素不变。
