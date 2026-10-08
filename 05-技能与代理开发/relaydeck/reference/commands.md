# relaydeck command cheat-sheet

Every command here is real (verified against the CLI). `relaydeck --help` and
`relaydeck <cmd> --help` are authoritative — when in doubt, run `--help`
rather than guessing a flag. `rdk` is an alias for `relaydeck`.

**Flag conventions that trip agents up:**
- Agent **type** is `-t/--type`; **workspace** is `-w/--workspace`.
- **Autonomy/permissions** are a *config key*: `-c autonomy=auto` — there is
  **no** `--autonomy` flag.
- Agent **ids**: lowercase letters/numbers/dashes, start with a letter, no
  underscores (e.g. `pr-reviewer`).
- Message **bodies** are positional shell args — single-quote them.

## On-ramp & health
| Command | What |
| --- | --- |
| `relaydeck --version` | Confirm it's installed |
| `relaydeck open [PATH]` | **One-gesture on-ramp**: find-or-register the workspace owning `PATH` (cwd default), ensure the daemon, open a viewer. `--web` → browser dashboard; `--no-view` → just ensure workspace+daemon (scriptable); `--name`, `-p/--plugin`, `--no-register` |
| `relaydeck init [PATH] [-p plugin]` | Alias for `workspace add` with defaults |
| `relaydeck daemon start \| stop \| restart \| status` | Daemon lifecycle |
| `relaydeck daemon logs` | Tail the daemon log |
| `relaydeck status [--agent <id>]` | Who you are, peers, unread inbox, **dashboard URL** |
| `relaydeck doctor` | Self-diagnostic (config, auth, DB, plugins) |
| `relaydeck update` | Upgrade relaydeck in place to the latest release |

## Workspaces
| Command | What |
| --- | --- |
| `relaydeck workspace add <path> -n <name> [-p messaging -p skills]` | Register a working dir |
| `relaydeck workspace list` | Registered workspaces + health |
| `relaydeck workspace info [name]` | Plugins + agents in a workspace |
| `relaydeck workspace set <name>` | Durable default workspace for this machine |
| `relaydeck workspace plugins <name> [--add p] [--remove p] [--set p]` | Edit a workspace's enabled plugins (no flags = print current) |
| `relaydeck workspace rm <name> [--yes]` | Unregister (leaves the directory; prompts without `--yes`) |

## Worktrees (parallel branches → workspaces)
| Command | What |
| --- | --- |
| `relaydeck worktree create <branch> --repo <path> [--base ref] [--existing] [--name n] [-p plugin] [--no-setup]` | New git worktree + register as workspace + run setup hook |
| `relaydeck worktree list` | Worktree workspaces with branch + git status |
| `relaydeck worktree remove <name>` | Teardown hook → `git worktree remove` → unregister |

