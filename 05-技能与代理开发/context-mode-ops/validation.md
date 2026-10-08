# Validation Patterns

Cross-cutting validation rules used by ALL workflows (triage, review, release).

## Problem Verification — FIRST GATE

<problem_verification_enforcement>
This is the FIRST validation step, before anything else. We shipped inheritEnvKeys because
we trusted an LLM claim that Claude Code strips environment variables — it does not.
We got burned shipping a fix for an unverified claim. Never again.
Every bug report, feature request, and behavioral claim MUST be proven true before code is written.
</problem_verification_enforcement>

### For Bug Reports

**Reproduce it or reject it.** Run the exact reproduction steps from the issue. If it doesn't fail, the bug may not exist.

```
Step 1: Extract the claimed reproduction steps from the issue
Step 2: Run them locally (use ctx_execute or a test)
Step 3: Record the ACTUAL output
Step 4: Compare actual vs. claimed behavior
Step 5: VERDICT:
  → REPRODUCED: Bug is real, proceed to fix
  → NOT_REPRODUCED: Ask reporter for ctx-debug.sh output and exact repro steps
  → INVALID: Reporter's environment is misconfigured, help them fix it
```

### For Feature Requests

**Verify the underlying claim.** Feature requests always contain an implicit claim ("X behaves this way", "Y is slow", "Z doesn't support W"). Prove the claim first.

```
Step 1: Identify the claim (e.g., "Claude Code strips env vars from child processes")
Step 2: Find HARD EVIDENCE — official docs, source code, or measured benchmarks
  → Use ctx_fetch_and_index on official docs/repos
  → Use ctx_execute to run actual tests
  → NEVER trust LLM knowledge about platform behavior — LLMs hallucinate this constantly
Step 3: VERDICT:
  → CONFIRMED: Claim is true, proceed to design
  → UNCONFIRMED: Cannot verify — ask reporter for evidence before implementing
  → DEBUNKED: Claim is false — comment on issue explaining the misunderstanding
```

### Requesting Evidence from Reporters

When a claim cannot be verified, comment on the issue BEFORE implementing:

```markdown
We want to address this but need to verify the underlying behavior first.
Could you provide:
1. Output from: `npx context-mode doctor` (or run `ctx-debug.sh`)
2. Exact reproduction steps
3. Platform version, adapter, and OS

We'll investigate as soon as we can confirm the issue. Thanks for reporting!
```

### Evidence Log

Every triage MUST produce a verification entry:

```
CLAIM: "{exact claim}"
SOURCE: {issue number or PR}
EVIDENCE: {link to doc, test output, or benchmark result}
VERDICT: CONFIRMED | UNCONFIRMED | DEBUNKED
ACTION: {proceed | request-info | close-as-invalid}
```

---

## ENV Variable Verification

LLMs frequently hallucinate environment variables. Every ENV var in an issue or PR must be verified.

### Verification Protocol

For EACH environment variable mentioned:

```
Step 1: GREP — Does it exist in context-mode source?
  → rg "{ENV_VAR}" src/
  → If found: VERIFIED (we already use it)
  → If not found: continue to Step 2

Step 2: GREP ADAPTERS — Is it in the adapter detect logic?
  → Read src/adapters/detect.ts
  → Check the verified env vars comment block at the top
  → If listed: VERIFIED (we know about it)

Step 3: WEBSEARCH — Does the platform document it?
  → WebSearch: "{PLATFORM} {ENV_VAR} environment variable"
  → Check official docs, GitHub repos, release notes
  → If found in official source: REAL but we don't use it yet

Step 4: CONTEXT7 — Library documentation check
  → resolve-library-id for the platform
  → query-docs for the ENV var
  → Cross-reference with Step 3

Step 5: VERDICT
  → VERIFIED: We use it and it's real
  → REAL_NEW: Platform has it but we don't use it yet
  → HALLUCINATED: No evidence it exists — flag it
  → DEPRECATED: Used to exist but was removed
```

### Known Verified ENV Vars (Reference)

| Platform | Verified ENV Vars | Source |
|----------|------------------|--------|
| Claude Code | `CLAUDE_PROJECT_DIR`, `CLAUDE_SESSION_ID` | src/adapters/detect.ts |
| Gemini CLI | `GEMINI_PROJECT_DIR`, `GEMINI_CLI` | src/adapters/detect.ts |
| OpenCode | `OPENCODE`, `OPENCODE_PID` | src/adapters/detect.ts |
| OpenClaw | `OPENCLAW_HOME`, `OPENCLAW_CLI` | src/adapters/detect.ts |
| Kilo | `KILO`, `KILO_PID` | src/adapters/detect.ts |
| Codex | `CODEX_CI`, `CODEX_THREAD_ID` | src/adapters/detect.ts |
| VS Code Copilot | `VSCODE_PID`, `VSCODE_CWD` | src/adapters/detect.ts |
| Cursor | `CURSOR_TRACE_ID`, `CURSOR_CLI` | src/adapters/detect.ts |
| Override | `CONTEXT_MODE_PLATFORM` | src/adapters/detect.ts |

