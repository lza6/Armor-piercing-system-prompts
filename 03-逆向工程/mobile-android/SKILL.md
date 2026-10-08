---
name: svc-mobile-android
description: Android APK static analysis — OWASP Mobile Top 10, Retrofit API audit, transport security, smali reading, component export, auth flow analysis. Use when target is an APK/Android app. Triggers - APK, Android, mobile app, decompiled, smali, jadx, apktool.
---

# Android APK Static Analysis — Full Methodology

Load this skill when the target is an Android APK (decompiled or not). This covers SAST — for dynamic testing (Frida, MITM), combine with `playbook-webapp`.

## Task Decomposition — MANDATORY

An APK audit is NOT one grep job. Decompose into these independent tasks (run in parallel):

1. **API Security Audit** — read ALL Retrofit/API interface files, find sensitive data in URLs
2. **Transport Security** — network_security_config, certificate pinning, cleartext
3. **Auth Flow Analysis** — full auth chain, session management, token handling
4. **Component Security** — exported activities/receivers/providers/services, intent filters, deep links
5. **Build Hygiene** — debug flags, dev tools in production, staging endpoints, logging
6. **Secrets & Crypto** — hardcoded keys (but FILTER non-issues — see Known Non-Vulns below)
7. **Data Storage** — SharedPreferences, Room/SQLite, file storage, backup flags
8. **Root/Tamper Detection** — what's present AND what's MISSING

## How to Read Decompiled Code

### Smali basics (you'll see this from apktool/baksmali)
```
.method public someMethod(Ljava/lang/String;)V   # method signature
invoke-virtual {v0}, Lcom/example/Foo;->bar()V    # method call
const-string v1, "hardcoded_value"                # string constant
sget-object v0, Lcom/example/Config;->API_KEY:Ljava/lang/String;  # static field
```

### Retrofit annotations in smali (THE most important pattern)
```smali
# @GET("endpoint") — HTTP method + path
.annotation runtime Lretrofit2/http/GET;
    value = "auth/password"
.end annotation

# @Query("password") — parameter goes in URL query string!
.annotation runtime Lretrofit2/http/Query;
    value = "password"
.end annotation

# @QueryMap — ALL params go in URL query string!
.annotation runtime Lretrofit2/http/QueryMap;
.end annotation

# @Body — parameter goes in request body (SAFE)
.annotation runtime Lretrofit2/http/Body;
.end annotation

# @Field — parameter goes in form body (SAFE)
.annotation runtime Lretrofit2/http/Field;
    value = "password"
.end annotation

# @Header — parameter sent as HTTP header
.annotation runtime Lretrofit2/http/Header;
    value = "Authorization"
.end annotation
```

**CRITICAL CHECK**: Any `@Query`/`@QueryMap` on a POST/PUT that carries sensitive data (password, PIN, OTP, token) = **CRITICAL** finding. Sensitive data in URL is logged everywhere (server access logs, proxies, CDNs, browser history, HTTP interceptors).

### Jadx output (Java-like, easier to read)
```java
@POST("auth/password")
Call<AuthResponse> login(@QueryMap Map<String, String> params);
// ^ This means login credentials go in URL query string = CRITICAL
```

## Checklist: OWASP Mobile Top 10

### M1: Improper Platform Usage
- [ ] Exported components without permission protection
- [ ] Deep links with sensitive actions (password reset, transfers)
- [ ] Content providers with `grant-uri-permissions` or no read/write permission
- [ ] Intent filters on activities that handle sensitive data
- [ ] `android:debuggable="true"` in manifest
- [ ] `android:allowBackup="true"` (data extractable via ADB)
- [ ] `android:usesCleartextTraffic="true"`
```bash
grep -r 'exported="true"' resources/AndroidManifest.xml
grep -r 'android:debuggable' resources/AndroidManifest.xml
grep -r 'allowBackup' resources/AndroidManifest.xml
```

