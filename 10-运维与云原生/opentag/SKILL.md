---
name: opentag
description: Deploy or operate OpenTag's supported paired setup when a user needs to bootstrap the self-hosted Docker Compose Control Plane and Slack Source App, configure or pair a local ACP Runner with a GitHub Project Target, start the Runner service, verify readiness, or diagnose a Slack mention that did not complete.
---

# OpenTag

## Supported route

```text
Slack Source App
  -> self-hosted Control Plane
  -> paired local Runner
  -> ACP Agent in one local checkout
  -> GitHub Project Target/publication
  -> truthful result in the originating Slack thread
```

Slack is the only Source App. GitHub is the Project Target and
publication/evidence provider. The Control Plane owns Slack ingress, custody,
and projection; the Runner owns local execution.

## Load the needed reference

- Compose deployment, Slack bootstrap, TLS, or pairing authority:
  `references/control-plane.md`
- Slack app URLs, events, scopes, channel binding, or missing mentions:
  `references/slack-setup.md`
- GitHub target, local checkout, publication, or readback:
  `references/github-setup.md`
- Codex or another supported ACP executor:
  `references/codex-runner.md`
- Any failed readiness, pairing, Run, or delivery step:
  `references/troubleshooting.md`

Load only the references whose branch applies.

## Guardrails

- Pair only with the exact HTTPS Control Plane origin the user operates or
  explicitly trusts.
- Keep Slack signing and bot secrets in Control Plane secret files. Its `.env`
  contains only their host-side file paths and non-secret Slack identifiers.
- Enter the bootstrap pairing authority and GitHub token through local secret
  input. Keep them out of chat, command arguments, logs, screenshots, and git.
- The ACP Agent edits only the assigned local checkout. OpenTag performs Slack
  delivery and GitHub publication through governed provider boundaries.
- Preserve unrelated working-tree changes. Confirm the checkout and target
  before enabling write-capable work.
- Run completion, enqueue, approval, and provider acceptance are separate
  facts. Preserve `outcome_unknown` until the original provider operation is
  reconciled.

## Paired workflow

1. Bootstrap the Control Plane and Slack installation from
   `deploy/compose/.env.example`, using file-backed Slack secrets, then start
   Compose behind TLS.
   Completion: the Compose project reports healthy, `/readyz` succeeds, and
   bootstrap logs show the intended Slack binding without secret plaintext.

2. Install the reviewed CLI and verify the local checkout and ACP login.

   ```bash
   npm install -g @opentag/cli@0.11.0
   opentag --version
   git -C /absolute/path/to/checkout status --short
   ```

   Completion: the CLI reports `0.11.0`, the user has accepted the checkout
   state, and the chosen ACP executor is locally authenticated.

3. Configure the Runner, trusted relay, checkout, ACP executor, and GitHub
   Project Target. Omit secret flags so the CLI prompts locally.

   ```bash
   opentag setup \
     --relay https://control.example.com \
     --project /absolute/path/to/checkout \
     --executor codex \
     --github-repository owner/repo \
     --project-target-id target_team
   ```

   Replace `target_team` with the active Slack binding's Project Target ID from
   Compose. Do not require a duplicate Runner environment variable for it.

   Setup performs the initial pair. For an existing configuration that is
   still unpaired, complete that same pairing with:

   ```bash
   opentag pair \
     --relay https://control.example.com \
     --trust-relay-origin https://control.example.com
   ```

   Completion: redacted config shows `paired_relay`, the exact trusted origin,
   a paired Runner registration, and the intended GitHub target ID; pairing has
   registered that target through the active Slack binding and verified exact
   Control Plane readback. The bootstrap token is not retained in Runner config.

4. Keep the Runner active in one supported mode.

   ```bash
   opentag start
   # or, after a global install:
   opentag service install
   opentag service start
   opentag service status
   ```

   Completion: the service reports running and ready, the runtime credential
   is accepted by the Runner Control Context endpoint, and the configured ACP
   executor is ready.

5. Verify before inviting work.

   ```bash
   opentag doctor
   opentag status
   opentag config show
   ```

   Completion: required checks pass, the relay and Runner identities match,
   the checkout maps to the intended GitHub target, and displayed secrets are
   redacted.

6. Mention the installed OpenTag app in the bootstrapped Slack channel with one
   bounded task.
   Completion: the signed Slack event creates one WorkThread and Run, the
   paired Runner claims one fenced Attempt, and the originating thread receives
   a concise result or an explicit actionable failure.

The setup is complete only when every completion criterion above holds on the
paired route. A local process exit, queued delivery, generated pull-request URL,
or Slack acknowledgement alone is insufficient.
