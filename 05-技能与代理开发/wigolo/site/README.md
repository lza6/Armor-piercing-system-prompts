# wigolo landing site

Next.js static site for [wigolo](https://github.com/KnockOutEZ/wigolo), deployed to GitHub Pages by `.github/workflows/site.yml` on pushes to `main` that touch `site/`.

```bash
npm ci
npm run dev        # local dev at localhost:3000 (no base path)
npm run build      # static export to out/
```

Environment (set by the Pages workflow; optional locally):

| Var | Purpose |
|---|---|
| `NEXT_PUBLIC_BASE_PATH` | `/wigolo` on GitHub Pages project hosting; unset locally |
| `NEXT_PUBLIC_SITE_URL` | Canonical URL for metadata/OG (`https://knockoutez.github.io/wigolo`) |
| `NEXT_PUBLIC_WEB3FORMS_KEY` | Access key for the quick-feedback form (web3forms.com). Unset → the form hides and only the GitHub links show. |

All `/public` asset references go through `asset()` from `src/lib/site.ts` so they work under the base path. Fonts are open-licensed and self-hosted at build time via `next/font` (Bricolage Grotesque · Instrument Sans · Azeret Mono).
