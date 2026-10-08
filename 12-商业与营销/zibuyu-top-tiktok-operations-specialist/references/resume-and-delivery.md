# Resume and delivery

All inline paths below are relative to the parent skill directory. Start with `python scripts/inspect_run.py <run-directory>`; read only the ledger/compile sections needed for the returned job actions. The snapshot is navigation only, not a validator result or live status. For a streaming child, inspect the parent and every planned color; one finished child cannot establish batch completion.

## Route from persisted evidence

- Existing record: use the sibling [poll guide](../../popboom/references/poll-and-reconcile.md). Never submit, migrate or regenerate a receipt for that accepted job.
- Unknown/interrupted submission: reconcile platform records and persisted request identity. No automatic retry even if no record is yet known.
- Planned job: load the full [production workflow](production-workflow.md) and relevant production contracts before preparing or submitting. An inspector recommendation is not authorization.
- Failed job: preserve exact error and include it in delivery. Route only the failed artifact to its owning skill. Paid retakes require explicit authorization and a new version/fingerprint; accepted IDs never repeat.
- Terminal succeeded job: use the original MP4, record source/path/hash and measured media parameters, and inspect representative beginning/middle/end frames plus action/voice/identity evidence. Inspect the actual garment construction and color, fixed model identity and continuity, not just the platform status. Load the planned claim/action/voice fields and the relevant quality rubric as needed.
- Missing captions: use the caption-only route below; preserve accepted compile bytes, immutable parent plan, prompt, receipt and video. Do not rerender for missing delivery copy.

Before mutating a ledger, preserve the previous revision and validate the exact transition with the sibling ledger validator. Historical query/report work stays historical. Preserve `resubmit_allowed: false`, record IDs, paid fingerprints and any rate-limit latch. If validator errors need schema detail, read only the applicable section of the [batch contract](../../popboom/references/batch-execution-contract.md).

## Caption-only requests and copy recovery

If valid copy already exists in the original compile, run the delivery validator and return that copy without rewriting files. If the user requests new/revised copy, write a separate `copy-supplement.json` only when an artifact is useful: bind it to the exact video/record and original compile SHA-256 when available, and include the source evidence, market, color, caption and five tags. Never edit an accepted compile or immutable streaming plan, regenerate its paid receipt, or use the supplement to claim the original production/publication gates passed. Existing validators do not consume this supplement; a publication request with revised copy needs the publishing skill's independently reviewed exact final table.

When the user provides only an existing video URL and asks only for copy, inspect available video/product evidence or use their supplied verified facts; unreadable sources remain unverified and unsupported claims are omitted. Return the requested German or American-English caption and exactly five unique relevant non-brand hashtags in one paragraph. Do not fabricate a compile/ledger, initiate production, or claim video QA. This narrow copy request ends after delivering the copy. If evidence is insufficient, ask only for the missing garment facts needed to write it accurately.

Progress/unknown/timeout/failure reports may be delivered immediately with their true status. The terminal delivery validator gates a claim of complete production delivery, not honest interim updates. Read each reference only once unless its content changed; resume/poll links are stage handoffs, not instructions to reread in a loop.

## Actual-video QA and failure handling

13. If the rendered video can be inspected, review garment identity/material, claim-to-part close-up accuracy, action-to-voiceover accuracy, action coverage, the three-phase performance arc, relaxed posture, motivated distance changes, exact two-arm/two-hand anatomy, endpoint-to-start hand continuity, persistent phone/prop ownership, lip-sync, expression/emotional progression, voice energy and intelligibility, lighting continuity, visible proof, CTA purity, and pairwise differentiation. Reject a take where the model stands at attention, repeatedly resets to both hips, performs disconnected equal-energy tasks, silently switches the phone hand, loses a held prop, stays emotionally flat, shows the wrong part, or speaks a claim without its planned action. Record exactly one quality verdict (`keep`, `fix_in_post`, `reroll`, `rewrite`, or `stop_or_rescope`), one `primary_failure_variable`, and evidence. If it cannot be inspected, record `unverified`.
14. Never start a paid retake without explicit user authorization. An authorized retake gets a new timeline version, prompt hash, and job fingerprint and changes one variable only. The same defect in two takes requires rewrite/decomposition. Never resubmit an already accepted `record_id`.
15. If PopBoom reports `failed`, stop paid execution for that color and preserve its visible error; still include that failed job in the final Copy Delivery Pack Gate.
16. After the final terminal ledger update, run the Copy Delivery Pack Gate. Return the validated status/video location or failure, creative-quality verdict, caption, and exactly 5 hashtags for every job. A link-only completion response is not a completed plugin workflow.

## Multi-Color Differentiation Completion Gate

Before PopBoom submission, require `planned_differentiation_passed` for every pair of color variants. After rendering, mark `rendered_differentiation_verified` only when the actual videos show at least one non-color audible difference and one non-color visible difference in scene, styling, proof, or core action for every pair. If the videos cannot be inspected or transcribed, mark them `rendered_differentiation_unverified`; prompt differences alone do not prove final-video differences.