Any ENV var NOT in this table must go through the full verification protocol.

## Adapter Test Matrix

### Full Matrix Run

```shell
# Run ALL adapter tests
npx vitest run tests/adapters/

# Individual adapter (for targeted testing)
npx vitest run tests/adapters/claude-code.test.ts
npx vitest run tests/adapters/gemini-cli.test.ts
npx vitest run tests/adapters/opencode.test.ts
npx vitest run tests/adapters/openclaw.test.ts
npx vitest run tests/adapters/kilo.test.ts
npx vitest run tests/adapters/codex.test.ts
npx vitest run tests/adapters/vscode-copilot.test.ts
npx vitest run tests/adapters/cursor.test.ts
npx vitest run tests/adapters/antigravity.test.ts
npx vitest run tests/adapters/kiro.test.ts
npx vitest run tests/adapters/zed.test.ts

# Detection logic
npx vitest run tests/adapters/detect.test.ts
npx vitest run tests/adapters/client-map.test.ts
```

### Report Format

```
ADAPTER TEST MATRIX
═══════════════════
claude-code     ✓ 5/5    gemini-cli      ✓ 4/4
opencode        ✓ 6/6    openclaw        ✓ 3/3
kilo            ✓ 4/4    codex           ✓ 3/3
vscode-copilot  ✓ 4/4    cursor          ✓ 3/3
antigravity     ✓ 2/2    kiro            ✓ 3/3
pi              ✓ 2/2    zed             ✓ 2/2
detect          ✓ 8/8    client-map      ✓ 6/6
───────────────────────────────────────────
TOTAL: {N}/{N} passed | 0 failed
```

## Core Module Tests

```shell
# Core tests
npx vitest run tests/core/routing.test.ts
npx vitest run tests/core/search.test.ts
npx vitest run tests/core/server.test.ts
npx vitest run tests/core/cli.test.ts

# Module tests
npx vitest run tests/store.test.ts
npx vitest run tests/executor.test.ts
npx vitest run tests/security.test.ts
npx vitest run tests/formatters.test.ts

# Hook tests
npx vitest run tests/hooks/

# Full suite
npm test
```

## OS Compatibility Checks

### Path Handling

```javascript
// WRONG — breaks on Windows
const configPath = homedir + "/.config/opencode/config.json";

// CORRECT — works everywhere
const configPath = path.join(homedir(), ".config", "opencode", "config.json");
```

Grep for potential issues:
```shell
# String concatenation with path separators
rg "homedir\(\)\s*\+" src/
rg '"/\.' src/
rg "'\\./" src/

# Direct slash usage in paths (should use path.join)
rg 'path\s*=.*"/' src/ --type ts
```

### Temp Directory

```javascript
// WRONG — hardcoded /tmp
const tmpFile = "/tmp/context-mode-output.txt";

// CORRECT — uses OS temp dir
const tmpFile = path.join(os.tmpdir(), "context-mode-output.txt");
```

Grep for hardcoded temp:
```shell
rg '"/tmp/' src/
rg "'/tmp/" src/
```

### Native Bindings (better-sqlite3)

Check that `better-sqlite3` is in `optionalDependencies` (not `dependencies`) and the code handles the case where it's not available:

```shell
rg "better-sqlite3" src/ --type ts
rg "optionalDependencies" package.json
```

### Process Spawn

```javascript
// WRONG — shell: true behaves differently on Windows
spawn("command", { shell: true });

// CORRECT — explicit shell selection
spawn("command", { shell: process.platform === "win32" ? "cmd.exe" : "/bin/sh" });
```

## Hook Format Validation

Each platform has different hook formats. Verify changes match:

| Platform | Hook Format | Key Differences |
|----------|------------|-----------------|
| Claude Code | `hooks.json` in plugin dir | `PreToolUse`, `PostToolUse`, `PreCompact`, `SessionStart` |
| Gemini CLI | `~/.gemini/settings.json` | `BeforeTool`, `AfterTool`, `PreCompress`, `SessionStart` + `matcher` |
| VS Code Copilot | `.github/hooks/*.json` | Same as Claude Code but separate file |
| Cursor | `.cursor/hooks.json` | No `SessionStart` (injects via file instead) |
| OpenCode | `opencode.json` | Uses `agents` section, not traditional hooks |
| OpenClaw | `openclaw.plugin.json` | Extension model, not hook-based |

## Security Checks

### Sandbox Escape

```shell
# File writing attempts through ctx_execute
rg "writeFile\|appendFile\|createWriteStream" src/executor.ts

# Path traversal
rg "\.\.\/" src/ --type ts

# Command injection vectors
rg "exec\(.*\$\{" src/ --type ts
rg "spawn\(.*\$\{" src/ --type ts
```

### Information Disclosure

```shell
# Sensitive paths
rg "process\.env\b" src/ --type ts | grep -v "test"

# Home directory exposure
rg "homedir\(\)" src/ --type ts
```

