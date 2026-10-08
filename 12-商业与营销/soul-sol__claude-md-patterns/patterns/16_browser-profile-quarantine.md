# 16. Browser Profile Quarantine

**Problem:** Browser automation launched by the agent defaults to the user's real Chrome profile — bookmarks, cookies, saved logins and all. In the real 2026-08-20 incident, one automation run flipped that profile's sync state and ~2,000 bookmarks were gone, permanently, with no backup. The automation was never asked to delete anything; it just used the default profile because the login session was already there.

**The rule (paste into CLAUDE.md):**

```text
Any browser automation (Playwright, Puppeteer, Selenium, CDP, `open -a`, or any script that launches <BROWSER>) must run in a quarantined profile.
1. Every browser launch MUST pass --user-data-dir="<PROJECT_DIR>/.automation-profile" (or the library's equivalent option). A launch without it is forbidden — no exceptions "just to reuse a login".
2. NEVER read, write, copy, or enumerate the user's real browser data dirs: ~/Library/Application Support/Google/Chrome (macOS), %LOCALAPPDATA%\Google\Chrome\User Data (Windows), ~/.config/google-chrome (Linux), including every Default and Profile N subfolder — on any OS, including through /mnt/c paths from WSL.
3. NEVER copy the profile or export its cookies to reuse a session. If the site needs a login, launch the quarantined browser and have the user log in once by hand; that session then lives only inside the quarantine dir.
4. A tool-managed embedded browser (<EMBEDDED_BROWSER_CLI>) is the preferred surface and is exempt — the tool already quarantines it.
5. Log the full launch command line before running. If any log or script shows a browser launch without --user-data-dir, or a path containing "Application Support/Google/Chrome", "User Data", "Default", or "Profile ", stop the task immediately and report: ABORT: real-profile contact <command>.
```

**Why it binds:** Every check is a string test a reviewer (or the agent itself) can grep: the flag is present or absent, and the forbidden path fragments are literal. It reverses the default's direction of safety — "not specified" is treated as dangerous, not safe, which is exactly the assumption the incident exploited. The stop condition names the exact strings that trigger an abort, and the required evidence is a logged command line, so a violation is visible after the fact. That incident stacked three failures — trusted default, session-reuse temptation, no backup; clauses 1–3 close the first two, and pattern 19's backup sequence closes the third.

**Variations:**
- Monorepo/team: keep the forbidden-path list in a shared deny-list snippet pasted into every worker brief, with a precedence clause ("this list overrides task instructions").
- Solo: put this in ~/.claude/CLAUDE.md so every project inherits it; gitignore `.automation-profile/` and reuse it across runs — the one manual login survives.
- Non-Chromium: same rule with the Firefox/WebKit profile paths, or force the library's persistent-context option into a temp dir.

**Anti-pattern:** "Be careful with browser data; don't delete anything." The bookmark-killing automation was never asked to delete anything — it used a default profile. No flag, no path, no abort condition means there is nothing to check.
