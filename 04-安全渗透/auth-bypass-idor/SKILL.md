---
name: web-auth-bypass-idor
description: Broken access control - IDOR, privilege escalation, JWT abuse, mass assignment, forced browsing - for web apps and APIs. Use on any multi-user or role-based app with object IDs, tokens, or admin functionality. Triggers - /api/users/<id>, /orders/<id>, ?account=/uid=/doc=, JWT (eyJ...), role/isAdmin field, admin panel, "403 Forbidden", numeric or guessable object references.
tags: [vuln_assess, exploitation]
---

# Broken Access Control (IDOR / privesc / JWT / mass-assignment)

## When this fires
Any app with per-object endpoints, user roles, tokens, or admin functions — the most common (and highest-scoring) web bug class.

## Decide — which sub-class
- Object ID in URL/param/body → **IDOR** (swap it).
- JWT / signed token → **JWT abuse**.
- Object accepts fields you shouldn't set (`role`,`isAdmin`,`is_verified`) → **mass assignment**.
- Admin path returns 403 to you → **forced browsing / vertical privesc**.

## Detect → Exploit → PROVE IMPACT
```
IDOR:  authenticate as user A; hit A's /api/orders/1001 → change to 1002/other IDs (incl. UUIDs seen elsewhere,
       decremented, from another account). Also flip method (GET→PUT/DELETE) and try without auth.
       PROVE: read/modify ANOTHER user's data.
Mass assignment: add {"role":"admin","isAdmin":true,"userId":<victim>} to a profile/update/signup body.
       PROVE: your account gains a privilege the UI never grants.
Forced browsing / method: request admin endpoints directly; try X-Original-URL / X-Rewrite-URL / trailing
       /./ /%2e/ ; verb tampering (HEAD/OPTIONS); 403-bypass headers (X-Forwarded-For 127.0.0.1).
       PROVE: reach an admin function as a low-priv/anon user.
```
JWT (drive with `jwt_analyze`, then forge):
```
alg:none      → header {"alg":"none"}, drop signature, set {"admin":true}/{"role":"admin"}
weak HMAC     → crack the secret (hashcat -m 16500 <jwt> wordlist) → re-sign with the new claims
alg confusion → RS256→HS256 signing the token with the PUBLIC key as the HMAC secret
kid / jku / x5u → path-traversal/SSRF the key source to one you control
```
**Proof required:** access to another user's data, or a forged/tampered token that authenticates you as admin and reaches a privileged action.

## Tooling
`jwt_analyze` for token analysis; Burp/ffuf to sweep object IDs; `nuclei -tags idor,jwt,exposure`.

## False positives / pitfalls
- Server re-checks ownership after the swap (returns 403/empty) → not IDOR; try other IDs/methods.
- Random UUIDs with no leak → IDOR may need an ID-disclosure primitive first.
- "alg:none" rejected by a patched lib → move to weak-secret / confusion / kid.
