---
name: remember
description: "Log a finding or pattern to persistent brain memory. Auto-fills from session context. Usage: /remember"
disable-model-invocation: false
---
Save current finding/pattern to brain memory.

## Flow
1. Read current session context — what target, endpoint, vuln class
2. Ask user to confirm or edit:
   - Target: (auto-detected)
   - Endpoint: (from session)
   - Vuln class: (from session)
   - Result: confirmed / rejected / partial
   - Severity: critical / high / medium / low
   - Bounty: $___
   - Notes: ___
3. Write to brain:
   - If confirmed: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> confirmed "<description>" "<details>"`
   - If rejected: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> exhausted "<what failed>" "<why>"`
4. Sync to global brain: `uv run python3 $CLAUDE_PROJECT_DIR/tools/global_brain.py learn technique "<pattern>"`
5. Track response if submitted: `uv run python3 $CLAUDE_PROJECT_DIR/tools/response_tracker.py log <id> <status>`

## Why This Matters
- /resume shows which endpoints you've tested and which remain
- Cross-target learning: patterns from target A inform hunting on target B
- Global brain accumulates technique knowledge across all engagements

## Top-Tier Recall Standard

Before writing memory, make it useful to a future agent that has no conversation context.

Use this shape:
```
target:
surface:
vuln_class:
primitive:
accounts_or_roles:
evidence_path:
request_summary:
response_marker:
impact:
status:
next_action:
```

If the item is rejected, preserve the blocker with the same care as a finding. High-quality negative memory prevents duplicate work and false confidence.
