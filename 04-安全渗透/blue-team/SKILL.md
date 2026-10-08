---
name: blue-team
description: >-
  Blue team defense using Deep Eye outputs for detection engineering, IR
  content, and hardening. Use for blue team, SOC, SIEM, detection engineering,
  threat hunting, IR triage, hardening, /blue-team.
---

# Deep Eye — Blue Team Skill

Deep Eye = controlled attack corpus for detection and control validation.

## Generate corpus

```bash
python deep_eye.py -u https://STAGING -v --formats json,sarif
```

Useful noisy checks: `sql_injection`, `xss`, `ssrf`, `ssrf_cloud`, `log4shell`, `lfi`, `crlf_injection`, smuggling modules.

## Detection loop

1. Take High finding (`payload`, `url`, `type`)
2. Write SIEM/WAF rule
3. Replay scan / single request
4. Measure FPs
5. Document owner

## Control validation

| Finding | Control |
|---------|---------|
| IDOR/BOLA | Object-level authz |
| JWT | Alg lockdown, signature verify |
| SSRF | Egress / metadata block |
| XSS | CSP + encoding |
| Secrets | Scanner + CI secret scan |

## Retest

```bash
python deep_eye.py -u URL --retest-new reports/prior.json
```

## Rules

Do not disable prod controls only to silence scans; coordinate SOC windows.
