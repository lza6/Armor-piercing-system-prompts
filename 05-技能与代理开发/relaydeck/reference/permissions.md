# Permissions, unblocking & safety

The single biggest threat to an unattended fleet is a **native prompt with no
human to answer it** — a worker hits "Do you trust the files in this folder?
[y/N]" or "press enter to continue" and silently sits there forever while you
think it's working. relaydeck gives you a detect → prevent → answer →
escalate stack. Use it deliberately.

## The lifecycle of a stuck agent

1. The harness prints a prompt and blocks on stdin.
2. relaydeck's **semantic-status engine** reads the rendered screen ~1×/s and
   flips the agent to `awaiting-input` (visible in `agent list`, on the
   dashboard, and as an `agent.status_changed` event). *Detection is
   automatic for every harness — it never types for you.* A fresh vendor hook
   or human signal temporarily takes precedence; see `reference/monitoring.md`.
3. From there, one of three things answers it: autonomy (set at spawn), the
   `autopilot` plugin (automatic), or you (`agent unblock`).

## Layer 1 — Prevent at spawn (autonomy)

Autonomy is a **config key**, set with `-c autonomy=<mode>` (there is **no**
`--autonomy` flag):

```sh
relaydeck agent create w -t codex-cli -w proj --purpose "build+test" \
    -c autonomy=auto
```

- `auto` (**default**) — run safe work autonomously, still guard genuinely
  dangerous ops. Best default for unattended runs.
- `bypass` — skip all approval/sandbox checks. Fastest, **no guardrails**;
  use only in a sandbox/throwaway workspace you trust.
- `locked` — fail-safe: only an allowlist runs, everything else is denied.
- `manual` — inject nothing; you pin the harness's own flags via `-c args=…`.

Regardless of mode, every harness auto-allows the `relaydeck` CLI itself
(peer messaging / replies / status), so fleet coordination never stalls.

Autonomy covers the harness's *own* approval prompts (tool/permission/
sandbox). It does **not** cover first-run OS-ish prompts like trust-this-
folder, license acceptance, or "update available" — those are Layers 2/3.

Operator-pinned harness flags win: if you pass e.g. `-c args="--permission-mode
plan"` (claude) or `-c sandbox=workspace-write -c approval_policy=never`
(codex), relaydeck respects them and stays out of the way.

## Layer 2 — Auto-answer the benign ones (autopilot)

The `autopilot` plugin watches for `awaiting-input` and auto-answers a
**small, curated, conservative** set of prompts, then HOLDS anything it
doesn't recognize for a human (it never guesses).

```sh
relaydeck plugin set autopilot mode benign        # default
relaydeck autopilot rules                          # exactly what it will answer
relaydeck autopilot test "Do you trust the files in this folder? [y/N]"
relaydeck autopilot status                         # mode + agents mid-prompt
```

Modes:
- `off` — detection only; you/HITL handle everything.
- `benign` (default) — only always-safe prompts: "press enter to continue"
  (→ Enter) and trust-this-workspace (→ y, because you already chose to run
  an agent here).
- `all-known` — also declines in-session update prompts (never swap the
  binary mid-run) and, **only if** `auto_accept_terms=true`
  (`relaydeck plugin set autopilot auto_accept_terms true`), accepts
  terms/license prompts.

Autopilot deliberately has **no "just press the default" rule** — the matcher
is case-insensitive, so it can't tell `[Y/n]` (default yes) from `[y/N]`
(default no), and blindly accepting an unknown default could be destructive.
Unrecognized prompts are emitted as `autopilot.held` and left for a human.

```sh
relaydeck events tail -f --type autopilot          # unblocked + held, live
```

## Layer 3 — Answer by hand / from your loop (agent unblock)

When `agent list` shows `awaiting-input` and autopilot held it (or is off),
you decide. **Always look first**, then answer:

```sh
relaydeck agent screen w                 # see exactly what it's asking
relaydeck agent unblock w --answer y      # type "y" + Enter
relaydeck agent unblock w --enter         # just Enter (e.g. "press enter")
relaydeck agent unblock w --key esc       # dismiss / cancel
relaydeck agent unblock w                 # NO action — only prints the screen
```

The no-flag form is the safe default: it shows the prompt and sends nothing,
so you never accept a dangerous default by reflex. Fully scriptable inside a
supervisor loop:

```sh
while :; do
  for a in $(relaydeck agent list -A -q); do
    if relaydeck agent list -A | grep -q "$a.*awaiting-input"; then
      relaydeck agent screen "$a"        # decide based on what you see
      # … apply unblock OR escalate to the human …
    fi
  done
  sleep 5
done
```

## Layer 4 — Escalate to a human (hitl)

For prompts that genuinely need a person (a real approve/deny, a destructive
op), hand it over:

```sh
relaydeck agent escalate w -m "wants to force-push main — needs a human"
relaydeck hitl ask "Deploy v2.3 to prod?"          # an agent asks directly
relaydeck hitl status                               # which channels are wired
relaydeck hitl test                                 # fire a test escalation
```

`escalate` / `hitl` emit a HITL escalation to every configured channel (the
dashboard bell always; Telegram/etc. if wired). For a *button* prompt
(tap-able Approve/Reject) use the **relaydeck-prompts** skill
(`relaydeck prompts ask … --wait`). Autopilot and hitl compose: autopilot
clears the benign noise, hitl/escalate raises the rest.

## Operator authentication (the daemon token)

The daemon is protected by a bearer token; the CLI uses it automatically.
When you script against the API or share access, manage tokens explicitly:

```sh
relaydeck auth show                    # location + redacted token
relaydeck auth token                   # print it raw (for $RELAYDECK_TOKEN / curl)
relaydeck auth issue --label ci --scope read-only  # mint a scoped Bearer token
relaydeck auth list                    # labels, scopes, last-used, expiry
relaydeck auth revoke <id>             # invalidate one immediately
relaydeck auth rotate                  # new daemon token, old invalidated
```

Prefer a **scoped issued token** for anything non-interactive; `rotate` the
daemon token if it leaks.

## Plugin trust

Plugins carry a trust level (`bundled > curated > local > untrusted`). An
`untrusted` plugin (a third-party entry point not in the curated registry or
lockfile) won't load unless approved on install or
`RELAYDECK_ALLOW_UNTRUSTED_PLUGINS=1`. `relaydeck plugin list` shows each
plugin's Source + Trust. See `reference/extending.md`.

## Decision table

| Prompt | Who answers | Result |
| --- | --- | --- |
| Harness tool/permission/sandbox approval | autonomy (`auto`/`bypass`) | runs without prompting |
| "press enter to continue" | autopilot `benign` | Enter |
| "trust the files in this folder?" | autopilot `benign` | y |
| "update available?" | autopilot `all-known` | declined (n) |
| "accept the terms?" | autopilot `all-known` + `auto_accept_terms` | y |
| Arbitrary `[Y/n]` default | **held** | you / hitl decide |
| "Delete production? (type DELETE)" | **held → escalate** | a human, always |

## Golden rules

- **Detection is free and automatic; answering is a deliberate choice.**
- **Look before you unblock.** `agent screen` first, every time.
- **Never auto-accept what you haven't vetted** — keep `auto_accept_terms`
  off unless you know what your harnesses prompt for.
- **`bypass` only in throwaway/sandboxed workspaces.**
- **Escalate, don't guess**, when a wrong answer is destructive.
