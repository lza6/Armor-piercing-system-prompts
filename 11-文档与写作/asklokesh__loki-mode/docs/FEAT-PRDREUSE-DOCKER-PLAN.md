# FEAT-PRDREUSE-DOCKER-PLAN

Implementation plan for the PRD-reuse + Docker feature batch. Anchored to verified file:line.
Designed for 4-5 parallel dev agents with zero file overlap. Founder scope locks:
- Docker dashboard: publish on a HOST PORT and AUTO-OPEN (like local loki start); show BOTH local and docker runs.
- Image cleanup: after pull, prune ONLY dangling/old asklokesh/loki-mode images NOT in use by a running container.
- loki stop: ALSO stops/removes the loki-mode docker container for this project (tracked via .loki docker state).

## Context: wave-4 uncommitted edits (build on top, do NOT revert)
Uncommitted W4 work in: autonomy/run.sh, autonomy/sandbox.sh, autonomy/prd-checklist.sh,
autonomy/spec-interrogation.sh, dashboard/migration_engine.py, dashboard/server.py,
loki-ts/src/council/voter_agents.ts, loki-ts/src/runner/build_prompt.ts, loki-ts/src/runner/council.ts.
Relevant: build_prompt.ts buildPromptForRunner now passes ctx.prdPath (a PATH). PRD-reuse Bun work depends
on this and must NOT touch build_prompt.ts.

## Existing scaffolding (EXTEND, not duplicate)
Bash route already implements generated-PRD reuse for the no-file case:
- .loki/generated-prd.md is the canonical generated-PRD path, byte-locked for parity (run.sh ANALYSIS_INSTRUCTION,
  build_prompt.ts:208, resume lines).
- decide_generated_prd_action() (run.sh:4892) returns reuse | update | generate | user_owned.
- persist_prd_signature_if_present() (run.sh:4983) writes .loki/state/prd-signature.json.
- run_autonomous() (run.sh:~13824) auto-detect block handles the empty prd_path case only.
- Bun route: runAutonomous (autonomous.ts:230) -> makeContext (autonomous.ts:641) sets ctx.prdPath = opts.prdPath. No reuse/persist.

Docker route already has host-aggregating dashboard:
- cmd_docker (autonomy/loki:~28748) -> docker-run.sh helpers.
- _loki_docker_register_host running/stopped brackets the blocking run; registers $(pwd) in ~/.loki/dashboard/projects.json.
- Dashboard /api/projects (server.py:~2623) aggregates local + docker; pid=None reads bind-mounted .loki/session.json.
- Dashboard Stop for docker works via STOP file (server.py:~2970).
- cmd_dashboard_start (autonomy/loki:~4038) + cmd_dashboard_open (~3982) are standalone host dashboard entries.

## Design decision locks
LOCK 1: Canonical PRD path = .loki/generated-prd.md (do NOT invent .loki/prd/current.md). Persist user content INTO it;
record provenance in .loki/state/prd-signature.json via a new `source` field.
LOCK 2: User PRDs resolve to reuse/user_owned, NEVER update. Stamp source:"user" at persist; short-circuit
decide_generated_prd_action to user_owned when source=user (except --fresh-prd). Signature-diff `update` stays scoped to source=generated.
This answers "update only if needed": user PRD = always as-is; generated PRD = existing signature logic.
LOCK 3: DOCKER-DASH uses the HOST dashboard, not the published container port. Container mounts only workspace; an
in-container dashboard sees ONE project and cannot satisfy "shows BOTH." Host dashboard already aggregates both.
Local loki start runs the dashboard on host port 57374 + auto-opens (run.sh:~10081-10099), so host dashboard IS "like local loki start."
LOCK 4: Ownership deconflicted by FILE with an interface contract (matrix below).

## FEATURE 1 - FEAT-PRD-REUSE
Semantics (both routes identical):
| Run | File arg? | Persisted PRD? | Behavior |
| 1st | yes | no | use file; persist to .loki/generated-prd.md, source=user |
| 1st | no | no | codebase-analysis generates .loki/generated-prd.md, source=generated (existing) |
| 2nd+ | no | yes | continue from persisted PRD (reuse; generated may update on drift, user never) |
| 2nd+ | yes | yes | new file overwrites .loki/generated-prd.md, source reset to user (brownfield) |

