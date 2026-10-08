# Product Timeline

This file is the source of truth for progress.

Rules:

1. Start every PR by reading this file.
2. Work on the first unchecked item unless the user explicitly overrides it.
3. If no PR workflow exists yet, leave items unchecked and report the next item.
4. After opening a PR, change the completed item from `[ ]` to `[x]`.
5. Add the PR number at the end of the item, for example `(#12)`.
6. Leave the next unchecked item visible in the final summary.

## Phase 0: Project Rails

Goal:

> Make the project understandable before building features.

Expected PRs:

- [ ] Add README.
- [ ] Add project blueprint.
- [ ] Add AGENTS.md.
- [ ] Add product timeline.
- [ ] Add development cycle docs.
- [ ] Add initial architecture doc.

Exit criteria:

- A new contributor can understand the project goal.
- An agent can find the next task without asking.
- PRs have a verification standard.

## Phase 1: MVP Skeleton

Goal:

> Create the smallest runnable product shape.

Expected PRs:

- [ ] Choose MVP technical stack.
- [ ] Add project scaffold.
- [ ] Add first smoke test.
- [ ] Add first user-facing workflow.
- [ ] Add basic documentation for local use.

Exit criteria:

- The project runs locally.
- The first workflow can be demonstrated.
- Tests or smoke checks prove the skeleton works.

## Phase 2: Useful MVP

Goal:

> Make the first workflow genuinely useful.

Expected PRs:

- [ ] Add core data model.
- [ ] Add validation.
- [ ] Add primary output view.
- [ ] Add error handling.
- [ ] Add user-facing examples.

Exit criteria:

- The first user can complete the main workflow.
- Failure modes are understandable.
- The docs match actual behavior.
