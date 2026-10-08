# Amazon Product And Review Evidence Contract

Use this contract whenever the user supplies an Amazon product URL, asks for review analysis, or wants product-page facts to influence an apparel script. It is the evidence-provenance contract for schemas `1.3` and `1.4` even when no marketplace URL is supplied.

## Collection Order And Access Boundary

1. Lock the requested URL, marketplace, parent ASIN when visible, selected child ASIN, color, size, `th`/`psc` selection, and capture time before extracting facts.
2. Prefer the user's already signed-in Chrome session when the Chrome control skill and its required browser tool are exposed. Use it for dynamic specifications, variation labels, direct review cards, and buyer images.
3. If Chrome control is unavailable, use the public Amazon page. Never claim that public-web access was logged-in Chrome control.
4. Never bypass a CAPTCHA, login, regional restriction, or anti-bot page. Ask the user to complete the browser step or provide screenshots when that evidence is necessary.
5. Record `complete`, `partial`, `blocked`, or `not_provided`. A blocked review endpoint does not block a visual-only script, but all review-derived statements must be removed.
6. Search snippets, Amazon AI summaries, and third-party pages may help discover a lead. They never become direct product or buyer evidence without an exact-ASIN Amazon source.

## Identity Lock

- A URL is not evidence by itself. Each evidence source must identify its URL or local artifact, parent product ID, child product ID when applicable, selected variant scope, locator, capture time, and content SHA-256.
- `exact_child` means the source is visibly tied to the selected child ASIN and variant. `parent_family` and `sibling_child` evidence must be labeled and cannot be presented as an exact selected-variant test.
- Do not substitute a similar item, different ASIN, editorial summary, reseller listing, or another color/size review while calling it the requested product.
- Amazon commonly aggregates reviews across a variation family. Preserve each review's displayed child ASIN/color/size. Family reviews may inform qualified pain analysis, not exact-variant promises.

## Schema 1.3 And 1.4 Research Bundle

Every schema `1.3` or `1.4` compile contains `shared_core.research_bundle` with:

```text
contract_id: amazon_product_review_evidence_v1
requested_url, canonical_url, marketplace
parent_product_id, child_product_id, selected_variant
captured_at, collection_route, status, limitations[]
evidence_sources[]
evidence_items[]
review_analysis { status, sampled_review_ids[], buyer_image_count, themes[], conflicts[], excluded_sources[] }
```

`evidence_sources[]` uses stable `source_id` values and one of:

- `amazon_catalog_attribute`: structured Amazon attribute for the locked product/variant.
- `amazon_seller_copy`: seller/manufacturer bullet or description.
- `amazon_customer_review`: one direct Amazon review with unique `review_id`, rating, explicit `review_date_status` (`captured` or `not_captured`), nullable date, verified-purchase state (`true`, `false`, or `unknown`), and displayed variant scope. Never invent a missing date or treat an absent badge as a negative badge.
- `amazon_customer_image`: one buyer image linked to its `parent_review_id`.
- `amazon_qa`: Amazon Q&A; discovery-only unless the answer has separately verified authority.
- `amazon_review_summary`: Amazon-generated/AI review summary; discovery-only.
- `third_party_summary`: reseller, search, editorial, or other external summary; discovery-only.
- `visible_reference`, `garment_label`, `user_provided`, or `verified_test`: non-Amazon evidence with the same provenance fields.

Do not store reviewer names, profiles, or full review bodies in the batch object or final generation prompt. Store review IDs, hashes, metadata, and short normalized propositions. Keep long quotations outside the paid prompt.

`evidence_items[]` maps a stable `evidence_id` to one source and declares:

- `assertion_kind`
- a short normalized `statement`
- `exact_product_match`
- `exact_variant_match`
- `conflict_status`
- `permitted_uses[]`
- `performance_demo_allowed`

Discovery-only, mismatched-product, conflicted, or blocked evidence cannot authorize a script claim. A customer image can support only visible color, fit, length, or construction observations. It cannot prove composition, softness, stretch, breathability, comfort, care, opacity performance, or durability.

## Source Strength And Language

- Amazon catalog attributes or a clearly readable garment label may support objective composition, weight, fit, length, stretch classification, and care facts for the locked variant.
- Seller copy supports only the seller's attributed qualitative statement. It never becomes a buyer opinion and never authorizes “buyers say”.
- Direct customer reviews support buyer-reported experience and pain. They do not automatically prove an objective material or performance property.
- One direct review permits only a single-review attribution such as “one buyer mentioned”.
- A sample theme requires at least three unique direct `review_id` values. Say “in the reviews sampled” rather than implying the entire rating population.
- Never use the page's total rating count as the number of reviews actually read.
- Mixed positive and negative evidence must remain `mixed`; do not collapse it into an absolute claim.

## Pain-To-Solution Gate

Persist `shared_core.pain_solution_map[]` before scriptwriting. Each entry contains:

