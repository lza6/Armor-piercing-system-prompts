# Slack Source App

Use this branch to configure the only supported source surface or diagnose a
missing Slack mention. Slack sends signed HTTPS requests to the self-hosted
Control Plane; the local Runner does not open a Slack connection.

## Configure the Slack app

In the Slack app console:

1. Enable the Events API.
2. Set the Request URL to
   `https://control.example.com/v1/providers/slack/events/<route-identity>`.
3. Subscribe to `app_mention` and the message event required for replies in the
   configured channel type.
4. Enable Interactivity & Shortcuts and set its URL to
   `https://control.example.com/v1/providers/slack/interactivity/<route-identity>`.
5. Grant the app the minimum scopes needed to read the configured conversation,
   post/update its thread projection, and add its receipt reaction.
6. Install the app to the intended workspace and invite it to the intended
   channel.

Use the exact route identity, team ID, app ID, channel ID, bot user ID, and
member/operator/approver/admin user IDs in the Compose bootstrap configuration.
Do not substitute display names for Slack IDs.

Completion: Slack accepts both HTTPS URLs, the app is installed and invited,
and the configured IDs refer to the same workspace, app, bot, and channel.

## Keep credentials in the Control Plane

Write the signing secret and bot token into separate protected host files. Put
only these paths in the Compose `.env`:

```text
OPENTAG_SLACK_SIGNING_SECRET_SOURCE_FILE=...
OPENTAG_SLACK_BOT_TOKEN_SOURCE_FILE=...
```

The credential values do not belong in the Runner config, ACP prompt, chat,
shell arguments, Compose environment values, screenshots, or git. The
`bootstrap-slack` service must store the mounted `file:/run/secrets/...`
references.

Completion: the running installation resolves both file references while
rendered Compose config, logs, and `opentag config show` reveal no Slack secret.

## Verify one mention

Wait until the Control Plane target exists and the paired Runner reports fresh
readiness. In the bootstrapped channel, send one bounded mention such as:

```text
@OpenTag inspect the failing test and summarize the likely cause
```

Use the installed app's actual display name. The Control Plane must verify the
raw signature and exact route before parsing or admitting the event.

Completion: one signed Slack delivery produces one WorkThread and Run, the
paired Runner claims one fenced Attempt, and the same Slack thread receives a
concise acknowledgement and truthful terminal or attention state.

## Truth boundary

A Slack acknowledgement proves neither Runner completion nor GitHub
publication. A final message must reflect durable Run and delivery evidence.
If provider I/O began but its result cannot be reconciled, retain
`outcome_unknown` and do not send the operation again automatically.
