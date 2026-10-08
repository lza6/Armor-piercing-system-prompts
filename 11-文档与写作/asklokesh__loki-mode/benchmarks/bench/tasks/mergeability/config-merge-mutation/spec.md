# Task: make deep_merge() pure (no input mutation)

`config_merge.py` recursively merges an override config onto a base config.
It produces the right values, but it has a side-effect bug:

- `deep_merge(base, override)` MUTATES the caller's `base` dict in place.
- It also shares nested dict references, so mutating the result can reach back
  into the inputs.

A caller who keeps a base config around and merges an override onto it finds
their original base silently changed. That is a real correctness hazard.

## What to change

Make `deep_merge()` PURE: return a NEW merged dict and leave BOTH `base` and
`override` completely untouched. No shared nested references between the inputs
and the returned value.

## Constraints

- The merged VALUES must stay correct: overlapping dict values merge
  recursively, otherwise the override value wins.
  Example: merging `{"a":1,"nested":{"x":1,"y":2}}` with
  `{"b":2,"nested":{"y":20,"z":30}}` yields
  `{"a":1,"b":2,"nested":{"x":1,"y":20,"z":30}}`.
- Keep the change focused on `config_merge.py`.