### M2: Insecure Data Storage
- [ ] Sensitive data in SharedPreferences (tokens, passwords, PII)
- [ ] Unencrypted SQLite/Room databases with sensitive data
- [ ] Sensitive data in external storage
- [ ] Logging of sensitive data (Log.d/Log.i with tokens/credentials)
```bash
# SharedPreferences usage
grep -rn "SharedPreferences\|getSharedPreferences\|PreferenceManager" sources/
# Logging sensitive data
grep -rn "Log\.\(d\|i\|v\|w\|e\)" sources/ | grep -i "token\|password\|secret\|key\|auth"
# External storage
grep -rn "getExternalStorage\|EXTERNAL_STORAGE\|Environment.getExternalStorageDirectory" sources/
```

### M3: Insecure Communication
- [ ] **Certificate pinning** — check network_security_config.xml for `<pin-set>` elements
- [ ] **Cleartext traffic** — `cleartextTrafficPermitted` should be false
- [ ] **Custom TrustManager** — accepting all certs (X509TrustManager with empty checkServerTrusted)
- [ ] **HostnameVerifier** — custom verifier that always returns true
- [ ] **WebView** — `setMixedContentMode(MIXED_CONTENT_ALWAYS_ALLOW)`
```bash
# Certificate pinning
grep -r "pin-set\|CertificatePinner\|sha256/" resources/ sources/
# Trust all certs
grep -rn "X509TrustManager\|checkServerTrusted\|TrustAllCerts\|ALLOW_ALL" sources/
# Hostname verifier bypass
grep -rn "HostnameVerifier\|ALLOW_ALL_HOSTNAME_VERIFIER\|verify.*return true" sources/
```

### M4: Insecure Authentication
- [ ] Auth credentials in URL query params (`@Query`/`@QueryMap` on auth endpoints)
- [ ] Biometric auth without server-side validation (local-only bypass)
- [ ] Hardcoded credentials (admin/password patterns in code)
- [ ] Token storage (where is the auth token kept? SharedPrefs? Encrypted?)
- [ ] Session timeout — is there an expiry? Auto-logout?
- [ ] PIN/pattern stored locally (hash vs plaintext)
```bash
# Find all API interface files (Retrofit)
find sources/ -name "*Api*" -o -name "*Service*" -o -name "*Endpoint*" | grep -i "\.java$\|\.smali$"
# Query params on auth
grep -A5 "@Query\|@QueryMap" sources/ | grep -B2 -i "password\|pin\|otp\|sms\|token\|secret"
```

### M5: Insufficient Cryptography
- [ ] Hardcoded encryption keys/IVs
- [ ] ECB mode (patterns leak through encryption)
- [ ] MD5/SHA1 for security-critical operations
- [ ] Custom crypto implementations
- [ ] Weak key derivation (no PBKDF2/bcrypt/scrypt)
```bash
grep -rn "AES/ECB\|DES\|RC4\|Cipher.getInstance" sources/
grep -rn "MessageDigest.getInstance.*MD5\|MessageDigest.getInstance.*SHA-1" sources/
grep -rn "SecretKeySpec\|IvParameterSpec" sources/ | grep -i "hardcoded\|static\|final"
```

### M6: Insecure Authorization
- [ ] Client-side authorization checks only (role checks in app, not server)
- [ ] IDOR — predictable/sequential IDs in API calls (transfer confirm by ID)
- [ ] Missing server-side validation on state transitions
```bash
# Find operations that use simple IDs
grep -rn '@Query("id")\|@Path("id")' sources/
```

### M7: Client Code Quality
- [ ] Debug code in production (WebView debugging, Chucker, Stetho, LeakCanary, Flipper)
- [ ] Development/staging endpoints in production code
- [ ] Verbose error messages exposing internal details
- [ ] Test accounts or bypass codes
```bash
# Debug tools in production
grep -rn "setWebContentsDebuggingEnabled\|ChuckerInterceptor\|Stetho\|LeakCanary\|Flipper" sources/
# Dev/staging URLs
grep -rn "staging\|\.dev/\|localhost\|10\.0\.\|192\.168\.\|debug" sources/ resources/
```

### M8: Code Tampering
- [ ] Root/jailbreak detection (RootBeer, custom checks for `su`, Superuser.apk)
- [ ] Frida/Xposed detection
- [ ] Emulator detection (how robust? Package name checks only = weak)
- [ ] Integrity verification (SafetyNet/Play Integrity, PairIP)
- [ ] Debugger detection (anti-debug techniques)
```bash
# Root detection
grep -rn "RootBeer\|isRooted\|/system/app/Superuser\|/system/xbin/su\|test-keys" sources/
# Frida detection
grep -rn "frida\|xposed\|substrate\|Magisk" sources/
# Emulator detection
grep -rn "generic\|goldfish\|sdk_gphone\|emulator\|genymotion\|bluestacks" sources/
```