Bash (Agent C, autonomy/run.sh ONLY):
- New persistence branch for explicit user PRD: when prd_path non-empty and not already .loki/generated-prd.md:
  mkdir -p .loki .loki/state; atomic copy prd_path -> .loki/generated-prd.md; write prd-signature.json with
  source:"user", prd_sha (reuse _loki_prd_file_hash run.sh:4869), generated_at, signature (compute_codebase_signature),
  origin_path; repoint prd_path=".loki/generated-prd.md"; export GENERATED_PRD_ACTION="user_owned".
- Extend decide_generated_prd_action (4892): read source from prd-signature.json. Precedence:
  --fresh-prd/LOKI_PRD_REGEN -> generate > source=user -> user_owned > existing generated logic.

Bun (Agent D, loki-ts/src/runner/autonomous.ts + NEW loki-ts/src/runner/prd_reuse.ts; NOT build_prompt.ts):
- New helper resolvePrdForRun(opts) called at top of runAutonomous (autonomous.ts:233 before makeContext). Mirrors 1a/1b:
  user file -> copy + persist source:user -> return generated path; empty + generated exists -> decideGeneratedPrdAction
  TS port -> return generated path for reuse/update/user_owned, undefined for generate; empty + none -> undefined.
  Set resolved path onto opts.prdPath before makeContext (autonomous.ts:655). No build_prompt.ts edit.
- Parity: TS prd-signature.json schema + decideGeneratedPrdAction must match bash exactly.

.loki state additions: .loki/generated-prd.md also holds persisted user PRD. prd-signature.json adds
source ("user"|"generated"), origin_path (when source=user).

AC: AC1 user file persists byte-equal + source:user. AC2 no-arg rerun reuses, no codebase analysis. AC3 new file
overwrites, source stays user, origin_path updates. AC4 no-arg first run still generates source:generated. AC5
--fresh-prd re-analyzes -> source:generated. AC6 bash/Bun identical source+action (parity test). AC7 no-arg rerun
after user PRD never enters GENERATED_PRD_UPDATE_MODE even if codebase changed.

## FEATURE 2 - FEAT-DOCKER-DASH (host dashboard auto-open)
Architecture (LOCK 3): host dashboard, auto-opened.
Agent A (autonomy/loki, inside cmd_docker, start path): between _loki_docker_register_host running and the blocking run:
1. Start/reuse host dashboard via cmd_dashboard_start (idempotent). Port: DASHBOARD_DEFAULT_PORT 57374 with fallback
   (Agent B loki_docker_pick_host_port). 2. Auto-open gated like run.sh:~10088 ([ -t 1 ] && not background && LOKI_NO_AUTO_OPEN!=1)
   via cmd_dashboard_open. 3. Container stays dashboard-OFF (docker-run.sh LOKI_DASHBOARD=false), no container port publish.
Agent B (autonomy/docker-run.sh): loki_docker_pick_host_port - probe 57374, increment to free port if bound, echo chosen port.
server.py (Agent E only IF a gap appears): /api/projects already aggregates docker, Stop already handles docker. Default: no change.

AC: AC8 loki docker start in TTY starts host dashboard + opens browser. AC9 dashboard lists docker run alongside local.
AC10 LOKI_NO_AUTO_OPEN=1/non-TTY/--bg no browser. AC11 second docker start reuses dashboard, both runs listed.

## FEATURE 3 - FEAT-DOCKER-PRUNE (scoped image cleanup after pull)
No explicit docker pull today (run auto-pulls only if absent). PRUNE needs explicit pull.
Agent B (docker-run.sh helpers) + Agent A (call site in cmd_docker): loki_docker_pull_and_prune, called from cmd_docker before run (start path):
1. docker pull $LOKI_DOCKER_IMAGE (default asklokesh/loki-mode:latest), capture image ID.
2. In-use set: docker ps --format '{{.Image}} {{.ImageID}}'.
3. Enumerate ONLY asklokesh/loki-mode: docker images --filter 'reference=asklokesh/loki-mode' --format '{{.ID}} ...'
   + dangling: --filter 'reference=asklokesh/loki-mode' --filter 'dangling=true' -q.
4. docker rmi each ID NOT the just-pulled :latest AND NOT in-use (best-effort).
5. NEVER docker image prune -a. Scope strictly reference=asklokesh/loki-mode.
6. Honest output: reclaimed count/bytes or "nothing to reclaim."
Gate: LOKI_DOCKER_PRUNE=${LOKI_DOCKER_PRUNE:-1} (default on, =0 opt-out; =0 also skips explicit pull).

AC: AC12 old asklokesh/loki-mode IDs removed, :latest remains. AC13 in-use image never removed. AC14 non-loki-mode
image never touched (decoy survives). AC15 LOKI_DOCKER_PRUNE=0 skips; honest output.

