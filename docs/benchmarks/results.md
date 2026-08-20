# Benchmark results

Measured on the production ASG deployment. See [methodology](methodology.md) for
definitions.

## Headline

> **~86% overall input-cost reduction** on production coding-agent traffic,
> driven by prefix-cache hit rates of **75–94% per session** (hourly peaks above
> 96%).

## Prefix-cache hit rate

| Scope | Rate | Notes |
|---|---|---|
| Per-session, steady state | 75–94% | single agent session, stable prefix |
| Hourly peak (high-traffic hour) | 96–98% | longest-lived sessions, warmest prefixes |
| Aggregate, mixed traffic | ~47–75% | includes short sessions and model mix |

Measured as `cached_tokens / (prompt_tokens + cached_tokens)`.

## Cost composition (why output matters)

Input tokens dominate the token count, but output tokens carry a higher unit
price. On production:

- Output tokens are ~0.6% of **token** volume, but ~8% of **cost** — a ~4×
  price-per-token amplification, further diluted by cache discounts on input.
- This is why the Dual-Mode output constraint exists: shrinking output volume
  buys an outsized cost reduction per token saved.

## Context folding

| Metric | Value |
|---|---|
| Representative single-fold saving | 399 tokens (12,104 → 11,705) |
| Folding applies | only to sessions long enough that cold history ≫ active zone |
| No-op behavior | short sessions / empty cold history are never folded |

## Request profile

Production traffic shows a mean of **~600 tokens per request** after
optimization, across mixed streaming and non-streaming tool-calling workloads.

## Savings formula (for projecting your own savings)

For a session with prefix length `P`, `N` turns, hit rate `H`, and cache discount
factor `R`:

```
input cost ≈ N × P × ( H × R + (1 − H) × 1 ) + non-prefix input + output
```

Because `H` sits at 0.75–0.94 and `R` is typically 0.1–0.25, the prefix term —
which would otherwise be the dominant cost — collapses. Example with
`P=4,500`, `N=40`, `H=0.9`, `R=0.1`:

```
without ASG: 40 × 4500 = 180,000 billed input tokens
with ASG:    40 × 4500 × (0.9×0.1 + 0.1) = 40 × 4500 × 0.19 = 34,200
→ ~81% reduction in prefix input cost
```

This is why the numbers compound: **normalization enables caching, caching is the
big lever, and the other layers (folding, Dual-Mode, tool-fix) shrink the
remaining non-cached volume.**

## Model coverage

Verified against DeepSeek, GLM, Qwen, MiniMax, OpenAI, Anthropic, and xAI
upstreams on the production deployment.

> ⚠️ These numbers are from one deployment and provider pricing at the time of
> measurement. Treat them as directional; measure your own traffic with the
> [methodology](methodology.md).
