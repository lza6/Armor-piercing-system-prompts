---
name: "mkl-write-regression"
description: "Write a focused regression test from an established bug reproduction and public behavior. Use when a fix needs a test that fails on the affected version; avoid mirroring the implementation or weakening assertions."
---

# Write a regression test

Read the reproduction, expected public behavior, and nearby test conventions. Derive the assertion from the behavior a caller needs, not from the proposed implementation.

Write the smallest test that captures the defect. Use deterministic inputs and existing project dependencies where practical. Do not mock away the faulty path. Place supporting fixtures beside the test and explain any environment requirements.

Run the test against the affected implementation before accepting it as a regression test. Inspect why it failed: an assertion about the reported behavior is evidence; an import error, collection failure, permission problem, or timeout is not the intended proof. Keep the test unchanged when evaluating the fix.

If the fix is already present, use an isolated checkout or an equivalent preserved baseline to test the pre-fix version. Preserve the user's current checkout and uncommitted work. Do not revert their working tree merely to get a red test.

Return the test location, why the assertion represents the bug, the baseline result, and the fixed-version result if tested. Name any checks not run. Do not alter the expected result just to make the suite green.

The source library's [independent example test](https://github.com/00200200/maintainer-skills-lab/blob/main/examples/bugfix/test_slug.py) asserts punctuation and whitespace behavior without reproducing the implementation's transformation steps.
