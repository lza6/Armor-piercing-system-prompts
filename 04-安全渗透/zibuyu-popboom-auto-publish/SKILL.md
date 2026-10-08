---
name: zibuyu-popboom-auto-publish
description: Schedule existing, verified Zibuyu PopBoom videos to bound TikTok accounts, then audit the platform's saved time, timezone, identity and copy. Reconcile existing schedules without duplicate submission; integrated publishing follows whole-batch production and copy delivery.
---

# Zibuyu PopBoom Auto Publish

Publish already completed videos through PopBoom using the user's account bindings and defaults. This skill does not authorize new video generation, paid retries, or real publication merely by being loaded.

For a resumed publishing action, first load its saved publishing ledger. If `schedule_id` or `log_id` exists, use `check_publish` and the [post-schedule audit](references/post-schedule-audit.md); report platform state and audit verdict separately. Do not rerun preparation, require a future timestamp for an already submitted action, or submit it again. The entry gates below govern new publishing preparation and writes, not read-only reconciliation of an existing action.

## Entry Modes and Whole-Batch Gate

- **Integrated mode:** preserve the original Zibuyu production workflow. Finish every planned video in the current batch, its quality review, and the user-visible video and copy delivery before entering publishing.
- Every in-scope video must have production status `succeeded`, final QA disposition `keep` (agent review or explicit user manual acceptance, with its basis retained), and its final market-language caption actually delivered to the user. A submitted task, a saved file, or copy drafted only in an internal ledger is insufficient.
- Pending, failed, unknown, rejected, or undelivered items block publishing for the whole batch. Do not publish early successes, silently exclude failures, or treat a partial batch as complete. Only an explicit user change can reduce the planned scope; record that change before re-evaluating the gate.
- During production, only retain any account/PID/date already supplied as publishing intent. Do not load this publishing workflow, query channels/products, prepare schedules, or delay production delivery for missing publishing information.
- After the gate passes, the integrated workflow's handoff starts publishing preparation without another invocation or request to enter this skill. If the user explicitly requested production only, manual delivery, or no publishing, stop at production delivery. Otherwise collect all still-missing account code, PID, date, or video locator in one concise request, reusing everything already known.
- **Independent mode:** accept the user's explicitly identified set of existing videos. Verify that set's completed status, usable assets, QA, and delivered final copy; do not reconstruct a production batch or regenerate videos. Videos belonging to an active integrated batch still obey its whole-batch gate.
- Reuse existing verifiable QA and delivery evidence. If evidence is missing, complete only the missing review/delivery and preserve any limitation; do not claim a pass from a platform success alone.

For a Zibuyu run with `batch-compile.json`/`ledger.json` or a parent `batch-plan.json`, resolve the owning plugin's `zibuyu-top-tiktok-operations-specialist` skill and its `references/post-production-publishing.md`. Run its `scripts/build_publish_handoff.py <absolute-run-directory>` and require a freshly passing whole-batch snapshot; a child path must include its parent batch. This conditional bridge applies in both entry modes. Independent existing videos without those production artifacts do not require this helper or a fabricated production ledger.

Before preparing real publication, inspect the current PopBoom tool schema. Prefer it over static notes. For Zibuyu apparel, retain the production workflow's current status, garment-fidelity, and final-copy requirements.

## Standing user-confirmed path

Use [confirmed-publishing-contract.md](references/confirmed-publishing-contract.md) for the caption, Beijing-time submission/readback, manual QA and non-redundant authorization rules. These settled rules supersede older unresolved-caption/timezone guidance. Do not ask the user to reconfirm them.

## Account Bindings

| Account code | TikTok username | Market | Timezone | Language | Secondary screenshot evidence |
| --- | --- | --- | --- | --- | --- |
| 德1 | `ryleighhsing9` | DE | `Europe/Berlin` | Deutsch | Active marketing account, shared; shop `Tiktok跨境23店_DE` |
| 德2 | `nanettefei5` | DE | `Europe/Berlin` | Deutsch | Active marketing account, shared; shop `Tiktok跨境23店_DE` |
| 德3 | `jasperping1` | DE | `Europe/Berlin` | Deutsch | Active marketing account, not visibly shared; no visible bound-shop label |
| 美1 | `arrage83` | US | `America/New_York` | English | Active marketing account, shared; shop `Tiktok美国跨境3店` |
| 美2 | `basildxg8ho` | US | `America/New_York` | English | Active marketing account, no visible bound-shop label |
| 美3 | `heathpfsu8o` | US | `America/New_York` | English | Active marketing account, no visible bound-shop label |

