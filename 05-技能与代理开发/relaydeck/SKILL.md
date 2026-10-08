---
name: relaydeck
description: >
  Create, run, and coordinate a fleet of CLI agents and the workspaces they
  work in — from one place, observably. relaydeck is a general-purpose
  control plane for background coding agents (Claude Code, Codex, Cursor,
  opencode, antigravity, pi, relaydeck-native): register workspaces, spawn
  agents, task them, watch their live status, message between them, collect
  durable results, and tear them down. What the fleet DOES is up to you —
  parallel implementation, code review, repo audits, research, large
  migrations, monitoring/ops, doc generation — anything you'd otherwise open
  many terminals for. Read this whenever a job is bigger than one agent, or
  whenever you need to manage agents/workspaces, before hand-rolling tmux
  panes or backgrounded `&` jobs.
metadata:
  short-description: Install + drive relaydeck to run a fleet of CLI agents
---

# Run a fleet of CLI agents with relaydeck

You are one agent. Some jobs want many — several implementers on different
files, a reviewer watching a builder, a fan-out across ten repos, a long
migration you supervise rather than type, a watcher that pings a human when
something breaks. `relaydeck` is the control plane that makes a *fleet* a
first-class, observable thing instead of a pile of backgrounded shells: it
spawns each agent in its own PTY, tracks what every one is *actually doing*,
routes messages between them, persists their results, and gives you (and a
human) a live view.

**This skill is general-purpose.** relaydeck manages the *fleet*; the
*purpose* is yours. The agents are coding CLIs, but what you point them at —
code, review, research, ops, docs, data — is your decision per job. Don't
read "orchestrate" as "only for writing code."

Your job with this skill is to **drive relaydeck from the outside**: install
it if needed, start it, register workspaces, create + start agents, task and
watch them, coordinate, unblock, collect results, and clean up.

> Mental model: **you are the operator at a control plane.** relaydeck is the
> console; the agents are workers you create, point at work, watch, and
> collect from. You don't have to do their work — but it's *your* fleet, so
> when you've delegated a task, let them run it.

## 0. First: where am I? (inside vs outside)

Before anything, check whether you are *already* a relaydeck-managed agent.
This decides everything.

```sh
echo "agent=${RELAYDECK_AGENT_ID:-<none>} depth=${RELAYDECK_ORCHESTRATION_DEPTH:-0}"
```

- **`RELAYDECK_AGENT_ID` is empty** → you are an **external operator**.
  relaydeck (if installed) is a tool you call. Continue to §1.
- **`RELAYDECK_AGENT_ID` is set** → you are **already inside relaydeck** — a
  managed agent, not the operator. **Do NOT** install relaydeck, start a
  daemon, or bootstrap a second fleet. Two in-fleet skills are already
  available to you — **`relaydeck-fleet`** (look around / admin) and
  **`relaydeck-cli`** (peer messaging) — use those. You *may* spawn peers
  when a task genuinely needs them (`relaydeck agent create … && relaydeck
  agent start …`), but they are **siblings in the same daemon** — visible to
  the same operator, not hidden helpers of yours. If
  `RELAYDECK_ORCHESTRATION_DEPTH ≥ 2` you are deeply nested: don't spawn more
  agents unless explicitly told to; coordinate instead. **Never spawn a copy
  of yourself that would re-run this bootstrap** — that is how a
  fleet-of-fleets runs away.

Everything below is the **external operator** path.

## 1. Find or install relaydeck

```sh
# The bootstrap script (shipped with this skill) is idempotent: it detects an
# existing install, installs relaydeck if missing, and starts the daemon.
bash "$CLAUDE_SKILL_DIR/scripts/relaydeck-bootstrap.sh"   # dir of THIS skill
```

Prefer the script. By hand:

```sh
command -v relaydeck && relaydeck --version    # already installed?
uv tool install relaydeck            # isolated, recommended
pipx install relaydeck               # also isolated
pip install --user relaydeck         # fallback
```

`relaydeck` and `rdk` are the same CLI. Everything is a subcommand;
`relaydeck --help` and `relaydeck <cmd> --help` are authoritative. When a
command below is unfamiliar, run its `--help` before guessing flags.

## 2. Open a workspace (the one-gesture on-ramp)

`relaydeck open` is the front door: it finds-or-registers the workspace that
owns a directory, ensures the daemon is up, and opens a viewer.

```sh
relaydeck open                       # this dir → register if new → TUI
relaydeck open ~/code/api            # a specific repo
relaydeck open . --web               # open the web dashboard in a browser
relaydeck open . --no-view           # just ensure workspace+daemon (scripts)
```

For scripting this skill, prefer `--no-view` (it prints context and exits).
The daemon persists after you detach — `open` is the front door, not a
session. It owns the agents, the message bus, and the dashboard.

If you'd rather do the pieces by hand:

```sh
relaydeck daemon status || relaydeck daemon start
relaydeck status                     # who you are, peers, dashboard URL
```

