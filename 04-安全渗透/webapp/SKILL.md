---
name: playbook-webapp
description: Web application pentest methodology + ROUTER to the per-vuln-class web skills. Load at the START of systematic web testing to get the phase flow (recon → map → test-by-OWASP-class → prove → report) and pick which web-<class> skill to load for each surface. Use on any web app / HTTP API engagement.
tags: [vuln_assess, exploitation]
---

# Web App Pentest — methodology + skill router

This is the ORDER of operations and the map to the deep skills. Each vuln CLASS has a dedicated `web-<class>` skill with exact payloads + detect→exploit→PROVE — load the one matching the surface; don't test from this index.

## 1. Recon + active scan FIRST
Fingerprint stack+version (whatweb/httpx, headers), CMS (wpscan/etc.), APIs (/swagger, /graphql, /.well-known). Then a content-matched `nuclei -u <t>` pass BEFORE exhaustive dir-fuzz (framework-runtime RCEs don't appear as routes) → `nuclei_parse`. Version + a known CVE → fetch & vet a public PoC (shared exploitation methodology).

## 2. Map (hidden surface, AFTER the scan)
`ffuf`/`gobuster` (→ `gobuster_parse`), vhosts (`ffuf -H Host:FUZZ`), params (`arjun`), auth endpoints (login/register/reset/OAuth), API routes. Review client-side JS for endpoints/secrets.

## 3. Test by class → load the matching skill
| Signal / surface | Load |
|---|---|
| param → DB query, SQL error/differential | **web-sqli** |
| input renders in a server template ({{7*7}}) | **web-ssti** |
| server fetches a URL you control (webhook/proxy/import) | **web-ssrf** |
| file param / download / `../` reflection | **web-lfi-traversal** |
| file upload sink | **web-upload-rce** |
| XML/SOAP/SAML/DOCX/SVG parsing | **web-xxe** |
| serialized blob (rO0/O:/VIEWSTATE/pickle) | **web-deserialization** |
| object IDs, roles, JWT, admin funcs | **web-auth-bypass-idor** |
| known framework+version CVE | shared exploitation methodology + searchsploit/nuclei |
Also-check (no dedicated skill yet): XSS (`dalfox`, `xss_detect`), command injection (`;id`/`$(id)`), CSRF, CORS/security-headers, crypto/secrets-in-JS, business-logic/race conditions.

## 4. PROVE + Report
A finding is `suspected` until you reproduce concrete impact (dumped canary row / `id` / file bytes / cloud creds / cross-user data) — then `add_vuln` `confirmed` with the evidence. Never mark a host resolved/"safe" without a completed active scan. Report: reproduction steps + request/response evidence + CVSS + OWASP-WSTG mapping.
