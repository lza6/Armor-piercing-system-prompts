---
name: red-team
description: >-
  Red team / adversary simulation with Deep Eye for web/API footholds. Use for
  red team, adversary simulation, ATT&CK mapping, kill chain, /red-team.
  Authorized engagements only.
---

# Deep Eye — Red Team Skill

Goal-driven simulation. Deep Eye = web/API recon and foothold sensor.

## Preconditions

RoE (objectives, crown jewels, no-go, detection expectations), legal auth, OPSEC plan.

## Kill chain (web-heavy)

```
Recon → Initial access (app) → Session/token abuse → Lateral (SSRF/cloud) → Objective
```

| Phase | Deep Eye |
|-------|----------|
| Recon | `enable_recon`, OpenAPI, crawl |
| Access | Core inject, `file_upload` |
| Authz | `idor`, `api_bola_deep`, `jwt_deep` |
| Pivot | `ssrf_cloud`, `cloud_misconfig` |
| Mobile | `mobile.enabled` + Frida/static modules |

```bash
python deep_eye.py -u https://APP --scope-nl "SCOPE" -v --formats json
```

## Purple handoff

| TTP | Deep Eye evidence | Telemetry expected | Gap | Fix |
|-----|-------------------|--------------------|-----|-----|

## Rules

Stay in RoE; no third-party pivots; no destructive malware; log for debrief.
