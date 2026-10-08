---
name: update-libs
description: Update the libs/ SDK submodules (openai-go, anthropic-sdk-go, go-genai) to new upstream/fork versions, adapt tingly-box to API changes, and verify with build + vet + tests. Use when asked to bump/update SDK submodules, sync libs to new tingly-dev/upstream tags, or fix breakage after a submodule bump.
---

# Update libs/ SDK submodules

## Wiring

- Submodules: `libs/openai-go`, `libs/anthropic-sdk-go`, `libs/go-genai`,
  wired into `go.mod` via `replace`.
- Version source of truth: `OPENAI_GO_VERSION` / `ANTHROPIC_SDK_GO_VERSION` /
  `GO_GENAI_VERSION` at the top of `Taskfile.yml` (branch or tag names).
- `task submodule:update` checks out those refs and rewrites `branch =` in
  `.gitmodules` to match — never hand-edit `.gitmodules`.

## Procedure

1. **Declare the target versions**: edit the three vars in `Taskfile.yml`,
   then run:
   ```bash
   task submodule:update
   ```
2. **Network fallback** if `git fetch` over HTTPS fails (SSH usually works):
   ```bash
   for d in libs/openai-go libs/anthropic-sdk-go libs/go-genai; do
     git -C $d config url."git@github.com:".insteadOf "https://github.com/"
   done
   ```
   Set it per-submodule — submodule repos don't inherit superproject config.

3. **Review the fork patches.** The `tingly-dev-*` branches carry a small set
   of deliberate patches over upstream. Verify each still exists and still
   makes sense; a patch may have been deleted on purpose:
   ```bash
   git -C libs/<sdk> fetch origin
   git -C libs/<sdk> log --oneline origin/<upstream-ref>..<tingly-dev-ref>
   git -C libs/<sdk> diff --stat origin/<upstream-ref>..<tingly-dev-ref>
   ```
   Fork patch commits are prefixed `tingly-dev` and explain their intent in
   the commit body — read it before deciding anything.

4. **Build-fix loop**:
   ```bash
   go build ./...
   ```
   Typical SDK breakage: plain fields become `param.Opt[T]`
   (`packages/param`). Write with `param.NewOpt(x)`, read with `x.Value`
   (`Opt[T]` is a value struct; absent → zero value, no nil deref). Note that
   identically-named fields on different structs may differ — one may stay
   plain while another became `Opt` — so check the struct, not the field name.

5. **Verify**:
   ```bash
   go vet ./internal/... ./cli/... ./ai/... ./imbot/...
   go test ./internal/protocol/...
   ```
   For any failure, first attribute it: SDK-related or environment?
   (Failing on network fetches, temp-HOME layout, etc. is unrelated to the
   bump.) Do not claim verification passed on failures you didn't triage.

6. **Evaluate behavior changes — do NOT blindly conform.** When a test fails
   on changed semantics (not just types):
   - Read the fork/upstream commit that changed the behavior and its stated
     intent.
   - Search tingly-box for production call sites relying on the old behavior.
   - Adapt our code/tests only if the product does not depend on the old
     behavior; otherwise surface the conflict to the user before changing
     anything.

7. **Commit in stages** (never one big commit; the user's preferred order):
   1. `chore(taskfile)`: Taskfile.yml version vars only.
   2. `chore(libs)`: submodule pointers + `.gitmodules` (follows the Taskfile).
   3. `refactor`/`fix`: API adaptation code + tests (always last).
   4. Optional: doc updates.
