# Pre-Continuation Checks

Run these checks **in order** every time you (the coordinator) are about to re-enter Phase 0 — whether triggered by a user reply, a timer firing, or the automatic loop.

**Harness note:** Checks 2 and 3 read from platform-provided session state (e.g. Claude Code's stdin JSON). On other harnesses this data may not be available — each check gracefully skips if its fields are absent.

Read the current session state from whatever mechanism your platform provides (e.g. Claude Code pipes session JSON to stdin). The relevant fields:

- `context_window.used_percentage` — 0–100, percentage of context window consumed (treat null as 0)
- `rate_limits.five_hour.used_percentage` — 0–100 (Claude Code: Pro/Max subscribers only)
- `rate_limits.five_hour.resets_at` — Unix epoch seconds when the 5-hour window resets
- `rate_limits.seven_day.used_percentage` — 0–100 (Claude Code only)
- `rate_limits.seven_day.resets_at` — Unix epoch seconds when the 7-day window resets

Each field may be independently absent. If absent, skip the corresponding check.

---

## Check 1: Context Window

If `context_window.used_percentage` **> `CONTEXT_COMPACTION_THRESHOLD`**:

1. Print: `"⚠️ Context window at {usage}% — compacting before continuing."`
2. Compact context using your platform's mechanism (e.g. `/compact` on Claude Code). Wait for it to complete.

---

## Check 2: Five-Hour Usage Limit

If `rate_limits.five_hour.used_percentage` is present and **> `API_FIVE_HOUR_THRESHOLD`**:

1. Convert reset epoch to human-readable time:
   ```bash
   # macOS
   date -r {resets_at}
   # Linux
   date -d @{resets_at}
   ```
2. Print: `"⏸ 5-hour usage limit at {usage}% — auto-pausing until reset at {reset_time}. BAD will resume automatically."`
3. **If `TIMER_SUPPORT=true`:** compute a cron expression from the reset epoch and schedule a resume:
   ```bash
   # macOS
   date -r {resets_at} '+%M %H %d %m *'
   # Linux
   date -d @{resets_at} '+%M %H %d %m *'
   ```
   Call `CronCreate`:
   - `cron`: expression from above
   - `recurring`: `false`
   - `prompt`: `"BAD_RATE_LIMIT_TIMER_FIRED (five_hour) — The 5-hour rate limit window has reset. Re-check five_hour.used_percentage; if now below API_FIVE_HOUR_THRESHOLD, continue with Pre-Continuation Check 3 (seven-day). If still too high, schedule another pause until the next reset time."`

   Save the job ID. Do not ask the user for input — resume automatically when `BAD_RATE_LIMIT_TIMER_FIRED` arrives.

4. **If `TIMER_SUPPORT=false`:** print the reset time and wait for the user to reply when they're ready to continue. Then re-check the limit before proceeding.

---

## Check 3: Seven-Day Usage Limit

If `rate_limits.seven_day.used_percentage` is present and **> `API_SEVEN_DAY_THRESHOLD`**:

1. Convert reset epoch to human-readable time:
   ```bash
   # macOS
   date -r {resets_at}
   # Linux
   date -d @{resets_at}
   ```
2. Print: `"⏸ 7-day usage limit at {usage}% — auto-pausing until reset at {reset_time}. BAD will resume automatically."`
3. **If `TIMER_SUPPORT=true`:** compute a cron expression from the reset epoch and schedule a resume:
   ```bash
   # macOS
   date -r {resets_at} '+%M %H %d %m *'
   # Linux
   date -d @{resets_at} '+%M %H %d %m *'
   ```
   Call `CronCreate`:
   - `cron`: expression from above
   - `recurring`: `false`
   - `prompt`: `"BAD_RATE_LIMIT_TIMER_FIRED (seven_day) — The 7-day rate limit window has reset. Re-check seven_day.used_percentage; if now below API_SEVEN_DAY_THRESHOLD, continue with Phase 0. If still too high, schedule another pause until the next reset time."`

   Save the job ID. Resume automatically when `BAD_RATE_LIMIT_TIMER_FIRED` arrives.

4. **If `TIMER_SUPPORT=false`:** print the reset time and wait for the user to reply when ready. Then re-check before proceeding.

---

Only after all applicable checks pass, proceed to re-run Phase 0 in full.
