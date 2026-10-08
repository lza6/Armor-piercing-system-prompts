---
name: file-a-task
description: File work into no_human (task_add) and check on it (task_status) via the no_human MCP bridge, instead of doing the work inline.
---

# File a task into no_human

Use the two MCP tools this plugin exposes — `task_add` and `task_status` — to
hand off work that should become a reviewed pull request, instead of making
the change yourself in this conversation.

## When to use

- The work is substantial enough to deserve its own branch, its own tests,
  and independent review — not a one-line edit you can just make inline.
- You want no_human to plan, code, test, have the result reviewed by a second
  model, and open a pull request, while you keep doing something else.
- Do **not** use this for small edits you can make directly in the current
  conversation — that is strictly slower and adds no value.

## Prerequisite: the local server must already be running

Both tools call the existing no_human HTTP API at `http://127.0.0.1:8420`.
The bridge does **not** retry or fall back if that address is unreachable —
it refuses to start at all (`SystemExit`, no auto-retry). Before using either
tool, make sure the server is running:

```bash
nh start
```

(The desktop app starts this for you automatically.) If a tool call fails
with a connection error, the fix is to start the server, not to retry the
call.

## Tool: `task_add`

Creates a new no_human task via `POST /api/tasks`.

**Parameters (all required strings):**

| Name | Type | Description |
|---|---|---|
| `title` | string | Short task title. |
| `description` | string | Full task description — inline the concrete context here (exact file paths, exact acceptance criteria), rather than referencing this conversation, since no_human cannot see it. |
| `repo_path` | string | Absolute path to the target repo checkout the task should run against. |

You do not pass `source` — the bridge sets it to `"mcp"` on every call it
makes, server-side, before the request is sent.

**Returns:** compact JSON with the new task's id and the source the server
actually stored (first-class alongside `"board"`/`"jira"`), e.g.:

```json
{"task_id":"task-abc123","source":"mcp"}
```

## Tool: `task_status`

Fetches a task's current full state via `GET /api/tasks/{id}`.

**Parameters (required string):**

| Name | Type | Description |
|---|---|---|
| `task_id_or_external_id` | string | Either the task's own id (or a unique prefix of it) or its external id. |

Resolution order: first tries the value as a task id (or id prefix); if that
404s, falls back to a client-side scan of `GET /api/tasks` for a matching
`external_id`, then re-fetches by the real id. If nothing matches either way,
the call raises — a nonexistent id is an error, not an empty result.

**Returns:** the complete task object as compact JSON, including at least
`id`, `external_id`, `source`, `title`, `description`, `status`,
`created_at`, `updated_at`, e.g.:

```json
{"id":"task-abc123","external_id":null,"source":"mcp","title":"Fix thing","description":"...","status":"in_progress","created_at":"2026-08-20T12:00:00Z","updated_at":"2026-08-20T12:05:00Z"}
```

`task_status` is read-only polling — call it again later to check progress;
it never changes the task.

## Writing a good ticket

Inline everything the coding agent will need directly into `description`:
exact file paths, exact acceptance criteria, any commands to run. Do not
reference "the conversation above" or anything in your own context — the
task runs in its own fresh session and cannot see it.

## Product boundary — no_human opens a PR and stops

**no_human plans, codes, tests, has the work reviewed by a second model,
opens the pull request, and stops there. Merge is always the human's
action — this skill must never instruct or attempt a merge, an approval, or
a push to a protected branch.** There is no tool exposed here that can merge
or approve anything, and none should be sought or improvised — do not run any
merge or approval command on the agent's behalf. Once `task_status` reports a
PR link, your job is to report that link back to the human — not to merge it,
approve it, or push further changes to it yourself.
