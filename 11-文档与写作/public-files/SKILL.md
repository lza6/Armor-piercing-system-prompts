---
name: public-files
description: Share static files (PDFs, images, zips, build artifacts) as direct, permanent CDN links.
---

# Public Files

Commands target the current Space by default; add `-s <spaceId>` to use another.

## Publish

Upload one file or one directory:

```bash
cohub public upload <source> [destination]
```

Examples:

```bash
cohub public upload ./report.pdf
cohub public upload ./report.pdf reports/latest.pdf
cohub public upload ./dist demo
```

Use `--json` for a manifest of every uploaded file (path, size, MIME type) plus the Space URL prefix.

Existing files are never replaced implicitly. When replacement is intended, use:

```bash
cohub public upload <source> [destination] --overwrite
```

## Inspect

```bash
cohub public ls [path]
cohub public ls -r [path]
cohub public url <path>
```

## Behavior notes

- Uploads are permanent: there is no delete command yet. Anything uploaded stays publicly accessible.
- A URL prefix is not a browsable directory; always link to a specific file.
- The CDN caches content for ~5 minutes. After `--overwrite`, the old version may remain visible briefly; query strings do not bypass the cache.

## Safety

- Never upload secrets, tokens, private logs, or internal-only files.
- Use relative asset paths in static sites.
- Use `--overwrite` only when replacing the existing public path is intentional.

## Finish

Return the direct public URL when the command provides one.
