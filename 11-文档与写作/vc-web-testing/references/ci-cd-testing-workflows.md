# CI/CD Testing Workflows

## GitHub Actions - Complete Workflow

```yaml
name: Test Suite
on:
  push:
    branches: [main]
  pull_request:

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'pnpm' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm run test:unit -- --coverage
      - uses: actions/upload-artifact@v4
        with: { name: coverage, path: coverage/ }

  e2e-tests:
    needs: unit-tests
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'pnpm' }
      - run: pnpm install --frozen-lockfile
      - run: npx playwright install --with-deps
      - run: npx playwright test --shard=${{ matrix.shard }}/4
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report-${{ matrix.shard }}
          path: playwright-report/
```

## Test Splitting

```bash
# By shard (equal files)
npx playwright test --shard=1/4

# By timing (Knapsack)
- uses: chaosaffe/split-tests@v1
  with:
    glob: 'tests/**/*.spec.ts'
    split-total: ${{ matrix.shard }}
```

## Caching

```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.local/share/pnpm/store
      ~/.cache/ms-playwright
    key: ${{ runner.os }}-${{ hashFiles('pnpm-lock.yaml') }}
```

## Flaky Test Management

```yaml
- run: npx playwright test --retries=2
- run: npx playwright test --grep-invert @flaky  # Quarantine
```

## Performance & Security Gates

```yaml
- run: pnpm install -g @lhci/cli && lhci autorun
- run: pnpm audit --audit-level=high
- uses: github/codeql-action/analyze@v3
```

## Merge Reports

```yaml
merge-reports:
  needs: e2e-tests
  steps:
    - uses: actions/download-artifact@v4
      with: { pattern: playwright-report-*, merge-multiple: true }
    - run: npx playwright merge-reports ./all-reports
```

## GitLab CI

```yaml
stages: [test, e2e]

unit:
  stage: test
  script: [pnpm install --frozen-lockfile, pnpm run test:unit]

e2e:
  stage: e2e
  parallel: 4
  script:
    - pnpm install --frozen-lockfile && npx playwright install --with-deps
    - npx playwright test --shard=$CI_NODE_INDEX/$CI_NODE_TOTAL
  artifacts:
    when: on_failure
    paths: [playwright-report/]
```

## Best Practices

- **Fail fast:** Unit tests before E2E
- **Parallelism:** Shard E2E across jobs
- **Cache:** pnpm store, Playwright browsers
- **Artifacts on failure:** Reports for debugging
- **Security gates:** pnpm audit, SAST before merge
