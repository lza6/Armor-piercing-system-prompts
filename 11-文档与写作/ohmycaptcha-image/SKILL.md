---
name: ohmycaptcha-image
description: Generate product visuals, README banners, documentation illustrations, and architecture diagrams for OhMyCaptcha. Use when the user asks for repository art, hero images, deployment diagrams, or marketing visuals for captcha infrastructure projects.
---

# OhMyCaptcha Image Skill

Generate clean, public-safe visuals for the OhMyCaptcha repository and documentation.

## Use cases

- README hero images
- Documentation illustrations
- Architecture diagrams
- Deployment flow visuals

## Principles

1. Keep visuals product-oriented and privacy-safe.
2. Never embed real tokens, private URLs, or customer data.
3. Prefer abstract infrastructure motifs: browser automation, API flows, tokens, multimodal vision, cloud deployment.
4. Style: polished, modern, minimal, open-source-friendly.
5. For README visuals, use 16:9 landscape ratio.

## Prompting template

Include in every image prompt:
- **Subject**: OhMyCaptcha as a self-hostable captcha-solving service
- **Motifs**: browser automation, API requests (`/createTask`, `/getTaskResult`), reCAPTCHA/hCaptcha/Turnstile widgets, multimodal vision model, SGLang local deployment
- **Style**: polished dark UI aesthetic with blue/indigo accents
- **Constraints**: no secrets, no vendor logos, no text resembling real keys

## Example prompt

> Create a polished open-source hero illustration for OhMyCaptcha, showing a modern self-hosted captcha-solving pipeline with browser automation, API flow cards, a local SGLang model server, and cloud deployment badges. Dark UI with blue/indigo accents. No secrets or vendor logos.

## Output recommendations

- 16:9 or wide landscape composition
- Dark or neutral background
- Cyan / blue / indigo accent palette
- Enough negative space for cropping
