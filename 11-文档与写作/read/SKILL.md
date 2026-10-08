---
name: read
description: Invoke when given any URL, web page link, or PDF to read. Fetches the content as clean Markdown via proxy cascade and saves to Downloads. Not for local files already in the repo.
metadata:
  version: "3.8.0"
---

# Read: Fetch Any URL or PDF as Markdown

Prefix your first line with 🥷 inline, not as its own paragraph.


Convert any URL or local PDF to clean Markdown and save it.

## Routing

| Input | Method |
|-------|--------|
| `feishu.cn`, `larksuite.com` | Feishu API script |
| `mp.weixin.qq.com` | Proxy cascade first, built-in WeChat article script only if the proxies fail |
| `.pdf` URL or local PDF path | PDF extraction |
| GitHub URLs (`github.com`, `raw.githubusercontent.com`) | Prefer raw content or `gh` first. Use the proxy cascade only as fallback. |
| Everything else | Run `scripts/fetch.sh {url}` (proxy cascade with Markdown-only fallback) |

After routing, load `references/read-methods.md` to get the exact commands for the chosen method, then execute them.

## Output Format

```
Title:  {title}
Author: {author} (if available)
Source: {platform}
URL:    {original url}

Summary
{3-5 sentence summary}

Content
{full Markdown, truncated at 200 lines if long}
```

## Saving

Save to `~/Downloads/{title}.md` with YAML frontmatter by default.
Skip only if user says "just preview" or "don't save". Tell the user the saved path.

After saving and reporting the path, stop. Do not analyze, comment on, or discuss the content unless asked. If content was truncated at 200 lines, say so and offer to continue.

## Gotchas

- If a web search plugin is installed (e.g., PipeLLM search), use it for URL discovery before fetching.
- r.jina.ai and defuddle.md require no API key
- Network failures: prepend local proxy env vars if available
- Long content: `| head -n 200` to preview first
- GitHub URLs: prefer raw content or `gh` CLI. Use `scripts/fetch.sh` only as fallback.
- Local fallback tools may return JSON internally, but the final output and saved file must still be Markdown.
- If all methods fail (proxies, local tools, and proxy env vars): stop and tell the user what was tried and what failed. Suggest they open the URL in a browser and paste the content, or provide an alternative URL. Do not silently return empty or partial results.
