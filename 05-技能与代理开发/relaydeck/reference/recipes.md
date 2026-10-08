# Fleet recipes

Composable patterns. All assume you've bootstrapped
(`scripts/relaydeck-bootstrap.sh`) and registered a workspace. Replace
ids/types/purposes to taste. Recall the flag conventions: `-t` type, `-w`
workspace, `-c autonomy=auto` for permissions, ids are
`lowercase-with-dashes`.

These aren't only for coding — the same machinery runs review, research,
migration, and monitoring. The *purpose* you give each agent is what differs.

## 1. Parallel fan-out (N specialists, one task)

Many agents, same brief, independent work — then collect durable results.

```sh
relaydeck workspace add . --name proj --plugin messaging --plugin skills
for spec in security:"security audit" perf:"performance audit" \
            a11y:"accessibility audit"; do
  id=${spec%%:*}
  relaydeck agent create "$id" -t claude-code -w proj \
      --purpose "${spec#*:}" -c autonomy=auto
done
relaydeck agent start security perf a11y
relaydeck workspace message 'Audit your specialty. Hand back findings with:
relaydeck agent result put "$RELAYDECK_AGENT_ID" --summary "<one line>" --body @findings.md'
for id in security perf a11y; do
  relaydeck agent wait "$id" --status complete-unread --timeout 900 \
    || relaydeck agent screen "$id"
  relaydeck agent result get "$id"        # durable — survives an agent crash
done
relaydeck agent rm security perf a11y --yes
```

## 2. Pipeline (stage → stage, each waits for the last)

```sh
relaydeck agent create impl   -t codex-cli   -w proj --purpose "implement" -c autonomy=auto
relaydeck agent create review -t claude-code -w proj --purpose "review"
relaydeck agent start impl review

relaydeck agent send impl 'Implement the change in ISSUE.md. Reply DONE when committed.'
relaydeck agent wait impl --status complete-unread --timeout 1200

relaydeck agent send review 'Review impl'\''s commit. Reply APPROVE or list blocking issues.'
relaydeck agent wait review --status complete-unread --timeout 600
relaydeck agent screen review             # read the verdict, branch your logic
```

## 3. Review quorum (accept only if ≥2 of 3 approve)

```sh
for r in r1 r2 r3; do
  relaydeck agent create "$r" -t claude-code -w proj --purpose "reviewer" -c autonomy=auto
done
relaydeck agent start r1 r2 r3
relaydeck workspace message 'Review PR #42. Reply: APPROVE or REJECT + reason.'
for r in r1 r2 r3; do relaydeck agent wait "$r" --status complete-unread --timeout 600; done
relaydeck workspace inbox --full          # tally the verdicts, decide in your logic
relaydeck agent rm r1 r2 r3 --yes
```

Cap at three — more reviewers rarely change a 2/3 quorum and just burn tokens.

## 4. Research / investigation (read, don't write)

Point agents at *understanding*, collect synthesized findings. Use `locked`
autonomy so they can't mutate anything.

```sh
for area in api:"how auth works" web:"how state syncs" infra:"deploy pipeline"; do
  id=${area%%:*}
  relaydeck agent create "study-$id" -t claude-code -w proj \
      --purpose "${area#*:}" -c autonomy=locked
done
relaydeck agent start study-api study-web study-infra
relaydeck workspace message 'Investigate your area. Put a findings memo:
relaydeck agent result put "$RELAYDECK_AGENT_ID" --summary "<area>" --body @memo.md'
for id in api web infra; do
  relaydeck agent wait "study-$id" --status complete-unread --timeout 900
  relaydeck agent result get "study-$id"
done
```

## 5. Large migration across branches (worktrees)

One agent per branch, isolated working trees off one repo — no trampling.

```sh
for br in migrate/api migrate/web migrate/worker; do
  relaydeck worktree create "$br" --repo ~/src/monorepo --plugin messaging
  name=$(echo "$br" | tr '/' '-')
  relaydeck agent create "mig-$name" -t codex-cli -w "$name" \
      --purpose "apply the codemod + run tests" -c autonomy=auto
  relaydeck agent start "mig-$name"
done
relaydeck agent list -A                    # watch every branch's agent at once
# … when each is green, review its worktree, merge, then:
relaydeck worktree remove <name>           # teardown hook + git worktree remove
```

## 6. Cross-repo fan-out (same job, many repos)

```sh
for d in ~/src/api ~/src/web ~/src/worker; do
  name=$(basename "$d")
  relaydeck workspace add "$d" --name "$name" --plugin messaging
  relaydeck agent create "bump-$name" -t codex-cli -w "$name" \
      --purpose "bump deps + run tests" -c autonomy=auto
  relaydeck agent start "bump-$name"
done
relaydeck agent list -A
```

## 7. Monitoring / ops (a long-lived watcher)

An agent that watches something and pings a human only when it matters.

```sh
relaydeck agent create watcher -t claude-code -w proj \
    --purpose "watch CI + prod health; escalate real problems" -c autonomy=auto
relaydeck agent start watcher
relaydeck agent send watcher 'Poll the build + error rate. On a real failure,
run: relaydeck agent escalate "$RELAYDECK_AGENT_ID" -m "<what broke>". Otherwise
relaydeck broadcast a one-line all-clear each cycle.'
# Your side: react to its broadcasts/escalations on the stream.
relaydeck events tail -f --type operator.broadcast
```

## 8. Supervisor loop (keep a fleet healthy)

Clear benign stalls, surface real ones; pair with autopilot so most clear
themselves. Run the loop in *your* session — agents keep running in the
daemon regardless; Ctrl-C the loop without killing the fleet.

```sh
relaydeck plugin set autopilot mode benign
relaydeck events tail -f --type autopilot.held &   # held prompts → your attention
while :; do
  relaydeck agent list -A
  relaydeck context-watch status                    # compact any that are filling
  sleep 15
done
```

## Patterns to avoid

- **Shadow-redoing a delegated task.** If you handed it to an agent, let the
  agent finish — don't start editing the same files yourself mid-flight.
- **Polling an inbox in a tight loop to "wait."** Use `relaydeck agent wait`
  (status, SSE-backed) — peer replies are pushed into sessions, not pulled.
- **Spawning a worker that re-runs this bootstrap.** That's a
  fleet-of-fleets. A spawned worker is already inside relaydeck
  (`RELAYDECK_AGENT_ID` set) and §0 stops it — keep it that way.
- **Forgetting teardown.** `relaydeck agent rm <ids...> --yes` the one-offs;
  `relaydeck worktree remove` the throwaway branches.
