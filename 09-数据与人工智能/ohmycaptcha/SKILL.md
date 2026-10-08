---
name: ohmycaptcha
description: Deploy, configure, validate, and integrate the OhMyCaptcha captcha-solving service. Use when working with YesCaptcha-style APIs, flow2api integration, reCAPTCHA/hCaptcha/Turnstile task creation, image classification, SGLang local model deployment, Render/Hugging Face cloud deployment, or OpenAI-compatible multimodal model setup. Also use when the user asks how to self-host a captcha-solving service or wants request/response examples for OhMyCaptcha.
---

# OhMyCaptcha Skill

Operational guidance for deploying and integrating the OhMyCaptcha service.

## Model architecture

OhMyCaptcha uses two model backends:

- **Local model** — self-hosted via SGLang/vLLM (e.g. `Qwen/Qwen3.5-2B`). Handles image recognition and classification tasks. Configured via `LOCAL_BASE_URL`, `LOCAL_API_KEY`, `LOCAL_MODEL`.
- **Cloud model** — remote OpenAI-compatible API (e.g. `gpt-5.4`). Handles audio transcription and complex reasoning. Configured via `CLOUD_BASE_URL`, `CLOUD_API_KEY`, `CLOUD_MODEL`.

## Supported task types (19 total)

### Browser-based (12)
`RecaptchaV3TaskProxyless`, `RecaptchaV3TaskProxylessM1`, `RecaptchaV3TaskProxylessM1S7`, `RecaptchaV3TaskProxylessM1S9`, `RecaptchaV3EnterpriseTask`, `RecaptchaV3EnterpriseTaskM1`, `NoCaptchaTaskProxyless`, `RecaptchaV2TaskProxyless`, `RecaptchaV2EnterpriseTaskProxyless`, `HCaptchaTaskProxyless`, `TurnstileTaskProxyless`, `TurnstileTaskProxylessM1`

### Image recognition (3)
`ImageToTextTask`, `ImageToTextTaskMuggle`, `ImageToTextTaskM1`

### Image classification (4)
`HCaptchaClassification`, `ReCaptchaV2Classification`, `FunCaptchaClassification`, `AwsClassification`

## Local model setup (SGLang)

```bash
pip install "sglang[all]>=0.4.6.post1"
# From ModelScope (China):
export SGLANG_USE_MODELSCOPE=true
python -m sglang.launch_server --model-path Qwen/Qwen3.5-2B --port 30000
```

Then configure OhMyCaptcha:
```bash
export LOCAL_BASE_URL="http://localhost:30000/v1"
export LOCAL_MODEL="Qwen/Qwen3.5-2B"
```

## Startup checklist

1. Install dependencies: `pip install -r requirements.txt && playwright install --with-deps chromium`
2. Start local model server (SGLang on port 30000)
3. Set env vars: `LOCAL_BASE_URL`, `CLOUD_BASE_URL`, `CLOUD_API_KEY`, `CLIENT_KEY`
4. Start service: `python main.py`
5. Verify: `curl http://localhost:8000/api/v1/health`
6. Test: create a reCAPTCHA v3 task against `https://antcpt.com/score_detector/` with key `6LcR_okUAAAAAPYrPe-HK_0RULO1aZM15ENyM-Mf`

## Response rules

1. Prefer the repository's documented behavior over assumptions.
2. Use placeholder credentials only. Never expose real secrets.
3. Be explicit about limitations:
   - `minScore` is compatibility-only
   - Task storage is in-memory with 10-min TTL
   - reCAPTCHA v2 and hCaptcha may require image classification fallback in headless environments
4. For deployment help, reference `docs/deployment/local-model.md`, `docs/deployment/render.md`, `docs/deployment/huggingface.md`.
5. For API usage, reference `docs/api-reference.md` and the usage guides under `docs/usage/`.
