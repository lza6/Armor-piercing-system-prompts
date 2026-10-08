# dcg cold-start hotspot table — 2026-08-05

**Scenario:** hook-mode (`PreToolUse`, JSON on stdin), process-per-invocation — the real path every agent Bash command takes. **Metric:** cold wall-clock latency, dcg cost isolated as `full_eval − DCG_BYPASS`. **Budget:** `HOOK_EVALUATION_BUDGET_MS = 1000`.

**Fingerprint:** Apple M4 Pro (10P+4E), 64 GiB, macOS 26.2, rustc 1.98.0-nightly, git `4dc38ba`. Shipping `release` binary (opt-level "z", regex family at opt-level 3, LTO, strip). 250 cold spawns/config, median + p95. Attribution by differential (config deltas cancel the process-spawn floor). Raw: `differential.txt`, `perf_baseline`-style p50/p95 above.

## Absolute baseline (p50 / p95, ms)

| case | p50 | p95 | dcg cost (p50 − bypass) |
|---|---|---|---|
| bypass (floor) | 10.3 | 15.0 | — |
| quick_reject (`ls -la`) | 11.6 | 18.0 | **1.3** |
| destructive (`git reset --hard`) | 17.0 | 18.6 | **6.7** |
| heredoc (`sh -c 'rm -rf …'`) | 18.0 | 21.0 | **7.7** |
| kubectl delete ns, 12 packs | 17.3 | 20.2 | **7.0** |
| multi-construct (worst observed) | 28.9 | 37.9 | **18.4** |

dcg is **35×–90× inside its 1000 ms budget.** The dominant *absolute* cost (~10 ms) is OS process spawn + dyld load of the 25 MB binary + stdin — outside the evaluation deadline and not dcg-evaluation-optimizable.

## Ranked hotspots (dcg-controllable cost)

| Rank | Location | Metric | Value | Category | Evidence |
|---|---|---|---|---|---|
| 1 | matched-pack regex compilation (`PackEntry::get_pack` → `LazyCompiledRegex` build for the pack a command's keywords hit) | delta, cold | **+5.4–5.8 ms per matched pack** | CPU (regex compile) | differential.txt P4−P1 (+5.39), P5−P3 (+5.80) |
| 2 | heredoc Tier-2/3 parse (tree-sitter/ast-grep init + parse) on `sh -c`/`python -c`/heredoc commands | delta, cold | **+6.0 ms** | CPU (parser) | differential.txt P6−P2 (+6.02) |
| 3 | base setup: `Config::load` + pack-enable resolution + keyword index for the default 3 packs | delta, cold | **+1.3 ms** | CPU/alloc | differential.txt P1−P0 (+1.33) |
| — | heredoc trigger-set (`RegexSet`) init | delta, cold | +0.34 ms | CPU | P2−P1 |

## Refuted hypotheses (negative evidence)

- **"More enabled packs slows every invocation"** — REJECTED. 3→12 packs changed the keyword-index/quick-reject cost by −0.14 ms (noise): P3−P1. The aho-corasick `EnabledKeywordIndex` scales for free; keyword pre-filtering means only the *matched* pack compiles. Enabling more packs is nearly free until one matches.
- **"Heredoc cost scales with pack count"** — REJECTED. Heredoc-with-12-packs vs heredoc-core: +0.04 ms (P7−P6). The cost is the parse, not pack fan-out.
- **"The keyword index / quick-reject is the hot path"** — REJECTED. A no-match command (quick reject) costs only 1.3 ms total; the index build is a fraction of that.

## Confirmed hypothesis

**Regex compilation of the matched pack is the #1 dcg-controllable cost**, ~5–6 ms per matched pack, paid fresh on every cold process because compilation is `LazyLock`/lazy-per-pattern and "first use" == "only use" in a one-shot process. This corroborates the #245/#248 finding recorded in `Cargo.toml` (the `opt-level = 3` override on the regex crates is the existing mitigation) and the deferred "build-time DFA serialization via regex-automata `from_bytes`" idea.

## Highest-EV optimization lever (for extreme-software-optimization)

**Build-time DFA serialization / lazier pattern compilation** for the always-on `core.git` + `core.filesystem` packs (matched by the most common destructive commands). `regex-automata`'s `DFA::from_bytes` deserializes a precompiled automaton in ~O(1) instead of rebuilding the NFA→DFA each cold start. Impact: could remove most of the ~5–6 ms rank-1 cost on the commands that hit it. Effort/risk: HIGH — it is an architectural change to the pattern-compilation layer of a security-critical matcher; every pattern's serialized form must be proven byte-identical in behavior, and `fancy-regex` (backtracking, ~15% of patterns) cannot be serialized this way. Confidence the win materializes: MEDIUM.

**EV caveat:** the absolute budget headroom is 35–90×. This lever shaves ~5 ms off a ~17 ms cold start whose ~10 ms floor is un-addressable OS spawn. It is the only lever above the `Impact × Confidence / Effort ≥ 2.0` bar, and even it is marginal given the headroom. Micro-optimizing the warm evaluation path (already sub-ms per the criterion benches) is below the bar.
