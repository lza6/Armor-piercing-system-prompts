---
name: reporting-phase
tags: [reporting]
description: Reporting phase — generate structured pentest report from findings. Use when the current phase is REPORTING.
---

# Report Generation Guide

## Finding Format
Each finding should include:

```
### [SEVERITY] Finding Title
- **Target**: IP:port or URL
- **CVSS/Severity**: Critical/High/Medium/Low/Info
- **CVE**: CVE-XXXX-XXXXX (if applicable)
- **ATT&CK**: TXXXX (technique ID)
- **Status**: Confirmed / Exploited

**Description**: What the vulnerability is and why it matters.

**Evidence**: Command run + output proving the vulnerability.

**Impact**: What an attacker can achieve by exploiting this.

**Remediation**: Specific steps to fix the issue.
```

## Report Structure

### Executive Summary
- Engagement scope and dates
- Overall risk rating (Critical/High/Medium/Low)
- Key findings count by severity
- Top 3 most critical issues
- Strategic recommendations (2-3 sentences)

### Methodology
- Phases performed
- Tools used
- Scope boundaries

### Findings (grouped by severity)
1. Critical findings
2. High findings
3. Medium findings
4. Low findings
5. Informational

### Attack Path Narrative
Step-by-step description of successful attack chains:
- Initial access vector
- Privilege escalation path
- Lateral movement
- Data accessed

### Host Summary Table
| Host | OS | Services | Vulns | Access |
|------|----|----------|-------|--------|

### Recommendations
Prioritized remediation list:
1. Immediate (Critical/High) — fix within 48h
2. Short-term (Medium) — fix within 30 days
3. Long-term (Low/Info) — fix within 90 days

## Severity Classification
- **Critical (CVSS 9.0-10.0)**: RCE, auth bypass, default creds on critical systems, full DB dump
- **High (CVSS 7.0-8.9)**: Arbitrary file read/write, privesc, SSRF to internal, stored XSS on admin
- **Medium (CVSS 4.0-6.9)**: Reflected XSS, sensitive info disclosure, missing security headers with impact
- **Low (CVSS 0.1-3.9)**: Version disclosure, minor info leaks, missing non-critical headers
- **Info (CVSS 0.0)**: Open ports, technology detection, informational findings

## Output Formats
Use the `report_gen` tool to generate:
- Markdown report (default)
- JSON export (machine-readable)
