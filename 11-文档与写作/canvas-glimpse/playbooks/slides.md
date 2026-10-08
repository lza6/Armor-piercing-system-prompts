# Playbook · slides

**Use when** the human explicitly asks for a **deck / presentation / talk / paced
walkthrough** — something they'll step through one idea at a time. This is the
exception, not the default.

**Don't** turn an explainer into slides by reflex. Reference material, dense
review, and research surfaces belong on a **scroll page** (every other playbook).
Slides trade density for pacing; only spend that trade when pacing is the point.

## Recipe

1. **Plan the story before the markup:** open with the point, build context, show
   evidence, close on the decision or next action.
2. **One idea per slide.** If a slide needs a paragraph, it's two slides or a
   scroll page.
3. **Vary composition** so consecutive slides don't feel like the same card
   repeated — a title slide, a big-number slide, a diagram slide, a two-column
   slide.
4. State the **screen-size assumption** and give obvious navigation (arrows +
   keyboard). Keep text sparse; let one visual carry each slide.

## Design & portability

- Fixed-aspect slides (e.g. a 16:9 `.slide` centered in the viewport) with
  `←/→` and click-to-advance; show `3 / 8` progress.
- Large type, strong alignment, deliberate whitespace — not dense paragraphs.
- Light + dark still required: a deck viewed in a dark room must not glare. Keep
  the same tokens; test both.
- No horizontal overflow — a slide is a fixed box; content that would exceed it is
  a sign the slide is overloaded.

## Snippet

```html
<style>
  .deck{max-width:900px;margin:0 auto;}
  .slide{display:none;aspect-ratio:16/9;background:var(--bg-elev);
    border:1px solid var(--line);border-radius:16px;box-shadow:var(--shadow);
    padding:clamp(24px,5vw,56px);flex-direction:column;justify-content:center;}
  .slide.on{display:flex;}
  .slide h2{font-size:clamp(24px,4vw,40px);letter-spacing:-.02em;}
  .nav{display:flex;gap:12px;align-items:center;justify-content:center;margin-top:18px;color:var(--ink-2);}
  .nav button{font:inherit;padding:8px 16px;border-radius:999px;border:1px solid var(--line);
    background:var(--bg-elev);color:var(--ink);cursor:pointer;}
</style>

<div class="deck">
  <section class="slide on"><div class="eyebrow">01 · thesis</div>
    <h2>Ship the backfill this week</h2></section>
  <section class="slide"><div class="eyebrow">02 · why now</div>
    <h2>2.1M images have no thumbnail</h2></section>
  <div class="nav"><button onclick="go(-1)">←</button>
    <span id="pos">1 / 2</span><button onclick="go(1)">→</button></div>
</div>

<script>
  let i=0; const s=[...document.querySelectorAll('.slide')];
  function go(d){ s[i].classList.remove('on'); i=(i+d+s.length)%s.length;
    s[i].classList.add('on'); document.getElementById('pos').textContent=(i+1)+' / '+s.length; }
  addEventListener('keydown',e=>{ if(e.key==='ArrowRight')go(1); if(e.key==='ArrowLeft')go(-1); });
</script>
```

**Pitfalls:** don't default to slides; don't paste a scroll outline into fixed
frames without rewriting the narrative; don't repeat the same slide composition
unless the repetition means something.

Worked example: [`examples/playbooks/slides.html`](../../../examples/playbooks/slides.html).