Only these six accounts are in scope; the user deferred the seventh. Model/account codes select these bindings, not arbitrary recent videos.

Resolve the full current `channel_id` with `list_channels(status="active", keyword="<TikTok username>")`. Verify the returned identity and active state; shop labels are secondary evidence. Missing, inactive, duplicate, or mismatched results require clarification before submission. Screenshot short values such as `ffd5b5cf` and `c517511b` are not verified complete channel IDs.

## Default Publishing Settings

- Count: use the completed, verified videos assigned to the selected account, with a normal daily cap of 3 scheduled posts per account. Check the available publishing ledger for existing allocations; do not assume a fresh batch means an empty day.
- Times: `07:00`, `12:00`, and `18:00` on the user-provided date in the account's timezone. With fewer than 3 videos, schedule only those confirmed videos; with more than 3 for an account/date, ask whether to carry extras to another date or leave them unpublished.
- Convert each local date/time using its named timezone into ISO 8601 with the actual date-specific offset, including daylight-saving rules. Every scheduled time must remain in the future. Do not silently roll dates forward, replace a passed slot, or omit `scheduled_time` and publish immediately.
- Inspect the live `scheduled_time` contract: an example ending in `+08:00` does not by itself restrict the API to Beijing time. The offset selects the time zone when no separate timezone parameter exists. Preserve the intended instant when changing representation: `2030-07-09T07:00:00-04:00` equals `2030-07-09T19:00:00+08:00`; changing only the suffix changes the intended instant. A valid input format does not prove correct platform interpretation.
- Order: Codex chooses unless the user specifies one. Missing or conflicting date/slot decisions are resolved after production delivery, before the final confirmation table.
- Product source: default to `list_channel_products(source="shop")` for the resolved channel.
- Shopping-cart title: omit `link_title` unless the user supplies one, allowing PopBoom to use and truncate `product_title` automatically.
- Video title: use the entire delivered caption plus exactly five hashtags as `video_title`; never replace it with a short title.
- Caption: reuse the final delivered body text and exactly 5 unique, relevant, non-brand hashtags in the account's market language. Never use `#Imily Bela` or `#ImilyBela`. Keep each caption and its tags together.
- `run_precheck`: default `false`.
- `is_ai_generated`: default `false`, following the user's explicit account defaults unless overridden for the batch.
- Cover: first frame, preferably `cover_timestamp_ms: 0`; omit all cover fields only if the current verified default already selects the first frame. Do not require a cover image URL.
- `music_id`: none by default.

## Resolve the Exact Videos and Product

When the context already identifies the completed videos, minimal input remains:

```text
模特/账号代号：
PID：
发布日期：
```

Use the current batch handoff or the user's explicit record IDs, run directory, local folder, or URL list. If the set is ambiguous, request one locator; do not search unrelated recent work and guess.

The live `list_channel_products.keyword` description may provide only fuzzy product-title matching, not exact PID search. A keyword hit alone is never PID verification. Use a previously verified channel-specific PID-to-product mapping or retrieve candidates/pages and check an exact returned product identifier or evidenced SKU/PID mapping. Confirm channel, market, and product identity; do not accept the first similar title. If no exact mapping can be established, resolve the ambiguity before publishing.

Use `product_id` and `product_title` from the verified PopBoom/TikTok product record. Keep the user's PID alongside the matched ID and the evidence of their relationship.

For PopBoom-generated videos, reuse a usable `check_task(record_id)` `video_url`; download only when it is missing/unusable or a local mirror is needed. For local videos, use the available PopBoom upload path within the authorized scope and retain its returned public URL. Do not upload or download a second copy when the verified asset is already usable.

## Full caption and Beijing scheduling

Set `video_title = caption_final` (the full delivered caption plus five hashtags). The mapping is settled under `popboom_beijing_caption_v1`; do not block on a missing caption argument or ask for mapping evidence again. Honor actual live API errors without inventing length limits.

Always convert account-local slots using date-specific DST to explicit Beijing `+08:00` before submission. Record account-local ISO, Beijing ISO and UTC. Parse bare PopBoom `check_publish` times as Beijing for those submissions under the confirmed contract, preserving the raw query. Explicit returned offsets take precedence.

