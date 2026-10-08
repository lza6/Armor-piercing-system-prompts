# Reranker Tokenizer Equivalence Corpus

60 (query, doc) pairs across 6 buckets — used by
`tests/integration/reranker-tokenizer-equivalence.test.ts` to verify that
HF `tokenizers` (Rust, Python) produces byte-identical
`input_ids`/`attention_mask`/`token_type_ids` as `@xenova/transformers` (JS)
on the same `tokenizer.json`.

## Buckets

1. **ASCII short** — common technical queries + matching short docs
2. **ASCII long truncating** — long docs >1500 chars to exercise `strategy='only_second'`
3. **Multilingual** — Chinese, Japanese, Korean, Arabic, Hebrew, German with umlauts, Spanish, Portuguese, French, Russian
4. **Emoji + ZWJ** — family glyphs, flag-modifier sequences, single emoji + skin tone, mixed text+emoji
5. **Special tokens in text** — literal `[CLS]`, `[SEP]`, `<s>`, `<|endoftext|>` etc. inside user text
6. **Edge cases** — empty query/doc, whitespace only, single char, rare codepoints, repetition

## Freeze policy

This corpus is **frozen after commit 7** (when `@xenova/transformers` is removed).
Once frozen, the snapshot at `tests/fixtures/reranker-tokenizer-snapshot.json`
(created in commit 4 from the xenova baseline) cannot be regenerated in-tree
without checking out a pre-commit-7 SHA. To add a new pair after freeze:

1. `git checkout <pre-commit-7-sha>` (last commit before xenova was removed)
2. Run the equivalence test in a "regenerate" mode (TODO: add `WIGOLO_REGEN_SNAPSHOT=1` env flag in commit 4)
3. Copy the new snapshot back to the working tree

Alternatively, accept the new pair has no historical baseline and add it to
the snapshot from Python output only (loses the xenova-equivalence guarantee
for that pair).