## FEATURE 4 - FIX-DOCKER-STOP (loki stop reaps the container)
Repro: container loki-<sha12> Up but loki stop says "No active session." cmd_stop (autonomy/loki:~2242) only checks .loki.
Container name deterministic: loki-<sha12 of workspace> (docker-run.sh:~204-214).
Agent A (autonomy/loki write+read) + Agent B (docker-run.sh helpers):
Write: before blocking run write .loki/docker/run.json {container, image, project_dir, started_at}; clear after.
  Helpers loki_docker_write_runstate / loki_docker_clear_runstate.
Read/reap (cmd_stop folder-scoped default, before "No active session"):
1. Read .loki/docker/run.json -> container; fallback recompute loki-<sha12 of $(pwd)> (loki_docker_container_name).
2. If docker ps -q -f name=^${container}$ non-empty -> docker stop then docker rm (best-effort; --rm may auto-remove).
3. Remove run.json. 4. Report reap, no "No active session" when docker run Up.
5. loki stop --all: also docker ps -q --filter ancestor=asklokesh/loki-mode -> stop/rm all (machine-wide, parity with --all PID).
Folder-scoped default stays folder-scoped. Preserves v7.7.30-34 stop-scoping.

.loki state additions: .loki/docker/run.json (NEW) {container, image, project_dir, started_at}.

AC: AC16 docker start then stop (same folder) stops+removes container, names it, no "No active session". AC17 run.json
deleted -> still reaps via recomputed name. AC18 stop in folder X does not stop docker run in folder Y. AC19 stop --all
stops every loki-mode container. AC20 no docker run + no local session -> existing "No active session" (no regression).

## FILE-OWNERSHIP MATRIX (zero overlap)
- Agent A (Docker orchestration): autonomy/loki ONLY. cmd_docker (dashboard start/open F2, pull+prune F3, write/clear
  run.json F4), cmd_stop (docker reconcile+reap F4). Calls Agent-B helpers by name; uses cmd_dashboard_start/open.
- Agent B (Docker helpers): autonomy/docker-run.sh ONLY. loki_docker_pick_host_port, loki_docker_pull_and_prune,
  loki_docker_write_runstate, loki_docker_clear_runstate, loki_docker_container_name. No call-site edits in autonomy/loki.
- Agent C (PRD-reuse bash): autonomy/run.sh ONLY. User-PRD persistence in run_autonomous, extend decide_generated_prd_action.
- Agent D (PRD-reuse Bun): loki-ts/src/runner/autonomous.ts + NEW loki-ts/src/runner/prd_reuse.ts. MUST NOT edit build_prompt.ts.
- Agent E (Tests + server.py iff needed): tests/** new files, loki-ts/tests/** new files; server.py only if a real DASH gap appears.
  Do not edit existing W4-touched test files.

A<->B: disjoint files, share function-boundary contract. C<->A: PRD-reuse-bash entirely in run_autonomous; cmd_start
already passes file arg to run.sh. D<->W4: D sets ctx.prdPath upstream; build_prompt.ts read-only for D.

Sequencing: B, C, D independent -> parallel immediately. A depends on B helper signatures (contract fixed up-front, A can
start against signatures). E writes tests against contracts in parallel, finalizes after A-D land.

## RISKS
R1 (DASH architecture): host dashboard satisfies host-port + auto-open + shows-both; container-port publish breaks shows-both. Host chosen (LOCK 3).
R2 (host-port conflict): cmd_dashboard_start idempotent; loki_docker_pick_host_port fallback. Container-port publish was already disabled for 57374 collision.
R3 (PRD update-only-if-needed): LOCK 2 source field. source=user always reuse/user_owned; source=generated existing signature logic. Hand-edited persisted PRD -> still user_owned.
R4 (prune over-aggression): triple-scoped (reference filter + exclude :latest ID + exclude in-use). AC14 decoy survives. rmi best-effort.
R5 (pull latency): pull-always v1, gated by LOKI_DOCKER_PRUNE. OQ: pull-always vs only-on-digest-change; recommend pull-always v1.
R6 (--rm + stop): docker stop triggers auto-removal; docker rm best-effort (already-gone = success).
OQ1: run.json write BEFORE blocking run, clear AFTER. Deterministic-name fallback covers the window.
OQ2: confirm Bun runAutonomous reached with opts.prdPath set from same arg as bash; parity test guards decision logic regardless.