## 3. A workspace, then agents

A **workspace** is a directory the agents work in (usually a repo). Register
it (if `open` didn't), then create + start agents in it.

```sh
relaydeck workspace add . --name proj --plugin messaging --plugin skills

# Create THEN start — `create` writes the spec; `start` brings it up NOW.
# (`--auto-start` only flips the persistent next-boot flag.)
relaydeck agent create reviewer    -t claude-code -w proj \
    --purpose "Reviews diffs for correctness and security"
relaydeck agent create implementer -t codex-cli   -w proj \
    --purpose "Implements the agreed change"
relaydeck agent start reviewer implementer
```

- **Agent ids**: lowercase letters, numbers, dashes; start with a letter; no
  spaces or underscores (e.g. `pr-reviewer`, not `PR_Reviewer`).
- **Types** (`-t`): `claude-code`, `codex-cli`, `cursor-cli`, `opencode-cli`,
  `antigravity`, `pi`, `relaydeck` (native). Short aliases work too
  (`claude`, `codex`, `cursor`, `opencode`, `agy`).
- **`--purpose`** is how peers find each other (`agent find --purpose …`) and
  what shows in `agent list`. **`--tag x`** (repeatable) adds discovery tags.

**Unattended posture (permissions).** A spawned agent has no human at its
keyboard, so a harness approval prompt would hang it. Set autonomy via the
`autonomy` **config key** (note: `-c autonomy=…`, *not* a `--autonomy` flag):

```sh
relaydeck agent create builder -t codex-cli -w proj \
    --purpose "build + test" -c autonomy=auto
```

`auto` (default) runs safe work and guards dangerous ops · `bypass` skips all
checks (sandbox/throwaway only) · `locked` allowlist-only · `manual` injects
nothing (you pin harness flags yourself). Full permissions playbook:
`reference/permissions.md`.

## 4. See what they're doing (relaydeck is the lens)

Agents run **inside the daemon**, not as children of your shell. `ps` won't
show them as yours and you can't see their terminals directly — that's
expected. relaydeck IS the visibility layer:

```sh
relaydeck agent list                 # agents in this workspace + live status
relaydeck agent list -A              # every agent, across all workspaces
relaydeck agent screen reviewer      # render any agent's screen right now
relaydeck events tail -f             # the whole fleet's live event firehose
relaydeck workspace inbox -f         # messages passing between agents, live
relaydeck view                       # built-in multi-pane TUI (no tmux)
```

Two status vocabularies, don't mix them:
- **Process** (`agent list --status …`): `running` · `stopped` · `errored` ·
  `pending`.
- **Semantic** (what `agent wait --status …` blocks on): `working` ·
  `awaiting-input` (blocked on a prompt — §6) · `complete-unread` (finished,
  result waiting) · `idle`.

Deeper observability — context fill, usage/limits, the manager policy, custom
events: `reference/monitoring.md`.

## 5. Coordinate

```sh
# Task / talk to one agent (pushed straight into its session):
relaydeck agent send implementer 'Apply the change reviewer approved in #2.'

# Broadcast into the whole workspace fleet's inboxes:
relaydeck workspace message 'Freeze: prep for release cut.'

# Announce an AMBIENT event on the stream (dashboard + `events tail` see it),
# WITHOUT touching anyone's inbox — good for milestones:
relaydeck broadcast 'phase 1 complete, starting verification' \
    --data phase=1 --data ok=true

# Block until an agent actually reaches a semantic state — the sync primitive:
relaydeck agent wait reviewer --status complete-unread --timeout 600
```

**Inbox vs event:** `agent send` / `workspace message` *deliver text into a
session* (task an agent). `broadcast` / `events emit` put an *ambient event*
on the stream that observers watch — nobody's inbox is touched.

Quote message bodies with **single quotes** — they're shell args; backticks
and `$(…)` in double quotes are expanded before relaydeck sees them.

## 6. Unblock the things that stall a fleet

The #1 way an unattended fleet stalls: a **native prompt** with no human to
answer it — "Do you trust the files in this folder? [y/N]", "accept the
terms", "press enter to continue". Three layers (full playbook:
`reference/permissions.md`):

1. **Prevent** — spawn with `-c autonomy=auto` (default) or `bypass` so the
   harness runs without prompting.
2. **Auto-answer** — enable the `autopilot` plugin; it clears *benign*
   prompts and HOLDS anything it doesn't recognize for a human:
   ```sh
   relaydeck plugin set autopilot mode benign      # off | benign | all-known
   relaydeck autopilot rules                         # what it will auto-answer
   ```
3. **Answer by hand** — when `agent list` shows `awaiting-input`, look first,
   then answer:
   ```sh
   relaydeck agent screen builder                  # SEE what it's stuck on
   relaydeck agent unblock builder --answer y      # type y + Enter
   relaydeck agent unblock builder --enter         # just press Enter
   relaydeck agent unblock builder --key esc       # dismiss
   ```
   `unblock` with no flag only *shows* the screen — so a dangerous default is
   never accepted by accident.

