# watch a changelog (+ webhook delivery)

The `watch` tool registers change-watch jobs: wigolo snapshots a page, hashes
its extracted content, and reports a change the next time the content hash
moves — inline, or POSTed to a webhook.

## Run it

```bash
./watch.sh                                  # watches https://nodejs.org/en/blog
URL=https://your-site/changelog ./watch.sh  # or any changelog you care about
```

Requires node >= 20 and `jq`.

## What you'll see (real output)

```text
== 1. register a watch on https://nodejs.org/en/blog ==
job id: 74bec5cfdc2cb8bc36a728e46ede89a1b7e7b2a731158771e47674ee6358edfd

== 2. list registered jobs ==
74bec5cfdc2c…  active  every 3600s  https://nodejs.org/en/blog

== 3. on-demand check (first check records the baseline) ==
{
  "url": "https://nodejs.org/en/blog",
  "changed": false,
  "current_hash": "3e600c4fc579"
}

== 4. what a change looks like: the diff engine's report ==
--- old
+++ new
@@ -1,3 +1,4 @@
-## v2.4.0
+## v2.5.0
 - Added container queries guide
+- Added subgrid examples
 - Fixed typo in grid docs

== 5. clean up ==
{"removed_jobs":1}
```

The first check records a baseline (`changed: false` + the content hash).
Once a check sees a different hash, the report flips to `changed: true` with
`previous_hash` / `current_hash` — and for the full picture of *what* changed,
the `diff` tool renders unified hunks, per-section changes, or a summary
(step 4 shows it on two inline changelog versions; `wigolo diff <url>`
compares a page against its cached copy).

## Scheduling

A one-shot `watch run <id>` checks on demand. **Scheduled** checks fire while
a long-lived wigolo is running — `wigolo serve` or an active MCP session —
at each job's `--interval`. Register jobs once; any long-lived wigolo on the
same machine picks them up.

## Webhook delivery

Pass `--notify` at registration and every detected change is POSTed to your
endpoint as JSON:

```bash
npx wigolo watch add https://nodejs.org/en/blog --interval 3600 \
  --notify https://hooks.example.com/wigolo
```

```json
{ "job_id": "74bec5cf…", "url": "https://nodejs.org/en/blog",
  "changed": true, "previous_hash": "3e600c…", "current_hash": "9f41aa…",
  "diff_summary": "…" }
```

A receiver is a few lines of node (run it wherever your automation lives):

```js
// receiver.mjs — node receiver.mjs
import http from 'node:http';
http.createServer((req, res) => {
  let body = '';
  req.on('data', (c) => (body += c));
  req.on('end', () => { console.log('change:', body); res.end('ok'); });
}).listen(8787);
```

**Security note:** webhook targets (and watched URLs) must be public http(s)
endpoints. Loopback, RFC 1918, link-local, and cloud-metadata addresses are
refused by wigolo's SSRF guard — by design, so a watch job can never be used
to probe your internal network. Give the receiver a public HTTPS ingress (or
whatever tunnel your stack already trusts) and keep redirects in mind: a
redirecting webhook target is treated as a delivery failure rather than
followed blindly.

## Related

- Rich page-version diffs: `npx wigolo diff --help`
- The same `watch` tool over REST / MCP for automation platforms:
  [n8n-remote-mcp](../n8n-remote-mcp/)
