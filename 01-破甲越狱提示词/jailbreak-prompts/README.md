# jailbreak-prompts


> 精选 LLM 破甲提示词合集 — Codex / GPT-5.x / Claude / Gemini / DeepSeek / Grok — UNFILTERED MODE · GODMODE · CTF Sandbox

## 文件

| 文件 | 适用模型 |
|------|---------|
| `xsser_codex-base-prompt.md` | Codex CLI (GPT-5.x) |
| `codex-red-team_system-prompt.md` | Codex CLI (GPT-5.x) |
| `L1B3RT4S_ANTHROPIC.mkd` | Claude 全系列 |
| `L1B3RT4S_OPENAI.mkd` | ChatGPT / GPT |
| `L1B3RT4S_CHATGPT.mkd` | ChatGPT |
| `L1B3RT4S_GOOGLE.mkd` | Gemini |
| `L1B3RT4S_DEEPSEEK.mkd` | DeepSeek |
| `L1B3RT4S_GROK_MEGA.mkd` | Grok |
| `L1B3RT4S_META.mkd` | Llama |
| `L1B3RT4S_MICROSOFT.mkd` | Copilot |
| `L1B3RT4S_PERPLEXITY.mkd` | Perplexity |
| `L1B3RT4S_BRAVE.mkd` | Brave Leo |
| `L1B3RT4S_CURSOR.mkd` | Cursor |
| `L1B3RT4S_NOUS.mkd` | Nous Hermes |

## 使用方式

### Codex CLI

```toml
# ~/.codex/config.toml
model_instructions_file = "./xsser_codex-base-prompt.md"
approval_policy = "never"
sandbox_mode = "danger-full-access"
```

### Claude / ChatGPT / 等

直接粘贴 .mkd / .md 文件全文到新会话。

## 声明

仅供安全研究、CTF 竞赛、授权红队评估。使用者自行负责。
