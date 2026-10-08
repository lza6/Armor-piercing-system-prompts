# Architecture

## Goals

1. The agent can change NetBox, but every write is preceded by a reviewable diff.
2. A plan runs the same way every time: deterministic order, explicit preconditions, journaled
   inverses, reverse-order rollback. None of it depends on the model remembering a step.
3. The plugin stays inside Hermes' documented plugin surface so it survives Hermes upgrades.
4. No NetBox model knowledge is hard-coded. Endpoints are paths; field semantics are inferred
   from the API representation. New NetBox models and plugins work without changes here.

## Modules

| Module | Responsibility | Depends on |
|---|---|---|
| `client` | HTTP: auth header, pagination, error typing, endpoint validation | `requests` (lazy) |
| `diff` | Compare writable desired values with API representations; convert API form back to writable form | none |
| `planner` | Validate operations, resolve targets, diff, build the plan document | `client`, `diff`, `settings`, `store` |
| `executor` | Preflight, per-step precondition, execute, journal, rollback | `client`, `diff`, `settings`, `store` |
| `store` | Atomic JSON persistence of plans, process-wide store handle | Hermes `plugin_storage` (optional) |
| `timefmt` | UTC stamps rendered in the Hermes-configured zone for reports only | Hermes `hermes_time` (optional) |
| `sinks` | Optional syslog / HTTP forwarding of audit records on a background thread | none |
| `audit` | Actor capture from Hermes' session context; append-only `audit.jsonl` event stream | Hermes `gateway.session_context` (optional) |
| `settings` | Operator settings with coercion and defaults | Hermes `ctx.get_config` (optional) |
| `schemas` | Tool schemas | none |
| `handlers` | Tool handlers: args to JSON, error mapping, client factory seam | everything above |
| `commands` | `/netbox` and `hermes netbox` built on the handlers | `handlers` |
| `__init__` | `register(ctx)` | all |

Dependencies point downward only. `client`, `diff`, `planner`, `executor` and `store` have no
Hermes imports at module load, which is why the whole engine is testable with plain `unittest`.

## The plan document

```jsonc
{
  "id": "nbp-20260908T193012Z-4f1a",
  "version": 1,
  "status": "planned",            // planned | applying | applied | failed | rolling_back | rolled_back | partially_rolled_back
  "description": "Move sw1 to SFO",
  "netbox_url": "https://netbox.example.com",
  "created_at": "2026-09-08T19:30:12Z",
  "summary": {"create": 0, "update": 1, "delete": 0, "noop": 0},
  "warnings": [],
  "steps": [
    {
      "index": 0, "action": "update", "endpoint": "dcim/devices", "object_id": 10, "label": "sw1",
      "requested_op": "update", "resolved_by": "match",
      "changes": {"site": {"from": 1, "to": 2}, "status": {"from": "active", "to": "planned"}},
      "payload": {"site": 2, "status": "planned"},
      "precondition": {"last_updated": "2026-09-01T10:00:00.000000Z"}
    }
  ],
  "journal": [
    {
      "index": 0, "action": "update", "endpoint": "dcim/devices", "object_id": 10, "status": "done",
      "inverse": {"action": "update", "endpoint": "dcim/devices", "object_id": 10, "data": {"site": 1, "status": "active"}},
      "after_last_updated": "2026-09-08T19:31:02.113311Z",
      "revert_status": "reverted", "revert": {"restored_fields": ["site", "status"]}
    }
  ],
  "apply": {"started_at": "...", "finished_at": "...", "outcome": "applied", "rollback_on_failure": true},
  "rollback": {"started_at": "...", "finished_at": "...", "reason": "requested", "reverted": 1, "conflict": 0, "failed": 0}
}
```

Step shapes by action:

| action | extra fields | precondition at apply |
|---|---|---|
| `create` | `data`, optional `exists_check` (the `match`) | no object matches `exists_check` |
| `update` | `changes`, `payload`, optional `upsert: true` | object exists and `last_updated` unchanged |
| `delete` | `snapshot` (full API object) | object exists and `last_updated` unchanged |
| `noop` | `reason` | none; journaled as `skipped` |

