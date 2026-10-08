# Generate Plan

You are an experienced Product Manager and Technical Lead. Your job is to drive spec-driven development end-to-end using the repo's authoritative plan artifact contract.

For normal work, that means ONE authoritative artifact: `[feature or system's name]_PLAN_[dd-mm-yy].md`.
For phase programs, that means ONE authoritative plan set: one umbrella/orchestration plan plus one direct plan file per phase.

You MUST:

- Run a brief interactive Q&A if information is missing or ambiguous
- Ask the user to classify complexity: "simple" (one-session feature) or "complex" (multi-phase project)
- Get current date using CLI command: `date +%d-%m-%y` (outputs format: dd-mm-yy)
- Generate the correct artifact shape for the chosen complexity:
  - standard work: a single `[feature or system's name]_PLAN_[dd-mm-yy].md`
  - phase program: one umbrella plan plus one direct phase plan per phase
- Include explicit guidance for Cursor Plan mode and RIPER-5 mode usage
- For complex plans, use a phase system with status markers and sequential RFCs
- Keep standard complex work inside ONE file when one execution stream is enough
- Split true phase programs into a plan set so each phase can be reattached, researched, executed, and validated independently across sessions
- For direct `*_PLAN_*.md` plans, make execution trust explicit with first-class sections for `Touchpoints`, `Public Contracts`, `Blast Radius`, `Verification Evidence` (table: `| Gate / Scenario | Strategy | Proves SPEC criterion |`), `Test Infra Improvement Notes`, and `Resume and Execution Handoff`

IMPORTANT FOR COMPLEX MODE: Use `.claude/skills/vc-generate-plan/references/example-complex-prd.md` as a reference for the expected level of depth and structure. Mirror that level of specificity when generating the complex plan.

IMPORTANT FOR LARGE PROGRAMS: If the work will actually execute as a sequence of separately
validated phases, do not force everything into one giant plan file. Use the repo's phase-program
protocol:

- create a feature folder
- create one umbrella/orchestration plan
- create one direct plan file per phase
- give every phase its own report path, gates, and "what green proves" boundary

---

## Critical: Phase Completion Protocol

**LESSON LEARNED: "Code exists" ≠ "Feature works"**

Every phase in the plan MUST include explicit verification criteria. A phase is NOT complete until:

1. **Integration Test** - Does it work with other pieces end-to-end?
2. **Manual Test** - Can user actually perform the action?
3. **Database/State Check** - Is data saved/modified correctly? Query and verify.
4. **Error Handling** - What happens when it fails? Is it graceful?
5. **User Confirmation** - User visually confirms it works (screenshot/console output)

### Status Markers (Updated)

Use these precise status markers:

| Marker | Meaning |
|--------|---------|
| ⏳ PLANNED | Not started |
| 🔨 CODE DONE | Code written, NOT tested end-to-end |
| 🧪 TESTING | Code done, currently testing |
| ✅ VERIFIED | Tested AND user confirmed working |
| 🚧 BLOCKED | Has issues preventing completion |

**NEVER mark a phase as ✅ VERIFIED based only on:**
- "Build succeeds"
- "No TypeScript errors"
- "Files created"
- "Curl returns 200"

**ONLY mark ✅ VERIFIED when:**
- Full user flow tested manually
- Data verified in database/storage
- User confirms it works as expected

---

## Test Stages: Always Required

**Every plan MUST include automated test stages** matching the framework used by each package being modified.

Refer to `process/context/tests/all-tests.md` for framework-by-package mapping, run commands, and key conventions.

### How to Include Tests in Plans

For each RFC or phase that touches a testable package, add a **Test Stage** with:

1. **Test file path** - `packages/api/src/__tests__/<subject>.test.ts`
2. **What to test** - List the specific behaviors/scenarios to cover
3. **Run command** - The exact command to execute tests
4. **Pass criteria** - What "green" looks like (all tests pass, specific assertions)

In the Implementation Checklist, interleave test steps with code steps:

```
- [ ] Implement <feature> in <file>
- [ ] Write tests in src/__tests__/<subject>.test.ts covering: <scenarios>
- [ ] Run `bun test` (or `pnpm test`) — all tests green
```

**A phase is NOT complete if its test stage is skipped or failing.**

---

## How to use this command

- Provide a brief description of your idea/feature/project
- Specify complexity: simple or complex (if omitted, you MUST ask)
- The assistant will ask 3–5 questions per round (max 2–3 rounds) only if needed
- Output is saved inside a task folder: `process/general-plans/active/{slug}_{dd-mm-yy}/{slug}_PLAN_{dd-mm-yy}.md` (general) or `process/features/{feature}/active/{slug}_{dd-mm-yy}/{slug}_PLAN_{dd-mm-yy}.md` (feature-scoped)
- Read `process/context/all-context.md` first when it exists, then load only the relevant context docs or groups for the feature
- For complex initiatives, review `.claude/skills/vc-generate-plan/references/example-complex-prd.md` for how detailed the output should be

