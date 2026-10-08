---
name: happier-release
description: Resolve Happier's private release authority and run an exact-SHA release or nightly through cheap admission, verified CI evidence, resumable immutable candidates, and terminal publication proof. Use for preparing, dispatching, recovering, or assessing a Happier release.
metadata: {"openclaw":{"homepage":"https://github.com/happier-dev/happier"}}
---

# Happier Release

Keep release policy simple: test source once, admit the operation cheaply, build each immutable candidate once, verify once per trust boundary, promote independent products in parallel where the workflow permits, and recover without rebuilding valid work.

When the same approved `dev` source must ship to both public channels, select
the conductor's `preview-and-production` target. It reuses one exact-SHA CI and
approval packet while the canonical channel workflow runs preview and
production concurrently. Do not try to reuse preview artifact bytes for
production: channel-specific binaries embed different feature-policy
environments. The fast path removes duplicate orchestration and operator wait,
not required channel-specific builds or artifact verification.

## Resolve authority first

Inspect the public machine-readable contract with:

```bash
node scripts/pipeline/run.mjs release-contract
```

Then resolve the private release authority for the absolute checkout:

```bash
hmaint release bootstrap --repo <absolute checkout> --json
```

Use the returned private skill and its instructions as authoritative. The public contract defines targets, profiles, and compatibility intent; private operating procedure stays outside this repository.

Do not publish from a dirty or protocol-incompatible maintainer-tools checkout. Bootstrap/preflight must bind the maintainer-tools commit and return structured admission failures before any release mutation. Do not modify or clean that checkout as an implicit part of releasing.

### Execute private authority where the credentials live

First establish the actual execution host (`uname -s`, `pwd -P`) and the absolute source checkout. A path under a mounted VM workspace does not by itself mean the agent process is running inside Linux.

- On the configured macOS host, run the provisioned `hmaint` executable directly so it can use the Mac Keychain and native signing/release prerequisites. If it is not on `PATH`, use the already-configured absolute maintainer-tools executable disclosed by the environment; do not scan unrelated home directories, install another copy, or guess a checkout.
- From the managed Linux development VM, keep repository work in the authoritative VM checkout and use the Stack execution-host/broker path for Mac-only authority. Where an exact Mac command is required, use the 0.3 launcher form `apps/stack/bin/hstack-exec --target=<configured-mac-target> --cwd=<repo-relative-dir> -- <command> ...`; use the configured target name (normally `mac-host`) and never copy Keychain secrets into the VM.
- `yarn ghops auth status` is the safe credential-path probe. In a managed Linux workspace it should report the Mac-host credential broker; it must not print the token. Failure of direct Keychain access from one process does not authorize falling back to a personal `gh` login.

Before porting or releasing, resolve and compare real paths, repository roots, branch/commit bases, and dirty state. Host checkouts and VM-mounted siblings with similar names may be distinct repositories; never assume that changes written to one are visible in the other.

## Admit without repeating CI

Before expensive candidate work, run the repository's cheap, non-mutating release preflight. It validates operation-specific inputs such as source/channel identity, version and notes projection, maintainer protocol compatibility, tools, credentials, external configuration, and selected runner/platform prerequisites. It must call canonical owners and must not repeat unit, typecheck, integration, or E2E work from source CI.

Consume an explicit successful exact-SHA CI run/attestation whenever available. Pass its numeric `ci_run_id`; the release verifier must bind repository, canonical workflow, event, branch, completion, success, and exact head SHA. Nightly release validation remains artifact-specific and risk-selected—it is not another full source-CI run. If source CI is absent, fail or defer quickly through the canonical fallback instead of keeping a release runner watching another workflow.

Source CI cannot prove behavior that requires an as-yet-unpublished signed candidate. Candidate identity, archive/signature checks, installer consumption of the candidate, CLI update continuity between real published versions, store submission, and promoted-reference checks therefore remain in the release graph after their required artifact exists. Run those independent candidate validations in parallel where their prerequisites allow, but never replace them with source mocks or waive artifact verification. A collector can expose every reachable error in one attempt; it cannot execute a consumer before the artifact or external state that consumer requires exists.

Keep `fast`, `release`, and `deep` profile ownership distinct as defined in [CI cleanup](../happier-ci-stabilize/references/ci-cleanup.md). Do not turn a profile or runner backend into a copied workflow.

## Recover through one owner

For a failing or slow run, apply `skills/happier-ci-stabilize/SKILL.md`. Its recovery table decides among:

- native failed-job rerun for a same-SHA safe transient failure;
- immutable-candidate resume after a control/test-only fix, preserving individually verified successful siblings;
- fresh release when source, packaging, dependencies, signing inputs, or candidate bytes changed.

Do not blindly retry an ambiguous publication mutation. Reconcile remote state through its canonical mutation owner first. Use one foreground monitor, bound to one run and attempt, with 5-20 minute polling for long operations. Close only from terminal release status, exact candidate identity, required validation, and promoted-reference evidence.

## Preserve issue availability evidence

Issue availability is a public release contract owned by `docs/issue-triage.md`. Snapshot only the earlier `stage:*` queues proved by the selected source topology before candidate binding, and advance that snapshot only after post-promotion verification:

- current `dev` nightly: source;
- `dev` -> `preview`: source/dev;
- `preview` -> `main`: preview;
- authorized direct `dev` -> `main`: source/dev.
- coordinated `dev` -> preview + main: snapshot source/dev once and advance it
  directly to stable only after both channel releases succeed.

A reconciliation failure does not roll back published artifacts, but remains a visible release-workflow failure. Retry only the idempotent label owner or leave issues at their prior stage for the next matching release. Never compensate by closing issues or claiming availability without release evidence.
