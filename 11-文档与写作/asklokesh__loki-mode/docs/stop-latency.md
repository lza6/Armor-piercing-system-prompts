# Stop latency

How fast can you stop a run. Every number here is derived from the source cited
beside it. Numbers marked MEASURED are enforced by `tests/test-stop-latency.sh`;
numbers marked DERIVED come from a documented timeout default and have not been
sat through end to end.

The short version: **`loki stop` is the supported mechanism and is bounded at
about 1 second. `touch .loki/STOP` is graceful, not immediate, and on the
default runner its worst case is 2 hours.** The product used to say the opposite.

## The supported stop

```bash
loki stop
```

**Worst case: about 1 second to SIGKILL. MEASURED.** `loki stop` signals the
run's whole process group: SIGTERM, a 1 second grace, then SIGKILL
(`autonomy/loki`, `_stop_group_by_pgid_files`). The provider subprocess and its
tool children die with the orchestrator. **This bound does not depend on what
the run was doing when you issued it** -- that timeout-independence is the
actual property worth relying on, and it is what the test asserts.

What "MEASURED" means here, precisely: `tests/test-stop-latency.sh` starts a
victim in its own process group that **installs a SIGTERM trap and would
otherwise sleep for 7200 seconds**, then drives `_stop_group_by_pgid_files`
against it and times the call. A cooperative victim would prove nothing -- it
would die on the SIGTERM whether or not the escalation existed. The suite
asserts the seconds scale rather than a sub-second figure, because it runs on
shared CI hardware and the claim that matters is "seconds, not the two hours
the victim asked for".

One caveat the suite records rather than hides: `loki stop` reaps by two
independent routes (`_kill_pid` on the recorded pid, which carries its own
`kill -9` escalation, and the process-group path above). Either alone kills the
victim, so an end-to-end `loki stop` cannot attribute the bound to the group
path. Verified by mutation: deleting the group SIGKILL left an end-to-end test
green. The suite therefore drives the group reaper directly for the attributing
assertion, and times the whole command separately for the number a user feels.

From the dashboard Stop button the bound is larger: `_killpg_project`
(`dashboard/server.py`) sends SIGTERM, polls for up to 5 seconds, then SIGKILL,
and a confirming reaper sweep follows it. Treat the dashboard button as
seconds, not sub-second, and prefer `loki stop` in automation.

## `touch .loki/STOP` is graceful, not immediate

The STOP file is a cooperative signal. The runner reads it at the **top of each
iteration**, before dispatching to the provider, and never during a provider
call: `check_human_intervention` (`autonomy/run.sh:25149`) is called from
exactly one site, `autonomy/run.sh:22301`. A STOP file written one second after
a dispatch is not observed until that provider call returns.

**Worst case on the bash runner: 7200 seconds plus iteration teardown. DERIVED**
from `LOKI_PROVIDER_CALL_TIMEOUT`, which defaults to 7200 (`autonomy/run.sh:880`).
Note that `LOKI_PROVIDER_IDLE_TIMEOUT` (default 120) does **not** lower this: the
deadline helper resets its activity clock on every output chunk, so a provider
that streams normally never trips the idle path.

Two places where the STOP file **is** fast, because the loop is already polling:

- during a pause (`handle_pause`): about 1 second, the poll interval
- during a retry backoff: one tick, since the wait loop rereads STOP each tick

Use `.loki/STOP` when you want the run to finish its current iteration and stop
cleanly. Use `loki stop` when you need it to stop now.

## Lowering the graceful bound

Set `LOKI_PROVIDER_CALL_TIMEOUT` to the longest single provider call you are
willing to wait through. The STOP-file worst case is that value plus a few
seconds of teardown. Lowering it below what a real iteration needs will cut off
useful work, so this is a stop-latency versus throughput trade, not a free win.

## Scope, and one place we are not at parity

These numbers cover the default bash runner. `docs/exit-codes.md` documents
exit-code parity between the bash and Bun (`LOKI_SDK_LOOP=1`) runners. **Stop
latency is not at that parity**, and saying so is more useful than implying it:

- The Bun runner records no process-group id, so `loki stop` reaches the runner
  and its direct provider child but may leave the provider's own tool
  subprocesses running. Prefer the bash route where you need a guaranteed
  whole-tree kill.

Closed in v9.28.0: the Bun provider call previously passed no timeout at all,
so its STOP-file path had **no upper bound**. It now honors
`LOKI_PROVIDER_CALL_TIMEOUT` like the bash route
(`loki-ts/src/runner/providers.ts`), using the SIGTERM-then-SIGKILL escalation
`shellRun` already implemented.

`loki stop` remains the supported mechanism on both routes.