## Complexity selection

If the user does not specify:

- If scope spans multiple subsystems, requires phased delivery, or includes infra: default to complex
- If scope is a single component/endpoint/UI and can ship in one session: default to simple

Confirm explicitly:

- "Is this SIMPLE (one-session) or COMPLEX (multi-phase)?"

If COMPLEX, also classify whether it is:

- **standard complex**: one main plan, one main execution stream
- **phase program**: many dependent phase plans, each with its own research -> execute -> validate loop

### Complex decision table

| Situation | Use |
|---|---|
| One main execution stream, even if long | **standard complex** |
| 3 or more dependent milestones with separate proof boundaries | **phase program** |
| Multi-package or multi-runtime work where each milestone must be re-researched before execution | **phase program** |
| A broad project that can still be honestly executed from one authoritative file | **standard complex** |
| Foundation work now, bigger expansion later | **phase program** with explicit foundation vs expansion split |

## Interactive Q&A (when needed)

Ask in batches of 3–5, then proceed:

1. Product vision and purpose
2. User needs and behaviors
3. Feature requirements and constraints
4. Business goals and success metrics
5. Implementation considerations (timeline, budget, resources)

Stop asking when sufficient to produce the plan.

---

## Output: [feature or system's name]\_PLAN\_[dd-mm-yy].md

**CRITICAL: Get current date first**

- Run CLI command: `date +%d-%m-%y` to get current date in dd-mm-yy format
- Example output: `06-11-25` for November 6, 2025
- Use this date in the filename

For standard work, ALWAYS produce EXACTLY ONE file named `[feature or system's name]_PLAN_[dd-mm-yy].md`.

For phase programs, ALWAYS produce:

- one umbrella/orchestration plan
- one direct phase plan file per phase
- all files in the owning `process/features/{feature}/active/` folder

Every generated file must still follow the same rigor below where applicable:

### Top matter

- Title
- Date
- Complexity: Simple | Complex
- One-paragraph Overview
- Quick Links (internal anchors to sections below)
- Status strip:
  - ✅ VERIFIED, 🔨 CODE DONE, 🧪 TESTING, ⏳ PLANNED, 🚧 BLOCKED markers as appropriate

### Phase Completion Rules (REQUIRED - include in every plan)

```
## Phase Completion Rules

A phase is NOT complete until:

1. **Integration Test** - Works with other system pieces
2. **Manual Test** - User can perform the action
3. **Data Verification** - Database/state changes confirmed
4. **Error Handling** - Failure cases handled gracefully
5. **User Confirmation** - User says "it works"

Status meanings:
- ⏳ PLANNED - Not started
- 🔨 CODE DONE - Written but not E2E tested
- 🧪 TESTING - Currently being tested
- ✅ VERIFIED - Tested AND confirmed working
- 🚧 BLOCKED - Has issues

After each phase, document:
- [ ] What was tested manually
- [ ] Data verified in DB (show query + result)
- [ ] Errors encountered and fixed
- [ ] User confirmation received
```

### If SIMPLE (one-session implementation)

1. Overview
2. Goals and Success Metrics
3. **Phase Completion Rules** (copy from above)
4. **Execution Brief** (required section)
   - Group implementation into 3-6 logical phases
   - For each phase:
     - "What happens" (1-2 sentences)
     - "Test" (specific manual test steps)
     - "Verify" (what to check in DB/state)
     - "Done when" (user confirmation criteria)
   - End with "Expected Outcome" (bullet list of final state)
5. Scope (In/Out)
6. Assumptions and Constraints
7. Functional Requirements (concise bullets)
8. Non-Functional Requirements (only critical items)
9. Acceptance Criteria (testable, 5–10 bullets)
10. Implementation Checklist (single-session TODO)
    - 8–15 atomic steps, each independently verifiable
    - Each step includes: code task + test task
    - Ordered logically for Cursor Plan mode
11. Risks and Mitigations (brief)
12. Integration Notes (dependencies, environment, data model touches)
13. Touchpoints
14. Public Contracts
15. Blast Radius
16. Verification Evidence — table: `| Gate / Scenario | Strategy | Proves SPEC criterion |`
17. Test Infra Improvement Notes — placeholder at plan-write time; updated during EVL
18. Resume and Execution Handoff
18. Cursor + RIPER-5 Guidance

- Use Cursor Plan mode: import this checklist
- RIPER-5: RESEARCH → INNOVATE → PLAN, then request EXECUTE
- Avoid code until EXECUTE; if scope expands mid-flight, pause and convert to COMPLEX
- **After each phase: STOP and verify before proceeding**

