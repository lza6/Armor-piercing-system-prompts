# Playbook · diagram

**Use when** the relationship is the point: a request flow, a system
architecture, a state machine, a dependency graph, a sequence of calls. Anything
you'd otherwise describe as "A sends X to B, which…" is clearer drawn.

**Don't** reach for a diagram when there's no topology — a list of steps with no
branching is a numbered list, and tabular data is a [table](table.md).

## Choose the technique

- **Mermaid** (default) when automatic node placement and edge routing matter
  more than rich node content — flows, sequences, state, ER, class diagrams. This
  is the precedent in `examples/architecture-overview.html`; the canvas Chrome has
  internet, so the CDN script is fine.
- **CSS grid / positioned HTML / SVG** when each node needs prose, code, a
  control, or heavy annotation — Mermaid nodes are cramped for that.
- **Hybrid** for big systems: one small Mermaid overview, then detailed HTML
  cards below it. Don't cram every file into one graph.

## Recipe

1. Lead with the **question the diagram answers** ("How does an upload become a
   thumbnail?"), not "here is the architecture."
2. Keep the first visual to the **core relationship**; push dense evidence (file
   refs, endpoint tables, ops notes) into cards below it.
3. Prefer **top-down** (`TB`) for multi-step flows unless the flow is genuinely
   short and linear (`LR`).
4. Use page-scoped class names — Mermaid owns generic ones like `.node`.
5. Label uncertain edges as questions so the reader can correct them.

## Light + dark (the Mermaid gotcha)

Mermaid bakes its theme **at `initialize()`** and won't restyle on a CSS theme
flip. Wire it to the toggle: stash each graph's source, and on theme change
re-`initialize` then re-`run`. **Don't ship the stock `neutral`/`dark` themes** —
they read as generic AI-Mermaid (flat gray nodes, gray edge-label boxes that
clash in dark). Use `theme:'base'` with `themeVariables` matched to the page
palette: accent-tinted `mainBkg`, accent `nodeBorder`, ink `primaryTextColor`,
and `edgeLabelBackground` set to the card surface so labels blend. Give `.diagram`
`overflow:auto`, and prefer **`LR`** for a cyclic/wide flow so every node stays
above the laptop fold. The example wires all of this up.

**Sequence diagrams use their OWN text vars.** `primaryTextColor` colors
flowchart node labels — but sequence-diagram arrow labels read from
`signalTextColor`, participants from `actorTextColor`, and notes from
`noteTextColor`, none of which reliably inherit high contrast. A theme that only
sets `primaryTextColor` renders a *readable flowchart and a faint, near-invisible
sequence diagram*. Set the sequence vars explicitly (the snippet does). And keep
the diagram off any dark code surface: Mermaid's SVG background is transparent, so
a global `pre{background:<dark>}` bleeds through — `base.html` now resets
`pre.mermaid{background:transparent}`; keep that if you hand-roll the page.

## Snippet

```html
<div class="card bleed">
  <div class="eyebrow">Flow · upload → thumbnail</div>
  <h2>How does an upload become a thumbnail?</h2>
  <div class="diagram" style="overflow:auto"><pre class="mermaid">
flowchart LR
  U([Client]) -->|upload| API[Upload API]
  API -->|store| S3[(Object store)]
  API -->|enqueue| Q[[Queue]] --> W[Resizer worker]
  W -->|write sizes| S3
  S3 --> CDN[(CDN)] -->|serve| U
  </pre></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.min.js"></script>
<script>
  document.querySelectorAll('.mermaid').forEach(n => n.dataset.src = n.textContent);
  // Brand-matched vars — swap the hexes for your page tokens.
  const vars = m => m === 'dark'
    ? { background:'transparent', mainBkg:'#1b2140', nodeBorder:'#93a4ff', lineColor:'#7c8bff',
        primaryTextColor:'#e6e8f0', textColor:'#e6e8f0', edgeLabelBackground:'#171923',
        // sequence diagrams read from these, NOT primaryTextColor — set them or labels go faint:
        actorBkg:'#1b2140', actorBorder:'#93a4ff', actorTextColor:'#e6e8f0', actorLineColor:'#7c8bff',
        signalColor:'#c9cfea', signalTextColor:'#e6e8f0',
        noteBkgColor:'#2a2140', noteBorderColor:'#c9a94a', noteTextColor:'#f2e7c4',
        labelBoxBkgColor:'#1b2140', labelTextColor:'#e6e8f0', sequenceNumberColor:'#171923' }
    : { background:'transparent', mainBkg:'#eef1ff', nodeBorder:'#4c5fd5', lineColor:'#7784d8',
        primaryTextColor:'#1a1b26', textColor:'#1a1b26', edgeLabelBackground:'#ffffff',
        actorBkg:'#eef1ff', actorBorder:'#4c5fd5', actorTextColor:'#1a1b26', actorLineColor:'#7784d8',
        signalColor:'#3a4260', signalTextColor:'#1a1b26',
        noteBkgColor:'#fff3c4', noteBorderColor:'#e0c15a', noteTextColor:'#5a4b00',
        labelBoxBkgColor:'#eef1ff', labelTextColor:'#1a1b26', sequenceNumberColor:'#ffffff' };
  function renderMermaid(mode){
    document.querySelectorAll('.mermaid').forEach(n => {
      n.removeAttribute('data-processed'); n.textContent = n.dataset.src;
    });
    mermaid.initialize({ startOnLoad:false, theme:'base', themeVariables: vars(mode) });
    mermaid.run();
  }
  window.__onThemeChange = renderMermaid;   // base.html calls this on every theme flip
  renderMermaid(document.documentElement.getAttribute('data-theme') === 'dark'
    || (document.documentElement.getAttribute('data-theme') !== 'light'
        && matchMedia('(prefers-color-scheme: dark)').matches) ? 'dark' : 'light');
</script>
```

**Pitfalls:** don't hand-build boxes-and-arrows from `<div>`s for a flow (no edge
routing, reads worse than Mermaid); don't let default diagram colors clash with
the page or become invisible in one mode — a dark `pre` background behind a
transparent Mermaid SVG, or an unset `signalTextColor`, both produce dark-on-dark
sequence labels; don't present unverified architecture as fact — cite the files
that back it. (`glimpse publish` now audits contrast and warns with an
`invisible-text` finding when text is near-invisible against its background.)

Worked example: [`examples/playbooks/diagram.html`](../../../examples/playbooks/diagram.html).
