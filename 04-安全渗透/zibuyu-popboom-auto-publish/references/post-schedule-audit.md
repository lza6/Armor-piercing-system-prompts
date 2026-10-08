# Post-schedule audit

Read this when submitting or reconciling a scheduled publication. This is a platform-readback audit, separate from production QA and authorization. Its purpose is to catch a successfully created task whose saved identity, copy or time is wrong or cannot be verified.

## Audit each row before continuing the batch

1. Preserve the approved request and stable action identity. Save the creation receipt and returned `schedule_id` or `log_id` immediately, retaining the actual ID type.
2. Call `check_publish` with that exact ID in its corresponding live-schema field. Save its complete unaltered response and an offset-aware observation timestamp. Bind the evidence to the actual completed tool call, queried ID and response digest where execution records are available; a locally written `tool: check_publish` label alone does not prove a query occurred. Creation receipts may only echo inputs; they cannot prove persisted scheduling values. New audit evidence must not predate the submission/receipt or the current reconciliation start, when applicable. Do not freshen an old response by changing a general action timestamp. Preserve historical verdicts with their observation time; do not invent a fixed expiry period.
3. Match the returned schedule/log ID, channel, product ID and complete caption against the approved row. Check the exact video using the returned TikTok file ID linked to the creation receipt/upload evidence, or another platform-provided asset identity. A shared product title or color name is insufficient. Retain exactly five approved tags; do not rewrite the caption during audit. Compare cover/music/AI settings if exposed; label absent optional fields as not returned, never as verified.
4. Audit the time as described below. Keep its verdict separate from the platform's waiting/published/failed state.
5. Only continue new submissions when the required identity, copy and time checks pass. If the first or any later row needs review or mismatches, hold the unsubmitted remainder and keep already returned IDs. Read-only queries and evidence collection may continue. Missing evidence does not justify canceling or recreating an existing task.
6. After the last submitted row, reconcile the complete intended set against the approved manifest: one retained action per video, no missing/extra/duplicate actions or platform IDs, unchanged approved request bindings, correct account/PID/caption bindings, and the intended count and slots on each date in the **account timezone**. Never declare the whole batch passed by checking only the remaining rows in a shortened ledger. A Beijing midnight may belong to the preceding US account date. This final reconciliation does not itself schedule a background monitor.

## Confirmed PopBoom convention

For explicit Beijing +08:00 submissions, interpret a bare check_publish schedule time as Asia/Shanghai under [confirmed-publishing-contract.md](confirmed-publishing-contract.md), preserving the original text and declaring this basis. This is a user-confirmed operational convention, not inferred provider metadata. The explicit-evidence fallback rules below apply to other platforms or submissions outside this convention. Do not stop or re-ask solely because PopBoom omits the offset.

## Compare instants, not clock text

- Record the account's IANA zone, intended local date/time, actual date-specific offset, exact submitted ISO value and equivalent UTC instant before dispatch. Resolve nonexistent/ambiguous DST local times before submitting. Do not infer a platform timezone from the account's market or this computer's timezone.
- The live API contract wins. An ISO example with `+08:00` is not a declaration that other valid offsets are unsupported. Do not invent a separate `timezone` argument. A permitted change of representation must convert the clock and date as well as the offset.
- Extract the **observed** scheduled time only from a query or platform schedule detail. If it has `Z` or an explicit offset, parse it and compare its UTC instant with the expected instant. Accept equivalent offsets; do not require the same clock digits. Reject invalid dates/offsets rather than normalizing them silently.
- For a timezone-free PopBoom readback of a Beijing submission, use the confirmed contract above. Outside that contract, use a timezone only when applicable platform metadata or evidence establishes its interpretation. Save the supporting evidence and resolve DST for that date. A local default, account alias, old example, creation echo, or a guess based on clock similarity is not evidence. Do not confuse a generic UI timezone report with the standing user-confirmed PopBoom contract.
- If the offset/interpretation remains unknown, set `needs_review` and report the raw observed time. Do not label a possible 12-hour difference as a proven execution delay/advance, and do not claim that the database or scheduler discarded a timezone without implementation evidence.
- If the verified observed instant differs, set `mismatch` with the signed difference (`observed_utc - expected_utc`). Compare at the precision the platform actually exposes; for second-precision schedules require exact seconds. Do not accept a large discrepancy under an arbitrary tolerance.