## TypeScript Validation

```bash
# Full type check
npm run typecheck

# Should report 0 errors
# If errors exist, they MUST be fixed before shipping
```

## Pre-Ship Checklist

Every change, regardless of workflow, must pass:

- [ ] **Problem verified** — CLAIM_VERDICT is CONFIRMED with hard evidence (this is gate zero)
- [ ] `npm run typecheck` — 0 errors
- [ ] `npm test` — all pass
- [ ] Adapter tests — all 12 pass (or N/A if untouched)
- [ ] ENV vars — all verified against real platform source
- [ ] Path handling — no hardcoded separators
- [ ] Hook format — matches target platform's schema
- [ ] No security regressions

## Fan-out Claim Verification — SECOND GATE

<fan_out_verification_enforcement>
STOP. Before dispatching N implementation agents on a survey result, you MUST
verify each of the N claims independently. Audit agents are LLMs; LLMs lie.
A survey that returns "10 adapters need this fix" is the moment to slow down,
not the moment to fork 10 parallel implementations.

RULE: One audit claim ≠ one verified bug. Each item gets its own CLAIM_VERDICT
before code is touched. The same Problem Verification gate that applies to
external bug reports applies — equally — to internal fan-out claims.
</fan_out_verification_enforcement>

### When this gate fires

Any time an EM-mode flow produces a multi-item list of suggested fixes — from
a survey agent, a triage sweep, a cross-adapter audit, a release pre-flight,
an architecture review — the list is a HYPOTHESIS, not a punch list. The N
items have NOT been verified just because one agent grouped them.

### The pattern

```
PHASE A — Reproduce each claim
  Spawn one verification agent PER item in parallel.
  Each agent MUST: read the platform source, attempt to reproduce the
  failure, return CLAIM_VERDICT = CONFIRMED | DEBUNKED | UNDETERMINED.
  File:line citations from real source are mandatory. No source citation
  = treat as UNDETERMINED.

PHASE B — Research only CONFIRMED items
  For each CONFIRMED item, the SAME agent (or a follow-up) MUST identify
  the concrete fix shape: file location, format, multi-window safety,
  feasibility tier (HIGH / MEDIUM / LOW). DEBUNKED items are dropped.
  UNDETERMINED items are tabled for the maintainer's empirical session.

PHASE C — Implement only verified items
  Dispatch implementation agents only for items that made it through A and
  B. The fanout factor at this gate is typically 5-20% of the original
  audit list — the rest were speculation that the gate caught.
```

### Why this gate exists

Real incident (v1.0.148-era stats accuracy work, 2026-05-24):

- A survey agent returned 10 adapters needing a `resolveCodexSessionCwd`-
  style on-disk fallback. The maintainer asked for proof.
- Phase A spawned 5 parallel verification agents (cursor, vscode-copilot,
  jetbrains-copilot, gemini-cli, opencode).
- Result: 2 DEBUNKED (cursor: passes cwd in hook stdin; opencode: native
  plugin bridge, no spawn), 3 CONFIRMED but with different fix shapes
  (vscode-copilot: trivial 1-line cascade addition; jetbrains: on-disk
  fallback infeasible due to multi-window collision; gemini-cli: upstream
  bug + docs workaround).
- Net engineering scope: **1 trivial line change** instead of 10
  parallel implementations.

Without this gate, we would have shipped 10 speculative fixes against
unreproducible claims — the same failure mode as the inheritEnvKeys
incident (an LLM said "Claude Code strips env vars from child processes",
we shipped a fix, and the platform behavior was never the claim).

### What a CLAIM_VERDICT must look like

```yaml
claim:
  description: "<one sentence stating the hypothesis precisely>"
verdict:
  status: CONFIRMED | DEBUNKED | UNDETERMINED
  evidence:
    - "file:line — what was found"
    - "file:line — supporting detail"
    - "file:line — counterexample (for DEBUNKED)"
  fix_shape:        # only when CONFIRMED
    location: "<file/format>"
    field_path: "<where cwd / target value lives>"
    multi_window_safety: "<mtime guard? unsafe? n/a?>"
    feasibility: HIGH | MEDIUM | LOW
  decision: SHIP | DOCS-ONLY | TABLE | DROP
```

### Anti-patterns to recognize

- **"The audit says 10 — let me spawn 10 implementation agents."** Don't.
  Spawn 10 verification agents first; THEN spawn implementation agents only
  for the survivors.
- **"This is the same kind of bug we just fixed, so it must apply here
  too."** Almost-same-shape bugs frequently have completely different
  root causes per-platform. Verify per-platform.
- **"The agent said X, I trust it."** LLMs are programmed to take the
  path of minimum energy. They will write plausible verdicts without
  actually reading the source. Require file:line citations and spot-
  check at least one.
- **"We don't have time to verify each one."** Verification is faster
  than reverting bad ships. Five verification agents in parallel finish
  in the time one implementation agent takes. The verification spend pays
  for itself if it catches even one DEBUNKED.
