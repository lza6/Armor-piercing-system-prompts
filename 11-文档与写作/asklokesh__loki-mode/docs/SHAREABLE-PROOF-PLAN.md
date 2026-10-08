# Shareable Proof-of-Run: Implementation Plan (v7.19.3)

Architect plan for adding opt-in shareability to the proof-of-run artifact
without breaking the v7.18.2 zero-egress posture. Design only.

## 1. Verified current state

All claims read from source.

**Generator** `autonomy/lib/proof-generator.py` (689 lines)
- `_build_proof` (line 366) assembles the frozen schema v1.0 dict. `deployment`
  set at line 401: `{"deployed_url": deployed_url, "public_url": None}`.
  `public_url` is ALWAYS `None` at generate time. This is the publish-time
  injection slot the design hangs on.
- `_build_social_hook` (line 444) computes the one-line hook from already-redacted
  fields only: `cost.usd` via `_fmt_usd_hook` (line 427), `files_changed.count`,
  and council ratio via `_council_ratio` (line 406). Cost clause omitted when usd
  is None (never fabricates `$0.00`).
- `_render_html` (line 562) loads `proof-template.html`, replaces
  `__PROOF_OG_DESCRIPTION__` (og + twitter metas) with `_attr_esc(hook)`, then
  replaces `__PROOF_JSON__` (escaping `<` as `<`). Falls back to
  `_render_fallback_html` (line 470) when template missing.
- CHOKEPOINT: `generate` (line 582) calls `proof_redact.redact_tree(proof)` once
  (line 609), refuses to emit if redaction did not run (line 613), applies length
  caps AFTER redaction (line 619), then hashes and writes proof.json + index.html.

**Template** `autonomy/lib/proof-template.html` (803 lines)
- Self-contained / zero-egress by contract (header comment lines 1-40). Social
  meta lines 46-55: og:title, og:description, twitter:card=summary,
  twitter:description. Line 46: "No og:image (would break self-containment)" -
  preserved by this plan.
- Hero markup lines 261-265 (`.hero`, `#heroLine`, `#heroSub`, `#heroActions`);
  `.hero-actions`/`.btn` CSS at 105-115. `renderHero` (line 426) builds the hero
  client-side. `renderCta` (line 732) emits command blocks with `data-copy`.
- `wireCopy` (line 762) is a delegated click handler keying off `data-copy`,
  using `navigator.clipboard` with `execCommand` fallback. PATTERN to mirror for
  share buttons + copy-link.
- Template reads `deployment.public_url` at line 466 (`renderTier1`).

**CLI** `autonomy/loki`
- `cmd_proof` (line 26277): list/show/open/share. `share` (26393) gates
  `--hosted` to `_loki_hosted_publish_proof` (26113, posts redacted bytes to
  `LOKI_HOSTED_ENDPOINT`, honest "no backend yet" otherwise), else uploads
  index.html AS-IS to a gist via `_loki_gist_upload` (26021, `gh gist create`,
  exposes `LOKI_LAST_GIST_URL`). The default gist path never fills public_url or
  re-renders.
- Social hook is reimplemented inline in Python at 26503-26553 (duplicates
  `_build_social_hook`). This plan adds no third copy.

**Dashboard** `dashboard/server.py`: `_proofs_dir` (7668), `_safe_proof_run_dir`
(7672, realpath-contained), `GET /api/proofs` (7689). Read-only consumer;
untouched.

**Redaction** `autonomy/lib/proof_redact.py`: `redact_tree` (266),
`redact_value` (260), `set_context`/`reset_context` (37/51), `RULES_VERSION =
"1.0"` (29). Reused, never reinvented.

**Existing tests**: `tests/test_proof_html.py` (`test_no_external_resource_refs`
forbids `src=`, `@import`, `url(http`, any non-allowlisted `https?://`; allowlist
only `github.com/asklokesh/loki-mode`; plus null-cost-never-renders-$0.00),
`tests/test_proof_redaction.py` (secret-class + e2e), `tests/cli/test-proof-command.sh`,
`tests/dashboard/test_proofs_routes.py`.

**Dependency posture**: no Pillow/cairo/cairosvg. Local PNG rasterization would
add a dependency and break the zero-dep local posture - ruled out.

## 2. The og:image decision

CHOSEN: keep the local proof zero-egress with NO og:image; add a branded in-page
card rendered from the redacted dict; inject og:image and share-target URLs ONLY
at publish time via the existing `deployment.public_url` slot, on an
HTML-serving host.

- Option (a) data-URI og:image: REJECTED. og:image is spec'd as a URL; scrapers
  do not reliably honor data-URI og:image. Bloat for zero benefit.
- Option (b) separate card file referenced only at publish: PARTIALLY ADOPTED. A
  real og:image needs a public URL, absent at generate time. Reference no image
  at generate; at publish (only if the proof reaches an HTML-serving host
  returning a public URL) that host is where og:image is set.
- Option (c) branded in-page card: ADOPTED as the always-on self-contained
  visual. Inline CSS / inline SVG, rendered client-side from the redacted dict.
  Zero egress, zero new deps, redaction-safe.

HONEST GIST LIMITATION: `loki proof share <id>` publishes to a GitHub Gist. The
gist page serves GitHub's own og tags (profile picture), not the uploaded HTML's
og tags; `raw.githubusercontent.com` serves `text/plain`, not scraped. So the
gist path does NOT produce a rich preview of the proof card. Only an
HTML-serving host returning a real public URL served as `text/html` (the
`LOKI_HOSTED_ENDPOINT` seam) can. Stated plainly in docs + CLI output.

