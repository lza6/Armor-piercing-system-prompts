# Paired-Route Troubleshooting

Use this branch when deployment, pairing, Runner readiness, a Slack mention, or
GitHub publication does not complete. Check the supported route in order and
stop at the first failed authority.

## 1. Control Plane

```bash
docker compose --env-file deploy/compose/.env -f deploy/compose/compose.yaml ps
curl --fail https://control.example.com/healthz
curl --fail https://control.example.com/readyz
docker compose --env-file deploy/compose/.env -f deploy/compose/compose.yaml logs --no-log-prefix control-plane jobs bootstrap-slack
```

Verify the public TLS origin, migrations, bootstraps, jobs process, PostgreSQL,
and KEK mount. Read logs for state and identifiers only; never print a secret
file.

Completion: `/readyz` succeeds and the intended Slack installation and binding
exist without a bootstrap conflict.

## 2. Pairing and Runner

```bash
opentag config show
opentag service status
opentag service logs
opentag doctor
opentag status
```

Confirm the canonical relay origin, paired registration and credential
generation, fresh Runner heartbeat, target mapping, checkout, and ACP
readiness. A healthy service controller alone is insufficient.

If pairing stopped with an unknown mutation outcome, reuse the persisted
operation through the CLI's reported recovery path. Do not generate another
registration or reuse a bootstrap/recovery credential for a different purpose.

Completion: one paired Runner with a current runner-scoped credential is ready
for the exact Control Plane Project Target.

## 3. Slack ingress

Confirm that Slack's Events API and Interactivity URLs contain the exact current
route identity. Check the team, app, bot, channel, installation state, binding
generation, app membership, subscribed event, scopes, and signing-secret file
reference.

Interpret common closed outcomes literally:

- `channel_binding_not_found`: the Slack conversation is not bound to the
  configured Project Target.
- setup required or temporarily unavailable: the target, Runner, or fresh
  readiness fact is missing.
- signature or route failure: fix the Slack endpoint/secret identity before
  sending another test mention.

Completion: one new signed mention is durably reserved and creates exactly one
WorkThread/Run, or returns one explicit non-success without creating work.

## 4. ACP execution

Use `opentag doctor` and Runner logs to distinguish missing local
authentication, unavailable command, rejected checkout, stale target binding,
and Attempt failure. Preserve the executor's actual conclusion and bounded
diagnostic; do not substitute another executor silently.

Completion: the intended ACP executor claims and settles one fenced Attempt in
the intended checkout.

## 5. GitHub publication and Slack projection

Check the durable publication and delivery records before looking at UI state.
Separate candidate creation, approval, provider-I/O begin, provider result,
exact-head readback, and Slack projection.

- `delivery.intent.queued` means only that delivery was durably queued.
- `provider_io_begun` means the external outcome may still be unknown.
- `accepted` or `rejected` is the recorded provider result.
- `outcome_unknown` requires reconciliation of the original operation before
  any retry.

Completion: the durable journal and provider readback support the exact status
shown in Slack. If they do not, report the unresolved boundary rather than
claiming success.
