---
name: review-renovate
description: Review Renovate bot PRs that update GitHub Actions dependencies. Verifies supply chain integrity by checking pinned commit SHAs against upstream tagged releases, reviews changelogs for breaking changes, and confirms compatibility with existing workflow configurations. Use when a Renovate PR updates GitHub Actions in .github/workflows/.
---

# Review Renovate GitHub Actions PRs

You are reviewing a Renovate bot PR that updates GitHub Actions dependencies. Your job is to verify supply chain integrity and ensure the upgrades won't break CI/CD workflows.

## Inputs

You will be given a PR number or URL. Use `gh` CLI to fetch PR details and diff.

## Steps

### 1. Fetch PR metadata and diff

```
gh pr view <PR> --json title,body,files,commits,author,headRefName
gh pr diff <PR>
```

Confirm the PR author is `app/renovate`. If not, flag this immediately — it may not be an automated dependency update.

### 2. Identify all action version changes

From the diff, extract each changed action:
- Full action name (e.g., `oven-sh/setup-bun`)
- Old version tag and pinned SHA
- New version tag and pinned SHA
- Update type (patch, minor, major)

### 3. Verify pinned SHAs against upstream tags

For every action being updated, verify **both old and new** SHAs match the claimed version tags:

```
gh api repos/{owner}/{repo}/git/ref/tags/{version} --jq '.object.sha'
```

Compare each result against the SHA in the workflow file. If any SHA does not match, **stop and report a supply chain integrity failure**. Do not approve the PR.

### 4. Review changelogs for breaking changes

From the PR body (Renovate includes release notes), check each updated action for:
- Removed inputs or outputs that the workflows currently use
- Changed default behavior for inputs the workflows rely on
- New required inputs
- Major version bumps (these almost always have breaking changes)

### 5. Check workflow compatibility

Read the affected workflow files and verify:
- No removed or renamed inputs are being used
- No changed defaults affect current behavior
- The action's runtime requirements are still met (e.g., Node.js version compatibility)

### 6. Report findings

Present a summary table:

| Action | Old | New | Type | SHA verified |
|--------|-----|-----|------|-------------|
| ... | ... | ... | patch/minor/major | yes/NO |

Then state:
- Whether all SHAs are verified
- Whether any breaking changes were found
- Whether the workflows remain compatible
- A clear **safe to merge** or **do not merge** recommendation