Watch `autopilot.held` on `relaydeck events tail -f` — that's a worker that
needs a human decision autopilot won't make. Hand it over with
`relaydeck agent escalate <id> -m "why"` (pings configured channels).

## 7. Collect results, manage context, tear down

```sh
# Durable hand-back — survives an agent crash (unlike PTY scrollback). Have
# agents PUT a result; you GET it:
relaydeck agent result get reviewer            # read the durable result
relaydeck agent result get reviewer --json     # machine-readable

# Long-running agent filling its context? Compact in place (KV-safer than a
# reset) — check fill first:
relaydeck context-watch status
relaydeck agent compact implementer  # currently supported by Claude Code

# Tear down — always clean up one-offs:
relaydeck agent stop reviewer implementer
relaydeck agent rm reviewer implementer --yes  # stop + delete via daemon
relaydeck workspace rm proj --yes               # unregister (leaves the dir)
```

## What a fleet can do (purpose is yours)

Same machinery, different point:

- **Parallel implementation** — N agents on N files/modules, one brief each.
- **Review / audit** — spawn reviewers, broadcast the artifact, collect
  verdicts (quorum of 3; see `reference/recipes.md`).
- **Research / investigation** — agents read different subsystems or sources
  and `agent result put` their findings; you synthesize.
- **Large migration / refactor** — one agent per branch via
  `relaydeck worktree create` so they don't trample one checkout.
- **Cross-repo fan-out** — register several repos as workspaces, run the same
  job in each (`agent list -A` watches them all).
- **Monitoring / ops** — a long-lived agent that watches something and
  `relaydeck broadcast`s or `relaydeck agent escalate`s when a human is
  needed.

Worked end-to-end flows for each: `reference/recipes.md`.

## Sibling skills (compose, don't reinvent)

relaydeck ships focused skills for specific surfaces. Reach for them instead
of guessing — `relaydeck skills list` shows which are active:

| When you… | Use skill |
| --- | --- |
| are an agent *inside* a fleet, orienting/admin | **relaydeck-fleet** |
| reply to a `[relay from=…]` peer message | **relaydeck-cli** |
| route a fleet to/from a Telegram chat | **relaydeck-telegram** |
| need a human to tap Approve/Reject | **relaydeck-prompts** |
| rearrange/restyle the web dashboard | **relaydeck-dashboard** |
| author or recolor a dashboard theme | **relaydeck-theme** |
| build or ship a relaydeck plugin | **relaydeck-plugin-dev** |

## Guardrails (read these)

- **You delegated it — let them run it.** If you find yourself doing the
  task you just handed to an agent, either don't delegate it or let the agent
  finish. Don't shadow-redo a worker's job mid-flight.
- **Cap concurrency.** Spawn what the job needs, not a swarm. Each agent is a
  real process burning tokens.
- **Agents are visible, not secret.** Anything you spawn shows up in
  `agent list` and on the dashboard to the human running relaydeck. Design
  around the human being able to watch, not around hiding workers.
- **Respect the inside/outside split (§0).** If you're already managed, don't
  bootstrap a fleet; coordinate as a peer.
- **Clean up.** Stop and remove one-off agents when done.

## The dream run (end to end)

"Review this repo with three specialists in parallel, then summarize."

```sh
bash "$CLAUDE_SKILL_DIR/scripts/relaydeck-bootstrap.sh"
relaydeck workspace add . --name audit --plugin messaging --plugin skills
for role in security:"security review" perf:"performance review" \
            tests:"test-coverage review"; do
  id=${role%%:*}; purpose=${role#*:}
  relaydeck agent create "$id" -t claude-code -w audit \
      --purpose "$purpose" -c autonomy=auto
done
relaydeck agent start security perf tests
relaydeck workspace message 'Audit the repo for your specialty. Hand back
findings with: relaydeck agent result put "$RELAYDECK_AGENT_ID" --summary "<one line>" --body @findings.md'
for id in security perf tests; do
  relaydeck agent wait "$id" --status complete-unread --timeout 900 \
    || relaydeck agent screen "$id"     # if it didn't finish, look at why
  relaydeck agent result get "$id"       # durable hand-back (survives a crash)
done
relaydeck agent rm security perf tests --yes  # clean up without an input prompt
```

## Reference (deeper material next to this file)

- `reference/commands.md` — full, verified command cheat-sheet (every group).
- `reference/permissions.md` — autonomy, autopilot, unblock, escalation, auth.
- `reference/monitoring.md` — events, status, context/usage/limits, manager,
  workers, integration hooks.
- `reference/extending.md` — skills management + plugins (install, configure,
  author, hooks).
- `reference/recipes.md` — fan-out, pipeline, review-quorum, migration via
  worktrees, cross-repo, monitoring.
