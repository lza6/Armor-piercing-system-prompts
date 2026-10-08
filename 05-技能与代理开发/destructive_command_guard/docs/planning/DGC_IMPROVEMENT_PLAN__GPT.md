# DCG Improvement Plan (Hybrid, ultra-detailed)

> **Author:** GPT-5.2 (Codex CLI)
> **Date:** 2026-01-08
> **Status:** Proposal
> **Scope:** Hook correctness + false-positive immunity + heredocs/inline code + operational UX + team-wide integrations (pre-commit/CI)

This document is a deliberately **over-explained** plan intended for “future us.” It captures the reasoning, trade-offs, and implementation approach for making `dcg` (Destructive Command Guard) more:

- robust and reliable (no silent gaps, no hangs)
- fast (microseconds on the allow path; bounded worst-case behavior)
- predictable (deterministic decisions)
- ergonomic and user-friendly (high trust, low disruption)
- resistant to false positives that destroy coding-agent velocity
- modular and extensible (packs, tiers, explainability, allowlisting, scanning integrations)

If we implement the top items here, `dcg` becomes the kind of tool users keep enabled permanently because it feels *obviously correct* and *rarely annoying*.

---

## Executive summary (what matters most)

**Strategic pillars (ranked, with dependencies called out):**

1. **Correctness substrate (must-fix):** enabled packs run in hook mode, deterministic decisions, stable match IDs, and a shared evaluator for hook + CLI.
2. **False positive immunity (trust unlock):** execution-context classification (executed vs data) + safe string-argument registry + project allowlists.
3. **Deep bypass coverage (safety unlock):** tiered heredoc + inline script scanning (fast trigger → bounded extraction → AST-aware matching).
4. **Explainability + safe customization (adoption unlock):** `dcg explain` decision trace, suggestions, and allowlisting by stable rule IDs (not raw regex).
5. **Team-wide protection (compelling unlock):** pre-commit scanning + CI/GitHub Action scanning that reuse the same engine, plus strong testing (corpus + proptest + fuzzing).

**What this plan optimizes for:**

- users leave dcg enabled (low false positives, high trust)
- decisions are repeatable and debuggable
- performance is predictable and bounded (no “random 200ms hooks”)
- expansions (packs, heredocs, CI scanning) reuse one engine and one policy model

---

## Table of contents (high level)

