# Playbook · code

**Use when** the artifact shows source: a snippet, a full file, a patch/diff, or
a before/after. The reader should be able to see *what changed and why* without
scrolling a raw patch in repo order.

**First, consider the [`explain`](../../explain/SKILL.md) skill instead.** For
walking through code you wrote or a subsystem you explored, `glimpse explain`
ships a purpose-built interactive viewer (Architecture / Data flow / clickable
Call stack with per-node snippets and "Ask about this"). You produce a JSON spec;
Glimpse renders it. Reach for *this* playbook when you're hand-authoring a canvas
artifact that happens to include code (a plan with a patch, a review with
before/after) rather than a dedicated walkthrough.

## Recipe

1. Put the **path, language, and the reason to look** immediately before each
   block — "`auth/service.py` — the constant-time check we're adding."
2. Keep evidence next to the claim: reference line numbers or annotate the exact
   lines, don't separate the point from the code that proves it.
3. For a change, show **before/after** or a **+/- diff**, not two disconnected
   files. Group multi-file changes by user-facing area, not patch order.
4. Show the **relevant slice**, not a whole 800-line file.

## Design & portability

- Render code in a themed `<pre>` with `overflow:auto; max-width:100%` — long
  lines scroll the block, never the page. Keep a dark code surface in both themes
  (it reads well and matches `examples/`), but ensure the surrounding chrome
  (path header, borders) follows the light/dark tokens.
- **Diffs:** tint added lines with the `--good` soft pair and removed with
  `--danger` soft; prefix with `+`/`-` so it survives when color is muted or
  printed. A left border per line reinforces it.
- **Before/after:** side-by-side in a `.grid` (each column `min-width:0`) on wide
  screens; it collapses to stacked on narrow — no horizontal overflow either way.
- Don't paste code as a screenshot, and don't hand-highlight with brittle
  per-token `<span>`s unless you keep it minimal.

## Snippet

```html
<style>
  .codehdr{font:600 12.5px/1 var(--mono);color:var(--ink-2);
    background:var(--bg-sunk);border:1px solid var(--line);border-bottom:none;
    border-radius:9px 9px 0 0;padding:9px 13px;overflow-wrap:anywhere;}
  .codehdr + pre{border-radius:0 0 9px 9px;margin-top:0;}
  .diff .add{display:block;background:var(--good-soft);color:var(--good);}
  .diff .del{display:block;background:var(--danger-soft);color:var(--danger);}
</style>

<div class="eyebrow">Diff · auth/service.py</div>
<h2>Make the password check constant-time</h2>
<div class="codehdr">auth/service.py · lines 30–34</div>
<pre class="diff"><code><span class="del">-    if pw != user.pw_hash:</span>
<span class="add">+    if not hmac.compare_digest(pw_hash, user.pw_hash):</span>
         raise Unauthorized()
     return self.sessions.mint(user.id)</code></pre>
```

**Pitfalls:** don't dump an unrelated giant file; don't separate a claim from its
lines; don't pick a code theme that vanishes in one mode.

Worked example: [`examples/playbooks/code.html`](../../../examples/playbooks/code.html).
