# Poll and reconcile existing PopBoom jobs

Paths in code spans are relative to the PopBoom skill directory. This route is read-only toward PopBoom: discover only the required `check_task` / conditional `download_video` live schema. It does not load the production manual or authorize upload/generation. If the exact tool is not callable, use tool discovery before reporting it unavailable. Use the configured trusted connection; do not invent endpoints or retrieve credentials from logs.

## Persistent state

1. Read the relevant ledger job and previous persisted state; for Zibuyu use the parent `inspect_run.py` snapshot and returned parent/child paths. A malformed or missing ledger is a blocker to state updates, not evidence the job was never submitted.
2. A known `record_id` must never return to `generate_video`, migration or new receipt generation. Preserve `resubmit_allowed: false`; select query/download/QA/report from evidence. For `submission_unknown`, `submission_started`, or an interrupted dispatch without a confirmed record, reconcile existing platform history with the exact stored identity. No automatic resubmission, even after timeout or a missing search result.
3. Persist raw query responses and their observation time, record ID and normalized status. Never replace a known terminal/accepted record with guessed status or infer success from a tool envelope. Preserve the immediately previous ledger snapshot before changes and validate each terminal transition with `scripts/validate_ledger.py --previous-ledger ...`; include exact batch compile where required. Use `--summary --result-file <run-local-result.json>` and inspect full errors on failure.
4. Do not reset configured concurrency (12, user source), effective inflight limits or rate-limit events. A 429/concurrency signal retains the batch-scoped latch with effective inflight 1; it never authorizes a new paid wave. For a streaming parent apply the latch across all children. Read the relevant persistent-ledger/rate-limit section of [batch execution](batch-execution-contract.md) when updating those fields.

## Poll cadence and completion

- Query active records round-robin: queued after 15s, then 30s, then at most every 60s; running about every 15s; use +/-20% jitter.
- Retry transient network, 429 and 5xx query errors at most three times with backoff and the same record. Stop if authentication/access fails. After 15 minutes without terminal evidence retain the record as `pending_timeout` for later resume; never regenerate.
- Preserve the successful `check_task.video_url` when usable. Use `download_video` at most once only if that URL is missing or a local mirror is required, persisting reason/attempt/result/artifact metadata. Zibuyu actual-media QA needs the original MP4 and observed evidence; a usable link alone cannot establish QA.
- Report meaningful completion/failure or action needed. Aggregate unchanged query results; do useful independent authorized work while waiting. Never conceal failures or mark uninspected media as accepted.
- Identity mismatch in a quality-gated batch blocks remaining releases. Keep completed record evidence; a corrected paid attempt requires its own authorized scope.

## Rejoin delivery

A run belongs to Zibuyu if the plugin was invoked, a sibling batch compile or parent batch plan exists, or the request resumes a Zibuyu apparel batch. For those runs read the parent's [Resume and delivery](../../zibuyu-top-tiktok-operations-specialist/references/resume-and-delivery.md), inspect original media, run its delivery validator and deliver every status/video or failure plus quality verdict and complete five-tag caption. Set `completion_reported` only after that package reaches the user. Do not enter publishing until the whole intended batch is accepted and delivered; respect production-only requests and separate publication authorization.

A generic standalone PopBoom task without Zibuyu context ends after its requested terminal status/link report. Missing evidence remains explicit.
