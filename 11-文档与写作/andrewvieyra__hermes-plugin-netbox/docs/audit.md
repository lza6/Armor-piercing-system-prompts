# Audit trail

Every plan, apply and rollback is recorded twice: inside the plan file, and as a stream of events in
`audit.jsonl`. The plan file is state that changes as the plan progresses. The stream is append-only
and is what a SIEM or log shipper should read.

All timestamps are UTC in ISO 8601 with a `Z` suffix. Nothing in either output contains the NetBox
token.

## Where

| Output | Location | Setting |
|---|---|---|
| Plan files | `<HERMES_HOME>/plugin-data/netbox/plans/<plan_id>.json` | none |
| Event stream | `<HERMES_HOME>/plugin-data/netbox/audit.jsonl` | `audit_log` (on/off), `audit_log_path` (absolute path override) |

Set `audit_log_path` to a location your collector already watches, for example
`/var/log/hermes/netbox-audit.jsonl`. The writer opens the file in append mode for every event, so
`logrotate` with `copytruncate` is safe, and several Hermes processes (gateway and CLI) can append to
the same file without coordination.

## The actor record

Attached to every event and stored in the plan under `requested_by`, `apply.actor` and
`rollback.actor`. Keys are always present; unknown values are `null` so field mappings stay stable.