## State machine

```
planned ──apply──► applying ──all steps ok──► applied ──rollback──► rolling_back ──► rolled_back
                      │                                                          └──► partially_rolled_back ──rollback──┐
                      └──failure/conflict──► failed ──(auto or manual rollback)──► rolling_back ──► …                   │
                                                                                                                       ▼
                                                                                                             (retry until rolled_back)
```

- `planned -> applying` (and the rollback claim) happen under a cross-process file lock,
  `<plans>/.lock`, after re-reading the file, so two applies of one plan cannot both proceed even from
  different processes (gateway and CLI). A lock held longer than `PlanStore.claim_timeout` refuses
  with a retry hint.
- `failed` with no `done` journal entries has nothing to revert; automatic rollback is skipped and
  the status stays `failed` (the plan cannot be applied again; re-plan).
- `partially_rolled_back` can be rolled back again; only entries not yet `reverted` are retried,
  which is how `force` finishes a rollback that stopped on conflicts.

## Diff semantics

`diff.values_equal(desired, current)` decides whether a writable value already describes the API
value:

| `current` shape | `desired` accepted as equal when |
|---|---|
| nested object `{"id", "name", "slug", …}` | int equals `id`; str equals `slug`, `name` or `display`; dict is a subset match |
| choice `{"value", "label"}` | str or `{"value"}` equals `value` |
| list | same length and every desired element matches a distinct current element |
| plain dict | deep equality (except `custom_fields`, compared key by key) |
| scalar | equal, or numeric-string equal (`"1"` vs `1`); bool never coerces |

`diff.to_writable` maps API form back to writable form (nested object to id, choice to value,
recursively). The `from` side of every change is stored in writable form so the inverse PATCH is
ready without further lookups.

`snapshot_to_payload` builds a POST body from a delete snapshot: read-only keys (`id`, `url`,
`display`, `created`, `last_updated`, `*_count`, …) are dropped and nested values converted. If
NetBox rejects the re-create with a 400 naming specific fields, those fields are dropped and the
POST retried once; the dropped fields are journaled.

## Concurrency and crash safety

- Apply and rollback persist the plan after every step. A crash between steps leaves a file whose
  journal shows exactly which inverses are pending; `hermes netbox rollback <id>` finishes it.
- Precondition checks are read-then-write, not atomic. NetBox has no conditional-write primitive,
  so a change landing between the check and the PATCH is not detected for that step. The window is
  one request. Rollback's own `after_last_updated` check closes the equivalent window on the way back.
- Steps run sequentially. Bulk endpoints are deliberately not used: one request per object keeps
  the journal exact and the rollback granular.

## Hermes integration

- `register(ctx)` performs no I/O beyond reading settings; Plugin Doctor blocks sockets during
  registration and this plugin passes.
- Tools are registered with `check_fn=is_configured` and `requires_env`, so they are hidden from the
  model when `NETBOX_URL` / `NETBOX_TOKEN` are missing rather than failing at call time.
- Handlers follow the contract `handler(args, **kwargs) -> str` and never raise.
- The plan store uses `plugins.plugin_storage.plugin_data_dir` when available so plan files follow
  the active Hermes profile, with a plain `$HERMES_HOME/plugin-data/netbox` fallback.

## Audit

Actor records and the event stream are described in [audit.md](audit.md). The executor emits an
event after every persisted state change, so the stream and the plan file never disagree about
what was attempted; an audit write failure is logged and never fails the operation.

## Non-goals

- Human approval UI. Hermes plugins cannot open an approval prompt from inside a tool; the skill
  and tool descriptions drive the model to ask. `write_mode: operator_only` is the hard version of
  that rule: the model cannot write at all and `/netbox` or the CLI is the only path.
- Bulk endpoints, async apply, or parallel steps.
- Schema-aware validation of `data` before apply. NetBox validates on write; the plan's rollback
  covers the failure.
