# 19. Destructive Ops: Look First, Back Up, Then Ask

**Problem:** `rm`, `mv`, overwrite, `truncate`, force-push, and migrations run on targets the agent located by pattern, not by inspection — and "the directory the user surely doesn't need" turns out to be the one they did. In the 2026-08-20 incident the damage was permanent for exactly one reason: no pre-modification backup existed. Carefulness was not the missing ingredient; a fixed sequence was.

**The rule (paste into CLAUDE.md):**

```text
Before any destructive or hard-to-reverse operation (rm, mv/overwrite, truncate, drop, git reset --hard / push --force / clean, schema migration), run this sequence — skipping any step cancels the operation:
1. LOOK: inspect the exact target and confirm it is what you think it is.
   Files/dirs: ls -la <TARGET>; stat <TARGET>; head -20 <TARGET> (list directories one level deep first).
   Git: git status --short; git log --oneline -3 before any force operation.
2. BACKUP: copy the target outside the operation's blast radius, timestamped:
   mkdir -p "<BACKUP_DIR>" && cp -R "<TARGET>" "<BACKUP_DIR>/$(basename <TARGET>).$(date +%Y%m%d_%H%M%S)"
   The backup must not be reachable by the same command that destroys the target.
3. APPROVE: for user-owned data (browser data, account state, published content, anything outside version control), state exactly what will change and get approval for THIS operation. Approval of the task type is not approval of this operation.
Then run the destructive command. If no backup is possible (external service with no export), the operation is human-only: report and wait.
After any accident: stop touching the system — automated "recovery" from an unknown state widens the damage.
```

**Why it binds:** The trigger verbs are named, so the rule fires on the command itself rather than on the agent's confidence. Each step is a concrete command whose output is evidence: a listing showing what was about to die, a timestamped backup path that can be re-checked later, an approval tied to this one operation — closing the loophole where "you asked me to clean up files" gets read as blanket consent for every future deletion. The incident grounding is explicit in the source chapter: the bookmark loss stacked three failures — trusted default, session reuse, no backup — and the missing backup is what converted minutes of inconvenience into permanent loss ("if a backup had existed, the damage would have ended as a few minutes of inconvenience"). The rule is deliberately two-sided: backup without approval is unauthorized modification; approval without backup is an unrecoverable accident.

**Variations:**
- Git repos: the backup step may be `git branch backup/<slug>` for tracked files; look-first becomes reviewing the `git diff` the reset would discard.
- Solo: shrink scope — look + backup for anything not under version control; commits are the backup for everything else.
- Production/shared: DB dump before every migration; prefer write-forward (append/new file) over in-place edits.

**Anti-pattern:** "Be careful with deletions; avoid data loss." No sequence, no backup command, no approval test — every destructive command looks necessary in the moment and runs unrehearsed.