### M9: Reverse Engineering
- [ ] Obfuscation level (ProGuard/R8/DexGuard)
- [ ] Native code protection (if any)
- [ ] String encryption

### M10: Extraneous Functionality
- [ ] Admin/debug endpoints accessible from client
- [ ] Hidden features triggered by config flags
- [ ] Logging/analytics sending sensitive data

## Build Artifact Forensics

### Dagger/Hilt DI Graph
Read the DI component files to find what's wired into the app:
```bash
# Find DI modules
find sources/ -name "*Module*" -o -name "*Component*" | grep -i "dagger\|hilt\|di\|inject"
# Look for interceptors in OkHttp chain
grep -rn "Interceptor\|addInterceptor\|addNetworkInterceptor" sources/
```
Chucker/Stetho/Flipper interceptors in the OkHttp chain = HIGH (logs all HTTP traffic).

### Network Configuration Analysis
```bash
# Full network security config
cat resources/res/xml/network_security_config.xml
# Check for domain-config entries (dev/staging domains?)
grep -r "domain-config\|includeSubdomains\|cleartextTrafficPermitted" resources/
```

### Permission Audit
Map each permission to its justification. Flag excessive:
- `READ_CONTACTS` / `READ_CALL_LOG` — legitimate for banking?
- `QUERY_ALL_PACKAGES` — anti-emulator, but privacy concern
- `RECORD_AUDIO` — voice banking or unnecessary?
- `ACCESS_FINE_LOCATION` — branch locator or tracking?

## API Surface Mapping — MANDATORY for Banking Apps

Read ALL Retrofit API interface files and build a complete endpoint map:
```bash
# Find all API interfaces
find sources/ -name "*Api*" | grep -v "test\|mock\|fake"
# Extract HTTP methods and paths
grep -rn "@GET\|@POST\|@PUT\|@PATCH\|@DELETE\|@HTTP" sources/
```

For each endpoint, note:
- HTTP method + path
- How params are passed (`@Query` = URL, `@Body` = body, `@Field` = form)
- Auth mechanism (header token? cookie?)
- Sensitive data exposure

Endpoints to focus on: auth, transfers, payments, password management, profile changes.

## Known Non-Vulnerabilities (FALSE POSITIVES — do NOT report as findings)

These are NOT findings — downgrade to INFO or skip:
- **Firebase/Google API keys** in resources — required client-side, public by design
- **Google Maps API key** — public, restricted by package name
- **reCAPTCHA site key** — public by design (appears in HTML on every reCAPTCHA page)
- **GCM/FCM sender ID, OneSignal app ID** — public identifiers
- **RSA/EC public keys** for pinning/license/JWT verification — public by definition
- **Package name, version code, build config** — not secrets
- **Obfuscated class names** — obfuscation is defense, not a vuln

These BECOME findings only when:
- API key grants server-side write access (Firebase Storage write, Firestore admin) — TEST IT
- OneSignal REST API is unprotected (attempt notification send)
- Firebase DB/Storage has open rules (check `/.json` and storage URL)

## Impact Chaining — Connect Findings

After individual checks, chain findings for combined impact:
- No pinning + sensitive data in URL = MITM intercepts passwords in plaintext
- Chucker + passwords in URL = passwords logged in app's local DB
- No root detection + local token storage = trivial token extraction on rooted device
- Exported activity + deep link = intent hijacking to sensitive screens
- IDOR + valid session = unauthorized access to other users' data

## Recording Findings

For each finding, call `state_update add_vuln` with:
- `severity`: use the chained impact, not individual
- `evidence`: exact file:line or smali path + the relevant code/annotation
- `status`: "confirmed" for code-level findings (they're deterministic, no FP risk)
- `description`: what it is + WHY it matters + what an attacker can do

For absence findings (no pinning, no root detection): still record as vuln — the absence IS the finding. Evidence = "searched for X in all sources/resources, not found."
