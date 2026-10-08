---
name: protocol:program-goal-charter-template
description: "Blank and filled Program Goal Charter template for phase-program umbrella plans. Defines the north-star, definition of done, scope tiers, and hard safety constraints for autonomous multi-phase sessions."
date: 15-06-26
metadata:
  node_type: reference
  type: protocol
  read_order: 10
  required: false
  read_when: "building a phase-program umbrella plan or needing the compressed session-goal block format"
---
# Program Goal Charter Template

This is the canonical template for the **Program Goal Charter** required by every phase program.

Read this after your project's planning context entrypoint when:

- building the umbrella orchestration plan for a phase program
- you need the durable "north star" the user would otherwise hand-paste every run
- you need the compressed session-goal block that launches an unattended long-running session

The charter is governed by `process/development-protocols/phase-programs.md` ("Program Goal Charter").

## What the charter is for

A phase program runs for a long time, often unattended, across many phases and sessions. The charter
captures the program-specific intent and safety boundaries once, durably, inside the umbrella plan, so
no agent has to re-derive (or the user re-paste) the north star, the definition of done, and the hard
safety constraints on every run.

## What the charter is NOT for

Do NOT re-paste execution discipline into the charter. The required 10-step per-phase loop,
re-research at phase entry, regression checkpoints, and honest phase status are already defined in
`process/development-protocols/phase-programs.md`. The charter references that protocol; it does not
duplicate it. Keep the charter to program-specific intent and safety only.

## Blank Charter Template

```text
# [PROGRAM NAME] — Program Goal Charter

North star:
- [one sentence stating the real end goal]

Definition of done:
- [the concrete capabilities an unattended agent must be able to perform when the program is complete]

What "verified" means (program level):
- [the exact bar for promoting work to ✅ VERIFIED for this program — gate surface, evidence, coverage]

Scope tiers → phase mapping:
- Tier 1 [name] → Phases [n, n, ...]
- Tier 2 [name] → Phases [n, n, ...]
- Tier 3 [name] → Phases [n, n, ...]
- This program retires Tiers [1-N].

Explicitly out of scope (deferred tier):
- [the tier and items intentionally not addressed by this program]

Hard safety constraints (non-negotiable, per phase):
- [program-specific irreversible/destructive boundaries, e.g. "never mutate prod X"]
```

## Compressed Session-Goal Block (printed in chat at kickoff)

After the umbrella plan and charter exist, print this block in chat (do NOT write it to a file). It is
the copy-pasteable launch packet for an unattended run. Keep it to roughly 8-12 lines.

**Hard rule:** the session-goal block MUST be under 4000 characters total — it is pasted into a
persistent `/goal` whose ceiling is ~4000 chars. If the program's safety constraints +
definition-of-done won't compress under 4000 chars, summarize and reference the charter's plan path
for the full detail rather than inlining everything.

```text
SESSION GOAL: [PROGRAM NAME]
Charter + umbrella plan: process/features/{feature}/active/{umbrella-plan}.md
Autonomy: Run autonomously under this persistent goal. Execute phases on your own
recommendation via the 10-step loop in phase-programs.md; report conflicts, errors, and
learnings in the phase report (the report is the communication channel, not a question).
Only pause for outward-facing / irreversible / costful / destructive actions
(see feedback_autonomous_phase_execution.md).
Hard stop conditions / safety constraints:
- [hard safety constraint 1 from the charter]
- [hard safety constraint 2 from the charter]
Next phase: process/features/{feature}/active/{next-phase-plan}.md
```

## Filled-In Reference Example

Real charter from the example-co.app "Full Product E2E" program. Note: execution-discipline prose is
intentionally absent — it lives in `process/development-protocols/phase-programs.md`.

The charter example below and the session-goal example further down both use example-co.app as the
sample program; they illustrate different aspects (charter structure vs compressed launch block)
and are not the same program instance.

