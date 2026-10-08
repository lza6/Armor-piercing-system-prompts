# Playbook · table

**Use when** you have many records that share the same fields and the reader
needs to **scan and compare** — endpoints, config keys, test results, files with
status, dependency versions. A table's job is to make the important column
readable without reading every cell.

**Don't** use a table when each record has a different shape or needs a paragraph
of explanation — that's cards. And a two-option tradeoff is a
[comparison](comparison.md), not a table.

## Recipe

1. Open with a **one-line summary** of what the rows prove or require
   ("14 endpoints; 3 still unauthenticated").
2. Order columns by the decision they support: **identity → evidence → status →
   action.** Put the column the reader scans for first.
3. Make **status legible without color alone** — a word or badge, not just a red
   cell (color is reinforcement, not the only signal; also helps in both themes).
4. Right-align numbers; keep units in the header, not every cell.
5. Keep raw detail reachable (a `<details>` row or a linked drill-down) rather
   than widening the table until it overflows.

## Light + dark & overflow

- Wrap every table in `.table-wrap{overflow-x:auto}` so a wide table **scrolls
  itself** instead of pushing the whole page sideways.
- Long paths / URLs / IDs in cells: `overflow-wrap:anywhere`, or truncate with
  `text-overflow:ellipsis` and put the full value in a `title=`.
- Sticky `th` (`position:sticky;top:0`) keeps headers visible while scanning long
  tables; give it a solid themed background so rows don't bleed through.
- Status badges should use the themed `--good/--warn/--danger` soft pairs so they
  stay legible in dark mode.

## Snippet

```html
<div class="eyebrow">Records · endpoints</div>
<h2>14 endpoints — 3 still unauthenticated</h2>
<div class="table-wrap">
  <table>
    <thead><tr><th>Method</th><th>Path</th><th>Auth</th><th style="text-align:right">p95</th></tr></thead>
    <tbody>
      <tr><td><code>POST</code></td><td style="overflow-wrap:anywhere"><code>/v1/images</code></td>
          <td><span class="chip" style="background:var(--good-soft);color:var(--good)">required</span></td>
          <td style="text-align:right">42&nbsp;ms</td></tr>
      <tr><td><code>GET</code></td><td style="overflow-wrap:anywhere"><code>/v1/images/:id</code></td>
          <td><span class="chip" style="background:var(--danger-soft);color:var(--danger)">none</span></td>
          <td style="text-align:right">8&nbsp;ms</td></tr>
    </tbody>
  </table>
</div>
```

**Pitfalls:** don't paste a terminal/markdown table and call it done; don't bury
the conclusion under an undifferentiated grid; don't rely on color as the only
status signal; don't let one long cell set the table's width.

Worked example: [`examples/playbooks/table.html`](../../../examples/playbooks/table.html).