```text
pain_point_id, pain_statement, basis, evidence_ids[], review_theme_id
status, selected_for_script, solution_claim_ids[]
```

- A review-derived pain must resolve to direct review evidence or a valid review theme.
- `script_eligible` pain requires at least one independent, allowlisted exact-variant solution claim from an exact-child catalog, inspected exact-child visual, garment label, verified user source, or verified test. Review evidence alone never proves its own solution. A complaint with no evidenced product solution stays `risk_only` and cannot become a conversion hook.
- Seller copy cannot create a buyer pain. Category inference may create a clearly labeled question hook, but not a fake buyer consensus.
- Every selected hook carries the same `pain_point_id` and one registered target-language pain term; every variant's `creative_delta.pain_focus_id` must resolve to it; the first product-proof beat after the hook must be one mapped solution claim.
- A direct-review/review-theme material-expectation pain may use `hook_semantics: evidence_backed_contrast` when its mapped solution is an independent, allowlisted exact-variant `texture` claim. The hook must be a short negative declarative contrast, not a question; it keeps only the pain's review evidence and uses no product-claim ID. The immediately following beat must be that mapped texture claim in a detail close-up with off-screen voiceover and one controlled `pinch_release`. The review establishes the expectation gap; the independent product evidence establishes the visible texture. If any part of this chain is missing, use a normal pain question/context hook or keep the complaint `risk_only`.
- For a selected-variant `direct` or `visual_only` claim, every bound evidence item must set `exact_variant_match: true` and come from a semantically compatible `exact_child` source. Sibling reviews and sibling buyer images may describe pain context but cannot become the selected variant's solution.
- Review themes count unique direct `review_id` values, not normalized evidence-item count. Several evidence items from one review still count as one review.

## Claim, Proof, Beat, Caption, And Hash Binding

For schema `1.3`:

- Every claim has `evidence_ids`, `assertion_kind`, and `claim_mode`.
- The claim-proof row and canonical proof beat carry the exact same evidence IDs. Evidence may not be swapped after the analysis stage.
- The hook carries its `pain_point_id` and exact pain evidence IDs.
- An `evidence_backed_contrast` hook previews only the mapped visible texture contrast; its exact claim/evidence/proof binding remains on the immediately following product-proof beat. It may not preview softness, comfort, breathability, stretch, composition, durability, or another performance/property claim.
- Captions declare `caption_claim_ids`, `caption_pain_point_ids`, `caption_claim_bindings`, and `caption_pain_bindings`. Each binding uses a registered target-language anchor that must occur in the caption; IDs alone never prove textual coverage. Caption copy may not introduce a claim or review theme absent from those bindings.

Amazon URLs are identity fields, not free text. Require HTTPS, an approved Amazon marketplace or Amazon image host, a canonical `/dp/{child_product_id}` or `/gp/product/{child_product_id}` path, and an exact review-ID match in direct-review URLs. Query-string mentions of `amazon.com`, unrelated hosts, and mismatched ASIN/review paths are invalid.
- The compiler adds `research_bundle_sha256` to each prompt rendering and director receipt. PopBoom binds it into the paid request fingerprint.
- The compiled Seedance prompt contains concise claims and actions, not reviewer identities, raw bodies, source URLs, or the internal research object.

## Numeric And Performance Gate

- Exact composition percentages are allowed only from an exact-product Amazon catalog attribute, readable garment label, verified test, or explicit user-provided label evidence with `assertion_kind: exact_composition`.
- Numeric performance claims still require `verified_test` evidence.
- Stretch, breathability, cooling, waterproofing, opacity performance, wrinkle resistance, or similar demonstrations require linked evidence with `performance_demo_allowed: true`.
- A `Low Stretch` catalog value must use `performance_demo_allowed: false`. Do not add a strong two-hand pull just because the word “stretch” appears.
- When a supported stretch demonstration is used, perform one coordinated gentle two-hand pull-release cycle with stable torso/camera and mouth out of frame. The action strength must match the evidence.

## Care And Conflict Rule

- Care claims require the selected child ASIN's catalog care field or a clear garment label.
- If a catalog field, seller bullet, label, or review conflicts, record both items and a conflict. Remove the care claim from script and caption instead of choosing the more marketable version.
- Shrinkage, fading, pilling, or wash durability mentioned in reviews remains buyer-reported risk. It cannot create a promise such as “will not shrink”.

## Minimum Pre-Script Analysis Output

Before hooks or voiceover, report internally and in the requested user-facing analysis:

1. Product identity and selected variant.
2. Objective product facts with source level.
3. Seller claims kept separate from objective attributes.
4. Review sample size actually read, variant scope, positive themes, negative themes, mixed themes, and buyer-image limits.
5. Conflicts and excluded sources.
6. Pain point -> evidence -> solution claim -> garment part -> close-up -> action -> visible endpoint.
7. Claims and pains deliberately excluded from the script.
