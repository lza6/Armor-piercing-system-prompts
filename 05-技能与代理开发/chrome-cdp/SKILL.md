---
name: chrome-cdp
version: 1.1.0
description: |
  Drive a real Chrome via the Chrome DevTools Protocol (CDP). Ensures a
  debuggable Chrome is running (launching it if needed) so the agent can
  navigate, read page content, inspect the DOM, run JavaScript, and take
  screenshots — without the user typing any launch command. Includes a
  first-class "review a live running app" flow: point Chrome at the app under
  development and inspect/screenshot the REAL page (what a static-artifact tool
  cannot do). Backed by Glimpse and the chrome-devtools MCP server. Use when
  asked to "use chrome", "open chrome", "drive my browser", "read the page in
  chrome", "review the running app", "screenshot the app", or "/chrome-cdp".
triggers:
  - chrome cdp
  - drive my browser
  - read the page in chrome
  - use chrome
  - review the running app
  - screenshot the app
---

# chrome-cdp

Connect this agent to a real Chrome through the Chrome DevTools Protocol.

## Ensure Chrome is debuggable

```bash
glimpse chrome        # launches Chrome with --remote-debugging-port (no-op if already up)
```

It uses a dedicated profile (Chrome 136+ blocks remote debugging on the default
profile). The endpoint is `http://127.0.0.1:9222`.

## Use it
- If the `chrome-devtools` MCP tools are loaded (`navigate_page`, `take_snapshot`,
  `click`, `evaluate_script`, …), use them directly.
- If not (e.g. the MCP server was just added and the agent hasn't restarted),
  drive Chrome via the bundled commands:
  - `glimpse read <url>` — navigate + dump page text
  - `glimpse shot <out.png> [url]` — screenshot
  - or raw CDP (Node has a built-in WebSocket): HTTP `/json` to list/open
    targets, WebSocket for `Page.navigate` / `Runtime.evaluate`.

## Notes
- This drives a **dedicated profile**, not the user's everyday Chrome (their
  logins/tabs). To use a logged-in site, log in once in that Chrome window.
- Anything in that Chrome is fully readable/controllable by the agent.
