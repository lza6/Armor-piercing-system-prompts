---
name: security-ops
description: >-
  Router for Deep Eye agent skills: pentest, bug-bounty, red-team, blue-team,
  ctf. Use for security ops, mixed intent, offensive vs defensive choice.
---

# Deep Eye — Security Ops Router

Load **one** skill from `.agents/skills/<name>/SKILL.md`.

| Intent | Skill |
|--------|-------|
| Authorized assessment | `pentest` |
| Program hunting | `bug-bounty` |
| Adversary simulation | `red-team` |
| Detection / harden | `blue-team` |
| Lab challenges | `ctf` |

## Always

Authorization for real systems; config via `config/config.yaml` or `python deep_eye.py --setup`.

## Entry

```bash
python deep_eye.py --setup
python deep_eye.py -u URL -v
python deep_eye.py -u URL --scope-nl "..."
```

Docs: `docs/SKILLS.md`, `CONFIGURATION.md`, `MODULES.md`, `AGENTS.md`.
