# Troubleshooting

First stop, always:

```bash
wigolo doctor        # names the broken component and the env var / command that fixes it
wigolo doctor --fix  # repairs the known failure classes automatically
```

## Symptom → fix

| Symptom | Fix |
| --- | --- |
| A component failed to download during `init` | `wigolo warmup --all` re-runs the downloads (or `--browser` / `--reranker` / `--embeddings` for just one). Failures don't block the rest of wigolo — components lazy-retry on first use. |
| Browser engine won't launch on Linux | `wigolo warmup --browser` installs the OS system libraries the browser engine needs (escalating with sudo where required); when it can't, the error prints the exact install command to run yourself, then re-run `wigolo warmup`. |
| `wigolo serve` exits: port in use | The daemon deliberately does not auto-rebind. The error names a free port to retry with, e.g. `wigolo serve --port 3334`. |
| `wigolo serve` refuses to start on a non-loopback host | Working as designed (fail-closed). Set `WIGOLO_API_TOKEN` / `WIGOLO_API_TOKEN_FILE`, or explicitly pass `--allow-unauthenticated`. See [self-hosting](./self-hosting.md#binding-beyond-loopback). |
| Fetch result says `blocked_by_challenge` | See [below](#blocked_by_challenge). |
| Search results feel thin / an engine seems dead | Degraded engines are *reported*, not hidden — check `engine_warnings`, `engine_telemetry`, and `engine_pool` in the response, and `wigolo doctor`'s per-engine table (it names the env var when an engine just wants a key, e.g. `WIGOLO_GITHUB_TOKEN`, `BRAVE_API_KEY`). |
| Results are stale | Pass `force_refresh: true` (news, prices, changelogs), or clear scoped entries: `wigolo cache clear --url-pattern="*example.com*"`. Lifetimes are tunable: `CACHE_TTL_SEARCH`, `CACHE_TTL_CONTENT`. |
| Everything fails behind a corporate proxy | Set `USE_PROXY=true` and `PROXY_URL` (credentials go to the OS keychain, not disk). See [configuration](./configuration.md#fetch-and-browser-engine). |
| A domain that used to work is misbehaving | wigolo learns per-domain fetch routing; a site redesign can invalidate what it learned. `wigolo tune show <domain>` to inspect, `wigolo tune reset <domain>` to relearn. |
| A watch job never fires | Watch checks run only while a daemon (`wigolo serve`) or MCP session is alive — a one-shot CLI call registers jobs but can't schedule them. |
| Browser engine download is slow or times out | Common on throttled or region-restricted links. Re-run `wigolo warmup --browser` — it retries and resumes. If the default download CDN is slow where you are, point the browser engine at a mirror: set `PLAYWRIGHT_DOWNLOAD_HOST=<mirror-url>` before warmup. |
| Embeddings model download fails (`TAR_BAD_ARCHIVE` / "unrecognized archive") | A truncated or corrupt download. wigolo now auto-clears the partial file and re-downloads once; if it still fails, `wigolo config --cleanup` and re-run `wigolo warmup --embeddings`. |
| Ranking model download fails (`fetch failed`) | A transient network blip. Re-run `wigolo warmup --reranker` — it retries with backoff. |
| A download fails with `self signed certificate in certificate chain` | You're behind a TLS-inspecting (corporate) proxy. Point Node at your organization's CA bundle — `NODE_EXTRA_CA_CERTS=/path/to/corp-ca.pem` — then re-run warmup. |
| `npm install` fails compiling a native dependency (often on Windows) | Your Node version has no prebuilt binary, so npm falls back to a source build. Use a supported LTS — **Node 20, 22, or 24** — where prebuilts exist, or install a C/C++ toolchain (Visual Studio Build Tools on Windows). |
| Downloads stall or fail on low disk | Components need ~1 GB free. Free space, point `WIGOLO_DATA_DIR` at a larger volume, or `wigolo config --cleanup` to reclaim a previous install. |

## A component failed during setup — is wigolo broken?

No. `init` exits 0 even when a download fails, and the **core** (search, HTTP fetch, crawl, extract, cache) needs no models and no browser at all. Failed components degrade gracefully — here is exactly what each one costs:

| If this failed… | What you lose | What still works |
| --- | --- | --- |
| Browser engine | JS-rendered pages fall back to plain HTTP fetch (some single-page-app content may be missing) | search, HTTP fetch, crawl, extract, cache, models |
| Embeddings model | semantic discovery — `find_similar` and semantic cache ranking fall back to keyword matching | search, fetch, crawl, extract, keyword cache |
| Ranking model | the ML re-rank pass (multi-engine rank fusion still applies) | everything else — results are just less finely ordered |
| No LLM key | `research` / `agent` / `search --format answer` return structured evidence instead of written prose | every keyless tool, which is the default |

Re-run `wigolo warmup --all` any time to retry the downloads, or just let each component lazy-load on first use.

## blocked_by_challenge

This label means the target sits behind an anti-bot challenge that did not clear within the challenge window. wigolo escalates through its fetch tiers (plain HTTP → TLS-impersonation tier → full browser engine), polls the challenge like a patient browser, and reuses previously solved clearances per domain — and when none of that works, it tells you so instead of returning the challenge page dressed up as content.

Two honest facts to calibrate expectations:

- **IP reputation is scored.** From datacenter IPs (VPS, CI, cloud), some challenge-protected sites will not clear even though the identical request works from a residential connection. That's a property of where you're running, not a knob wigolo forgot.
- **The opt-in lever is a proxy** whose IP reputation matches your legitimate-research use — see [self-hosting](./self-hosting.md#the-datacenter-ip-reality). Credentials are keychain-stored, and politeness (robots.txt, per-domain rate limits) still applies.

## Platform notes

**Node version.** wigolo runs on **Node 20, 22, or 24** (LTS). Very new or unusual Node builds may not have prebuilt native binaries yet and will try to compile from source (which needs a C/C++ toolchain) — stick to an LTS to avoid that.

**Windows.** Supported on Node 20+. The data dir is `%USERPROFILE%\.wigolo`. Set env vars with your shell's syntax (`$env:WIGOLO_SEARCH="hybrid"` in PowerShell); everything else — commands, flags, ports — is identical to the Unix docs.

**Linux (minimal images / containers).** The browser engine needs a handful of OS libraries; `wigolo warmup --browser` installs them (with sudo where available) and otherwise prints the exact command to run. **Python is _not_ required** — it is used only by the optional search-engine sidecar, so a "Python 3 not found" note is safe to ignore for core use.

**macOS (Apple Silicon / Intel).** Fully supported — models and browser included.

**Linux on ARM (arm64).** Core search, fetch, crawl, extract, and cache work normally. **Semantic features are currently unavailable on linux-arm64** — the embeddings model's tokenizer has no prebuilt ARM binary yet, so `find_similar`, embeddings, and semantic cache ranking fall back to keyword matching. If you need semantic features on Linux today, run on an x64 host; this is tracked for a future release.

## Slow, proxied, or offline networks

- **Slow or region-restricted link.** The model and browser downloads are the slow part, and they resume on re-run. `wigolo init --no-warmup` skips all downloads up front — each component then lazy-loads on first use. For a throttled browser-engine CDN, set a `PLAYWRIGHT_DOWNLOAD_HOST` mirror before warmup.
- **Corporate proxy.** Set `USE_PROXY=true` and `PROXY_URL` (credentials go to the OS keychain, not disk). Behind a TLS-inspecting proxy, also set `NODE_EXTRA_CA_CERTS` to your CA bundle so downloads validate.
- **Air-gapped / offline.** Run `wigolo warmup --all` on a connected machine, then copy its `~/.wigolo` to the target to seed the models and browser. Note that search and fetch still reach the live web at query time — only the model/browser _downloads_ can be pre-seeded.

## Where logs live

wigolo writes **all** logs to stderr (structured JSON by default; `LOG_FORMAT=text` for human eyes, `LOG_LEVEL=debug` to turn up detail). There is no hidden log directory:

- CLI runs: logs appear in your terminal's stderr; redirect with `2>wigolo.log`.
- MCP hosts: the host captures server stderr into its own MCP log location.
- `wigolo serve` under systemd/Docker: journal / container logs.
- The only file wigolo itself writes events to is the opt-in telemetry NDJSON (`~/.wigolo/telemetry/`), and only when `WIGOLO_TELEMETRY=1`.

## FAQ

**What's the business model? Will this start charging me?**
wigolo is free, open-source software under AGPL-3.0. There's no hosted tier, no metered API, no key to buy — it's local software; your machine does the work. That's the point of it.

**What does AGPL mean for me, plainly?**
Using wigolo as a tool — personally, in your company, wired into every agent you run — carries zero obligation. The license's share-alike clause applies only if you *modify wigolo itself and run the modified version as a network service for others*: then you share those modifications. Building products that merely call wigolo is not that.

**Is scraping with this ethical?**
wigolo defaults are built around being a polite client: robots.txt respected by default, per-domain rate limits and crawl delays, page budgets sized for research rather than bulk harvesting, and honest labeled failures instead of hammering at walls. Reliability work here means reading pages the way a real browser does — it is not a cloaking toolkit, and the docs won't teach you to build one.

**How stable is this?**
Public beta at 0.2.0. The documented surface is held to a test suite of roughly 7,600 automated tests; beta is about the polish bar and API-shape confidence, not known instability. Real limitations that exist are written down here rather than discovered in production — see the challenge ceiling above.

**Why is the install so big?**
Because the intelligence is local. The download is dominated by the on-device embedding + ranking models (~250 MB) and the optional browser engine binary (~0.5–1 GB) that JS-rendered fetching needs. That's the trade for keyless, private, no-per-call-cost operation. `init --no-warmup` defers all of it; `wigolo config --cleanup` reclaims it.

[← Docs index](./README.md) · [Next: Privacy & security](./privacy-security.md)
