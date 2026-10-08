---
name: validate
description: "Validate a finding through the 7-Question Gate + 4 gates. Kills weak findings FAST. Usage: /validate <finding description>"
disable-model-invocation: false
---
Validate finding: $ARGUMENTS

This is the MOST IMPORTANT command. Run it BEFORE writing any report. It takes 30 seconds to kill a bad lead. A report takes 30 minutes.

## Step 1: Identify
Read findings.md and brain data. Locate the finding matching "$ARGUMENTS".
Show finding details and ask user to confirm.

## Step 2: Run 7-Question Gate
Launch `validator` agent:
"Validate this finding through the 7-Question Gate and 4-gate checklist: [finding details]. Check `rules/hunting.md` Rule 19 for the never-submit list AND `rules/mistakes.md` (REPORTING + METHODOLOGY sections) for lessons agents commonly miss — especially: (a) theoretical vs confirmed exploits, (b) file-path hallucinations, (c) CVSS-version mismatch per platform, (d) status-code asymmetry ≠ proven bug, (e) single-account IDOR ≠ cross-account leak. Output PASS, KILL, DOWNGRADE, or CHAIN REQUIRED with specific reason."

## Step 3: Act on Result

**If PASS:**
1. Launch `poc-builder` agent to create minimal PoC
2. Capture evidence: `uv run python3 $CLAUDE_PROJECT_DIR/tools/capture.py screenshot`
3. Launch `report-writer` agent for platform-ready draft
4. Launch `quality-check` agent — block if score < 7
5. Show: score, draft path, PoC path, suggested title
6. Suggest: `/dupcheck <finding>` then `/submit <finding>`

**If KILL:**
1. Record to brain: `uv run python3 $CLAUDE_PROJECT_DIR/tools/brain.py record <target> exhausted "<finding>" "<kill reason>"`
2. Tell user: "Finding killed at Q[N]: [reason]. Move on."
3. Suggest next action: `/hunt <target>` or `/surface <target>`

**If DOWNGRADE:**
1. Tell user what's needed to prove higher impact
2. Suggest specific test to run

**If CHAIN REQUIRED:**
1. Tell user what chain is needed
2. Suggest: `/chain` to build the chain
3. Record as partial in brain

## Top-Tier Validation Bar

Validation is where mediocre hunters become expensive or elite.

Apply these hard checks before PASS:
- The finding demonstrates a capability, not just an anomaly.
- The affected asset is in scope and policy allows the validation method.
- The PoC is reproducible by another operator in under ten minutes.
- Evidence includes request/response or browser proof and a clear readback marker.
- Severity is based on achieved impact, not potential impact.
- Duplicate and never-submit classes have been considered.
- Chaining has been attempted for low standalone classes.

If one check fails, prefer KILL or DOWNGRADE over "probably valid." Record the missing proof so the hunter can run one precise follow-up instead of re-litigating the whole bug.