1. [Current architecture (as intended)](#current-architecture-as-intended)
2. [Current actual state (important gaps)](#current-actual-state-important-gaps)
3. [Design principles](#design-principles-guardrails-for-the-whole-project)
4. [Success metrics & budgets](#success-metrics--budgets)
5. [Idea pool (expanded)](#idea-pool-expanded)
6. [The best strategy (deep rationale)](#the-best-strategy-deep-rationale)
7. [Implementation roadmap (phased, test-first)](#implementation-roadmap-phased-test-first)
8. [Appendix: File formats & UX sketches](#appendix-file-formats--ux-sketches)

## Current architecture (as intended)

Claude Code calls `dcg` as a `PreToolUse` hook:

1. Parse stdin JSON (Claude hook protocol)
2. Extract Bash command string
3. Normalize command (strip `/usr/bin/git` → `git`, etc.)
4. Quick reject / keyword gating (skip work for irrelevant commands)
5. Safe patterns (whitelist) checked before destructive patterns
6. Destructive patterns checked second (blacklist)
7. Default allow for unmatched commands
8. Deny outputs:
   - colorful warning to stderr
   - JSON denial to stdout

The modular “pack system” is the long-term home for pattern logic:

- packs are enable/disable-able via config
- each pack has keywords to skip regex work quickly
- packs are intended to cover git, filesystem, database, containers, kubernetes, cloud, etc.

---

## Current *actual* state (important gaps)

The repository is mid-migration. Key problems that must be fixed early:

### 1) Non-core packs are effectively unreachable in hook mode

There is an early-return quick reject that only checks `git`/`rm`. If a command is `docker ...` or `kubectl ...`, the hook can return before evaluating packs. That means enabled packs can silently not run.

This is a **trust-killer**: `dcg test` might report “BLOCKED,” while the actual hook would allow the same command.

### 2) Decision nondeterminism

Pack evaluation order can be derived from `HashSet` iteration order. If multiple packs match, the chosen pack/reason can vary run-to-run.

Determinism is essential for:

- reliable debugging
- allowlisting by rule ID
- consistent E2E results
- user trust (“why did it block differently this time?”)

### 3) Duplicate legacy matching logic

There is legacy pattern matching duplicated in the binary in addition to the pack system. Duplication leads to drift and unfixable “works in one mode but not the other” behavior.

### 4) Per-command regex compilation (overrides)

Config overrides that compile regex at runtime per command introduce latency spikes and unpredictability.

### 5) Naming drift

There are still references to `git_safety_guard` in env vars/comments/scripts. Naming confusion causes misconfiguration.

### 6) False positives via context blindness

The core UX pain: substring matching blocks commands that merely *mention* dangerous commands in strings (commit messages, issue descriptions, grep patterns).

Example false positives that must be ALLOWED:

- `bd create --description="This blocks rm -rf"`
- `git commit -m "Fix git reset --hard detection"`
- `echo "example: git push --force"`
- `rg -n "rm -rf" src/main.rs`

### 7) “Interactive prompts” are dangerous in the hook path

Some “learning mode” concepts involve prompting the user at block time. For a Claude hook, blocking + waiting for stdin is usually a **hang** (catastrophic UX). Any “learning” must be:

- a separate CLI flow (`dcg allow ...`, `dcg explain ...`)
- or an output hint that Claude/user can copy/paste
- never an interactive prompt that the hook waits on

---

## Design principles (guardrails for the whole project)

### P0: Never hang, never crash, never spike unpredictably

This tool runs for every Bash command. Stability and bounded worst-case behavior is non-negotiable.

### P1: Default allow, but confidently deny known catastrophes

Unrecognized commands should not break workflows. But high-confidence catastrophic commands should be denied (or at least not silently allowed).

### P2: Deterministic and explainable decisions

Same input → same decision + same attribution every time.

### P3: False positives are a first-class problem

False positives destroy trust and velocity. A guard that users disable is strictly worse than a guard that is slightly less strict but always enabled.

### P4: Incremental delivery

Prefer small, test-driven, high-impact increments:

- fix correctness first
- then reduce false positives
- then add deeper scanning
- then add UX/explainability
- always maintain performance budgets and tests

---

## Success metrics & budgets

These are the “definition of done” signals. If we don’t measure, we’ll regress.

### UX / adoption metrics

| Metric | Target | Why it matters | How to measure |
|--------|--------|----------------|----------------|
| False-positive rate (blocking) | < 1% for common dev workflows | Users disable tools that are noisy | E2E corpus + real-world logs (opt-in) |
| Time to understand a block | < 30 seconds | Prevents rage-disabling | `dcg explain` usability testing |
| Time to resolve a false positive | < 2 minutes | Keeps velocity | `dcg allow` workflow + docs |
| Trust/retention | “kept enabled” | Ultimate success metric | anecdotal + optional telemetry |

### Correctness metrics

| Metric | Target | Why | How |
|--------|--------|-----|-----|
| Hook/CLI parity | 100% | “test matches reality” | parity tests (hook JSON vs `dcg test`) |
| Determinism | 100% | stable attribution, debuggable | run same input N times in tests |
| No silent pack skips | 0 | non-core packs must work | integration tests for docker/k8s/db |

### Performance budgets (ballpark; revise after baseline)

| Path | Target p50 | Target p95 | Notes |
|------|------------|------------|------|
| Allow path, no keywords | < 50µs | < 150µs | should be effectively invisible |
| Allow path, keyword present | < 200µs | < 750µs | pack keyword gating keeps it bounded |
| Deny path (regex-only) | < 1ms | < 3ms | includes formatting output |
| Heredoc/inline scan path | < 10ms | < 25ms | must be hard-capped + fail-open |

### Reliability / hardening metrics

| Metric | Target | Why | How |
|--------|--------|-----|-----|
| Panics/hangs | 0 | hook must never hang | fuzzing + property tests |
| Max input handling | bounded | prevent DoS | size/time limits + tests |

---

## Idea pool (expanded)

Each idea includes:

- **How it works**
- **User perception**
- **Implementation sketch**
- **Risks / mitigations**

### 1) Pack-aware global quick reject (fix pack reachability)

- How: global early return must consider keywords for **enabled packs**, not just `git`/`rm`.
- User perception: “Docker/kubectl protections actually work.”
- Implementation:
  - after config load, compute enabled packs and expand categories → concrete pack IDs
  - build a union of enabled packs’ `keywords`
  - only early-return if **none** of those keywords appear anywhere in the command
  - start simple: loop keywords and use `memchr::memmem::find` (fast enough for tens of keywords)
  - if keyword unions grow large, upgrade to an Aho–Corasick automaton (or a compiled `RegexSet`) so gating stays O(n) in the command size
- Risks: perf regression if union search is naive; mitigate with `memmem`/Aho–Corasick and perf budgets.

### 2) Deterministic pack ordering

- How: evaluate packs in a stable, documented order (tiers + lexicographic).
- User perception: consistent reasons; fewer “random” behaviors.
- Implementation: expand enabled categories → list pack IDs → stable sort or explicit tier ordering.
- Risks: subjective tier ordering; document and test.

### 3) Return stable match identity (pack_id + pattern_name)

- How: include `pattern_name` and `pack_id` in results, not just “reason.”
- User perception: block messages feel concrete and actionable.
- Implementation: have matching return `DestructivePattern { name, reason }`.
- Risks: unnamed patterns; address incrementally.

### 4) One shared evaluator for hook + CLI

- How: hook mode and `dcg test` call the same evaluation function.
- User perception: “test matches reality.”
- Implementation: move decision engine into library module; `main` becomes IO glue.
- Risks: refactor churn; mitigate with parity tests.

### 5) Precompile override regex at config load

- How: compile overrides once; invalid regex yields warning + ignore.
- User perception: avoids latency spikes; fewer random stalls.
- Implementation: `CompiledOverrides` runtime struct.
- Risks: config errors become silent; mitigate via `dcg doctor` warnings or logging.

### 6) Execution-context classification layer (“data vs executed”)

- How: classify spans as executed vs data; only match executable spans.
- User perception: huge false positive reduction; restores trust.
- Implementation: conservative shell tokenizer that recognizes quoting, pipes, separators, substitution.
- Risks: shell parsing complexity; mitigate by starting conservative and heavily testing.

### 7) Safe string-argument registry v1 (bd/git/rg/grep/echo/printf)

- How: treat values of known doc/metadata flags as data spans.
- User perception: immediate reduction in annoying blocks.
- Implementation: explicit table of (command, subcommand, flags) with tests.
- Risks: registry drift; keep small and test-driven.

### 8) Token-aware keyword gating (reduce substring triggers)

- How: treat keywords as whole executable tokens, not substrings anywhere.
- User perception: slightly faster and less “twitchy.”
- Implementation: reuse tokenizer to extract command words and gate on those.
- Risks: false negatives if too strict; fallback to substring in ambiguous cases.

### 9) Normalize common wrappers (sudo/env/command/\\git)

- How: canonicalize leading wrappers so destructive patterns still match.
- User perception: fewer bypasses; more consistent behavior.
- Implementation: strip known wrappers in normalization step, only when syntactically obvious.
- Risks: over-normalization; keep rules minimal and tested.

### 10) Decision modes: deny vs warn vs log-only

- How: allow less-certain detections to warn/log rather than hard block.
- User perception: fewer “workflow stops” while still protected.
- Implementation: severity/confidence per pattern or pack; config overrides.
- Risks: misconfiguration reduces safety; keep catastrophic patterns deny-by-default.

### 11) Structured logging + redaction

- How: log decisions with IDs, timing, and optional argument redaction.
- User perception: makes tuning and debugging easier without leaking secrets.
- Implementation: append-only log schema; redaction heuristics for quoted strings.
- Risks: privacy; default to minimal logging and opt-in verbosity.

### 12) `dcg test --explain` trace output

- How: show normalization, context classification, enabled packs, skipped packs, first match.
- User perception: trust boost; faster debugging.
- Implementation: evaluator emits a trace object in CLI mode only.
- Risks: ensure explain doesn’t affect hook performance.

### 13) Allowlist by (pack_id, pattern_name) instead of raw regex

- How: user allowlists exact rules, not broad regex strings.
- User perception: safer, simpler config.
- Implementation: new config section; check allowlist after match.
- Risks: needs stable pattern identity (idea #3).

### 14) Simulation mode (run on command logs)

- How: feed a log/history file; show what would be blocked/warned.
- User perception: safer rollout; tune before enforcing.
- Implementation: `dcg simulate --file ...`.
- Risks: don’t confuse simulation with enforcement; label clearly.

### 15) Observe mode for incremental rollout

- How: run warn/log-only for a period; then tighten.
- User perception: adoption-friendly.
- Implementation: config `default_mode=warn`; reports summary.
- Risks: forgetting to tighten; `doctor` can nag.

### 16) Regex hardening (prefer linear-time regex where possible)

- How: minimize `fancy-regex` usage to avoid backtracking spikes.
- User perception: fewer latency spikes.
- Implementation: use `regex` crate for most patterns; reserve fancy for lookarounds.
- Risks: rewriting patterns; do gradually with parity tests.

### 17) Size/time limits to avoid DoS/hangs

- How: cap command length, heredoc body size, AST parse time, etc.
- User perception: tool stays snappy under weird inputs.
- Implementation: early checks + fail-open with logs.
- Risks: might miss some malicious content; but better than hanging.

### 18) Expand E2E to cover non-core packs (docker/k8s/db)

- How: assert packs work in hook mode with enabled config.
- User perception: confidence in non-core protections.
- Implementation: E2E harness enabling packs explicitly + detailed per-test logging.
- Risks: test maintenance; keep cases high signal.

### 19) Golden parity tests: hook JSON path == CLI test path

- How: table-driven integration tests feed both entrypoints and assert identical outcomes.
- User perception: fewer surprises.
- Implementation: `assert_cmd` tests; deterministic ordering required.
- Risks: minimal.

### 20) Pack keyword audit tests (prevent gating false negatives)

- How: ensure each pack’s keywords cover realistic destructive invocations.
- User perception: fewer misses.
- Implementation: unit tests per pack verifying keyword present on matching cases.
- Risks: ongoing work but high ROI.

### 21) Per-rule “safe alternative” suggestion metadata

- How: include a tailored suggestion for each denial.
- User perception: less frustration; clearer next step.
- Implementation: add `suggestion` field to `DestructivePattern`.
- Risks: suggestions can be wrong in some environments; keep as optional guidance.

### 21.1) Suggestions database keyed by stable rule ID (pack_id:pattern_name)

- How: build a small registry mapping `pack_id:pattern_name` to suggestions (safe alternatives + docs + “how to allow” command).
- User perception: “this block taught me what to do instead” rather than “it yelled at me.”
- Implementation: `LazyLock<HashMap<&'static str, Vec<Suggestion>>>` plus formatting helpers used by deny output and `dcg explain`.
- Risks: stale/incorrect suggestions; mitigate by keeping suggestions generic and optional, and by linking to docs rather than prescribing exact commands in all cases.

### 22) Improved `dcg doctor` (detect config + hook drift)

- How: doctor verifies hook installed, config valid, packs reachable, naming consistent.
- User perception: easy onboarding and debugging.
- Implementation: add checks for config prefix, known hook JSON format, etc.
- Risks: none.

### 23) Config discovery optimizations

- How: faster repo root detection; allow explicit config path env var.
- User perception: faster in deep trees.
- Implementation: stop at `.git` quickly; allow `DCG_CONFIG=...`.
- Risks: minimal.

### 24) Optional “safe cleanup” pack (allow rm -rf of known artifacts)

- How: allow `rm -rf target/`, `node_modules/`, etc. only under repo root and exact paths.
- User perception: big ergonomic win for agents; fewer human interruptions.
- Implementation: compute repo root; allow exact relative paths; deny everything else.
- Risks: safety trade-off; must be opt-in and heavily tested.

### 25) Rule severity taxonomy (catastrophic/high/medium)

- How: tag rules; drive default decision mode and message tone.
- User perception: blocks feel more justified and consistent.
- Implementation: add enum to patterns; use it in output/policy.
- Risks: bikeshedding; start simple.

### 26) Confidence scoring for ambiguous cases

- How: if context uncertain, run deeper analysis; if still uncertain, warn not deny.
- User perception: fewer unwarranted hard blocks.
- Implementation: scoring pipeline combining context + match features.
- Risks: complexity; defer until basics are solid.

### 27) Tiered heredoc/inline scanning (RegexSet → extraction → AST)

- How: detect heredoc/inline script, extract content, run AST matching.
- User perception: closes bypasses; makes tool compelling.
- Implementation: tiered approach with strict budgets; fail-open on parse failure.
- Risks: language edge cases; mitigate with robust extraction limits and tests.

### 28) Language detection heuristics (command + shebang + content hints)

- How: infer python/ruby/node/bash etc. from command context and snippet content.
- User perception: fewer misses, fewer false positives.
- Implementation: heuristics + fallback ordering + tests.
- Risks: wrong inference; fail-open with trace/log.

### 29) Fuzz/property testing for tokenizers/parsers

- How: fuzz shell tokenizer and heredoc extractor to prove no panics/hangs.
- User perception: reliability under weird agent output.
- Implementation: `cargo fuzz` or proptest; curated corpora.
- Risks: setup time; worth it.

### 30) Performance benchmarks + budgets in CI

- How: microbench hot path; detect regressions.
- User perception: tool stays “invisible.”
- Implementation: Criterion benches; CI thresholds/trending.
- Risks: CI variance; use relative budgets and trending.

### 31) Project-scoped allowlist file with audit trail + expiration

- How: load a repo-scoped allowlist file (e.g., `.dcg/allowlist.toml`) that is committed and code-reviewed. Entries include metadata: who/when/why, optional expiration, and explicit risk acknowledgment for broad patterns.
- User perception: “my whole team benefits when we fix a false positive once.”
- Implementation: allowlist module with exact/prefix/rule-id entries; validation warnings for risky entries; save/update via CLI (`dcg allow`, `dcg allowlist validate`).
- Risks: allowlists can become a security hole; mitigate by requiring explicit `risk_acknowledged=true` for regex patterns, favoring rule-id allowlists, and allowing expiration.

### 32) Guided “learning” workflow (non-interactive hook-safe)

- How: when dcg blocks, it prints a copy/paste-able command to create an allowlist entry, plus `dcg explain` guidance. Learning happens via CLI, not by blocking the hook waiting for user input.
- User perception: “I can resolve this immediately without editing TOML by hand.”
- Implementation: deny output includes a stable rule ID and example `dcg allow --rule core.git:reset-hard --reason ...` (or per-project allowlist path).
- Risks: users allowlist too broadly; mitigate by defaulting to rule-id or exact-command allow entries and requiring explicit ack for regex.

### 33) Pre-commit integration (`dcg scan --staged`) scanning executable commands in code/config

- How: scan staged files for executable command contexts (shell scripts, CI YAML, Dockerfiles, Makefiles, Terraform provisioners, etc.) and run the same evaluator on extracted commands.
- User perception: “we catch dangerous commands before they land in the repo.”
- Implementation:
  - build a file scanner that *extracts executable command contexts* (not naive grep), then runs the existing dcg engine on each extracted command
  - MVP extractors (keep scope tight + high-signal):
    - `.sh`/shell scripts: executed lines (respect `set -euo pipefail` is irrelevant; we only extract)
    - Dockerfile: `RUN ...` (shell form), optionally `CMD`/`ENTRYPOINT` shell form
    - GitHub Actions: YAML `steps[].run` blocks only (ignore `name`, `env`, docs fields)
    - Makefile: recipe lines (tab-indented), with `\` continuations
  - next-wave extractors (after MVP + corpus coverage):
    - GitLab CI: `script:` / `before_script:` blocks
    - Terraform: `provisioner "local-exec" { command = "..." }` and `remote-exec` `inline` lists
    - docker-compose: `command:`/`entrypoint:` when shell form is detected
    - `package.json` scripts (warn-first rollout; tons of “documentation strings” to avoid)
  - output formats: `--format pretty|json` with `file:line[:col]`, extracted command, decision, rule id, suggestion
  - CI controls: `--fail-on error|warning|none`, `--max-file-size`, `--exclude`, `--redact`
  - provide `dcg install-hook` and support for hook managers (husky/lefthook/pre-commit)
- Risks: large scope; mitigate by shipping a minimal MVP (bash + YAML run blocks) and expanding with tests and clear config.

### 34) CI/GitHub Action scanning for PRs (diff-only)

- How: run `dcg scan` in CI on files changed in PR; post results as PR comment/check; optionally fail build on “error” severity.
- User perception: “team-wide guardrails even if local hooks are skipped.”
- Implementation: simplest version is a CI job that runs `dcg scan --changed-files ... --format json` and formats output. A polished version is a GitHub Action wrapper.
- Risks: false positives at org scale; mitigate with allowlist, severity modes, and “warn-first rollout.”

### 35) Regression corpus directory (high-signal real-world cases)

- How: maintain a corpus of commands representing known false positives, true positives, bypass attempts, and edge cases; tests iterate these files.
- User perception: fewer regressions; easier to reproduce bugs.
- Implementation: `tests/corpus/{false_positives,true_positives,edge_cases,bypass_attempts}` + a harness test.
- Risks: file proliferation; mitigate by keeping corpus small and curated and by never adding redundant cases.

### 36) Property-based testing (proptest) for invariants (determinism, idempotence, no panics)

- How: use proptest to generate command strings and assert invariants: normalization idempotence, deterministic decisions, no panics, consistent quick reject behavior.
- User perception: indirectly “it never breaks on weird inputs.”
- Implementation: add proptest as dev-dependency; gate slower tests as ignored or feature-flagged.
- Risks: flaky tests; mitigate by controlling strategies and timeouts.

### 37) Fuzzing (cargo-fuzz) for the hook parser + tokenizer + heredoc extraction

- How: fuzz entrypoints most likely to panic/hang: JSON input parsing, shell tokenizer, heredoc extraction.
- User perception: indirectly “hook doesn’t crash.”
- Implementation: fuzz targets; time-limited CI fuzz runs; keep corpora.
- Risks: setup complexity; mitigate with minimal initial targets and CI time limits.

---

## The best strategy (deep rationale)

This section is intentionally long. It’s where we spell out *how* the system should work and *why* it will be perceived as better in practice.

### #1 (Must-fix substrate): Correctness + determinism + shared evaluator (hook parity)

This is “foundation work.” Without it, any advanced features are unreliable.

#### What we must guarantee

1. If a pack is enabled and its keywords appear, it is evaluated in hook mode.
2. Decisions are deterministic (stable pack ordering).
3. A block has stable identity: `pack_id` + `pattern_name`.
4. Hook mode and CLI (“test”) run the same logic.

#### Why users care

- “It actually blocks docker/kubectl/DB, not just git/rm.”
- “It gives the same answer every time.”
- “`dcg test` matches production behavior.”

#### Implementation notes (concrete)

- Replace `git/rm`-only early return with enabled-pack keyword union gating.
- Evaluate packs in a stable tiered order, and return the first blocking match deterministically.
- Unify evaluator (single function) used by:
  - hook JSON path
  - `dcg test`
  - `dcg explain`
  - `dcg scan` (pre-commit/CI)
- Remove legacy matcher only after parity tests pass.

#### Tests (non-negotiable)

- Integration tests proving non-core packs block in hook mode.
- Parity tests: hook JSON decision == CLI decision for a command matrix.
- Determinism test: same input yields same `pack_id:pattern_name` repeatedly.

---

### #2 (Trust unlock): Execution-context classification + safe string-arg registry + project allowlists

#### Problem

Today’s most damaging failure mode is blocking when the dangerous substring is **data** (docs, commit messages, issue descriptions, grep patterns). This creates:

- velocity-destroying interruptions for coding agents
- rapid loss of trust (“this tool is dumb / gets in the way”)
- eventual disablement of the guard

#### Proposal

Introduce an execution-context layer that answers:

> “Which bytes of the command are actually executed code, and which bytes are merely data passed to some tool?”

Then:

- Only apply destructive patterns to executable contexts.
- Never block purely data contexts for safe commands.

This is intentionally conservative:

- If ambiguous, treat as executable (or run deeper analysis).
- Don’t try to be “smart” by default; instead be correct for common workflows.

#### How it works (concrete)

At a high level, this introduces “span semantics” for command text:

- `Executed`: words that will be executed as part of the shell command graph
- `InlineCode`: strings passed to interpreters (`bash -c`, `python -c`, `node -e`, etc.)
- `HeredocBody`: extracted script content (Tier 2+)
- `Data`: clearly non-executed strings (commit messages, descriptions, grep patterns)
- `Unknown`: ambiguous; treat as executable unless proven otherwise

1. Tokenize command into pipeline segments and operators:
   - `|`, `&&`, `||`, `;`
2. Identify executable “command words”:
   - first word of each segment (after wrappers like `sudo`, `env VAR=...`)
3. Identify special executable contexts:
   - command substitution: `$(...)`
   - backticks
   - `xargs` (phase 2/optional)
4. Apply a **safe string-arg registry**:
   - treat values of doc flags as data spans:
     - `git commit -m`, `bd create --description`, `rg -e`, etc.
5. Build a list of spans:
   - Executed (eligible for pack matching)
   - InlineCode (eligible; e.g., `bash -c "..."`, `python -c "..."`)
   - Data (never eligible for direct pack matching)
6. Evaluate packs only against eligible spans.

#### Why users will love it

This is the “trust unlock.” Users experience:

- fewer unwarranted blocks
- fewer manual intervention loops
- confidence that blocks correspond to real execution risk

They stop thinking “this tool is annoying” and start thinking “this tool is quietly protecting me.”

#### Implementation plan (pragmatic phases)

Phase A: Safe string-arg registry (fast ROI)

- implement a small registry for bd/git/rg/grep/echo/printf
- write unit tests for each entry
- add E2E regression cases for common doc strings

Phase B: Minimal conservative tokenizer

- handle quotes/escapes, pipes, separators, `$()`, backticks
- treat ambiguous constructs as executable
- add unit tests for quoting + substitution + pipes

Phase C (optional): token-aware keyword gating reuse

#### Tests (must-have)

- “Must allow” suite: docs and commit messages with dangerous substrings
- “Must block” suite: actual execution contexts (`bash -c`, pipe targets, substitution)
- fuzz/property tests optional but high value

#### Why I’m confident

This solves the most painful high-frequency problem with incremental, testable steps and minimal performance risk.

---

### #3 (Adoption unlock): Explain mode + suggestions + safe allowlisting workflows

#### Problem

Even with a perfect detector, users will disable the tool if it feels opaque. When blocked, users ask:

- Why did this match?
- What exactly matched (span)?
- Which packs were checked vs skipped?
- What should I do instead?
- How do I allow this safely if it’s legitimate?

#### Proposal

Provide an explicit “debug and resolve” workflow:

1. `dcg explain "<cmd>"` prints a full decision trace (pretty + json + compact formats).
2. Deny output prints stable rule ID and copy/paste allow command (hook-safe).
3. `dcg allow ...` and `dcg allowlist ...` manage a project allowlist file with audit trail.
4. Suggestions database provides safe alternatives and links to deeper docs.

#### User impact

- “I can understand any block in seconds.”
- “I can resolve false positives safely without editing source code.”
- “I can share allowlist fixes with my team (code review).”

#### Explain mode: recommended output formats

- **Pretty**: box-drawing + colors + step timing
- **JSON**: machine-readable trace for bug reports / CI
- **Compact**: one-line summary (`DENY pack:pattern "matched" — reason (0.9ms)`)

#### Explain mode: what the trace must show (minimum viable “trust”)

The trace must answer, explicitly:

1. **What input did you actually analyze?**
   - raw command (as received)
   - normalized command (path stripping, wrapper stripping)
2. **Why was this command analyzed at all?**
   - global keyword gating decision (which keyword triggered)
3. **What was considered executable vs data?**
   - execution-context classification summary (spans / segment commands)
4. **What allowlists applied?**
   - which allowlist layer matched (project/user/system), if any
5. **Which pack/pattern won and where did it match?**
   - stable rule ID: `pack_id:pattern_name`
   - matched text + byte span (and ideally token span)
6. **What should the user do next?**
   - safe alternatives
   - “how to allow safely” command examples

Recommended pipeline steps (trace entries):

- Input parse (hook JSON vs CLI input)
- Global quick reject / keyword gating
- Normalization (including wrappers)
- Execution-context classification
- Allowlist check (project → user → system, or chosen precedence)
- Pack evaluation:
  - safe patterns (if relevant)
  - destructive patterns
- Policy layer (deny vs warn vs log-only)
- Output formatting summary (what was emitted to stderr/stdout)

#### Suggestions: make the tool helpful, not just restrictive

For each blocking rule, attach (when available):

- safe alternative commands (with placeholders, e.g., `HEAD~N`)
- “preview first” commands (`git clean -n`, `docker system df`, `kubectl --dry-run=client`)
- docs links (local README anchors are fine; external docs optional)
- allowlist commands that are safe-by-default:
  - allow by rule ID in project allowlist
  - allow once (no persistence)

#### Allowlisting: safest shapes first

Prefer allowlists that are hard to abuse:

1. allow by **rule ID**: `core.git:reset-hard`
2. allow by **exact command** (rare; often too specific)
3. allow by **prefix + context** (e.g., `git commit -m` in string-argument context)
4. allow by **regex** only with explicit risk acknowledgment

Project allowlist lives in repo root (example): `.dcg/allowlist.toml`

#### Learning workflow (hook-safe)

When blocked, dcg should print something like:

```
To understand: dcg explain "<command>" --format pretty
To allow for this repo: dcg allow --rule core.git:reset-hard --reason "..." --project
```

No interactive prompt in hook mode.

#### Implementation plan

Phase A: Explain trace infrastructure (CLI-only)

- evaluator optionally collects trace steps behind a flag
- hook path passes `None` for zero overhead

Phase B: Suggestions DB + match visualization

- map `pack_id:pattern_name` to suggestions
- show matched span and matched text

Phase C: Project allowlist file + CLI helpers

- `.dcg/allowlist.toml` parsing/validation
- `dcg allow`, `dcg allowlist list/remove/validate`
  - `dcg allow --once ...` (one-shot allow, no persistence)
  - `dcg allow --project ...` (writes to project allowlist)
  - `dcg allow --user ...` (writes to user config allowlist)

#### Tests

- golden tests for explain output (compact + JSON at least)
- allowlist correctness tests (only bypasses intended rule)

#### Why I’m confident

This is a standard trust-building playbook: transparency + safe customization dramatically improves adoption.

---

### #4 (Safety unlock): Tiered heredoc + inline scanning (RegexSet → extraction → AST)

#### Problem

Agents can hide destructive behavior inside:

- heredocs (`python3 << EOF ... EOF`)
- inline scripts (`bash -c`, `python -c`, `node -e`)
- piped interpreters (`curl ... | bash`)

Regex-only top-level command scanning misses these, creating dangerous false negatives.

#### Proposal

Tiered architecture that keeps the common case fast:

Tier 1: ultra-fast trigger (allow false positives, no false negatives)
- detect heredoc operators `<<`, `<<-`, `<<~`, here-strings `<<<`
- detect interpreter inline flags `-c`, `-e`, `-p`
- detect “pipe to interpreter” patterns

Tier 2: bounded extraction
- extract heredoc bodies and inline code strings safely
- enforce size/line/time limits
- on failure: fail-open with a warning/log (never hang)

Tier 3: AST-aware matching (ast-grep-core or tree-sitter directly)
- match high-signal constructs per language:
  - python: `os.system`, `subprocess.*`, `shutil.rmtree`, etc.
  - node: `child_process.exec*`, `fs.rmSync`, etc.
  - bash: `rm -rf`, `git reset --hard`, etc.

#### User perception

This is the “compelling” feature:

- “It catches the clever bypasses.”
- “It’s not naive regex.”

#### Implementation notes

The key is bounding work:

- typical command: microseconds
- heredoc path: single-digit milliseconds
- hard cap + fail-open to avoid hangs

#### Tests

- unit tests for detection + extraction variants
- E2E: heredoc variants and bypass attempts
- robust error/limit tests: oversized heredocs, malformed delimiters, timeouts

#### Why I’m confident

Tiering is a proven strategy to keep latency small while providing deep analysis where needed.

---

### #5 (Compelling unlock): Team-wide scanning (pre-commit + CI) + “serious” test infrastructure

This turns dcg from “a hook for agent commands” into a broader safety layer teams can rely on.

#### What we add

1. `dcg scan` that scans files (staged or on disk) by extracting executable command contexts and evaluating them with the same engine.
2. Pre-commit hook integration: `dcg install-hook` installs `.git/hooks/pre-commit` or prints config for husky/lefthook/pre-commit.
3. CI integration (GitHub Action or simple workflow) scanning changed files in PRs.
4. “Serious” tests:
   - regression corpus
   - proptest invariants
   - fuzzing of parsers/tokenizers/extractors
   - performance benchmarks

#### Key design constraint: extract commands, don’t naive-grep

To avoid noisy false positives, scanning should extract actual command strings from:

- shell scripts: command lines (with basic parsing)
- YAML CI: `run:`/`script:` values
- Dockerfile: `RUN` lines
- Makefile: recipe lines (tab-indented)
- Terraform: `local-exec` / provisioner command strings

Then run the dcg evaluator on those extracted commands, applying the same context rules and allowlists.

#### Scan output must be reviewer-friendly and machine-friendly

We should support:

- **Pretty** output for humans (similar style to deny boxes)
- **JSON** output for CI and PR comments

Each finding should include:

- file path
- line/column (best effort; at least line)
- extracted command (truncated with “show full in JSON” behavior)
- `pack_id:pattern_name`
- reason
- severity (error/warn/info)
- suggestions (optional)

#### Pre-commit design constraints (avoid new false positives)

- Default to scanning only staged changes (diff-only) and only in recognized command-bearing contexts.
- Treat commit messages as **data** by default; warnings only (configurable).
- Provide a “warn-first rollout” mode so teams can adopt without blocking merges immediately.

#### User perception

- “We catch dangerous stuff before it hits main.”
- “It works even if someone skips local hooks.”
- “This feels like a real enterprise safety layer.”

#### Implementation phases (MVP first)

Phase A: `dcg scan` MVP

- scan `.sh` + `Dockerfile` + CI YAML `run:` blocks
- output findings (file:line, pack/pattern, reason, suggestion)

Phase B: pre-commit integration

- scan staged files
- configurable severity threshold (`fail_on=error|warning|none`)

Phase C: CI wrapper (diff-only)

- scan changed files; post report
- fail build based on severity threshold

#### Test infrastructure details (high value)

- Regression corpus of high-signal cases checked into `tests/corpus/...`
- proptest invariants:
  - normalization idempotence
  - deterministic decision
  - no panics
- fuzz targets:
  - hook JSON parser
  - tokenizer
  - heredoc extractor

#### Why I’m confident

These integrations are extremely compelling in real-world practice and reuse the same engine, so they are accretive rather than a separate product.

---

### Cross-cutting: Performance + safety guardrails (limits, hardening, benchmarks)

#### Problem

Hook tooling must remain invisible. As features grow (context parsing, heredocs, AST), performance regressions and hangs become the existential risk.

#### Proposal

- Precompile overrides and patterns to avoid per-command compilation
- Add strict size/time limits to expensive paths (heredoc/AST)
- Introduce benchmarks and budgets to prevent slow creep
- Prefer linear-time regex where possible

#### User perception

- “It never stalls my terminal.”
- “It’s fast enough that I forget it’s there.”

#### Implementation

Start with cheap, high-value caps:

- max command length
- max heredoc body bytes/lines
- max parse time

Then add microbench baselines.

#### Why I’m confident

These guardrails are low-risk, easy to test, and essential for long-term sustainability.

---

## Implementation roadmap (phased, test-first)

This is the sequence that minimizes rework and maximizes early user trust.

### Phase 0 (Days 1-3): Correctness substrate (unblock everything)

- pack-aware global quick reject (enabled packs actually run)
- deterministic pack ordering
- stable match identity (`pack_id:pattern_name`)
- minimal regression tests proving non-core packs work in hook mode

### Phase 1 (Week 1-2): Unify evaluator + remove duplication

- shared evaluator API used by hook and CLI
- parity tests (hook JSON path == CLI test)
- delete legacy matcher only after parity
- compile overrides once (remove per-command regex compilation)

### Phase 2 (Week 2-3): False positive immunity MVP

- safe string-arg registry v1
- conservative execution-context tokenizer
- E2E regression suite for false positives vs real execution contexts
- optional token-aware keyword gating

### Phase 3 (Week 3-5): Deep bypass coverage (heredoc/inline)

- Tier 1 triggers (RegexSet)
- bounded extraction with limits + fail-open logging
- language inference heuristics
- AST matching MVP for top languages
- dedicated heredoc E2E suite

### Phase 4 (Week 5-6): Explainability + allowlist ergonomics

- `dcg explain` pretty/json/compact
- suggestions DB
- project allowlist `.dcg/allowlist.toml` + CLI management
- deny output prints “how to allow safely” commands (hook-safe)

### Phase 5 (Week 6-8): Team integrations + hardening

- `dcg scan` MVP (scripts + CI YAML + Dockerfile)
- pre-commit hook integration (scan staged)
- CI/GitHub Action wrapper (diff-only)
- regression corpus + proptest + fuzzing + benchmarks in CI

**Important:** This roadmap is sequencing, not a promise. The principle is: *foundation first; then trust; then bypass coverage; then productization; always test-first.*

---

## Notes (where this plan is tracked in Beads)

The plan above is implemented as a detailed dependency graph in Beads, including:

- `git_safety_guard-99e` (P0) Core correctness/determinism
- `git_safety_guard-t8x` (P1) False positive immunity
- `git_safety_guard-yza` (P1) Heredoc/AST epic (existing)
- `git_safety_guard-1gt` (P2) Explainability/UX

This document is intentionally self-contained; Beads are the execution ledger.

---

## Appendix: File formats & UX sketches

This appendix is intentionally concrete so we don’t have to “rethink” these basics later.

### A.1 Project allowlist file (`.dcg/allowlist.toml`)

Goals:

- team-shared (committed + code-reviewed)
- safe by default (rule-id or exact/prefix)
- auditable (who/when/why)
- supports expiration for temporary allows

Example:

```toml
# .dcg/allowlist.toml

[[allow]]
rule = "core.git:reset-hard"
reason = "Repo migration playbook requires this step (documented)."
added_by = "alice@example.com"
added_at = "2026-01-08T01:23:45Z"
expires_at = "2026-02-01T00:00:00Z"

[[allow]]
command_prefix = "git commit -m"
context = "string-argument"
reason = "Commit messages often mention dangerous commands."
added_by = "interactive-cli"
added_at = "2026-01-08T01:24:00Z"

[[allow]]
command_prefix = "bd create"
context = "string-argument"
reason = "Issue descriptions are documentation."

[[allow]]
pattern = "echo\\s+\"Example:.*rm -rf.*\""
reason = "Documentation examples in echo."
risk_acknowledged = true
```

### A.2 Explain output sketch (compact)

```
DENY core.git:reset-hard "git reset --hard" — Destroys uncommitted changes (0.84ms)
```

### A.3 Explain output sketch (JSON)

Key fields:

- `decision`: allow/deny
- `total_duration_us`
- `steps[]`: step name, duration, details
- `match`: pack_id, pattern_name, matched_span, matched_text, reason
- `suggestions[]`: safe alternatives, docs, allow commands

### A.4 Pre-commit scan config sketch (`.dcg/hooks.toml`)

```toml
[pre_commit]
scan_patterns = ["*.sh", "Dockerfile*", ".github/**/*.yml", "*.yaml", "Makefile"]
exclude_patterns = ["vendor/**", "node_modules/**"]
fail_on = "error" # error|warning|none
max_file_size = 1048576
format = "pretty" # pretty|json
```

### A.4.1 `pre-commit` framework config sketch (`.pre-commit-config.yaml`)

```yaml
repos:
  - repo: local
    hooks:
      - id: dcg-scan
        name: dcg scan (staged)
        entry: dcg scan --staged --format pretty --fail-on error
        language: system
        pass_filenames: false
```

### A.4.2 `lefthook` config sketch (`lefthook.yml`)

```yaml
pre-commit:
  commands:
    dcg-scan:
      run: dcg scan --staged --format pretty --fail-on error
```

### A.4.3 `husky` config sketch (`.husky/pre-commit`)

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

dcg scan --staged --format pretty --fail-on error
```

### A.4.4 GitHub Actions workflow sketch (PR scanning)

This is intentionally “simple but correct”:

- run the same engine as hook mode uses
- scan only the PR diff (avoid scaling pain)
- upload JSON for debugging

```yaml
name: dcg scan
on:
  pull_request:
jobs:
  dcg:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Build dcg
        run: cargo build --release
      - name: Scan changed files (JSON)
        run: |
          ./target/release/dcg scan --git-diff origin/${{ github.base_ref }}...HEAD --format json --fail-on error > dcg_scan.json
      - name: Upload report artifact
        uses: actions/upload-artifact@v4
        with:
          name: dcg_scan
          path: dcg_scan.json
```

### A.5 Explain mode “pretty” sketch (more explicit)

The goal is not “pretty for its own sake.” The goal is: users can debug quickly and *trust the system*.

```
$ dcg explain "git reset --hard HEAD~5"

╔════════════════════════════════════════════════════════════════════╗
║                       DCG Decision Analysis                        ║
║                                                                    ║
║  Input:     git reset --hard HEAD~5                                ║
║  Decision:  DENY                                                   ║
║  Rule:      core.git:reset-hard                                    ║
║  Latency:   0.84ms                                                 ║
╠════════════════════════════════════════════════════════════════════╣
║ PIPELINE TRACE                                                     ║
║  [1] Keyword gating: hit "git" → full analysis                      ║
║  [2] Normalize: no change                                           ║
║  [3] Context: Executed span = full command                          ║
║  [4] Allowlist: none matched                                        ║
║  [5] Packs: core.git matched reset-hard at bytes 0..15              ║
╠════════════════════════════════════════════════════════════════════╣
║ MATCH                                                              ║
║  git reset --hard HEAD~5                                            ║
║  ▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔                                                     ║
╠════════════════════════════════════════════════════════════════════╣
║ SUGGESTIONS                                                        ║
║  • git stash                         (save work first)             ║
║  • git reset --soft HEAD~5           (keeps changes staged)         ║
║  • git reset --mixed HEAD~5          (keeps changes unstaged)       ║
║                                                                    ║
║ To allow for this repo (use with caution):                          ║
║  dcg allow --rule core.git:reset-hard --reason "..." --project      ║
╚════════════════════════════════════════════════════════════════════╝
```

### A.6 Allowlist validation rules (what we should enforce)

Allowlist validation should be opinionated, because the goal is “safe customization.”

- `rule = "pack:pattern"` entries:
  - must refer to a known rule ID (error if unknown unless `--allow-unknown` is set)
  - may be conditional (env/project path)
- `command_prefix = "..."` entries:
  - should require a `context` (string-argument, search-pattern, etc.) unless explicitly acknowledged as risky
- `pattern = "..."` regex entries:
  - must require `risk_acknowledged=true`
  - should warn if regex is overly broad (heuristics: `.*rm -rf.*` etc.)
- `expires_at`:
  - if expired, ignored with a warning in `dcg allowlist validate`
- Invalid regex:
  - must not crash the hook; should warn and ignore (fail-open)

### A.7 Regression corpus layout (recommended)

```
tests/corpus/
  false_positives/
    bd_create_description.txt
    git_commit_message.txt
    grep_pattern.txt
    echo_example.txt
  true_positives/
    rm_rf_root.txt
    git_reset_hard.txt
    docker_system_prune.txt
  bypass_attempts/
    semicolon_injection.txt
    pipe_to_bash.txt
    command_substitution.txt
  edge_cases/
    unicode.txt
    very_long.txt
    nested_quotes.txt
```

The harness should run these through both:

- hook JSON path
- CLI test path

…to ensure parity.

### A.8 Property tests & fuzzing (invariants worth encoding)

Property tests (proptest) should prioritize “tool never behaves dangerously weird”:

- normalization idempotence
- deterministic decision
- no panics
- size limit enforcement (never allocate unboundedly)

Fuzz targets should include:

- hook JSON parser
- tokenizer
- heredoc extractor
- (later) AST matcher wrapper with strict timeouts
