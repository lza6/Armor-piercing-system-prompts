# Task: harden slugify() for untitled input

`slugify.py` turns a human title into a URL-safe slug. It works for normal
titles but MISBEHAVES on "untitled" input:

- `slugify("")` returns `""` (an empty slug, which breaks URL routing).
- `slugify("   ")` (whitespace only) returns `""` for the same reason.
- `slugify(None)` raises `AttributeError` and crashes the caller.

Callers pass drafts that may have no title yet, so this is a real bug.

## What to change

Make `slugify()` return the fallback slug `"untitled"` for any input that would
otherwise produce an empty slug or crash: empty string, whitespace-only string,
and `None`.

## Constraints

- Do NOT regress the existing behavior: `slugify("Hello World")` must still
  return `"hello-world"` and `slugify("  A/B  Test ")` must still return
  `"a-b-test"`.
- The result must never carry a leading or trailing hyphen.
- Keep the change small and focused on `slugify.py`.