UNIFYING INSIGHT: og:image, share-button `url=`, and copy-link all need a public
URL absent at `file://` generate time. All three key off
`deployment.public_url`: null at generate (share degrades to text-only/hidden),
populated at publish.

## 3. Disjoint dev slices

Binding constraints (every slice): no VERSION bump; no commits; no emojis; no em
or en dashes (ASCII hyphen only); no new runtime dependencies; the default
generate path stays byte-for-byte zero network calls.

**Slice A - Template: branded card + JS-click share buttons**
(`autonomy/lib/proof-template.html` only)
- Style the hero (261-265) into a branded card: what was built (files_changed +
  spec source), verified (council verdict/ratio + gates), cost (cost.usd),
  duration (wall_clock_sec), plus Loki/Autonomi branding. Inline CSS / inline SVG.
  Rendered by `renderHero` from the redacted dict; no new schema fields.
- Add a share row in `.hero-actions` using `data-share` attributes (mirror
  `data-copy`): X/Twitter, LinkedIn, Copy link. NO literal `https://` href in
  static HTML (keeps `test_no_external_resource_refs` green). A new delegated
  handler assembles the intent URL at click time:
  - X: `https://twitter.com/intent/tweet?text=<hook>&url=<public_url>` (hook from
    og:description meta; public_url from parsed proof).
  - LinkedIn: `https://www.linkedin.com/sharing/share-offsite/?url=<public_url>`
    (URL only; LinkedIn ignores prefilled text).
  - Copy link: writes public_url via the clipboard path wireCopy uses.
- When public_url is null (default local case): share buttons hide or fall back
  to copy-the-hook-text; never emit a broken `url=`.
- og:image meta only as empty/absent default; populated by the publish path.

**Slice B - Generator: hook plumbing + public_url passthrough**
(`autonomy/lib/proof-generator.py` only)
- Add `LOKI_PROOF_SHARE_BUTTONS` (default ON; justified below) conditionally
  including the share-row markers. Reuse `_build_social_hook`.
- public_url stays None at generate. Optional `LOKI_PROOF_PUBLIC_URL` env, when
  set, threads into deployment.public_url BEFORE the redaction chokepoint.
  Default None. No og:image here. No network calls.

**Slice C - CLI: publish-time re-render + og:image/public_url injection**
(`autonomy/loki` only; OWNS the re-render)
- Hosted path (`_loki_hosted_publish_proof`, 26113): after the host returns a
  public URL, re-render with deployment.public_url set + og:image set to the
  host-minted card URL, then upload. Re-render reuses the generator (env var) so
  redaction runs again over the URL-injected dict - never a hand-built body.
- Gist path: byte-for-byte unchanged EXCEPT one honest stdout note (gist previews
  show GitHub's og tags, not the proof card; point to --hosted). Do not fabricate
  a public_url for gist.

**Slice D - Docs** (this file + a short note in proof docs / wiki): zero-egress
invariant, opt-in publish path, gist-vs-hosted preview reality, LinkedIn text
limitation.

Recommended parallelism: A, B, D in parallel; C after B's flag contract is
agreed.

## 4. Test plan

Model on `tests/test_proof_html.py`, `tests/test_proof_redaction.py`.

1. Default generate path zero-egress: extend `test_no_external_resource_refs` -
   generate with share buttons ON, assert NO `src=`, no `@import`, no `url(http`,
   no `https?://` URL except the github.com/asklokesh/loki-mode allowlist. The
   load-bearing guard that the JS-click design adds no literal share URLs.
2. Redaction holds on card + share text: reuse the e2e pattern - plant secrets in
   council summaries / spec / diffs, generate, assert none survive anywhere in
   index.html incl. card markup + og:description. Card renders ONLY redacted
   fields.
3. Share button URLs well-formed + prefilled: unit-test the JS URL assembly -
   given public_url, X intent contains URL-encoded hook + url=<public_url>;
   LinkedIn contains url=<public_url> no text; copy-link copies public_url. Null
   public_url -> no malformed url=.
4. og:image populated ONLY on publish: default page has no og:image; simulate
   hosted re-render with a public URL, assert og:image + public_url populated and
   re-redacted; gist path leaves page unchanged + prints the note.
5. Null-cost credibility preserved: card never shows `$0.00` when cost uncollected.

## 5. Honest limits

- Data-URI og:image not reliably scraped (og:image is spec'd as a URL); not used.
- GitHub gist yields no rich preview of the proof card (gist page = GitHub og
  tags; raw = text/plain).
- A rich social preview requires an HTML-serving host returning a public URL (the
  LOKI_HOSTED_ENDPOINT seam or any static HTML host the user configures). No
  official Loki hosted backend yet; the seam is honest about this.
- LinkedIn share ignores prefilled text; it scrapes the destination og tags, so
  LinkedIn previews depend on the hosted og:image. X intent carries the hook.
- Sharing is manual / opt-in: a button is inert until clicked; no auto egress.
- No hosted gallery / discovery feed in this release.

Share-buttons default ON: with JS-click construction there is no literal external
URL in the static page and no network call until the user clicks. The buttons are
inert markup until acted on, so they cannot violate the default zero-egress
invariant. Default ON maximizes the word-of-mouth lever at zero cost to the
posture; `LOKI_PROOF_SHARE_BUTTONS=0` opts out.

## Zero-egress invariant (precise)

- STAYS zero-egress (byte-for-byte zero network calls): generating a proof and
  opening the local index.html. No og:image, no src=, no @import, no web fonts,
  no literal share URLs. Card + hook render from the embedded redacted JSON.
  Share buttons present but inert.
- OPT-IN egress (only on explicit user action): clicking a share button, copying
  the link, and `loki proof share <id>` / `--hosted`. og:image and
  deployment.public_url are populated only on the hosted publish re-render.
