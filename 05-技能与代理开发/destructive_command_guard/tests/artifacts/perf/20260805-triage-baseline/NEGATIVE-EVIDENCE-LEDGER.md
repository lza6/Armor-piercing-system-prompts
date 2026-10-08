# dcg optimization — negative-evidence ledger (2026-08-05)

Applying `extreme-software-optimization`'s gate — **implement only Score ≥ 2.0 (Impact × Confidence / Effort), one lever, behavior proven unchanged** — to the ranked hotspots from `HOTSPOTS.md`. Baseline: dcg runs **35×–90× inside its 1000 ms hook budget**; cold p50 11.6–28.9 ms of which ~10 ms is the un-addressable OS process/dyld floor.

## Scored opportunity matrix

| # | Target | Impact | Conf | Effort | Score | Verdict |
|---|--------|:-:|:-:|:-:|:-:|---|
| 1 | Build-time DFA serialization for always-on `core.*` (`regex-automata::DFA::from_bytes`) to erase ~5 ms matched-pack compile | 4 | 2 | 5 | **1.6** | REJECT — below bar |
| 2 | Precompile/cache the heredoc tree-sitter parser to erase ~6 ms parse | 3 | 2 | 5 | **1.2** | REJECT |
| 3 | Trim base-setup config-discovery syscalls (~1.3 ms) | 2 | 3 | 3 | **2.0** | REJECT — not pure waste (see below) |
| 4 | Micro-optimize warm evaluation (quick-reject, normalize, pack match) | 1 | 3 | 2 | **1.5** | REJECT — already sub-ms (criterion) |
| 5 | Shrink dyld floor by feature-gating tokio/reqwest/rusqlite out of the hook binary | 3 | 2 | 5 | **1.2** | REJECT |

**No target clears Score ≥ 2.0 at acceptable correctness risk.** Details:

## Evidence gathered and what it disproved

- **Compilation is already non-redundant.** `PackEntry::get_pack` (`OnceLock`) builds each pack once per process; `LazyCompiledRegex.compiled` (`OnceLock`) compiles each pattern once, lazily on first match. There is **no repeated or wasted compilation to remove** — the ~5 ms is the irreducible NFA→DFA construction cost of the matched pack's patterns, and the `regex` family is already built at `opt-level = 3` (the #245 mitigation). *Verified by reading `src/packs/mod.rs:886-911` and `src/packs/regex_engine.rs:317-382`.*
- **The base-setup 1.3 ms is legitimate config discovery, not waste.** `Config::load` stats the explicit/system/user config paths and `find_repo_root` walks up for `.git`/`.dcg.toml`. These syscalls *are* the layered-config and project-policy feature; removing them changes behavior (fails the isomorphism proof). *Verified: `src/config.rs:3738-3820`.*
- **Enabling more packs is nearly free** (differential P3−P1 = −0.14 ms). The aho-corasick `EnabledKeywordIndex` scales; only the *matched* pack compiles. No batching/index win available — it is already batched.
- **Warm evaluation is already sub-millisecond** (criterion `hook_latency`): the match itself is not a hotspot; only cold compilation is.

## Why target #1 is deferred, not done

Build-time DFA serialization is the only lever that could meaningfully cut the largest controllable cost (~5 ms → <1 ms on matched commands). It is **rejected for now** because:
1. **Correctness risk on a security matcher.** Every serialized automaton must be proven byte-behavior-identical to the live-compiled regex across the full corpus. A single divergence is a false negative — the exact failure class five adversarial review rounds were just spent eliminating.
2. **`fancy-regex` (~15% of patterns, all lookahead/backtracking) cannot be serialized this way** — it has no DFA form, so the win is partial.
3. **The budget headroom is 35–90×.** Shaving ~5 ms off a ~17 ms cold start whose ~10 ms floor is fixed OS cost is imperceptible to users and does not approach the deadline.

**Revisit condition:** if the hook deadline is ever tightened toward the observed cold p95 (~20–38 ms), or if a platform with a much cheaper process floor makes compilation the dominant term, target #1 becomes worth its effort and risk. Approach then: serialize only the linear-engine `core.*` patterns behind a build-time codegen step with a golden differential harness asserting byte-identical match/҂non-match over the full `tests/corpus` before the serialized form is trusted; keep live compilation as the fallback and the source of truth.

## Conclusion

The disciplined outcome of the campaign: **dcg's hot path is already optimized for its budget with no isomorphism-safe lever above the EV bar.** The profilable `release-perf` build, the ranked hotspot table, and this ledger are committed so a future latency-driven pass can start from evidence rather than re-derive it. Motion for its own sake here would violate the skill's own anti-patterns and risk correctness on a security tool — so the campaign correctly stops at zero code changes to the evaluation path.