| Field | Meaning |
|---|---|
| `kind` | `model` when the model called a tool, `operator` when a human used `/netbox` or `hermes netbox` |
| `via` | `tool`, `slash` or `cli` |
| `platform` | Gateway platform: `signal`, `telegram`, `discord`, `bluebubbles`, `cli`, … |
| `source` | Hermes session source (`gateway`, `cli`, `tui`, `desktop`, `api_server`, …) |
| `profile` | Hermes profile, when multiplexing |
| `chat_id`, `chat_name`, `chat_type`, `thread_id`, `scope_id` | Where the conversation happened; `chat_type` is `dm`, `group`, `channel` or `thread` |
| `user_id`, `user_id_alt`, `user_name` | The platform user who sent the message (`user_id_alt` is the platform's stable alternate id, such as a Signal UUID) |
| `message_id` | The triggering message, where the platform exposes one |
| `session_key`, `session_id`, `task_id` | Hermes session identifiers, for joining with Hermes' own logs |
| `cron` | `true` when the turn ran inside a Hermes cron job |
| `os_user` | The OS account running Hermes |
| `request` | The user's message that led to the call, truncated to 500 characters; `null` when `audit_include_request` is off |

Identity comes from Hermes' per-session context, which the gateway binds for every turn. In a plain
CLI session most platform fields are `null` and `os_user` is the meaningful identity.

## The plan file

Besides the actor records, each plan carries an `audit` block written at planning time:

```json
"audit": {
  "host": "hermes-01",
  "os_user": "hermes",
  "pid": 41233,
  "plugin_version": "0.2.0",
  "hermes_version": "0.21.0",
  "hermes_home": "/home/hermes/.hermes",
  "netbox_version": "4.3.0"
}
```

and every journal entry records the HTTP call it made:

```json
"request": {"method": "PATCH", "path": "/api/dcim/devices/10/", "status": 200}
```

A reverted entry adds `revert_request` for the inverse call.

## NetBox's change log

After every apply and rollback the plugin reads NetBox's object-changes for the run's time window
(widened by a few seconds for clock skew) and attaches the matching records to each journal entry:

```json
"netbox_changes": [
  {"id": 8123, "request_id": "0b7c4d2e-…", "time": "2026-09-08T19:35:50.412Z", "changed_object_type": "dcim.device"}
]
```

Rollback writes get `revert_netbox_changes`. The `apply` and `rollback` blocks carry a summary,
`{"linked": true, "matched": 3, "unmatched": 0, "window": [since, until]}`, or `{"linked": false,
"error": …, "status": 403}` when the token cannot read the log. NetBox 4.x serves the log at
`core/object-changes`, 3.x at `extras/object-changes`; both are tried. The lookup is one extra request
per phase, never fails the operation, and can be turned off with `link_changelog: false`.

With the ids in place, a plan's journal and NetBox's change log reference each other: from NetBox
you can search by `request_id` to see the exact object diffs; from the audit stream, the
`changelog_linked` event lists the request ids for the run.

## Events

One JSON object per line. Common fields on every event:

| Field | Meaning |
|---|---|
| `ts` | UTC timestamp |
| `schema` | Event schema version, currently `1` |
| `plugin` | `netbox` |
| `event` | Event name, below |
| `plan_id` | The plan, or `null` for a rejected plan or an unknown id |
| `netbox_url` | The NetBox instance |
| `actor` | The actor record |
| `host`, `pid` | Where the event was produced |

| Event | When | Extra fields |
|---|---|---|
| `plan_created` | A plan was built and saved | `description`, `summary`, `steps`, `warnings` |
| `plan_rejected` | Operations failed validation or resolution; nothing saved | `description`, `operations`, `errors`, `first_error` |
| `plan_checked` | `dry_run` re-verified preconditions | `conflicts`, `applicable` |
| `apply_refused` | Preflight refused: wrong status, stale plan, other NetBox, deletes disabled, unknown id, second apply, lock held, or `write_mode` | `reason`, `dry_run`, `write_mode` when that was the cause |
| `apply_started` | Plan claimed and execution begins | `steps`, `summary`, `rollback_on_failure` |
| `step_done` | One write succeeded | `index`, `action`, `endpoint`, `object_id`, `label`, `http_status`, `request` |
| `step_skipped` | A `noop` step | `index`, `action`, `endpoint`, `object_id`, `label` |
| `step_conflict` | The object changed since planning; run stops | `index`, …, `error` |
| `step_failed` | NetBox rejected the write; run stops | `index`, …, `error`, `http_status`, `request` |
| `apply_finished` | Outcome of the run | `outcome` (`applied` or `failed`), `status`, `done`, and on failure `failed_step`, `failure_kind`, `error` |
| `rollback_refused` | Rollback preflight refused, or `write_mode` | `reason`, `force`, `write_mode` when that was the cause |
| `plans_pruned` | Retention removed plan files (`plan_id` is null) | `max_age_days`, `count`, `plan_ids` |
| `changelog_linked` | NetBox change-log lookup after an apply or rollback | `phase`, `linked`, `matched`, `unmatched`, `request_ids`; or `error`, `http_status` |
| `rollback_started` | Reverting begins, newest entry first | `entries`, `force`, `reason` (`requested`, or `automatic after failure at step N`) |
| `revert_reverted`, `revert_conflict`, `revert_failed` | Outcome of one inverse | `index`, `action`, `endpoint`, `object_id`, `label`, `error`, `new_object_id`, `http_status`, `request` |
| `rollback_finished` | Outcome of the rollback | `status`, `reverted`, `conflict`, `failed`, `force`, `reason` |

A complete successful lifecycle emits `plan_created`, `apply_started`, one `step_*` per step,
`apply_finished`. A failure adds `step_failed` or `step_conflict`, then, when automatic rollback is
on, `rollback_started`, one `revert_*` per completed step, `rollback_finished`.

## Example

```json
{"ts":"2026-09-08T19:35:50Z","schema":1,"plugin":"netbox","event":"step_done","plan_id":"nbp-20260908T193512Z-4f1a","netbox_url":"https://netbox.example.com","actor":{"kind":"model","via":"tool","platform":"signal","source":"gateway","profile":null,"chat_id":"+15550001111","chat_name":"Andrew","chat_type":"dm","thread_id":null,"scope_id":null,"user_id":"+15550001111","user_id_alt":"9b1c…","user_name":"Andrew","message_id":"1757360110","session_key":"signal:dm:+15550001111","session_id":"7c0e…","task_id":"7c0e…","cron":false,"os_user":"hermes","request":"Move sw1 to SFO and mark it planned"},"host":"hermes-01","pid":41233,"index":0,"action":"update","endpoint":"dcim/devices","object_id":10,"label":"sw1","status":"done","http_status":200,"request":{"method":"PATCH","path":"/api/dcim/devices/10/","status":200}}
```

## Ingestion notes

- To push events instead of tailing the file, configure `audit_sinks` (syslog or HTTP). See [sinks.md](sinks.md).

- The stream is newline-delimited JSON. Point Filebeat, Fluent Bit, Vector or the Splunk forwarder at
  the file with a JSON codec; every line is a self-contained record.
- Map `ts` to your timestamp field (`@timestamp` in Elastic).
- Useful searches: `event:apply_started AND actor.kind:model AND actor.chat_type:group` (a model
  wrote to NetBox from a group chat), `event:apply_refused` (attempted writes that were stopped),
  `event:step_failed OR event:revert_failed` (needs a human), `actor.cron:true AND event:apply_*`
  (unattended writes).
- Join with NetBox's change log on `netbox_url`, `request.path` and time window to see the resulting
  object diffs from NetBox's point of view.