## Agents — lifecycle
| Command | What |
| --- | --- |
| `relaydeck agent create <id> -t <type> -w <ws> --purpose "<role>" [--tag x] [-c autonomy=auto] [--system-prompt "…" \| --system-prompt-file F] [--auto-start]` | Define an agent (doesn't start it) |
| `relaydeck agent start <id> [<id>…]` | Bring up now |
| `relaydeck agent stop <id> [<id>…]` | Stop running agent(s) |
| `relaydeck agent restart <id> [<id>…]` | Stop then start with the agent's configured session flags |
| `relaydeck agent rm <id> [<id>…] [--yes] [--keep-history]` | Stop + delete through the daemon; history is purged unless retained explicitly |
| `relaydeck agent edit <id> [--purpose …] [--add-tag x] [--system-prompt …] [--show]` | Update meta/prompt (no flags → $EDITOR) |

`-t` types: `claude-code`, `codex-cli`, `cursor-cli`, `opencode-cli`,
`antigravity`, `pi`, `relaydeck` (+ aliases `claude`/`codex`/`cursor`/
`opencode`/`agy`).

## Agents — observe & discover
| Command | What |
| --- | --- |
| `relaydeck agent list [-w <ws>] [-A] [--status running\|stopped\|errored\|pending] [-q]` | Agents + live status (`-q` = ids only, scriptable) |
| `relaydeck agent find [--purpose <substr/regex>] [--tag <t>] [-A]` | Discover peers by purpose/tag |
| `relaydeck agent screen <id>` | Render an agent's current screen as text |
| `relaydeck agent events <id> [-f] [--type <substr>]` | One agent's event log |
| `relaydeck attach <id>` | Attach to an agent's PTY (like `tmux attach`) |

## Agents — coordinate
| Command | What |
| --- | --- |
| `relaydeck agent send <id> '<body>'` | Push a message into an agent's session |
| `relaydeck agent wait <id> --status <s> [--not-status <s>] [--timeout <sec>]` | **Block** until a semantic status (`working`/`awaiting-input`/`complete-unread`/`idle`). Exit 0 reached · 1 timeout · 2 usage · 3 transport |
| `relaydeck agent unblock <id> [-a <answer> \| --enter \| --key <k>]` | Answer a stuck native prompt (no flag = only show the screen) |
| `relaydeck agent compact <id>` | Compact context in place when supported (currently Claude Code) |
| `relaydeck agent escalate <id> [-m msg]` | Hand to a human now (HITL escalation to your channels) |

`--key` names: `enter, esc, ctrl-c, tab, up, down, left, right, y, n, space, backspace`.

## Agents — durable results
| Command | What |
| --- | --- |
| `relaydeck agent result put <id> --summary "…" [--body @file \| --body -] [--key K]` | Hand back a **durable** result (survives a crash). From inside an agent: `put "$RELAYDECK_AGENT_ID" …` |
| `relaydeck agent result get <id> [--key K] [--all] [--json]` | **Collect results** — the reliable hand-back path |
| `relaydeck agent viewed <id>` | Mark a result read (the read-transition) |
| `relaydeck agent transcript <id>` | An exited agent's last screen (crash recovery; opt-in via `RELAYDECK_TRANSCRIPT_BYTES`) |

## Messaging & events
| Command | What |
| --- | --- |
| `relaydeck workspace message '<body>' [--agent <id>] [--from <id>] [--wait <sec>]` | Broadcast into inboxes (or one agent) |
| `relaydeck workspace inbox [-f] [--full] [--agent <id>]` | Read messages passing through |
| `relaydeck reply <msg-id> '<body>'` | Threaded reply to a `[relay …]` line (infers recipient) |
| `relaydeck message show <msg-id>` | Read one message by id |
| `relaydeck broadcast '<msg>' [--type T] [--data k=v]` | Ambient event on the stream (not inbox) |
| `relaydeck events emit <type> [--data k=v] [-m msg]` | Emit a custom event |
| `relaydeck events tail -f [--type <substr>] [--agent <id>]` | The fleet firehose (history needs `--agent`, no `-f`) |

**Inbox vs event:** `workspace message` / `agent send` *deliver text into a
session*; `broadcast` / `events emit` put an *ambient event on the stream*
the dashboard, `view`, and `events tail` watch — nobody's inbox is touched.

## Permissions & unblocking → `reference/permissions.md`
| Command | What |
| --- | --- |
| `relaydeck plugin set autopilot mode <off\|benign\|all-known>` | How aggressively autopilot auto-answers |
| `relaydeck autopilot status \| rules` | Mode + active episodes · the allowlist |
| `relaydeck autopilot test "<screen text>"` | Dry-run the matcher (no agent touched) |
| `relaydeck auth show \| token \| rotate \| issue \| revoke \| list` | Daemon auth token + scoped Bearer tokens |

## Monitoring → `reference/monitoring.md`
| Command | What |
| --- | --- |
| `relaydeck context-watch status` | Each agent's context-window fill + warn/critical |
| `relaydeck usage [<agent>]` · `relaydeck usage-limits status` | Token usage · session/weekly windows per agent |
| `relaydeck manager status` | Manager policy + recent fleet-health actions |
| `relaydeck hitl status \| test` · `relaydeck hitl ask "<q>"` | HITL channels · test escalation · request a human |
| `relaydeck workers list \| logs <w> \| tail <w> \| retry <w>` | Daemon background workers |
| `relaydeck integration list` | Show each harness's semantic-status source (hook or always-on engine) |
| `relaydeck integration install claude \| uninstall claude` | Manage Claude's vendor hook; classifier entries have nothing to install |
| `relaydeck view [-w <ws>]` | Built-in multi-pane TUI |

## Skills & plugins → `reference/extending.md`
| Command | What |
| --- | --- |
| `relaydeck skills list \| show <name> \| doctor` | Inventory + health |
| `relaydeck skills add <source> [-w ws] [--skill name] [--mode symlink\|copy\|reference]` | Import skill(s) from any source |
| `relaydeck skills link <path> -w <ws> [--alias a]` | Import an external skill dir into a workspace |
| `relaydeck skills install [--target claude\|codex\|both] [--force]` | Install THIS bundled `relaydeck` skill into your skill roots |
| `relaydeck plugin list \| show <name> \| info <name>` | Loaded plugins + settings + provenance |
| `relaydeck plugin enable/disable <name>` | Toggle a plugin globally |
| `relaydeck plugin set <name> <key> <value>` · `unset <name> <key>` | Configure a plugin setting |
| `relaydeck plugin new <name> [--pattern …] [--local \| --workspace ws]` | Scaffold a plugin |
| `relaydeck plugin install <src> [--editable]` · `uninstall <name>` | Install/remove a plugin |
| `relaydeck plugin bundle [name]` | Recommended plugin bundles |

## Models
| Command | What |
| --- | --- |
| `relaydeck provider list` | Model catalogs from provider plugins |
| `relaydeck defaults list \| get <role> \| set <role> <spec> \| unset <role>` | Default model per role (classifier, voice, image, …) |
| `relaydeck preset list` | Named model presets |
| `relaydeck preset create <name> -p <provider> -m <model>` | Create a preset |
| `relaydeck preset edit <name> [-p provider] [-m model]` · `rm <name>` | Edit or remove a preset |
| `relaydeck recipe list \| show <name>` | Reusable system-prompt addenda |

## See everything at once
- `relaydeck view` — built-in multi-pane TUI (sidebar + live PTY + messages +
  events). `Ctrl-B D` to detach (the agents keep running).
- The **web dashboard** (URL from `relaydeck status`) — real-time fleet view.
  Reshape it with the **relaydeck-dashboard** skill.
