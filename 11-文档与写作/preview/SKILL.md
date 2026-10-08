---
name: preview
description: Preview Markdown, LaTeX, PDF, or code artifacts in the browser or as PDF. Use when the user wants to review a written artifact, export a report, or view a rendered document.
---

# Preview

Use the `/preview` command to render and open artifacts.

## Commands

| Command | Description |
|---------|-------------|
| `/preview` | Preview the most recent artifact in the browser |
| `/preview --file <path>` | Preview a specific file |
| `/preview-browser` | Force browser preview |
| `/preview-pdf` | Export to PDF via pandoc + LaTeX |
| `/preview-clear-cache` | Clear rendered preview cache |

## Fallback

If the preview commands are not available, use bash:

```bash
open <file.md>          # macOS — opens in default app
open <file.pdf>         # macOS — opens in Preview
```
