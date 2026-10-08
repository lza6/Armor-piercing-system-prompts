---
domain: harness
description: "vc-autoresearch domain config for the RIPER-5 coding harness. Formalizes the semi-manual self-improvement loop (self-improvement-loop.md) as a single bounded, logged command."
date: 10-06-26
predecessor: process/context/coding-harness-evals/self-improvement-loop.md
---

# vc-autoresearch — harness domain

Corpus: instruction files that guide the live orchestrator and its subagents.
Verify: run target harness scenarios and check exit code.
Guard: deterministic suite must stay green after every fix batch.
Frozen: vc-system-behavior spec files are NEVER modified.

## Domain Config

domain: harness
corpus:
  - .claude/agents/*.md
  - CLAUDE.md
  - .claude/skills/*/SKILL.md
  - process/development-protocols/*.md
  EXCLUDING: process/development-protocols/vc-system-behavior/**
frozen_files: process/development-protocols/vc-system-behavior/**
verify: pnpm test:runtime-harness
target: 0
guard: pnpm test:runtime-harness:unit
max_iterations: 10
research_agents: 2
fix_agents: 2
consecutive_all_clear: 2
auto_run: false

## Corpus Explanation

- `.claude/agents/*.md` — agent instruction files (most direct fix surface for misbehaviors)
- `CLAUDE.md` — orchestrator-level rules and routing
- `.claude/skills/*/SKILL.md` — skill reference docs agents load (additive fix surface)
- `process/development-protocols/*.md` (excluding vc-system-behavior/) — shared protocol prose

## Frozen Files (HARD CONSTRAINT)

process/development-protocols/vc-system-behavior/**

These are frozen RIPER-5 spec files. They document correct behavior; they do NOT configure live agent behavior. Misbehavior findings are recorded as learnings in iteration reports. The spec is never edited. Fix agents MUST refuse any instruction that would modify a file matching this pattern.

## Verify Command

pnpm test:runtime-harness

Run with the runtime-harness scenario selector env var scoped to the target scenario(s) that surface the misbehavior. Loop terminates when the verify command exits 0 (all targeted scenarios PASS).

## Guard Command

pnpm test:runtime-harness:unit

Runs the deterministic unit suite (unit mock scenarios; fast, $0, no live API calls). If the guard fails, the fix batch is reverted and a regression flag is logged. The extended live suite is NOT part of the guard — it is the `verify:` step.

## Research Thread List (per agent)

Agent 1 — agent files + CLAUDE.md:
- Does the agent file contain a mandatory bash gate for the misbehaving behavior?
- Is the instruction prose-only (cognitive) vs bash-enforced (mechanical)?
- What is the minimal bash gate that would prevent the misbehavior?

Agent 2 — skill SKILL.md files + protocol docs:
- Does the relevant SKILL.md have the correct trigger or check that the agent bypassed?
- Is the protocol doc prose actionable, or does it rely on cognitive recall?

## Fix Agent Assignment

Fix agent 1: agent .md files and CLAUDE.md
Fix agent 2: SKILL.md files and protocol docs (excluding frozen vc-system-behavior/)

## Predecessor Playbook

See process/context/coding-harness-evals/self-improvement-loop.md for the manual loop this domain automates. Key differences:
- Manual loop: human operator reads verdicts, decides edits, runs next drive
- harness domain: vc-autoresearch loop bookkeeper handles iteration counter, plateau detection, TSV logging, and parallel fix dispatch automatically

## Invocation Examples

Invoke after a live drive surfaces F-CTX-DISC (orchestrator uses ls instead of find):
  vc-autoresearch domain: harness corpus: .claude/agents/*.md CLAUDE.md verify: "<scenario-selector>=tier0 pnpm test:runtime-harness"

Invoke to fix F-PLAN-TIN (missing Test Infra Improvement Notes):
  vc-autoresearch domain: harness corpus: .claude/agents/vc-plan-agent.md verify: "<scenario-selector>=plan-artifact pnpm test:runtime-harness"

Full-corpus scan (all harness instruction files):
  vc-autoresearch domain: harness