## Copy Delivery Pack Gate

Treat the caption package as a required production artifact, not an optional postscript. This gate applies to new batches, interrupted batches, revision-0 resumes, direct PopBoom resumes, and already-rendered jobs.

After all jobs in a ledger are terminal and after the final ledger validation, run:

```text
python scripts/validate_delivery_pack.py <absolute-path-to-batch-compile.json> <absolute-path-to-ledger.json>
```

Use a verified Python runtime. On Windows, do not trust a `python.exe` app-execution alias: load the bundled workspace dependencies and invoke the returned absolute Python path. If no working Python runtime is available, report the gate as blocked; never bypass it with a link-only response.

Claim complete production delivery only when the command exits `0`, returns `valid: true`, and returns one delivery item for every ledger job. Use the returned `copy_ready_caption` verbatim unless the user asks for a rewrite. Each item must contain one non-empty market-language caption followed in the same paragraph by exactly 5 unique hashtags.

- Include the matching model, color, terminal status, `record_id` when present, usable video URL or verified local path, and failure reason when failed with each delivery item.
- Preserve German for 德1/德2/德3 and natural American English for 美1/美2/美3.
- For every Zibuyu apparel job, provide exactly five unique hashtags. Choose all five from product-, market-, occasion-, style-, buyer-search-, or TikTok Shop-relevant tags; do not force a fixed brand hashtag. Do not use `#Imily Bela`; replace it with a relevant non-brand tag so the total remains exactly five.
- If the delivery validator fails because copy is missing or invalid, recover existing valid copy or use the separate caption-only route above. Preserve the failed original gate explicitly; do not mutate accepted compile bytes, rerender, resubmit, or manufacture a passing receipt.
- Do not set a job's `completion_reported: true` merely because its status/link was announced. For this plugin, that flag means the user has received the complete status/link, quality verdict, caption, and exactly-5-hashtag package.
- If execution is interrupted after links were reported but before copy was delivered, keep `completion_reported: false` and `next_action: report` so the next resume repairs the handoff without another paid call.

## Completion Gate

Treat PopBoom completion as a task-status gate. In this plugin workflow, default completion means:

- the generation request was submitted;
- the user was told the request was submitted;
- PopBoom later reported `succeeded` or the UI showed completion;
- the user was told generation is complete.

Creative-quality acceptance is a separate gate. Report it only when the actual video was inspected and received `keep`; otherwise report its explicit verdict or `unverified`.

The production workflow is not complete until the Copy Delivery Pack Gate also passes and its complete output is delivered. A response containing only PopBoom statuses, record IDs, or video links is incomplete. Production completion is preserved if publishing later needs input or is blocked. For the integrated workflow, additionally track publishing preparation, confirmation, submission, verified scheduling, and actual publication separately under the post-production reference; a schedule receipt is not proof of a live TikTok post.

## Deliverables

For each color, return:

- PopBoom submission status and `record_id` when the request is submitted;
- PopBoom completion status and video URL or local file path when generation completes;
- rendered creative-quality verdict, one primary failure variable, and evidence when inspectable; otherwise `unverified`;
- the exact market-language voiceover lines with line-by-line Chinese translations whenever a prompt/script package is delivered for review;
- one copy-ready caption paragraph combining the conversion caption and exactly 5 relevant hashtags.

Before composing the final answer, materialize these fields with `scripts/validate_delivery_pack.py`. Do not reconstruct them from memory when a validated `batch-compile.json` already contains the copy.

The caption and hashtags must match the model market/language:

- `德1`, `德2`, and `德3`: German.
- `美1`, `美2`, and `美3`: natural American English.

Do not split the caption and hashtags into separate bullets, tables, or explanations unless the user asks for a separate production report.

## Hard Prohibitions

- Do not use external prompt-model workflows outside this plugin's bundled skills.
- Do not upload model portraits/photos into PopBoom reference material.
- Do not pass a clothing three-view downstream when it contains any visible realistic human-identity pixels, even when the prompt says not to transfer them.
- Do not hand-author, reorder, or paraphrase a paid PopBoom request after validation. Construct the call only from the exact validator-accepted ordered `outbound_request`; any changed prompt, reference URL/order, fixed-model identity digest, model, duration, ratio, or resolution requires a new validation and fingerprint.
- Do not expose API keys or bearer tokens in logs, scripts, captions, or final answers.
- Do not invent fabric composition, certifications, medical/health effects, exact performance numbers, discounts, reviews, sales volume, or shipping claims. Do not present seller copy, Amazon AI summaries, third-party summaries, buyer images, variation-family reviews, or a similar ASIN as exact selected-variant buyer fact.
- Do not bypass schema `1.4` evidence/action/market/quality validation, downgrade to an older schema for a new paid submission, use a non-exact v5 resume, send raw reviews/internal research/beat JSON/rationale/Chinese review translations to PopBoom, or claim that a `succeeded` task passed creative QC.
- Do not initiate any paid retake without explicit user authorization or change multiple variables in one retake.
