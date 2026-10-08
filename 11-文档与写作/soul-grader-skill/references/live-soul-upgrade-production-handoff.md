# Live SOUL upgrade for production handoff

Use this reference when a SOUL grade turns into a live deployed-profile identity upgrade, especially for a friend/client/private-beta handoff where the gateway is already running.

## Pattern

1. **Read the live identity surface first.** Confirm the profile `SOUL.md` path, whether it is symlinked to the workspace repo, and the adjacent `AGENTS.md`/`CLAUDE.md`, configs, runbooks, and manifest state.
2. **Back up before editing.** Create a timestamped backup of the live/profile-visible `SOUL.md` outside the repo or beside the profile with mode `0600`.
3. **Rewrite SOUL as identity and judgment, not a runbook.** Include mission, positive role boundaries, core thesis, optimization hierarchy, relationship posture, voice, truthfulness posture, and success orientation. Put approval matrices, evidence thresholds, escalation procedures, detailed definitions of done, commands, ports, service PIDs, and volatile state in `AGENTS.md` / `CLAUDE.md`, runbooks, manifests, or system controls.
4. **Set the user's target score and loop.** If the user already gave a target score, use it. If not, ask what score they want to achieve before rewriting. Use `soul-grader` and require an explicit score table after every rewrite. Scope comes after the rubric. System blockers are reported separately and only from direct system evidence. Continue until the target score is met, or until the user explicitly says “done” / accepts the current score.
5. **Save the grade report as a durable artifact.** Put a concise report in the repo/runbook area when the SOUL is part of a handoff, including score, deployability, blocker status, and remaining non-SOUL blockers.
6. **Secret-scan the changed identity/report files.** Scan for token/key/private-key shapes and avoid printing `.env`, `auth.json`, or private candidate/customer data.
7. **Commit/push the repo-backed identity.** If the workspace remote lacks noninteractive GitHub credentials, use an existing approved token through a temporary `GIT_ASKPASS` file, never by embedding the token in the remote URL or committing it. Remove temporary token files afterward.
8. **Verify the live profile actually sees the new SOUL.** Check symlink target and hash equality between workspace and profile paths; a good repo commit does not prove the running profile loaded it.
9. **Refresh the gateway safely.** Existing gateway sessions can keep cached prompts. If a direct `systemctl --user restart hermes-gateway-<profile>.service` is blocked because the current process is inside the active gateway, use a detached/profile-scoped helper or other approved outside-the-gateway path to restart only the affected service and verify the new PID.
10. **Smoke the behavior, not just the file.** Run a fresh profile chat with an exact prompt that exercises the new identity, positive ownership boundaries, judgment, and relationship posture. Test detailed gates separately against the companion operating agreement and system controls.
11. **Record the operation in the manifest.** Include backup path, commit, grade score, hash/symlink verification, restart scope, final service state, smoke response/marker, and remaining handoff blockers. Do not include secrets.

## Pitfalls

- Do not call a production handoff ready just because the SOUL score is high; Discord/Telegram access, allowlists, and real user-path smoke tests may still block handoff.
- Do not overfill SOUL with procedure because a grader rewards specificity. Put detailed workflow in skills/runbooks and keep SOUL as compact constitution.
- Do not ask for a target score if the user already supplied one; use the supplied threshold and keep iterating until it is met or the user says “done.”
- Do not claim a gateway has loaded a new SOUL until a fresh/new session or post-restart smoke proves the behavior.
- Do not route around Hermes gateway-safety blocks with unsafe inline restarts. Use a detached helper/planned restart path and verify only the affected profile service.
