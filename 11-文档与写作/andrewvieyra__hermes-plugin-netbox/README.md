# hermes-plugin-netbox

NetBox change management for [Hermes Agent](https://hermes-agent.nousresearch.com/).
The agent can read anything in NetBox, but it can only **write through a plan**: a field-level
diff you review, an apply step that journals every write, and a rollback that replays the
journal in reverse.

[![CI](https://github.com/andrewvieyra/hermes-plugin-netbox/actions/workflows/ci.yml/badge.svg)](https://github.com/andrewvieyra/hermes-plugin-netbox/actions/workflows/ci.yml)
![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue)
![License MIT](https://img.shields.io/badge/license-MIT-green)

```
you    > Move sw1 to SFO and mark it planned.
hermes > [netbox_query]  resolves site "sfo" -> id 2
hermes > [netbox_plan]   Plan nbp-20260908T193012Z-4f1a (planned) — Move sw1 to SFO
                         [0] update dcim/devices #10 "sw1"
                               site: 1 -> 2
                               status: active -> planned
                         Summary: 0 create, 1 update, 0 delete, 0 noop
         Apply this plan? (yes/no)
you    > yes
hermes > [netbox_apply]  [0] update dcim/devices #10 -> done
         Applied. sw1 is now in SFO with status planned.
```

## Why a plugin

Reading NetBox from an agent is easy: a skill wrapping `curl`, or NetBox Labs' read-only MCP
server, both work. Writing is different. "Precisely every time" needs logic that does not depend
on the model remembering to do it: resolve targets, diff against live state, refuse to touch an
object that changed since planning, record an inverse for every write, revert in reverse order on
failure. That logic lives in Python here and runs the same way on every call.

## How it works

```
 netbox_plan                      netbox_apply                       netbox_rollback
 ───────────                      ────────────                       ───────────────
 validate operations              preflight: status, age, URL,       for each journal entry,
 resolve id / match  ──GET──►     deletes allowed                    newest first:
 diff desired vs live             for each step:                       re-read object
 record last_updated                re-read, compare last_updated      compare last_updated
 save plan (planned)                write ──POST/PATCH/DELETE──►       run inverse ──►
                                    journal inverse + new stamp        journal outcome
                                  failure ► revert done steps        status: rolled_back |
                                  status: applied | failed             partially_rolled_back
```

A plan is a JSON file under `~/.hermes/plugin-data/netbox/plans/`. It is written after every step
of apply and rollback, so an interrupted run leaves a journal that `netbox_rollback` (or the CLI)
can finish. See [docs/architecture.md](docs/architecture.md) for the full model and guarantees.

## Install

Requirements: Hermes Agent 0.21 or newer, Python 3.10+, `requests` (already in the Hermes venv),
a NetBox 3.5+ instance and an API token with the permissions you intend to grant the agent.

```bash
hermes plugins install andrewvieyra/hermes-plugin-netbox
```

Hermes prompts for `NETBOX_URL` and `NETBOX_TOKEN` and stores them in `~/.hermes/.env`. Then:

```bash
hermes plugins enable netbox
```

Start a new session. The tools show up in the `netbox` toolset and `/netbox status` confirms
connectivity.

Manual install works too: clone this repository to `~/.hermes/plugins/netbox`, add the two
variables to `~/.hermes/.env`, and enable the plugin.

### Configuration

Settings live under `plugins.entries.netbox.settings` in `~/.hermes/config.yaml`:

```yaml
plugins:
  enabled: [netbox]
  entries:
    netbox:
      settings:
        allow_delete: false      # delete operations are refused unless true
        max_operations: 200      # cap per plan
        max_query_results: 500   # cap per netbox_query call
        max_plan_age_hours: 24   # older plans are refused by netbox_apply
        audit_log: true          # append events to audit.jsonl
        audit_log_path: ""       # empty = <HERMES_HOME>/plugin-data/netbox/audit.jsonl
        audit_include_request: true  # record the triggering message (truncated) in the actor
        write_mode: full         # full | operator_only | read_only
        plan_retention_days: 90  # prune plan files older than this; 0 keeps them forever
        link_changelog: true     # attach NetBox object-change ids to the journal after apply/rollback
        audit_sinks: []          # optional syslog / HTTP forwarding, see docs/sinks.md
```

Environment variables:

| Variable | Required | Meaning |
|---|---|---|
| `NETBOX_URL` | yes | Base URL, e.g. `https://netbox.example.com` |
| `NETBOX_TOKEN` | yes | API token. Legacy tokens are sent as `Token …`, v2 `nbt_…` tokens as `Bearer …` |
| `NETBOX_VERIFY_SSL` | no | `false` to skip certificate verification (self-signed labs) |
| `NETBOX_TIMEOUT` | no | Request timeout in seconds (default 30) |

## Tools

| Tool | Writes | Purpose |
|---|---|---|
| `netbox_query` | no | List objects with filters, fetch one by id, or check `status` |
| `netbox_plan` | no | Validate and diff a list of operations; save a plan |
| `netbox_apply` | yes | Apply a plan with preconditions, journal and auto-rollback; `dry_run` re-checks only |
| `netbox_rollback` | yes | Revert an applied or partially applied plan |
| `netbox_plans` | no | List plans or show one with its diff, journal and rollback result |

The bundled skill `netbox:workflow` (see [SKILL.md](SKILL.md)) tells the model to show the diff and
ask before applying. The tool descriptions repeat that rule, and in unattended contexts the skill
instructs the model not to apply at all.

### Operations

`netbox_plan` takes `description` and an ordered `operations` list:

```json
{"op": "create", "endpoint": "dcim/devices", "match": {"name": "sw8"},
 "data": {"name": "sw8", "site": {"slug": "nyc"}, "role": 1, "device_type": 1, "status": "planned"}}
{"op": "update", "endpoint": "dcim/devices", "id": 10, "data": {"status": "active"}}
{"op": "update", "endpoint": "dcim/devices", "match": {"name": "sw2", "site": "nyc"}, "data": {"serial": "X1"}}
{"op": "delete", "endpoint": "ipam/ip-addresses", "id": 20}
```

- `endpoint` is the API path relative to `/api/`: `dcim/devices`, `ipam/prefixes`, `plugins/bgp/sessions`.
- `id` or `match` locates the target. `match` is a set of NetBox filter parameters and must hit
  exactly one object.
- `create` with `match` is an upsert: an existing match becomes an `update` (or a `noop`).
- `data` is the desired state in writable form: integer ids or lookup dicts for related objects,
  values for choice fields, full lists for tags, a partial dict for `custom_fields`. Only fields
  present in `data` are compared and written.

More in [examples/](examples/).

### Slash command and CLI

Inside a session, `/netbox` gives an operator the same actions without going through the model:

```
/netbox status
/netbox plans [status]
/netbox show <plan_id>
/netbox check <plan_id>
/netbox apply <plan_id> [--no-rollback]
/netbox rollback <plan_id> [--force]
/netbox prune [--days N] [--dry-run]
```

The same subcommands exist as `hermes netbox …` in the shell, useful for rolling back after a
session has ended. Both accept an unambiguous fragment of the plan id, so `/netbox apply 4f1a`
works from a phone; the full id remains the canonical form in files and tool output.

## Safety model

- **Read-only until a plan exists.** `netbox_query`, `netbox_plan` and `netbox_plans` never write.
- **Every write is diffed first.** Updates PATCH only the fields that differ; unchanged
  operations become `noop` steps.
- **Optimistic concurrency.** Each update and delete records the object's `last_updated` at
  planning time and refuses to run if it differs at apply time. Upserts refuse to run if a match
  appeared. A conflict stops the plan like an error does.
- **Journaled inverses.** Before the next step starts, the previous step's inverse is on disk:
  delete the created object, PATCH the previous values, or re-create from a snapshot.
- **Automatic rollback on failure**, in reverse order, on by default. The outcome of every revert
  is journaled too.
- **Rollback respects later edits.** An object modified after the apply is skipped as a conflict
  unless `force` is set.
- **Deletes are opt-in.** `allow_delete` defaults to false. Rollback of a delete re-creates the
  object with a new id and cannot restore objects that NetBox cascaded.
- **Plans are bound.** A plan applies at most once, only against the NetBox URL it was built for,
  and only within `max_plan_age_hours`. The claim is taken under a cross-process file lock, so the
  gateway and the CLI cannot run the same plan concurrently.
- **No secrets in plans.** Plan files and the audit stream hold object data as returned by the API and never the token.
- **Everything is attributed.** Each plan, apply and rollback records who asked, from which platform and chat, and the HTTP calls made. See [docs/audit.md](docs/audit.md).

### Write modes

Hermes plugins cannot open an approval prompt from inside a tool, so in the default `full` mode the
confirmation step is the model asking you in chat and then calling `netbox_apply` after you say yes.
That is behaviour, driven by the skill and the tool descriptions, not a lock. `write_mode` adds the lock:

| `write_mode` | Who can write | Use it when |
|---|---|---|
| `full` | The model, after asking; and `/netbox`, `hermes netbox` | One trusted user in a direct message, no automation |
| `operator_only` | Only `/netbox` in a session and `hermes netbox` in a shell. Model calls to apply and rollback are refused with the exact command to relay | Group chats, messaging gateways, cron jobs present, prompt-injection concerns, "a human must cause every write" |
| `read_only` | Nobody. The apply and rollback tools are not registered for the model; `/netbox apply` refuses too | Drift audits, demos, first deployments |

Plans persist across every Hermes surface on the same `HERMES_HOME`, so `operator_only` supports
"plan from Signal, apply from the laptop." Refusals are written to the audit stream.

Whatever the mode, scope the API token to the permissions you want the agent to have. A read-only
token is the one safeguard NetBox enforces itself.

## Audit trail

Every plan, apply and rollback is recorded for review and for SIEM ingestion:

- **In the plan file:** who requested the plan (`requested_by`), who applied and who rolled back
  (`apply.actor`, `rollback.actor`), the environment it ran in (`audit`: host, OS user, Hermes and
  plugin versions, NetBox version), and the HTTP call behind every journal entry.
- **Linked to NetBox's change log.** After each apply and rollback the matching object-change
  records, with NetBox's `request_id`, are attached to the journal, so the two audit trails reference
  each other.
- **In an append-only event stream**, `<HERMES_HOME>/plugin-data/netbox/audit.jsonl`, one JSON
  object per line: `plan_created`, `apply_started`, `step_done`, `apply_finished`, `rollback_*`,
  and the refusals and rejections that never reached NetBox.

The actor record distinguishes a model tool call from a human using `/netbox` or `hermes netbox`,
and carries the gateway platform, chat, user, session, and the message that triggered the call.
All timestamps are UTC. Point your collector at the file, or set `audit_log_path` to a path it
already watches. Schema and examples: [docs/audit.md](docs/audit.md).

Installs without a log shipper can forward events directly: `audit_sinks` sends each event to a
syslog server (UDP, TCP or TLS, JSON or CEF) and/or an HTTP collector such as Splunk HEC, from a
background thread that never delays a NetBox operation. See [docs/sinks.md](docs/sinks.md).

## Compatibility

- NetBox 3.5 through 4.x. The `fields` query parameter needs NetBox 4.0+; `brief` works everywhere.
  Custom-field partial updates and nested lookup dicts follow NetBox 4 semantics.
- Hermes Agent 0.21+. The plugin uses only the documented `register(ctx)` surface:
  `register_tool`, `register_command`, `register_cli_command`, `register_skill`, `get_config`.

## Development

```bash
make test        # unit tests against the in-memory fake NetBox, no network
make lint        # ruff
make doctor      # hermes plugins doctor . --ci
HERMES_AGENT_ROOT=~/.hermes/hermes-agent make test-hermes   # load through Hermes' real PluginManager
```

The fake NetBox in `tests/fake_netbox.py` implements pagination, filters on nested objects,
choice fields, `last_updated` stamps, DRF-style 400 bodies and an injectable failure hook, so
apply, conflict and rollback paths are exercised end to end without a server.

Layout:

```
plugin.yaml   manifest (tools, env, settings schema)
__init__.py   register(ctx)
schemas.py    tool schemas the model sees
handlers.py   tool handlers: parse args -> planner/executor -> JSON
commands.py   /netbox slash command and `hermes netbox` CLI
client.py     NetBox REST client (auth, pagination, errors)
diff.py       desired-vs-live comparison, writable conversion
planner.py    operations -> plan (resolve, diff, preconditions)
executor.py   apply with journal, rollback from journal
store.py      plan persistence under plugin-data
audit.py      actor capture and the audit.jsonl event stream
timefmt.py    UTC-to-local rendering for reports (Hermes' timezone setting)
sinks.py      optional syslog / HTTP forwarding of audit events
settings.py   operator settings
SKILL.md      bundled skill: the workflow the model follows
```

## License

MIT. See [LICENSE](LICENSE).
