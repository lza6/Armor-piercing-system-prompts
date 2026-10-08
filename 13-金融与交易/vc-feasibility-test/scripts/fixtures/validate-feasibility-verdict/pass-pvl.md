---
slug: pvl-spike-example
date: 14-06-26
verdict: VIABLE
originating-phase: pvl
---

# Feasibility Verdict — Does the plan-validate loop accept a VERDICT from the pvl phase?

## Hypothesis
A VERDICT artifact produced during the inner PVL phase (not SPEC or INNOVATE) is a valid
originating context for a feasibility probe and must be accepted by the validator.

## Mechanism Under Test
The `originating-phase` field in the VERDICT frontmatter. Specifically whether `pvl` is
a recognised value alongside `spec` and `innovate`.

## Probe Family
1 — Static analysis / code inspection

## Probe Cost Class
`cheap-local`. No container, live provider, or browser required.

## Probe Method
```bash
node .claude/skills/vc-feasibility-test/scripts/validate-feasibility-verdict.mjs \
  .claude/skills/vc-feasibility-test/scripts/fixtures/validate-feasibility-verdict/pass-pvl.md
```
Confirmed `originating-phase: pvl` passes the validator with exit 0.

## Evidence Captured
```json
{
  "file": "...",
  "sectionsFound": ["Hypothesis", "Mechanism Under Test", "Probe Family", "Probe Cost Class", "Probe Method", "Evidence Captured", "Verdict", "Resulting Design Constraint"],
  "verdictValue": "VIABLE",
  "costClass": "cheap-local",
  "originatingPhase": "pvl",
  "failures": []
}
```

## Verdict
VIABLE

## Resulting Design Constraint
- **What this licenses:** VERDICT artifacts with `originating-phase: pvl` are fully valid and
  accepted by the D1 validator. Probes triggered during inner PVL need no special treatment.
- **What this forbids:** any approach that rejects `pvl` as an originating phase or treats it as
  less valid than `spec` or `innovate`.
- **What remains uncertain (known-gap):** whether a fourth originating phase (e.g. `execute`)
  should ever be added — left as a future extension decision.