## Final Manifest and Bound Confirmation

After the entry gate, identity checks, future-time checks, and caption gate pass, persist a concrete manifest containing the batch/selected-set identity, each video and QA/copy evidence, exact publishing parameters, and full caption mapping. Compute a deterministic content hash for its final version.

Integrated mode stores `manifest.json`, `approval.json`, and `ledger.json` in `<production-run>/publishing/`. Independent mode uses a stable `$CODEX_HOME/zibuyu-publish-runs/<publish_run_id>/` directory for the same three publishing files; this is a publishing record, not a fabricated production ledger. Save and report the directory/ID so a resume can find it. Reuse a known directory when resuming, and consult existing records for that account/date before allocating new slots; never create a fresh identity to bypass an uncertain earlier action.

Show a concrete publishing table for traceability; reuse the user's explicit same-scope publishing instruction as authorization without asking again; long captions may occupy linked, clearly identified rows in the same reviewable artifact:

```text
批次/范围 | 账号代号 | TikTok账号 | channel_id | 视频/record_id | video_url
PID | product_id | product_title | video_title | 完整caption及5标签 | caption传输字段
link_title | 当地日期/时区 | scheduled_time | run_precheck | is_ai_generated | 封面 | music_id
```

Include the table/manifest version or hash. Bind authorization to that exact content and selected set. A prior explicit approval of the same concrete table remains valid while its content is unchanged; do not ask again per video. Production-only approval is not publishing approval, but an explicit request to publish the identified set with account/PID/date is sufficient; persist it bound to the concrete manifest without another routine confirmation.

Changes to videos, accounts, products, complete copy, exact times, or publishing parameters require authorization for the affected scope. Reuse an existing explicit correction request that already identifies that scope; do not request redundant permission. A different offset spelling of the same instant is not a different intended publish time, but still record the exact outbound payload and any manifest revision. Do not silently work around an API error or claim an unverified audit passed.

## Submit, Persist, and Verify

Prepare only supported live `publish_video` arguments, including:

```text
channel_id, video_url, video_title, product_id, product_title, scheduled_time
run_precheck=false, is_ai_generated=false
```

Add the verified full-caption mapping and approved optional fields using the live schema. Use at most one cover field: `cover_timestamp_ms`, `cover_uri`, or `cover_image_url`.

Before each submission, persist its exact request, approved manifest hash, and a stable action identity in the batch's publish ledger. Include video identity, channel, product, scheduled time, and caption hash in that identity. This is local duplicate protection, not a claim that the API supports an idempotency parameter.

- Submit only the approved rows. Check existing action records first; never create a duplicate because a prior response was slow or a session resumed.
- Persist each returned `schedule_id` or `log_id` immediately. When an ID exists, query it with `check_publish`; do not resubmit that action while its outcome is unresolved.
- A timeout, disconnect, or ambiguous response after dispatch becomes `submission_unknown`. Do not automatically retry `publish_video`. Query a known ID; without an ID, reconcile available records or report the specific unresolved action before any further submission of it.
- Keep `scheduled`, `published`, `failed`, and `submission_unknown` distinct. A returned ID is a receipt, not proof of either successful scheduling or publication.
- After each submission, immediately query its exact ID and apply the [post-schedule audit](references/post-schedule-audit.md) before dispatching the next row. Use actual query evidence and the confirmed Beijing readback convention; a bare time alone does not require another question. Missing/malformed values are `needs_review`; a proven time or identity difference is `mismatch`. Stop remaining new submissions, preserve their unsubmitted state, and continue useful read-only reconciliation when either occurs.
- Persist raw query evidence, observation time, expected and observed timestamps/timezone evidence, UTC comparison and field-level findings. Keep platform state (`scheduled`/`published`/`failed`) separate from audit status (`passed`/`needs_review`/`mismatch`). Only say "排期已审核通过" when every intended row passes; otherwise say "平台已创建排期，时间/内容待审核" or identify the proven mismatch. An actual publication remains a publication fact even if its audit fails.
- Reuse successful rows and their evidence; a failed or unresolved row does not justify rerunning the batch or creating a paid generation task.

Keep publishing preparation bounded: reuse the delivered handoff, query each needed channel/product once per preparation pass, and refresh only stale or changed facts. Do not replace this reviewed workflow with `create_hosting_task`, which combines paid generation and automatic publication.
