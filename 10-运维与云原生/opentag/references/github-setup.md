# GitHub Project Target

Use this branch to configure the local checkout, the Control Plane target, or
governed publication/readback. Slack supplies every human request; GitHub
supplies the bound target and publication evidence.

## Establish one exact target

Confirm the intended checkout before setup:

```bash
git -C /absolute/path/to/checkout status --short
git -C /absolute/path/to/checkout remote get-url origin
```

Preserve unrelated changes. Confirm that `origin` names the intended
`owner/repo`, the base branch is correct, and the local git identity can create
the Runner-owned branch used for publication.

Run paired setup without a token argument so the GitHub credential is entered
through the local secret prompt:

```bash
opentag setup \
  --relay https://control.example.com \
  --project /absolute/path/to/checkout \
  --executor codex \
  --github-repository owner/repo \
  --project-target-id target_team
```

Use the repository guide at `docs/platforms/github.en.md` when selecting a
fine-grained token. Limit it to the target repository and the readback and
draft-pull-request operations the deployment enables.

Completion: redacted Runner config names the exact GitHub repository, checkout,
remote, base branch, and ACP executor without exposing the token.

## Match the Control Plane binding

Pass the exact `OPENTAG_SLACK_PROJECT_TARGET_ID` from the bootstrapped Control
Plane to setup. Pairing registers the target only when that ID is already
referenced by an active Slack installation and binding, then verifies provider,
owner, repository, default branch, executor, digest, and Runner through the
authoritative Control Context readback.

Completion: the Slack binding, Control Plane Project Target, and Runner local
allowlist resolve to one target identity and current binding generation.

## Publication authority

The ACP Agent may edit and verify only the assigned local checkout. OpenTag
alone owns Slack delivery and GitHub access: it freezes the base revision, records the
candidate and branch ownership, requires the configured approval, then performs
each GitHub publication step through the paired publication protocol.

Treat these as distinct facts:

- local files changed;
- a publication candidate was recorded;
- the exact candidate was approved;
- branch publication began;
- GitHub accepted a draft pull request;
- the expected head and required evidence were read back.

Completion: any published draft pull request matches the approved repository,
base, branch, and head SHA, and the durable receipt records the exact provider
outcome.

## Ambiguous outcomes

A generated URL, local process success, or queued intent is not provider proof.
When a GitHub request may have reached the provider but the response is lost,
retain `outcome_unknown`. Reconcile the remote branch or pull request by exact
operation identity and expected head before deciding whether another attempt is
safe.
