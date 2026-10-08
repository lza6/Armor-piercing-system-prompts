# Report Writing

## Title Formula
`[Vulnerability] in [Component] Enables [Impact]`

Under 15 words. Title Case. Impact-forward. No URLs.

| Bad | Good |
|---|---|
| XSS in search | Stored XSS in Comment Renderer Executes JavaScript in Admin Context |
| IDOR found | IDOR in User API Exposes PII of All Platform Users |
| SQL injection | Blind SQL Injection in Search Filter Enables Full Database Extraction |

## Structure

1. **Summary** (2-3 sentences): What's broken, what attacker can do, who's affected.
2. **Steps to Reproduce**: Numbered. ONE action per step. Exact URL, method, headers, body.
3. **Impact**: What attacker walks away with. How many users. Business impact.
4. **PoC**: Self-contained file. Screenshots at each step. Video if multi-step.
5. **CVSS 4.0**: Full vector string with justification per metric.
6. **Remediation**: 1-2 sentences. Developer-actionable. Specific fix.

## Style Rules
- Human tone, technical but triager-accessible
- Lead with impact, not process
- No padding ("I discovered...", "During my testing...")
- Every sentence adds information
- Never submit without PoC + evidence

## Common Mistakes
- Theoretical bugs ("could allow...")
- Screenshots of Burp instead of clear steps
- CVSS overclaiming
- Same bug class on multiple endpoints as one report (should be separate)
- Missing evidence attachment
