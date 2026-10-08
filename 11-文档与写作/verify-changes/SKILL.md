---
name: verify-changes
description: Verify a code change in this repo before committing or opening a PR. Use after editing collector (backend), web (frontend), shared package, or the Go agent — picks the minimal sufficient check set per touched area.
---

# Verify Changes

Run the smallest check set that covers what you touched. All commands run from the repo root unless noted.

## By touched area

### apps/collector (backend)

```bash
pnpm --filter @neko-master/collector exec tsc --noEmit
pnpm --filter @neko-master/collector test
```

Single test file / case (much faster while iterating):

```bash
pnpm --filter @neko-master/collector test -- src/modules/auth/auth.service.test.ts
pnpm --filter @neko-master/collector test -- -t "should validate token format"
```

Lint (CI-relevant; `console.log` is forbidden in collector source — use `console.info/warn/error`):

```bash
pnpm --filter @neko-master/collector lint
```

### apps/web (frontend)

```bash
pnpm --filter @neko-master/web exec tsc --noEmit
pnpm --filter @neko-master/web lint
```

For production-impacting changes (new routes, error boundaries, next.config, i18n files) also run the full build — it catches what tsc alone misses:

```bash
pnpm --filter @neko-master/web exec next build
```

### packages/shared

Shared types are consumed by both apps — verify both compile:

```bash
pnpm --filter @neko-master/shared build
pnpm --filter @neko-master/collector exec tsc --noEmit
pnpm --filter @neko-master/web exec tsc --noEmit
```

### apps/agent (Go probe)

```bash
cd apps/agent
go vet ./... && go build ./... && go test ./...
sh -n nekoagent && sh -n install.sh
```

## Checklist before commit

- [ ] Checks above pass for every touched area
- [ ] New behavior has a test (collector uses Vitest; see `src/__tests__/helpers.ts` for `createTestDatabase()` / `createTestBackend()`)
- [ ] UI changes: both locales translated and dark mode checked (see the `ui-conventions` skill)
- [ ] DB writes/schema touched: run the `add-stats-dimension` / `db-conventions` checklists

Note: `.husky/pre-push` runs collector `tsc --noEmit` + web `build` when pushing to `main` — running them yourself first avoids a slow failed push.
