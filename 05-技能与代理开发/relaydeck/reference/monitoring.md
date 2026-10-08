# Monitoring & observability

relaydeck is the lens on a fleet that runs inside the daemon, not in your
shell. Everything observable flows through one **event stream** plus a few
status commands. This is how you (and a human, and policy plugins) know
what's happening without attaching a terminal.

## The event stream (one firehose, many readers)

The same stream feeds the web dashboard, the `view` TUI, and the CLI:

```sh
relaydeck events tail -f                       # the whole fleet, live (SSE)
relaydeck events tail -f --type agent.         # only agent.* events
relaydeck events tail -f --agent reviewer      # only one agent
relaydeck events tail --agent reviewer --limit 50   # recent history (no -f)
relaydeck agent events reviewer -f             # one agent's log, live
```

> History is per-agent: `events tail` without `-f` requires `--agent` (there
> is no global history endpoint). Use `-f` for the live global firehose.

Emit your own events to make orchestration milestones visible (audit + hooks):

```sh
relaydeck broadcast 'phase 2 starting' --data phase=2
relaydeck events emit deploy.started --data service=api --data version=2.3
relaydeck events emit build.failed -m "tsc errors in web/"
```

**Event families worth watching** (substring-match with `--type`):

| Prefix | Means |
| --- | --- |
| `agent.status_changed` | semantic status flipped (working ↔ awaiting-input ↔ idle…) |
| `agent.result` | an agent handed back a durable result |
| `agent.context` | context-window fill crossed warn/critical/recovery |
| `agent.message_failed` | a peer message exhausted its retries |
| `autopilot.held` / `autopilot.*` | autopilot needs / made a decision |
| `usage_limits.threshold` / `.exceeded` | a session/weekly token window tripped |
| `manager.action` | the manager policy took (or recommended) an action |
| `hitl.*` | a human was asked / escalated |

These are all bridged onto the live stream, so a policy decision or a held
prompt is auditable in real time — not buried in a log.

## Status snapshots

```sh
relaydeck status                  # you + peers + unread + dashboard URL
relaydeck agent list             # this workspace: agent + semantic status + purpose
relaydeck agent list -A          # every agent across all workspaces
relaydeck agent screen <id>      # render an agent's current screen as text
relaydeck workspace inbox -f     # messages flowing between agents, live
relaydeck view                   # built-in multi-pane TUI (sidebar + PTY + msgs + events)
```

## Context-window awareness (don't let an agent overflow)

A long-running agent fills its context and degrades. `context-watch` tracks
each agent's fill (latest prompt tokens vs the model's catalogued context
window) and emits `agent.context` on warn/critical/recovery.

```sh
relaydeck context-watch status        # per-agent fill % + warn/critical state
```

When one goes critical, **compact in place** (KV-safer than a reset) — or, if
the harness can't compact, capture work and restart fresh:

```sh
relaydeck agent compact <id>                       # in-place summarize+trim
# harness has no in-place compaction →
relaydeck agent result put <id> --summary "checkpoint" --body @state.md
relaydeck agent restart <id>                       # restart as configured
```

In-place compaction is currently implemented for Claude Code. A normal
`agent restart` preserves the agent's configured resume/continue flags; it
does not promise a fresh conversation.

Models with no catalogued context window are reported honestly as unknown —
context-watch never guesses.

## Usage, rate limits & budgets

```sh
relaydeck usage [<agent>]             # token usage / metering stats
relaydeck usage-limits status         # per-agent session (5h) + weekly windows
```

`usage-limits` emits `usage_limits.threshold` / `.exceeded` as an agent
approaches/crosses a window, and can roll up to a provider-account total. Pair
with the manager (below) to act on it automatically; providers that don't
report usage (e.g. cursor) show `source=unknown`.

## The manager (policy layer)

`manager` is the plugin that turns health signals into auditable actions: it
reacts to `agent.context`(critical), `usage_limits.exceeded`, and
`agent.message_failed` and emits `manager.action`.

```sh
relaydeck manager status              # policy + recent fleet-health actions
relaydeck plugin show manager         # current settings + sources
```

Configure what it does (recommend-by-default; opt into stronger actions):

```sh
relaydeck plugin set manager on_context_critical compact   # off|recommend|compact|fresh-session|pause
relaydeck plugin set manager on_usage_exceeded   pause
relaydeck plugin set manager on_message_failed   recommend
```

Everything the manager does is a `manager.action` event on the stream — so a
human can audit every automatic decision after the fact.

## Daemon background workers

The daemon runs internal workers (inventory rescans, sweeps). When something
feels stale, inspect them:

```sh
relaydeck workers list                # status, last tick, last error
relaydeck workers logs <worker>       # recent ring-buffer log lines
relaydeck workers tail <worker>       # follow a worker's logs
relaydeck workers retry <worker>      # re-arm one stuck in crash_loop/errored
```

## Semantic-status sources

The built-in semantic engine watches every running PTY and derives `working`,
`awaiting-input`, `complete-unread`, and `idle` from the rendered screen. It
is always on and needs no installation. Claude Code can additionally use a
vendor-side hook for deterministic lifecycle signals; hookless harnesses are
listed as `classifier` entries for catalog compatibility but use the engine.

```sh
relaydeck integration list                 # hook vs always-on engine per harness
relaydeck integration install claude       # idempotent vendor hook
relaydeck integration uninstall claude
relaydeck integration cleanup-all          # before uninstalling relaydeck
```

The hook improves source precision for Claude; the engine remains the
universal fallback and reclaims stale hook/manual signals. These sources power
the status column, workspace roll-up, and `agent wait` synchronization.

## Audit trail

```sh
relaydeck audit tail                  # recent daemon audit records
relaydeck automation list             # automations with run history
```

Use these for "what happened and who/what did it" after a run — the durable
record behind the live stream.