```text
# Full Product E2E (example-co.app) — Program Goal Charter

North star:
- Make example-co.app's full-product E2E system reliable enough that a future agent can autonomously
  implement a feature, write/extend its tests, run them headlessly with complete evidence, diagnose
  and self-heal failures in a bounded loop, run highest-risk slices against production-fidelity
  targets, clean up all state safely, and hand off only after behavior is genuinely verified —
  including cross-tenant isolation — without a human babysitting the run.

Definition of done (the unattended agent must do all of these):
1. Implement a product change across web/tRPC/D1/R2/Cloudflare Stream/middleware.
2. Author Bun integration + Playwright specs against isolated disposable targets (per-run E2E SQLite
   file, isolated port, miniflare-local D1/R2) — never local.db or prod.
3. Run headless with full evidence (screenshots, video, traces, console, network, tRPC, server logs →
   redacted evidence bundle on every failure).
4. Diagnose: classify failures (product vs test vs harness drift vs stale-command vs schema drift)
   with a preflight doctor catching env problems (stale port 3301 listener, missing Clerk env,
   un-built schema, missing Chromium).
5. Self-heal in a bounded loop (real loop driver, bounded retries, BS_TEST_ATTEMPT
   increment/read, fail-closed at bound).
6. Clean up safely (E2E DB file, ports, R2/Stream state restored; prod never mutated; cleanup
   verified).
7. Hand off honestly (promote to ✅ VERIFIED only when phase gates AND regression checks pass with
   evidence; no green-by-scope-widening).

What "verified" means (program level):
- Default safe gate surface (test:safe → doctor → typecheck incl tests → lint → bun test w/ coverage
  → test:e2e:guards → isolated product specs → test:fidelity) runs green from a clean checkout, with
  no manual env override, redacted evidence on every failure, cross-tenant isolation + Clerk
  auth-failure branches covered, and highest-risk surfaces (D1 semantics, R2 serve, /api/upload
  routes, Cloudflare Stream contract) exercised against miniflare real bindings.

Scope tiers → phase mapping:
- Tier 1 autonomy blockers → Phases 00, 03, 04, 05, 06.
- Tier 2 fidelity & flakiness → Phases 01, 02, 07.
- Tier 3 coverage holes → Phases 08, 09, 10.
- This program retires Tiers 1-3.

Explicitly out of scope (Tier 4, deferred):
- Live-provider/costful gates (real Stream uploads/transcode, real-network media smoke — manual at
  deploy, per-run approval).
- Real prod D1/R2 mutation tests (read-only --remote schema snapshot diff allowed).
- Multi-user/multi-org auth E2E.
- Hosted/auto-deploy beyond the GH Actions gate.
- Broad repo lint cleanup beyond lint-in-test:safe.

Hard safety constraints (non-negotiable, per phase):
- Never mutate prod D1 (session-db), prod R2 (session-images), or Cloudflare Stream — only
  E2E-owned disposable targets; the DB-safety guard must make prod access impossible-by-accident.
- Never write the evidence bundle before redaction exists (live CLERK_SECRET_KEY + real session
  token in playwright/.auth/user.json).
- Never run live Stream/costful gates without explicit per-run approval.
- Leave unrelated dirty files / other agents' work untouched.
- Keep process/plan/context commits separate from execution commits; commit each phase before
  moving on.
```

### Filled-In Session-Goal Block (example)

A compressed launch packet for the example-co.app "Full Product E2E" program. This is the autonomous
session-goal variant: per-phase approval is standing-granted, and irreversible/costful actions are
deferred-and-reported rather than executed or paused-on (see
`process/development-protocols/phase-programs.md`, "Autonomous Session-Goal Variant"). It stays under
the ~4000-char `/goal` ceiling.

```text
ENTER EXECUTE MODE — Full Product E2E Program Goal

North star: Make the full-product E2E reliable enough that a future agent can autonomously implement
a feature, write/extend its tests, run them headlessly with full evidence, diagnose and self-heal
failures in a bounded loop, clean up all state safely, and hand off only after behavior is genuinely
verified — no human babysitting.

Definition of done (unattended agent can do all):
1. Implement a change across web/API/DB/container.
2. Author specs exercising it like a real user, against isolated disposable targets, never shared dev state.
3. Run headless, capturing screenshots, video, traces, console, network, server logs into a redacted evidence bundle per failure.
4. Diagnose: classify failures (product vs test vs harness vs stale-command); preflight doctor catches env problems before the run.
5. Self-heal in a bounded loop: real driver retries a bounded count, feeds evidence back, stops cleanly at the bound.
6. Clean up safely: every run leaves DB rows, E2E-owned containers/volumes/ports as found; production state never mutated; cleanup verified.
7. Hand off honestly: ✅ VERIFIED only when phase gates AND regression checks both pass with recorded evidence.

Verified (program level): default safe gates green from clean checkout, coverage measured, critical surfaces covered.

Scope (retire Tiers 1–3):
- T1 autonomy blockers → Phases 00, 03, 04.
- T2 flakiness → Phases 01, 02, 05.
- T3 coverage holes → Phases 06, 07.

OUT of scope (Tier 4): live/costful provider gates (per-lane approval only); broad lint; CI automation.

Hard safety (every phase): interact freely with the shared E2E container per your project's container
test docs — exec, read logs, rebuild image, recreate/restart via the managed script, create/remove
E2E-owned disposable targets. NEVER wipe named volumes, mutate prod data/storage, push production
images/deploy, or run live/costful gates without per-lane approval — defer-and-report those. Keep
process/plan commits separate from execution commits; commit each phase before moving on.

Discipline: fully autonomous — proceed phase-to-phase via the 10-step loop WITHOUT pausing for
approval. Self-decide; if something breaks, diagnose, write a new plan/fix, and continue. Done when
T1–3 are ✅ VERIFIED and T4 documented as deferred.
```
