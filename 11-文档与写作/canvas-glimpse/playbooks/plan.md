# Playbook · plan

**Use when** the human needs to inspect *what you're about to do* before you do
it: a technical design, an implementation plan, a PRD, a refactor proposal, a
migration. The goal is that another developer could read it and implement the
proposal — and that the human can catch a wrong assumption before code exists.

**Don't** use the full plan shape for a single small design choice — that's a
[comparison](comparison.md) or a one-question [input](input.md).

## Recipe

1. **Goal → current state → desired behavior**, up top, in that order.
2. **The proposed approach**, focused on the high-level decisions and *why*.
   Don't only enumerate the ambiguous bits — state the actual proposal.
3. **Steps / phases**, sequenced. Number them only because order genuinely
   matters here (it's a real sequence, not decoration).
4. **Risks, migration/back-compat concerns, and open questions** at the end.
   When a decision is still open, follow the [comparison](comparison.md) shape and
   give the human options to choose from — or escalate it as an
   [input](input.md).
5. Verify every claim against the codebase before presenting it as fact. If the
   plan touches UI, **mock the experience** (or screenshot the real one) rather
   than describing it in prose.

## Design notes

- A plan is a **reading surface** — keep prose at the ~680px measure, break it
  into clearly-titled cards, and make the proposal skimmable (lead sentence per
  section carries the point).
- Use a **phase strip** or numbered steps for sequence, a small
  [diagram](diagram.md) for the shape of the change, and callouts (`.note.warn`)
  for risks.
- Keep resolved questions *out* — when a decision lands, fold it into the plan
  and delete the open question; don't leave a stale "TBD".

## Snippet

```html
<div class="eyebrow">Plan · thumbnail backfill</div>
<h1>Backfill thumbnails without downtime</h1>
<p class="lede">Goal: every existing image gets a <code>webp</code> thumbnail,
  with no read downtime and no destructive writes.</p>

<div class="grid">
  <div class="card"><h3>Now</h3><p>Thumbnails generated on upload only; ~2.1M
    originals have none.</p></div>
  <div class="card"><h3>Target</h3><p>100% coverage; reads fall back to on-the-fly
    resize until backfilled.</p></div>
</div>

<div class="card">
  <h2>Approach</h2>
  <ol>
    <li><b>Add</b> a nullable <code>thumb_key</code> column (safe, online).</li>
    <li><b>Backfill</b> in batches of 1,000 off a work queue; idempotent by id.</li>
    <li><b>Cut reads over</b> once coverage &gt; 99%, keeping the fallback.</li>
  </ol>
  <div class="note warn"><b>Risk:</b> resize CPU on workers — throttle batch rate
    on queue depth. <b>Open:</b> keep originals forever, or expire at 1y?</div>
</div>
```

**Pitfalls:** don't omit failure modes / migration / back-compat; don't leave
resolved questions lying around; don't describe a UI you could show.

Worked example: [`examples/playbooks/plan.html`](../../../examples/playbooks/plan.html).