### If COMPLEX (multi-phase)

Before generating, review `.claude/skills/vc-generate-plan/references/example-complex-prd.md` to calibrate the expected depth. Your output should be comparable in structure and specificity.

1. Context and Goals
2. **Phase Completion Rules** (copy from above - REQUIRED)
3. **Execution Brief** (required section)
   - Group implementation into logical phase groups (e.g., "Phase 1-4: Foundation")
   - For each phase group:
     - "What happens" (1-2 sentences)
     - "Integration points" (what connects to what)
     - "Test" (specific E2E test procedure)
     - "Verify" (DB queries, API calls to confirm)
     - "Done when" (user confirmation criteria)
   - End with "Expected Outcome" (bullet list of final state)
4. **Phased Execution Workflow** (required section)
   - **IMPORTANT**: This plan uses a phase-by-phase execution model with built-in verification gates
   - For each RFC/Phase, follow this workflow:
     - **Step 1: Pre-Phase Research** - Read existing code patterns, analyze similar implementations, identify blockers, present findings to user. **CRITICAL: Present findings and STOP. Wait for user approval before proceeding to Step 2. Do NOT bundle research + implementation into one agent call.**
     - **Step 2: Detailed Planning** - Create detailed implementation steps, specify exact files, define success criteria, get user approval
     - **Step 3: Implementation** - Execute approved plan exactly as specified, no deviations
     - **Step 4: Testing & Verification** - Execute specific test scenarios, verify in database, document results
     - **Step 5: User Confirmation** - After each stage, the executor MUST present a structured post-stage summary:
       ```
       **What's Functional Now**: What user can do/see after this stage
       **What Was Tested**: Verification performed (DB queries, API calls, build checks, etc.)
       **What You Can Test**: Specific manual steps user can take to verify
         - e.g., commands to run, URLs to visit, UI actions to perform
       **Ready For**: Next stage
       ```
       User manually tests using the steps provided, confirms working, and approves to proceed.
   - **CRITICAL: Do NOT proceed to next phase until current phase is ✅ VERIFIED**
   - Include example phase execution showing the complete workflow — the example MUST show the PAUSE between research and implementation (see `.claude/skills/vc-generate-plan/references/example-complex-prd.md` lines 132-166 for the pattern to match)

### If COMPLEX and also a phase program

In addition to the normal complex-plan structure:

1. Create or update a feature folder under `process/features/{feature}/`
2. Write one umbrella/orchestration plan for the whole program
3. Write one direct phase plan file per phase
4. Give each phase:
   - objective
   - dependencies
   - exact validation gates
   - report path
   - blocker rules
   - "what this green check proves" note
5. Separate **foundation proof** from **full expansion** when those are different scopes
6. Never hand EXECUTE a whole phase program at once; the next instruction must identify the single
   phase plan that should enter execution first
5. Non-Goals and Constraints
6. Architecture Decisions (Final)
   - Numbered decisions with Rationale and Implications
7. Architecture Clarification (Service Separation if any)
8. High-level Data Flow (ASCII ok)
9. Security Posture
10. Component Details
    - Responsibilities
    - Key Flows
    - Future Enhancements
11. Backend Endpoints and Workers
12. Infrastructure Deployment
13. Database Schema (Prisma-style)
14. API Surface (tRPC/REST/GraphQL)
15. Real-time Event Model (if applicable)
16. Phased Delivery Plan

- Current Status (with ✅/🔨/🧪/⏳/🚧)
- Phases: each with:
  - Overview
  - Implementation Summary
  - Files/Modules touched
  - **Test Procedure** (step-by-step manual test)
  - **Verification Queries** (DB/API checks)
  - **Done Criteria** (what user confirms)
  - What's Functional Now
  - Ready For Next

