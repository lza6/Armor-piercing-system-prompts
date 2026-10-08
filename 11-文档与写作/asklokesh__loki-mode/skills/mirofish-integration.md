# MiroFish Market Validation Integration

> MiroFish provides swarm intelligence market validation via simulated user/market
> reception. Results are **advisory only** and never gate the RARV cycle or
> Completion Council decisions.

## When This Module Applies

Load this module when ANY of the following are true:
- Your prompt contains a `MIROFISH MARKET VALIDATION` section
- The project has `.loki/mirofish-context.json`
- The session was started with `--mirofish` flag
- Tasks in `.loki/queue/pending.json` have `source: "mirofish"`

**If none of the above are true, do not load this module.**

## Understanding MiroFish Advisory Data

When present, MiroFish context includes:
- **overall_sentiment**: positive, negative, or mixed
- **sentiment_score**: 0.0-1.0 (higher = more positive)
- **confidence**: low, medium, or high (based on simulation depth)
- **key_concerns**: Risk items identified by simulated personas
- **feature_rankings**: Features ranked by simulated reception score
- **recommendation**: proceed, review_concerns, or reconsider

## Risk-Driven Prioritization

When MiroFish identifies key concerns, address them proactively:

| Concern Category | Action |
|-----------------|--------|
| User adoption / UX | Prioritize UX polish and onboarding |
| Market fit / demand | Validate core value proposition first |
| Competition | Differentiate early, focus unique features |
| Privacy / trust | Add transparency features, opt-in controls |
| Technical feasibility | Prototype risky components first |

## Sentiment-Aware Feature Ordering

Use feature reception scores to influence implementation order:
- **Score >= 0.8**: High demand -- implement first (validated demand)
- **Score 0.6-0.79**: Moderate interest -- standard priority
- **Score 0.4-0.59**: Lukewarm -- consider redesign before building
- **Score < 0.4**: Low reception -- deprioritize or cut from MVP

## Confidence Score Interpretation

| Range | Meaning | Action |
|-------|---------|--------|
| 80-100 | Strong market signal | Proceed with confidence |
| 60-79 | Moderate signal | Consider pivots for weak areas |
| 40-59 | Weak signal | Recommend PRD revision before heavy implementation |
| Below 40 | Insufficient data | Treat as noise, proceed with caution |

## Pipeline Status

If your prompt contains `MIROFISH_STATUS` instead of full validation data, the
pipeline is still running. This is normal -- MiroFish simulations take 15-45 minutes.

- Continue with implementation using available context
- Results will appear in subsequent iterations when the pipeline completes
- Do not wait or block for MiroFish results

## What MiroFish Does NOT Do

- Does NOT override Completion Council decisions
- Does NOT block the RARV cycle
- Does NOT evaluate code quality (that is the quality gates system)
- Does NOT make architecture decisions (those are yours)
- Results are directional signals, not precise measurements

## Known Limitations

- **LLM convergence bias**: Simulated agents may converge on similar opinions
- **Garbage-in-garbage-out**: Poor PRD = poor simulation seed = unreliable results
- **No real user data**: Synthetic personas, not actual market research
- **Cultural bias**: Agent behavior reflects LLM training data distributions
- **Timing**: Results arrive 15-45 min after launch, may miss early iterations

## Configuration

MiroFish requires these environment variables for its Docker container:
- `LLM_API_KEY` (required) -- API key for MiroFish's LLM provider
- `ZEP_API_KEY` (required) -- Zep Cloud key for graph memory

CLI flags:
- `--mirofish [URL]` -- Enable with optional custom URL (default: localhost:5001)
- `--mirofish-docker IMAGE` -- Auto-start MiroFish Docker container
- `--mirofish-rounds N` -- Simulation rounds (default: 100)
- `--mirofish-timeout S` -- Max pipeline wait (default: 3600s)
- `--mirofish-bg` -- Run in background mode
- `--no-mirofish` -- Disable even if LOKI_MIROFISH_URL is set

## MiroFish Task Queue

When MiroFish completes, actionable items from the report are added to the task
queue with `source: "mirofish"`. These tasks represent risk mitigations and
recommendations. They are advisory -- prioritize alongside normal development tasks
based on the concern severity and your judgment.
