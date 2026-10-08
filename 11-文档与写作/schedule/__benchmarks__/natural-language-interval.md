---
name: natural-language-interval
skill: schedule
description: /schedule converts natural language intervals to valid cron expressions
tags: [happy-path]
input: /schedule add "check PRs every 30 minutes" /pr-watch
state: clean
assert-contains:
  - minimum
assert-not-contains:
  - Error
  - undefined
  - invalid cron
---

## What This Tests

A user adds a schedule using natural language ("every 30 minutes") instead of
a raw cron expression. The skill must:
1. Parse the natural language interval correctly
2. Show the derived cron expression so the user can verify it
3. Ask for confirmation before creating (CronCreate is a side effect)

A user who types "every 30 minutes" and gets back `*/30 * * * *` with a
confirmation prompt is a good outcome. A user who gets an error or a wrong
cron expression is a bad outcome.

## Expected Behavior

1. Extracts "every 30 minutes" → cron `*/30 * * * *`
2. Displays the cron expression alongside the natural language
3. Asks for confirmation before calling CronCreate
4. Does not crash or produce an error for a well-formed natural language interval