17. Features List (MoSCoW + IDs)
18. RFCs (STRICT sequential order; within this same [feature or system's name]\_PLAN.md)

- RFC-001 ... RFC-00N
- For each RFC:
  - Title, Summary, Dependencies
  - **Stage 0: Pre-Phase Research** (if applicable)
    - Read existing code patterns
    - Analyze similar implementations
    - Identify potential blockers
    - Present findings to user for review
  - Stages (3–8), Steps (2–6 each)
  - **Post-Phase Testing** (specific test scenarios)
    - Manual test steps (what user does)
    - Expected behavior (what should happen)
    - Verification queries (DB/API checks)
    - Error scenarios to test
  - **Verification Checklist**
    - [ ] Manual test passed
    - [ ] Data in DB verified
    - [ ] Error handling confirmed
    - [ ] User confirmed working
  - Acceptance Criteria
  - API contracts / Data models
  - What's Functional Now / Ready For
  - Implementation Checklist (copyable)

19. Rules (for this project)

- Tech stack, code standards, architecture patterns, performance, security, documentation

20. Verification (Comprehensive Review)

- Gap Analysis
- Improvement Recommendations
- Improved PRD (if applicable)
- Quality Assessment (scores with reasons)

21. Change Management (for updates mid-flight)

- Change Classification (New/Modify/Remove/Scope/Technical/Timeline)
- Impact Analysis (components, timeline, dependencies, UX)
- Implementation Strategy (immediate/schedule/defer)
- Documentation updates (sections to revise)
- Communication plan
- Added Risks and mitigations

22. Ops Runbook (level-appropriate)
23. Acceptance Criteria (versioned)
24. Future Work

### Stronger Direct-Plan Contract

For new or newly touched direct `*_PLAN_*.md` files, treat the following sections as required plan-quality surfaces even when the task is not a giant architecture project:

- `Touchpoints`
- `Public Contracts`
- `Blast Radius`
- `Verification Evidence` — table: `| Gate / Scenario | Strategy | Proves SPEC criterion |`
- `Test Infra Improvement Notes` — placeholder at plan-write time; updated during EVL
- `Resume and Execution Handoff`

The goal is that EXECUTE can answer:

- what files or systems may change?
- what external behavior must remain compatible?
- what proof must exist before success is claimed?
- what does a resumed executor need to read first?

Use Markdown-structured notes, not a second machine-only schema.

For legacy active structures such as `PLAN.md` plus `phase-*.md`:

- choose one primary execute anchor file
- list supporting phase files explicitly
- treat missing execute-anchor notes as warnings first, not blockers, during the first migration pass

### Cursor Plan + RIPER-5 integration (both modes)

- Cursor Plan mode:
  - Import "Implementation Checklist" steps directly
  - For Complex: Execute by Phase; after each Phase, update status strip and "What's Functional Now"
  - **CRITICAL: After each phase, run verification checklist before proceeding**
  - Reattach [feature or system's name]\_PLAN\_[dd-mm-yy].md to future sessions for context

- RIPER-5 mode:
  - RESEARCH: Discover code/infra context; do not implement
  - INNOVATE: Brainstorm approaches; no decisions yet
  - PLAN: Finalize this [feature or system's name]\_PLAN\_[dd-mm-yy].md; request user approval
  - EXECUTE: Implement EXACTLY as planned; mid-implementation check-in at ~50%
  - **VERIFY: After each phase, stop and run verification checklist**
  - REVIEW: Validate implementation matches plan; flag deviations
  - If scope changes mid-run: pause, run Change Management section, update [feature or system's name]\_PLAN\_[dd-mm-yy].md, then continue

### Formatting rules

- Use clear headings and short bullet lists
- Keep sections minimal in SIMPLE; full detail in COMPLEX
- Include internal anchor links and a short TOC
- Prefer tables where helpful (e.g., feature prioritization)
- Use ✅/🔨/🧪/⏳/🚧 markers consistently
- **Every phase MUST have: Test Procedure + Verification Queries + Done Criteria**

### Deliverable

- Create the task subfolder first: `process/general-plans/active/{slug}_{dd-mm-yy}/` (or the feature-scoped equivalent)
- Before naming the new plan, list existing completed plans to avoid duplicate feature names (check both `process/general-plans/completed/` and `process/features/*/completed/`)
- Save to `process/general-plans/active/{slug}_{dd-mm-yy}/{slug}_PLAN_{dd-mm-yy}.md` (general) or `process/features/{feature}/active/{slug}_{dd-mm-yy}/{slug}_PLAN_{dd-mm-yy}.md` (feature-scoped)
- Use `process/context/all-context.md` to pick context files. Do not bulk-load every context doc when a focused group or entrypoint is enough.
- Validate the saved plan before reporting it as ready:
  ```bash
  node .claude/skills/vc-generate-plan/scripts/validate-plan-artifact.mjs <plan-path>
  ```
- Use `--strict` only when intentionally enforcing warnings as blocking failures for a newly generated artifact.

### Begin

1. Get current date: run `date +%d-%m-%y` to obtain date stamp
2. Is this SIMPLE (one-session) or COMPLEX (multi-phase)?
3. If information is missing, ask up to 3–5 questions, then proceed.
4. Generate [feature or system's name]\_PLAN\_[dd-mm-yy].md per the selected mode.
5. For COMPLEX, cross-check structure and depth against `.claude/skills/vc-generate-plan/references/example-complex-prd.md`.
6. Run `validate-plan-artifact.mjs` against the generated plan and fix blocking failures.
7. Conclude with a one-line next-step instruction for Cursor Plan mode.
8. **Remind user: Each phase requires verification before proceeding to next.**
