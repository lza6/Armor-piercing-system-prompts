---
name: netbox-workflow
description: Plan, review, apply and roll back NetBox changes with the netbox_* tools. Load when the user asks to read, add, change, move, rename, retire or bulk-edit anything in NetBox (devices, interfaces, IPs, prefixes, sites, racks, circuits, VMs).
version: 0.4.0
metadata:
  hermes:
    tags: [netbox, dcim, ipam, infrastructure, change-management]
    requires_tools: [netbox_plan, netbox_apply]
---

# NetBox change workflow

Five tools, one rule: **nothing is written to NetBox without a plan the user has seen.**

| Tool | Writes? | Use it to |
|---|---|---|
| `netbox_query` | no | Look objects up, resolve names to ids, check `status` |
| `netbox_plan` | no | Turn desired changes into a diff and a `plan_id` |
| `netbox_apply` | yes | Execute a reviewed plan; auto-rollback on failure |
| `netbox_rollback` | yes | Revert an applied plan from its journal |
| `netbox_plans` | no | List plans or show one plan's diff and journal |

## Procedure

1. **Understand the target.** Query first. Resolve every related object (site, role, device type,
   tenant, VRF, tags) to an integer id with `netbox_query` and filters such as
   `{"slug": "nyc"}` or `{"name__ic": "core"}`. Use `fields` or `brief` to keep responses small.
2. **Build the plan.** Call `netbox_plan` with an ordered `operations` list. Use `match` instead of
   `id` when the user named the object ("the device sw1") so the plan is self-describing. Use
   `create` + `match` when the user says "make sure X exists" (upsert).
3. **Show the diff.** Paste the returned `diff` text to the user verbatim. Point out deletes,
   ambiguous-looking changes, and any `warnings`. If the response says nothing to apply, stop.
4. **Get explicit confirmation.** Ask a direct question ("Apply plan nbp-…? yes/no") and wait.
   In an unattended session (cron, webhook) do not apply; leave the plan for a human.
5. **Apply.** `netbox_apply` with the `plan_id`. If it fails, report the failed step, the error,
   and the rollback outcome from `report`. Do not retry blindly: re-query, re-plan, re-confirm.
6. **Verify.** Query the touched objects once more and confirm the result matches the intent.
7. **Rollback on request.** `netbox_rollback` with the `plan_id`. Objects someone else edited after
   the apply are reported as conflicts; only use `force: true` if the user explicitly accepts
   overwriting those edits.

## Write modes

The operator sets `write_mode` in `config.yaml`. Read the refusal message and act on it:

- `full` (default): you may call `netbox_apply` after the user confirms.
- `operator_only`: `netbox_apply` and `netbox_rollback` refuse model calls. Tell the user the exact
  command from the refusal, `/netbox apply <plan_id>` in this session or `hermes netbox apply <plan_id>`
  in a shell, and stop. Do not retry the tool.
- `read_only`: the apply and rollback tools are not available. Plans are still useful as a diff of what
  would change; say so and stop.

## Writing `data`

- Related objects: integer id (`"site": 3`) or a lookup dict (`"site": {"slug": "nyc"}`).
  Never a bare slug string for a foreign key: NetBox rejects it at apply time.
- Choice fields: the value, not the label (`"status": "planned"`).
- Tags: a list of ids or lookup dicts (`"tags": [{"slug": "core"}]`). The list is the full desired set.
- Custom fields: a partial dict (`"custom_fields": {"owner": "netops"}`); only listed keys change.
- Only fields present in `data` are compared and written. Omit anything you do not intend to change.

## Pitfalls

- A `match` must hit exactly one object. Two hits fail the plan: add a second filter (`site`, `role`).
- Filters are NetBox query parameters: `site` takes a slug, `site_id` an id, `name__ic` a substring.
- Deletes are disabled unless the operator set `allow_delete: true`. NetBox cascades deletes; a
  rollback re-creates the object with a new id and cannot restore cascaded children.
- Plans expire (`max_plan_age_hours`, default 24) and refuse to run if any target changed since
  planning. That is the safety net working; re-plan rather than force.
- Order operations so dependencies come first (create the site before the device in it). Steps run
  in order and roll back in reverse.

## Example

```json
{
  "description": "Move sw1 to SFO and mark it planned",
  "operations": [
    {"op": "update", "endpoint": "dcim/devices", "match": {"name": "sw1"},
     "data": {"site": {"slug": "sfo"}, "status": "planned"}},
    {"op": "create", "endpoint": "extras/journal-entries",
     "data": {"assigned_object_type": "dcim.device", "assigned_object_id": 10,
              "kind": "info", "comments": "Relocated to SFO per ticket NET-42"}}
  ]
}
```
