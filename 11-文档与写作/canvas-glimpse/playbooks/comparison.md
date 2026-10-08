# Playbook · comparison

**Use when** the human is weighing options, tradeoffs, or **current vs. target**.
Three shapes:

- **Before / after** — the same system changing over time.
- **Option cards** — mutually exclusive directions to choose between.
- **Scorecard** — options rated against explicit, comparable criteria.

If the goal is to actually *pick* one, pair this with an [input](input.md) control
that sends the chosen option back.

## Recipe

1. **Name the decision** at the top ("Queue: SQS vs. Postgres-backed?").
2. Show the **concrete shape of each side** — real behavior, real cost — not a
   vague pro/con blur. Align corresponding rows so differences pop.
3. Make the **cost as visible as the benefit.** Don't make every option look
   equally good if one clearly wins.
4. **Recommend only when the evidence supports it** — and say why. Surface the
   assumption that would flip the recommendation.

## Design & overflow

- Put options in a `.grid` (`min-width:0` children) so 2–3 columns collapse to
  stacked on narrow screens without overflow.
- Give the recommended option a quiet accent border/badge; keep the others plain
  — hierarchy, not fireworks.
- For a scorecard, use a `.table-wrap` table with a `✓ / — / ✗` glyph **plus**
  color (not color alone), so it reads in both themes and when printed.

## Snippet

```html
<div class="eyebrow">Decision · job queue</div>
<h1>SQS or a Postgres-backed queue?</h1>

<div class="grid" style="grid-template-columns:repeat(auto-fit,minmax(min(280px,100%),1fr))">
  <div class="card" style="border-color:var(--accent)">
    <div class="chip">Recommended</div>
    <h3>SQS</h3>
    <p><b>Benefit:</b> scales to spikes, zero ops.</p>
    <p><b>Cost:</b> another dependency; at-least-once delivery → make workers idempotent.</p>
  </div>
  <div class="card">
    <h3>Postgres-backed</h3>
    <p><b>Benefit:</b> one datastore, transactional with the job row.</p>
    <p><b>Cost:</b> polling load; caps out well below SQS throughput.</p>
  </div>
</div>
<div class="note"><b>Why SQS:</b> traffic is spiky and we already run idempotent
  workers. Flip to Postgres if volume stays under ~50 jobs/s.</div>
```

**Pitfalls:** don't compare vague summaries when you have concrete examples;
don't bury the assumption that changes the answer; don't present a foregone
recommendation as a neutral toss-up.

Worked example: [`examples/playbooks/comparison.html`](../../../examples/playbooks/comparison.html).
