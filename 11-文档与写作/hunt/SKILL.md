---
name: hunt
description: Invoke when debugging any error, crash, unexpected behavior, or failing test. Finds root cause before applying any fix. Not for code review or new features.
metadata:
  version: "3.8.0"
---

# Hunt: Diagnose Before You Fix

Prefix your first line with 🥷 inline, not as its own paragraph.


A patch applied to a symptom creates a new bug somewhere else.

**Do not touch code until you can state the root cause in one sentence:**
> "I believe the root cause is [X] because [evidence]."

Name a specific file, function, line, or condition. "A state management issue" is not testable. "Stale cache in `useUser` at `src/hooks/user.ts:42` because the dependency array is missing `userId`" is testable. If you cannot be that specific, you do not have a hypothesis yet.

## Rationalization Watch

When these surface, stop and re-examine:

| What you're thinking | What it actually means | Rule |
|---|---|---|
| "I'll just try this one thing" | No hypothesis, random-walking | Stop. Write the hypothesis first. |
| "I'm confident it's X" | Confidence is not evidence | Run an instrument that proves it. |
| "Probably the same issue as before" | Treating a new symptom as a known pattern | Re-read the execution path from scratch. |
| "It works on my machine" | Environment difference IS the bug | Enumerate every env difference before dismissing. |
| "One more restart should fix it" | Avoiding the error message | Read the last error verbatim. Never restart more than twice without new evidence. |

## Progress Signals

When these appear, the diagnosis is moving in the right direction:

| What you're thinking | What it means | Next step |
|---|---|---|
| "This log line matches the hypothesis" | Positive evidence found | Find one more independent piece of evidence to cross-validate |
| "I can predict what the next error will be" | Mental model is forming | Run the prediction; if it matches, the model is correct |
| "Root cause is in A but symptoms appear in B" | Propagation path understood | Trace the call chain from A to B and confirm each link |
| "I can write a test that would fail on the old code" | Hypothesis is specific and testable | Write the test before applying the fix |

Do not claim progress without observable evidence matching at least one of these signals.

## Hard Rules

- **Same symptom after a fix is a hard stop.** The previous hypothesis was wrong. Re-read the execution path from scratch.
- **After three failed hypotheses, stop.** Surface to the user: what was checked, what was ruled out, what is unknown. Ask how to proceed.
- **Never state environment details from memory.** Run detection first: `sw_vers`, `xcodebuild -version`, `node --version`, `rustc --version`. State the actual output.
- **External tool failure: diagnose before switching.** When an MCP tool or API fails, determine why first (server running? API key valid? Config correct?) before trying an alternative.
- **Pay attention to deflection.** When someone says "that part doesn't matter," treat it as a signal. The area someone avoids examining is often where the problem lives.
- **Visual/rendering bugs: static analysis first.** Trace paint layers, stacking contexts, and layer order in DevTools before adding console.log or visual debug overlays. Logs cannot capture what the compositor does. Only add instrumentation after static analysis fails.
- **Fix the cause, not the symptom.** If the fix touches more than 5 files, pause and confirm scope with the user.
- **Grep before touching.** Before modifying any file or calling any identifier in a fix, grep to confirm it exists at the expected location. No results = re-examine the execution path. Do not edit or reference from memory.
- If you catch yourself writing a fix before finishing the trace, or thinking "let me just try this," stop.

## Confirm or Discard

Add one targeted instrument: a log line, a failing assertion, or the smallest test that would fail if the hypothesis is correct. Run it. If the evidence contradicts the hypothesis, discard it completely and re-orient with what was just learned. Do not preserve a hypothesis the evidence disproves.

## Gotchas

| What happened | Rule |
|---------------|------|
| Patched client pane instead of local pane | Trace the execution path backward before touching any file |
| Same error after 4 patches, each burying the real cause | Same symptom = stop and re-read the whole execution path from scratch |
| Diagnosed "macOS 26 beta," it was a stable release | Run `sw_vers` first; never state versions from memory |
| MCP not loading, switched tools instead of diagnosing | Check server status, API key, config before switching methods |
| Wrote the fix before finishing the trace | "Let me just try this" = incomplete hypothesis. Stop. |
| Restarted 8 times without reading the actual error response | Read the last error verbatim before restarting |
| Orchestrator said RUNNING but TTS vendor was misconfigured | In multi-stage pipelines, test each stage in isolation |
| Used `format_size` (nonexistent), should have been `bytes_to_human` | Grep for the function name before writing a call to it |

## Outcome

```
Root cause:  [what was wrong, file:line]
Fix:         [what changed, file:line]
Confirmed:   [evidence or test that proves the fix]
Tests:       [pass/fail count, regression test location]
```

Status: **resolved**, **resolved with caveats** (state them), or **blocked** (state what is unknown).
