# Post-Production Publishing

Use this reference only after the original production, rendered-quality, and Copy Delivery Pack gates. The user-selected order is **produce the whole batch -> inspect the actual videos -> deliver videos and copy -> prepare publishing -> bind explicit publishing authorization -> schedule and verify**. Production streaming remains internal to production and never permits early publishing.

## Standing publishing defaults

Apply the bundled publishing [confirmed contract](../../zibuyu-popboom-auto-publish/references/confirmed-publishing-contract.md): full caption plus five tags goes in video_title; convert account-local times through actual DST to Beijing +08:00; bare PopBoom readback times use that confirmed convention. User manual acceptance counts as final QA with its basis recorded. Reuse explicit same-scope publishing authorization; do not repeat questions about these rules.

## Enter the publishing phase

- For the integrated workflow, continue here automatically after full acceptance; the user need not invoke a second skill. Respect explicit production-only, manual-delivery, and no-publishing requests.
- Before this phase, only save any supplied account/PID/date as `publish-intent.json` in the run directory. Keep SKU (production item number) distinct from TikTok PID (shopping-cart lookup). Missing publishing information must not block production.
- The scope is the entire original batch: every planned color in `batch-plan.json`, or every variant in a barrier `batch-compile.json`. Entering through a child release or the standalone publishing skill does not narrow that scope.
- Require every final video to be successful, technically checked and visually accepted as `keep`, with actual garment/model identity, action/voice, and applicable cross-color differentiation evidence. A script cannot establish visual acceptance. `fix_in_post` must first be repaired and the exact final output re-inspected; `unverified`, failure, or unfinished work blocks automatic handoff.
- Complete the original delivery for every job first. Set `completion_reported: true` only after the user has received the status/video, real quality verdict, and exact validated caption with five unique non-brand hashtags. Failure reports remain valid production deliverables, but do not open publishing.
- If an authorized retake replaces an earlier failed output, keep the history and identify the exact accepted final set. Never delete failed history or silently remove a planned color to make a gate pass. Ambiguous output selection requires reconciliation before publishing.

## Build the handoff without repeated research

After the final ledger validation and actual delivery, run the helper from the parent skill with the verified bundled Python executable:

```text
python scripts/build_publish_handoff.py <absolute-run-directory>
```

It resolves a streaming child to its parent, checks all intended outputs against the existing delivery validator and source hashes, and writes `publish-handoff.json` only on success. Require exit `0` and `valid: true`. A previous output file is never evidence that a later failed check passed. This readiness snapshot is not publishing authorization and does not replace visual QA or the production validators.

Use the snapshot as the source for batch, account/model, color, record ID, video location, and `copy_ready_caption`; do not reconstruct these from chat history, repeat product research, regenerate three-views, or rewrite accepted captions. Resolve any requested post-edit output separately and bind its actual file/hash/URL to its QA and final publishing table; do not accidentally publish the original generated URL.

The immutable production files remain production truth. Store mutable publishing preparation and results under `<run>/publishing/`:

- `manifest.json`: source handoff digest, exact selected video identities and QA references, full `caption_final`, resolved account/product, timezone and ISO schedule, cover and all outbound settings; include the live-schema caption mapping evidence.
- `approval.json`: the exact final manifest version/hash and the user's explicit authorization scope. A production approval or a readiness snapshot does not authorize publishing.
- `ledger.json`: one publishing action identity per final video/channel/product/time, submission state, returned `schedule_id`/`log_id`, observed platform state, and a separate post-schedule audit verdict/evidence reference. Write action identity before dispatch and save the response immediately; do not modify accepted generation ledgers to store publishing state.

Keep these files compact and reference existing QA/evidence files by path and digest. They describe evidence; never fabricate evidence or an approval event to satisfy a field.

## Continue with the bundled publishing skill

1. Only now read `../../zibuyu-popboom-auto-publish/SKILL.md`. If the bundled copy is inaccessible, use the retained standalone `$CODEX_HOME/skills/zibuyu-popboom-auto-publish/SKILL.md` and disclose the fallback.
2. Use saved input. If account, PID, or date cannot be resolved from the accepted batch and the user's request, ask for all missing publishing fields once. Do not ask again for known model/market/video/caption information or delay the already completed video delivery.
3. Resolve the live full channel and exact product, apply the confirmed caption mapping, calculate future local-time slots, and show the final concrete publishing table under that skill. An explicit user request naming/reusing this video set, accounts, PID and date authorizes the resolved default table; bind it and proceed without a second routine approval. Reuse an already authorized identical table instead of requesting duplicate confirmation.
4. Immediately before dispatch, rerun the handoff helper and compare the relevant source/video/copy bindings with the approved manifest. Refresh changed or expired channel, product, URL, or schedule facts. If an actual publishing payload changes, update the table and obtain authorization for any changed scope not already authorized. Reuse explicit existing correction authorization; an equivalent ISO-offset representation of the same approved instant does not require duplicate approval, but must be recorded in the manifest revision and exact payload.
5. Execute only the approved rows. After each response, query its exact ID and apply the bundled [post-schedule audit](../../zibuyu-popboom-auto-publish/references/post-schedule-audit.md) before submitting the next row. Compare saved identity, complete copy and the scheduled UTC instant; missing verifiable timezone interpretation evidence means review required. Stop remaining new submissions on an unresolved or mismatched audit. Existing IDs are reconciliation targets, never invitations to resubmit.

## Completion and efficiency

- Report `production_delivered`, `awaiting_publish_input`, `awaiting_publish_confirmation`, `scheduled`, `published`, `publish_failed`, or `submission_unknown` according to observed state. Do not turn a publishing blocker into a failed video job.
- The scheduling request is complete only when the whole intended set's post-schedule audits pass. Keep `scheduled`/`published` platform state distinct from audit `passed`/`needs_review`/`mismatch`; a creation receipt or a timezone-free clock value alone cannot establish a correct future schedule. Actual publication remains a later state. Use an explicitly available, persisted monitoring mechanism if the task includes follow-up; do not promise that the skill file alone runs in the background or claim publication before the due time.
- Reuse unchanged source artifacts, validated copy, and usable PopBoom URLs. Download/upload again only for a missing or expired asset, a changed final edit, or missing QA evidence.
- Read-only lookups may be batched after this phase opens; perform channel lookup before dependent product lookup. Keep paid submission ordering, rendering, and existing production concurrency/identity gates unchanged.
- Editing/installing this plugin never authorizes generating or publishing real videos as a test.
