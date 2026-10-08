---
name: analyze
description: "Analyze recon output with AI to suggest high-value targets and attack strategies. Usage: /analyze <target>"
disable-model-invocation: false
---
AI-powered analysis of recon data for: $ARGUMENTS

## Process
1. Read all recon data: `ls recon/` and read key files
2. Read brain data: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py brief $ARGUMENTS`
3. Read tech stack intel: `uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py suggest <detected-stack>`
4. Read hacktivity patterns: `uv run python3 $CLAUDE_PROJECT_DIR/tools/intel_engine.py analyze`

## Analysis Tasks (do all of these)

### Crown Jewel Mapping
What's the most valuable thing an attacker could access on this target?
- Financial data? → hunt IDOR on payment/billing endpoints
- User PII? → hunt IDOR on profile/export endpoints
- Admin access? → hunt auth bypass on admin endpoints
- Infrastructure? → hunt SSRF → cloud metadata

### Attack Path Ranking
Given the tech stack and recon output, rank the top 5 attack paths by:
1. Likelihood of vulnerability existing (based on tech stack patterns)
2. Impact if exploited (based on endpoint function)
3. Competition (based on hacktivity — avoid heavily-reported vuln classes)
4. Your past success (from brain patterns)

### Blind Spot Detection
What has NOT been tested? What endpoints have no brain data?
Cross-reference recon output against brain tested endpoints.
Flag untested high-value endpoints.

### Output
```
ANALYSIS: target.com
═════════════════════

Crown Jewels: [what's most valuable]

Top 5 Attack Paths:
1. [endpoint] × [vuln class] — likelihood: HIGH, impact: CRITICAL
2. ...

Blind Spots (untested P1 surface):
- /api/v2/payments/* — NO DATA in brain
- /api/v2/admin/* — NO DATA in brain

Recommendation: /hunt target.com --vuln-class [best bet]
```

## Top-Tier Operator Addendum

Treat `/analyze` as a thesis generator, not a summary command. The output must make the next hour of hunting obvious.

1. Build a weighted table before recommending anything:
   - `asset_value`: revenue, PII, admin, secrets, infrastructure, tenant boundary
   - `exploit_likelihood`: stack age, exposed methods, auth complexity, parser surface, prior bug class fit
   - `novelty`: low hacktivity overlap, new endpoint, changed JS, unusual integration, weak vendor pattern
   - `proof_path`: exact request needed to prove impact, required accounts, required evidence artifact
   - `policy_friction`: rate limits, forbidden data access, third-party scope, credential validation rules
2. Prefer attack paths with a short proof path over impressive theory. A boring IDOR with two accounts and a readback beats a speculative SSRF with no egress signal.
3. Include negative evidence. If `/api/admin/*` looks valuable but all routes are 403 with no differential, say that and explain what would change the ranking.
4. Separate `P1 now`, `P2 if time`, and `Kill for this session`. Top-tier analysis saves time by deleting tempting dead ends.
5. Every recommendation must name the next command and the exact first test: `/hunt target --vuln-class idor` plus the endpoint pair, account pair, and field to compare.