Example: target `2030-07-09T07:00:00-04:00` is submitted as `2030-07-09T19:00:00+08:00`. A genuine PopBoom query `2030-07-09 19:00:00` matches under the confirmed Beijing convention; `2030-07-09 07:00:00` is twelve hours early.

## Deterministic time check

Use the bundled helper for the time comparison; it performs no network or publishing operations and does not write the manifest or ledger:

```text
python scripts/audit_schedule_time.py <audit-input.json> --output <audit-result.json>
```

Paths are relative to the publishing skill. Prefer the configured bundled Python runtime when it supplies the IANA timezone database; some Windows system Python installations do not. The helper returns exit 0 only for `passed`; exit 2 means `needs_review` or `mismatch`. Read its result; the helper cannot establish that evidence is truthful, verify identity/copy, or replace the whole audit. Only mark the full row passed after those separate checks.

Example input (fictional IDs):

```json
{
  "schedule_id": "example-schedule",
  "expected_time": "2030-07-09T07:00:00-04:00",
  "submitted_time": "2030-07-09T19:00:00+08:00",
  "platform_contract": "popboom_beijing_caption_v1",
  "expected_timezone": "America/New_York",
  "observed_time": "2030-07-09 19:00:00 +0800",
  "observed_at": "2030-07-08T12:00:00Z",
  "evidence_kind": "check_publish",
  "evidence_ref": "evidence/check-example.json"
}
```

For an existing publication record, `log_id` may replace `schedule_id`; keep the actual returned ID type. Optional `evidence_not_before` is an offset-aware lower bound for a fresh submission/reconciliation audit. For a timezone-free value, the optional `observed_timezone` and `timezone_evidence_ref` must refer to the applicable platform evidence above. Do not populate them from the host's local timezone. If an IANA timezone database is unavailable, the helper reports review required rather than guessing a DST offset.

The workbench keeps the original response in `verificationEvidence = {"tool": "check_publish", "callId": "<actual tool call ID>", "attemptId": "<runner attempt ID>", "queriedAt": "<offset-aware ISO>", "result": "<complete raw MCP response or structured JSON>"}` and computes `postScheduleAudit` separately. It binds that evidence to a completed call in the runner's publishing event log, matching query ID and response digest. Its audit clock comes from the runner's recorded receipt time, not a rewritten action or evidence timestamp. Preserve the corresponding `creationEvidence` with the actual creation-call identity when using the returned video/file ID as the asset mapping.

If using the time-only Python helper with this evidence, extract `observed_time` from the matched `result`, use the verified query observation time as `observed_at`, and retain a reference to the whole evidence object. Never use the local approved time as the extracted observed time. An unbound tool label, locally assembled result or an old event without a verifiable observation time remains review required; the Python helper's arithmetic result cannot establish source authenticity.

## Persist and report

Under the existing run's publishing directory, save a post-schedule audit per original action/ID with:

- Approved request/manifest reference and hash; account zone, expected local ISO and UTC.
- Raw creation and query evidence references/hashes, returned video/schedule IDs, and offset-aware `observed_at`.
- Raw observed time, explicit offset or interpretation evidence, normalized UTC when known, and signed delta.
- Identity/copy field comparisons, missing optional fields, and `passed`, `needs_review` or `mismatch` with concrete reasons.

Keep platform state separate: "waiting" proves a task is pending, not that its time is correct. Show intended and observed times in separate columns. Never fill observed fields from the request/manifest or publish a calculated Beijing column as though the platform returned it. Existing historical reports without time evidence need review; do not rewrite immutable submission history to make them pass.

## Repair and follow-up

Use an exposed edit operation on the existing ID when available and within the user's authorized correction scope. Persist the old and new values, then query and audit again. If only cancellation/recreation is possible, verify cancellation of the original before any replacement and record the link between IDs; never issue `publish_video` as an assumed update. If no supported mutation exists, report the concrete limitation and leave the audit unresolved.

After a scheduled time, if the user requested follow-up, query the actual publication record and distinguish planned time from actual published time. Do not infer publication from time elapsed. Create a persisted monitoring mechanism only when requested; a skill rule itself never runs unattended.
