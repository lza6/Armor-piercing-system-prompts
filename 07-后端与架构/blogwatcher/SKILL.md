---
name: blogwatcher
description: "Monitor blogs and RSS/Atom feeds for updates using the blogwatcher CLI."
homepage: https://github.com/Hyaxia/blogwatcher
---

# blogwatcher

Track blog and RSS/Atom feed updates with the `blogwatcher` CLI.

Install

- Go: `go install github.com/Hyaxia/blogwatcher/cmd/blogwatcher@latest`

Quick start

- `blogwatcher --help`

Common commands

- Add a blog: `blogwatcher add "My Blog" https://example.com`
- List blogs: `blogwatcher blogs`
- Scan for updates: `blogwatcher scan`
- List articles: `blogwatcher articles`
- Mark an article read: `blogwatcher read 1`
- Mark all articles read: `blogwatcher read-all`
- Remove a blog: `blogwatcher remove "My Blog"`

Example output

```
$ blogwatcher blogs
Tracked blogs (1):

  xkcd
    URL: https://xkcd.com
```

```
$ blogwatcher scan
Scanning 1 blog(s)...

  xkcd
    Source: RSS | Found: 4 | New: 4

Found 4 new article(s) total!
```

Notes

- Use `blogwatcher <command> --help` to discover flags and options.

## PilotDeck Migration Note

- Source: /var/folders/27/xyyzc_n172l3jjmnxgqmhhzh0000gn/T/tmp.AyWDWGKoS4/openclaw/skills/blogwatcher
- Review status: candidate for PilotDeck native skills pack.
- Platform-specific OpenClaw/Hermes metadata was removed or should be ignored during review.
