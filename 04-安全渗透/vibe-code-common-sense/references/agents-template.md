# Agent Instructions

You are working in a project that uses Vibe Code Common Sense.

## Required Workflow

1. Read `docs/PRODUCT_TIMELINE.md` before starting.
2. Select the first unchecked item unless the user explicitly overrides it.
3. Keep the change PR-sized and focused.
4. Do not silently invent architecture; document decisions.
5. Run the relevant verification command before claiming completion.
6. Update `docs/PRODUCT_TIMELINE.md` when the PR is opened.
7. Mark the completed item `[x]` and add the PR number.
8. Mention the next unchecked item in the final summary.

## Do Not

- Do not skip the tracker.
- Do not bundle unrelated features.
- Do not claim completion without evidence.
- Do not overwrite user changes.
- Do not continue from memory after a merge; sync main and reread the tracker.

## PR Standard

Every PR should include:

- one clear purpose;
- docs updated when behavior or direction changes;
- tests or documented checks;
- product timeline update;
- a concise summary with verification evidence.
