# Self-Hosted Control Plane

Use this branch to deploy the supported single-host Compose profile, bootstrap
its Slack installation, or diagnose pairing authority. The detailed source of
truth is `deploy/compose/README.md`; use
`docs/control-plane-deployment.md` for TLS, upgrades, backup, and recovery.

## Prepare the deployment

Work from a reviewed OpenTag checkout:

```bash
cp deploy/compose/.env.example deploy/compose/.env
```

Replace every placeholder. Generate independent database, owner, bootstrap
pairing, recovery, fencing, and login-throttle secrets. Keep `.env` outside git.

Create three protected host files:

- the 32-byte relay-content KEK;
- the Slack signing secret; and
- the Slack bot token.

Set only their host-side paths in `.env`:

```text
OPENTAG_RELAY_CONTENT_KEK_SOURCE_FILE=...
OPENTAG_SLACK_SIGNING_SECRET_SOURCE_FILE=...
OPENTAG_SLACK_BOT_TOKEN_SOURCE_FILE=...
```

The Slack credential values belong only in those files. Compose mounts them as
`/run/secrets/...`; `bootstrap-slack` stores fixed `file:` references rather
than plaintext. On native Linux, follow the Compose guide's UID/GID `10001`
readability rules without exposing the parent directory.

Completion: `.env` contains Slack IDs and secret-file paths but contains
neither Slack credential value.

## Bind Slack to the intended target

Fill every `OPENTAG_SLACK_*` identifier. Set
`OPENTAG_SLACK_PROJECT_TARGET_ID` to the stable ID that will be registered for
the paired Runner. Start with `proposal_only`; use `pull_request` only when an
explicit approver is configured.

The Slack route identity determines the public endpoints:

```text
https://control.example.com/v1/providers/slack/events/<route-identity>
https://control.example.com/v1/providers/slack/interactivity/<route-identity>
```

Completion: the team, app, channel, bot, member roles, route identity,
publication mode, and Project Target ID match the intended Slack app and local
Runner target.

## Start and verify Compose

```bash
docker compose --env-file deploy/compose/.env -f deploy/compose/compose.yaml config
docker compose --env-file deploy/compose/.env -f deploy/compose/compose.yaml up --build -d
docker compose --env-file deploy/compose/.env -f deploy/compose/compose.yaml ps
docker compose --env-file deploy/compose/.env -f deploy/compose/compose.yaml logs --no-log-prefix migrate bootstrap-admin bootstrap-slack
```

Terminate TLS at a reverse proxy before exposing Slack endpoints. Then check:

```bash
curl --fail https://control.example.com/healthz
curl --fail https://control.example.com/readyz
```

Completion: migrations and both bootstraps completed, the Control Plane and
jobs are running, `/readyz` succeeds through the trusted HTTPS origin, and no
secret value appears in rendered config or logs.

## Pair the Runner and register the target

Have the user enter the bootstrap pairing authority locally. Pair only to the
same canonical HTTPS origin used above, and supply the exact
`OPENTAG_SLACK_PROJECT_TARGET_ID` as `--project-target-id`. The Control Plane
accepts registration only when that ID is referenced by the active Slack
installation and binding; pairing computes the digest and verifies exact
repository, Runner, executor, branch, and credential-generation readback.

Completion: the Runner has a runner-scoped credential, its bootstrap authority
is removed from local config, the Control Plane target matches the local
checkout binding, and the Slack binding reports ready rather than setup
required.

## Evidence boundary

Compose health proves only this local deployment. Pairing proves only Runner
registration. Neither proves ACP execution, Slack delivery, GitHub publication,
or provider completion. Back up PostgreSQL and the exact KEK/version as one
recovery set; an unmatched database or KEK is not a valid restore.
