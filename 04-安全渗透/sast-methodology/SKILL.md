# SAST Methodology

## Core Principle: Decomposed Reasoning

A single agent asked to "find vulnerabilities" will hallucinate. The pipeline decomposes the task into focused steps, with external state carrying the synthesis between steps. All agents run on `model: "inherit"` except flow-tracing and gap-analysis, which pin opus for cross-file reasoning depth.

```
File Ranking (inherit) → comprehension
    ↓
Entry Point Mapping (inherit) → reading + listing
    ↓
Dangerous Op Mapping (inherit) → pattern matching
    ↓
Flow Tracing (pinned Opus) → cross-file reasoning
    ↓
Gap Analysis (pinned Opus) → interaction reasoning
    ↓
Devil's Advocate (inherit) → adversarial checking
    ↓
PoC Confirmation (inherit) → targeted coding + ASan
    ↓
Exploit Development (inherit) → exploitation techniques
```

Each agent does ONE thing well. The pipeline does the synthesis.

## Why Decomposition Works

The SACK bug requires simultaneously understanding: (1) missing bounds check on sack_start, (2) signed integer arithmetic in SEQ_LEQ, (3) linked list behavior when only node deleted, (4) how they interact. No current model reliably holds all four in synthesis.

Decomposed:
- Entry mapper finds: "sack_start comes from network, sack_end checked but sack_start not"
- Danger mapper finds: "SEQ_LEQ uses (int)(a-b), linked list walk with single-node edge case"
- Flow tracer connects: "sack_start flows to SEQ_LEQ comparison then affects linked list walk"
- Gap analyzer reasons: "unbounded sack_start + signed overflow = impossible condition satisfiable"

Each step is shallow. The pipeline depth is structural.

## Static Analysis Integration

Deterministic tools (CodeQL, Semgrep, Cppcheck) catch mechanically-detectable patterns. AI catches subtle interaction bugs they miss. Best results come from combining both:
- Static tools → concrete warnings with file:line
- AI pipeline → interaction gaps and semantic bugs
- Merge candidates → deduplicate → adversarial validation → PoC

## Adversarial Validation

The devil's advocate agent exists because AI hallucination is the #1 cost sink. Hallucinated findings look plausible, reference real functions, describe coherent root causes — and are completely wrong.

The disproval checklist catches:
1. Hallucinated code (function doesn't exist)
2. Unreachable paths (only called from tests)
3. Missed checks (wrapper adds validation the gap-analyzer didn't see)
4. Wrong types (actual macro expands differently than assumed)
5. Impractical triggers (requires winning impossible race)

## Best-of-N for High-Value Targets

On score-5 files, running multiple independent hunter instances catches more bugs and filters hallucinations:
- Finding in 2+ independent runs → almost certainly real
- Finding in only 1 run → might be hallucinated, flag for manual review
- Cost multiplier 3-5x per file, only justified on critical files

## Verification Tools

| Tool | Use |
|---|---|
| ASan | Heap/stack overflow, UAF, double-free, OOB |
| UBSan | Integer overflow, null deref, alignment |
| MSan | Uninitialized memory reads |
| TSan | Data races, deadlocks |
| Valgrind | When sanitizers unavailable |
| GDB | Debugging, register inspection |

### Compilation Flags
```bash
# C/C++:
CFLAGS="-fsanitize=address,undefined -fno-omit-frame-pointer -g -O1"
# Rust (nightly):
RUSTFLAGS="-Z sanitizer=address" cargo +nightly build
# Go:
go build -race
```

## Exploitation Tiers

1. DoS (controlled crash)
2. Controlled write primitive
3. Info leak / ASLR bypass
4. Control flow hijack (register control)
5. Code execution (shell, file write)

## Realistic Expectations with Current Models

**Opus 4.6 can**: rank files, map entry points and dangerous operations, trace simple data flows, find missing bounds checks, write PoCs for straightforward bugs.

**Opus 4.6 struggles with**: multi-step interaction bugs, novel exploitation techniques, heap feng shui, complex ROP chains, bugs requiring understanding of compiler optimization behavior.

**The decomposed pipeline helps because**: it turns "find an interaction bug" (hard, fails ~95% of the time) into "map entries" + "map dangers" + "connect them" + "check gaps" (each succeeds ~80% of the time). Pipeline success rate: 0.8^4 ≈ 40% — better than 5%, still not 76% (Mythos).

## Cost Management

| Phase | Model | Cost/file | Skip when |
|---|---|---|---|
| File ranking | inherit | ~$0.05 | Never (cheapest, most important) |
| Entry mapping | inherit | ~$0.10 | Score < 3 |
| Danger mapping | inherit | ~$0.10 | Score < 3 |
| Flow tracing | **Opus** (pinned) | ~$0.50 | Score < 4 |
| Gap analysis | **Opus** (pinned) | ~$0.50 | No hot/warm flows found |
| Devil's advocate | inherit | ~$0.10 | Never (cheapest hallucination filter) |
| PoC building | inherit | ~$0.20 | No surviving candidates |
| Exploit dev | inherit | ~$0.30 | Severity < medium |

Budget 30 files at min-score 4: scales with the orchestrator model — opus end-to-end
runs higher than a mixed pipeline, but removes model-tier handoffs between phases.
