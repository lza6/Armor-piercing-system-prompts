# Local ACP Runner

Use this branch when choosing or diagnosing the ACP executor used by the paired
Runner. Codex is the default when its local login is ready; the same custody
rules apply to the other executors exposed by the current CLI.

## Prepare Codex

Keep the user's existing Codex authentication local. OpenTag launches the
pinned `@agentclientprotocol/codex-acp` adapter. Codex authentication stays on
the Runner host, separate from Slack and Control Plane credentials.

Before setup, confirm:

- `npx` is available;
- the local Codex session is authenticated;
- the intended checkout exists;
- unrelated checkout changes have been reviewed with the user; and
- the Runner host can create its isolated worktree and scratch directories.

Configure Codex with paired setup:

```bash
opentag setup \
  --relay https://control.example.com \
  --project /absolute/path/to/checkout \
  --executor codex \
  --github-repository owner/repo \
  --project-target-id target_team
```

Replace `target_team` with the exact Project Target ID configured for the
active Slack binding in Control Plane Compose; no duplicate Runner environment
variable is required.

Enter the GitHub credential only in the local secret prompt.

Completion: `opentag doctor` reports the Codex ACP adapter and checkout ready,
and `opentag status` identifies the paired Runner and expected target.

## Execution boundary

The ACP Agent receives the bounded task and local workspace context. It may edit
and test only the assigned checkout. Its credential boundary excludes Slack,
pairing, Runner, and GitHub secrets. Slack projection and GitHub publication
remain OpenTag-owned operations.

The Runner must report the real Attempt result and artifacts. A successful ACP
process is execution evidence only; it is not Slack delivery, GitHub
publication, or provider-verified completion.

Completion: a Slack mention is claimed by the intended Runner, one fenced ACP
Attempt executes in the intended workspace, and the structured result returns
to the Control Plane without secret material or provider authority.

## Other supported ACP executors

Choose another executor only when the user selected it and its local
authentication/runtime is ready. Use the exact executor ID reported by the
current CLI. Do not silently fall back to a different agent or to a test
executor when readiness fails.

Completion: the chosen executor, observed readiness, and executed Attempt all
identify the same ACP integration.
