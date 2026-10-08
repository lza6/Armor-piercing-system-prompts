# Nanochat bottleneck diagnosis and next experiment

## Conclusion

The primary bottleneck is **insufficient pretraining**, with **model size** setting a secondary ceiling. The evidence does not support treating SFT data quality or the SFT recipe as the main cause of weak benchmark capability, although the current SFT duration/selection and greedy decoding likely amplify repetition.

## Project evidence

- Architecture: depth 6, 73,531,646 total parameters. Nanochat's scaling-law count is 23,199,960 parameters (`transformer_matrices + lm_head`).
- Pretraining: 81,920,000 tokens, only **3.53 tokens per scaling parameter**.
- The base validation BPB was still improving at the endpoint: 1.1878 at step 4,000, 1.1743 at 4,500, and 1.1658 at 5,000. There is no plateau indicating that the model had exhausted the data/compute available to it.
- Base CORE evidence was near chance on the small evaluation: Wikidata 0.0000, OpenBookQA 0.2500, Winogrande 0.5625 (0.125 centered), Operators 0.0000.
- SFT validation BPB fell from 1.0174 to 0.7389, but the final sample answered “Paris” and then entered a numeric repetition loop. This is compatible with fitting the held-out SFT token distribution without acquiring broad knowledge or robust free-running generation.

## Literature interpretation

- Hoffmann et al., *Training Compute-Optimal Large Language Models* (2022), find that parameters and training tokens should grow in roughly equal proportions. Their canonical Chinchilla point is about 20 tokens per parameter. The exact ratio should not be transplanted blindly to nanochat's unusual parameterization, but 3.53 is far below both this literature reference and nanochat's code-level default target of 12.
- Zhou et al., *LIMA: Less Is More for Alignment* (2023), support the “superficial alignment” hypothesis: most knowledge and reasoning come from pretraining, while SFT mainly teaches the model how to express existing capabilities. They also show that validation perplexity can keep improving after generation quality peaks.
- Eldan and Li, *TinyStories* (2023), show that very small models can generate coherently when the domain and data distribution are deliberately simplified. That result argues against size being the sole cause of repetition, but it does not imply that a 73.5M model can perform strongly on broad MMLU/GSM8K-style tasks.
- Gunasekar et al., *Textbooks Are All You Need* (2023), show that curated, educational data can make small models much more sample-efficient. This makes pretraining data quality an important later axis, but quantity/coverage is the first bottleneck to remove here because the run ended while loss was still falling and at a very low token ratio.

## Recommended next experiment

Run a one-factor **pretraining-token ablation**:

1. Keep the d6 architecture, tokenizer, pretraining mixture, optimizer, batch size, SFT mixture, SFT steps, and decoding/evaluation settings unchanged.
2. Train from scratch to nanochat's `--target-param-data-ratio=12`: **278,396,928 tokens / 16,992 steps**, versus the current 81,920,000 tokens / 5,000 steps.
3. Save/evaluate checkpoints at 5,000, about 11,000, and 16,992 steps. At each checkpoint report base validation BPB, the same CORE tasks, fixed-prompt generations, and repetition statistics.
4. Apply the identical 1,500-step SFT recipe to the final base checkpoint and report ChatCORE, per-task MMLU/GSM8K accuracy or exact match, plus the same fixed generations.

This experiment cleanly tests the leading hypothesis. A substantial gain supports descending to an SFT early-stopping/data-curation round. If base loss improves but benchmark scores and generations barely move, the next bottleneck is model capacity or pretraining-domain coverage; compare a larger depth at matched compute rather than tuning SFT first.

## Sources

- Hoffmann et al. (2022), [Training Compute-Optimal Large Language Models](https://arxiv.org/abs/2203.15556)
- Zhou et al. (2023), [LIMA: Less Is More for Alignment](https://arxiv.org/abs/2305.11206)
- Eldan and Li (2023), [TinyStories: How Small Can Language Models Be and Still Speak Coherent English?](https://arxiv.org/abs/2305.07759)
- Gunasekar et al. (2023), [Textbooks Are All You Need](https://arxiv.org/abs/2306.11644)
