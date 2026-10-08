# Hunting Methodology

## The 5-Phase Non-Linear Workflow

### Phase 1: Understand the Target (before touching anything)
1. Read program scope, policy, safe harbor
2. Read 5+ disclosed reports in hacktivity
3. Map crown jewels: what would hurt the company most?
4. Understand the business domain — what features handle money, PII, auth?

### Phase 2: Map the Surface
1. Subdomain enumeration → live hosts → tech stack detection
2. JS bundle analysis → API endpoints, secrets, internal URLs
3. Run `/surface` for P1/P2/Kill ranking

### Phase 3: Hunt (the actual testing)
1. Pick P1 target from surface ranking
2. Select vuln class based on tech stack:
   - Rails/Django/Laravel → IDOR, mass assignment
   - Express/Node → prototype pollution, path traversal
   - Spring Boot → Actuator, SSTI
   - Next.js → SSRF via Server Actions
   - GraphQL → introspection, IDOR via node(), mutation auth bypass
3. Test with concrete payloads (see /hunt)
4. Apply the Sibling Rule on every endpoint
5. 20-minute rotation if no progress
6. Build a **depth matrix** before declaring a class exhausted:
   - Dimensions: `entrypoint × method × content-type × encoding × bypass`
   - Minimum 30 combinations on P1 surface (seed with `uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py matrix <class>`)
   - Do not stop at the first blocked payload — mutate and continue
7. Run **cross-context variants** for every promising input:
   - URL / query, JSON, form-urlencoded, multipart, GraphQL variables
   - Header / cookie mirrors, reflected values, stored values, async jobs / webhooks
8. Execute **encoding ladders** systematically:
   - raw → URL → double-URL → unicode escape → mixed-case / separator insertion
   - Keep the semantic payload constant through each ladder step
   - Then **stack encodings** in a single payload: `html-entity+URL`
     (`%26lt%3Bscript%26gt%3B`), `URL+html-entity`, `unicode-escape+URL`,
     `base64+URL`. WAFs typically decode once; targets decode twice, so a
     payload that looks benign after a single decode still executes at the
     sink.
9. Execute **auth-state permutations**:
   - unauthenticated, low-priv user A, low-priv user B, high-priv, expired token, stale session, cross-tenant
   - Compare response deltas (status, length, timing), not only status codes
10. Treat every bypass as a **family**, not a one-off:
    - For WAF / filter blocks, try separator insertion, case toggling, alternate delimiters, parser differentials, and protocol / host normalization tricks
11. **Log negative evidence** (what failed and why) via `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> recon "coverage-<class>" "<details>"` so autopilot resume avoids repeating exhausted paths.

### Phase 4: Validate + Chain
1. Run 7-Question Gate on any signal
2. If PASS → check A→B chain table
3. If CHAIN REQUIRED → build the chain or drop it
4. If KILL → move on immediately

### Phase 5: Report + Submit
1. Quality check (score ≥ 7)
2. Dupcheck against hacktivity
3. Submit with PoC + evidence + CVSS 4.0

## Wide vs Deep Route Selection

**Wide route** (recon-heavy): New target, unknown surface, no prior data.
- Run `/pipeline` for broad coverage first
- Then `/surface` to prioritize

**Deep route** (hunt-heavy): Known target, mapped surface, returning hunter.
- Run `/resume` to see what's untested
- Pick the highest-ROI untested endpoint
- Go deep on one vuln class

## Developer Psychology

Developers make CLASS mistakes, not random ones:
- If they forgot auth on endpoint A, they probably forgot on B and C
- If they use sequential integer IDs anywhere, they use them everywhere
- If input validation is weak in one form, check ALL forms
- New features (< 30 days) have the weakest security
- Acquired companies (different code, different team) = fresh attack surface

## Time Management

| Rule | Action |
|---|---|
| 5-minute rule | No interesting signals after 5 min → skip target |
| 20-minute rotation | No progress in 20 min → rotate vuln class or endpoint |
| 1-hour rule | Stuck on one target for 1 hour → switch programs entirely |
| A→B time box | 20 min per B candidate, max 3 candidates |
| Exhaustion rule | A class is "exhausted" only after the depth matrix baseline + sibling coverage (see Phase 3 steps 6-11) |

## ROI Ranking by Bug Class

| Bug Class | Competition | Avg Payout | Verdict |
|---|---|---|---|
| IDOR | Medium | High | Best ROI — always test first |
| Auth bypass | Medium | High | Second priority |
| Business logic | Low | High | Unique to each target |
| Race conditions | Low | Medium-High | Under-tested |
| OAuth/OIDC chains | Low | High | Complex but high payoff |
| SSRF → cloud | Medium | Very High | If you find DNS callback |
| Cache poisoning | Low | High | Rare skill |
| XSS | Very High | Medium | Skip unless you have a chain |
| Open redirect | Very High | Low | Only with OAuth chain |
| Missing headers | Infinite | $0 | Never submit |
