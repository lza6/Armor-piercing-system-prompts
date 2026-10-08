# Product Timeline

`docs/PRODUCT_TIMELINE.md` is the source of truth for project progress.

It should be specific enough that an agent can choose the next task without
guessing.

## Format

```markdown
## Phase 1: MVP Skeleton

Goal:

> Create the smallest runnable product shape.

Expected PRs:

- [ ] Choose MVP technical stack.
- [ ] Add project scaffold.
- [ ] Add first smoke test.

Exit criteria:

- The project runs locally.
- The first workflow can be demonstrated.
```

## Merge Tracking

After a PR exists:

```markdown
- [x] Add first smoke test. (#12)
```

The PR number is important. It proves the item moved.
