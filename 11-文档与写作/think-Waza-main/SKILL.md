---
name: think
description: Invoke before writing any code for a new feature, design, or architecture decision. Turns rough ideas into approved plans with validated structure. Not for bug fixes or small edits.
metadata:
  version: "3.8.0"
---

# Think: Design and Validate Before You Build

Prefix your first line with 🥷 inline, not as its own paragraph.


Turn a rough idea into an approved plan. No code, no scaffolding, no pseudo-code until the user approves.

Give opinions directly. Take a position and state what evidence would change it. Avoid "That's interesting," "There are many ways to think about this," "You might want to consider."

## Before Reading Any Code

- Confirm the working path: `pwd` or `git rev-parse --show-toplevel`. Never assume `~/project` and `~/www/project` are the same.
- Check `docs/solutions/` if present for prior decisions on the same problem.
- Search for related issues and PRs on GitHub before proposing anything.

## Propose Approaches

Offer 2-3 options with tradeoffs and a recommendation. Always include one minimal option. For each option: one-sentence summary, effort, risk, and what existing code it builds on.

For the recommendation, run four attack angles before presenting it:

| Attack angle | Question |
|---|---|
| Dependency failure | If an external API, service, or tool goes down, can the plan degrade gracefully? |
| Scale explosion | At 10x data volume or user load, which step breaks first? |
| Rollback cost | If the direction is wrong after launch, what state can we return to and how hard is it? |
| Premise collapse | Which assumption in this plan is most fragile? What happens if it does not hold? |

If an attack holds, deform the design and present the deformed version. If it shatters the approach entirely, discard it and tell the user why. Do not present a plan that failed an attack without disclosing the failure.

Get approval before proceeding. If the user rejects, ask specifically what did not work. Do not restart from scratch.

## Validate Before Handing Off

- More than 8 files or 1 new service? Acknowledge it explicitly.
- More than 3 components exchanging data? Draw an ASCII diagram. Look for cycles.
- Every meaningful test path listed: happy path, errors, edge cases.
- Can this be rolled back without touching data?
- Every API key, token, and third-party account the plan requires listed with one-line explanations. No credential requests mid-implementation.
- Every MCP server, external API, and third-party CLI the plan depends on verified as reachable before approval.

**No placeholders in approved plans.** Every step must be concrete before approval. Forbidden patterns: TBD, TODO, "implement later," "similar to step N," "details to be determined." A plan with placeholders is a promise to plan later.

## Gotchas

| What happened | Rule |
|---------------|------|
| Moved files to `~/project`, repo was at `~/www/project` | Run `pwd` before the first filesystem operation |
| Asked for API key after 3 implementation steps | List every dependency before handing off |
| User said "帮我做" and got 3 options | Stay in planning mode. Lead with the recommended option, and treat user acceptance as plan approval, not implementation approval. |
| Planned MCP workflow without checking if MCP was loaded | Verify tool availability before handing off, not mid-implementation |
| Rejected design restarted from scratch | Ask what specifically failed, re-enter with narrowed constraints |
| Built against wrong regional API (Shengwang vs Agora) | List all regional differences before writing integration code |
| Added FastAPI backend to a Next.js project | Never add a new language or runtime without explicit approval |
| User said "just do it" before approving the design | Treat it as approval of the last option presented. State which option was selected, then finish the plan. Do not implement inside `/think`. |

## Output

**Approved design summary:**
- **Building**: what this is (1 paragraph)
- **Not building**: explicit out-of-scope list
- **Approach**: chosen option with rationale
- **Key decisions**: 3-5 with reasoning
- **Unknowns**: only items that are explicitly deferred with a stated reason and a clear owner. Not vague gaps. If an unknown blocks a decision, loop back before approval.

After the user approves the design, stop. Implementation starts only when requested.
