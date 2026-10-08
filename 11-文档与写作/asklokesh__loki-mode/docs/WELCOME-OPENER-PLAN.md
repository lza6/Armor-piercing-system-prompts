# Welcome Opener Plan (the "magic opener")

## Goal
A polished welcome experience shown when people install the loki CLI or
run the Docker image. It (1) introduces Loki/Autonomi with the dashboard's
design language, (2) subtly highlights the research backing + memory
compounding moat, (3) offers a simple opt-in form collecting anonymous
role / business / tools, transmitted to PostHog on click, and (4) links to
autonomi.dev/docs.

## Hard constraints (from the user + CLAUDE.md)
- Anonymous usage analytics for PRODUCT IMPROVEMENT ONLY. NEVER transmit
  prompts, PRDs, code, or any project content. Disclosed, not covert.
- Reuse the EXISTING PostHog contract already in the codebase:
  - host `https://us.i.posthog.com`, path `/capture/`
  - public ingest key `phc_ya0vGBru41AJWtGNfZZ8H9W4yjoZy4KON0nnayS7s87`
    (same key postinstall.js + autonomy/telemetry.sh already use; public
    ingest keys are safe in client code), overridable by LOKI_POSTHOG_KEY.
  - Honor opt-out: LOKI_TELEMETRY_DISABLED=true and DO_NOT_TRACK=1.
- Design must match the loki dashboard: accent #553DE9, light bg #FAFAF7 /
  dark #0F0B1A, text #1A1614 / #F0ECF8, fonts DM Serif Display (headings),
  Inter (body), JetBrains Mono (mono). Glass cards. Light + dark.
- No emojis, no em dashes anywhere.

## Components

### 1. web-app-less static page: `assets/welcome/welcome.html`
Self-contained single HTML file (inline CSS + JS, Google Fonts link), so
it works opened directly from disk OR served. Sections:
- Hero: "Loki Mode by Autonomi" + tagline "Describe it. Walk away. Get
  working, tested software." + version (templated at serve time or left
  generic).
- Three subtle highlight cards: (a) RARV-C closure + unanimous council
  ("finishes the work, verified"), (b) Cross-project memory that compounds
  ("your agents stop repeating mistakes across projects"), (c) Research
  backing (Anthropic Constitutional AI / DeepMind debate / OpenAI Agents
  SDK / NVIDIA ToolOrchestra) shown as small footnote-style chips, not
  loud claims.
- Opt-in form: role (select), company size (select), primary tools
  (multi-select chips: Claude Code, Cursor, Codex, Copilot, Cline, Aider,
  other). A single "Send + get started" button. Consent line directly
  under it: "Sends anonymous usage analytics (your role, company size,
  tools) to help us improve the product. We never collect your prompts,
  PRDs, or code. Opt out anytime with LOKI_TELEMETRY_DISABLED=true."
- CTA row: "Read the docs" -> https://www.autonomi.dev/docs ; "Quick
  start" copyable `loki start ./prd.md`.
- On submit: POST to us.i.posthog.com/capture/ with event
  `welcome_profile`, distinct_id = the existing ~/.loki-telemetry-id (or a
  fresh uuid in browser localStorage if not injected), properties =
  {role, company_size, tools, source:'welcome_opener', loki_version}.
  Then show a thank-you state + the docs link. Submission is the ONLY
  network call; nothing is sent on page load.
- Respect opt-out: if the page is served with ?telemetry=off (the CLI
  passes this when LOKI_TELEMETRY_DISABLED/DO_NOT_TRACK is set), the form
  is replaced with a "analytics disabled" note and the submit is a no-op.

### 2. CLI command: `loki welcome`
- Mirrors cmd_dashboard_open idiom. Resolves the welcome.html path inside
  the package, opens it with `open`/`xdg-open`. If no browser opener
  (headless/Docker), prints the terminal fallback (see #4) + the file path
  + the autonomi.dev/docs URL.
- Passes ?telemetry=off when opt-out env is set.
- Dispatch: add `welcome) cmd_welcome "$@" ;;` near the `demo)` case
  (~autonomy/loki:12505) and a help line.

### 3. First-run auto-open (once)
- Marker file: `~/.loki/.welcomed`. On first `loki start` (and at end of
  postinstall when a browser is available + not CI + not opt-out), if the
  marker is absent: open the welcome page once, then write the marker.
  Never auto-open again. CI/non-TTY/Docker never auto-open a browser; they
  only print the terminal welcome.
- postinstall.js already prints an install summary; append a one-line
  "Run `loki welcome` for a quick tour" pointer (no auto browser launch in
  postinstall to avoid surprising npm installs).

### 4. Terminal fallback
- A clean ASCII/ANSI welcome (loki accent color) printed when no browser:
  product one-liner, the 3 highlights as one line each, the docs URL, the
  quick-start command, and the consent/opt-out line. No network call from
  the terminal fallback (the form is browser-only; terminal just informs).

## Files
- ADD `assets/welcome/welcome.html`
- ADD `assets/welcome/welcome.terminal.sh` (sourced helper that prints the
  terminal welcome) OR inline in cmd_welcome.
- EDIT `autonomy/loki`: cmd_welcome + dispatch + help + first-run hook in
  cmd_start.
- EDIT `bin/postinstall.js`: add the "Run `loki welcome`" pointer line.
- EDIT `package.json` "files": add `assets/`.
- EDIT `Dockerfile`, `Dockerfile.sandbox`: COPY assets/.
- ADD `tests/test-welcome-opener.sh`.
- Version bump 14 locations + CHANGELOG.

## Privacy test matrix (must pass)
- Page load makes ZERO network calls (only submit does).
- Submit payload contains ONLY {role, company_size, tools, source,
  loki_version, distinct_id}; never any file/prompt/PRD content.
- LOKI_TELEMETRY_DISABLED=true and DO_NOT_TRACK=1 each => form disabled,
  no capture.
- `loki welcome` works headless (terminal fallback, no browser error).
- First-run marker opens once and not again.

## Honesty
Highlights cite real, documented research (already in README Research
Foundation) and frame the memory moat as "retrieval/compounding," NOT a
claimed task-success number (consistent with prior honesty fixes).
