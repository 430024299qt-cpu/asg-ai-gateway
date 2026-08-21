# ASG Benchmark & Honest Numbers

> Last updated: 2026-08-21
> Methodology: production traffic replay on the live deployment (HK region)

---

## Summary

| Metric | Value | Notes |
|--------|-------|-------|
| Overall token-cost reduction | **~86%** | Measured across all optimization layers combined |
| Prefix-cache hit rate (peak, h9) | **98.2%** | Hourly peak; typical session 75–94% |
| Dual-Mode output savings | **91.7% KV hit** | Structural output constraint injection |
| Cumulative tokens saved (L2 folding) | **353,788** | Compressed conversation history |
| Non-stream latency overhead | **~1100ms** | Down from ~2800ms after P0-1 optimization |

---

## What each layer saves

### L1 — Request normalization

Canonicalizes message arrays, sorts tool schemas, strips volatile fields (timestamps, random seeds, session IDs). This raises the provider's automatic prefix-cache hit rate, so identical prefixes across sessions are served from cache instead of being re-processed.

**Measured impact:** Prefix-cache hit rate improved from ~48% (no normalization) to 75–94% per session.

### L2 — Context folding

Compresses older conversation turns into compact `[FOLD]` markers when the context exceeds a threshold. The model still sees the essential information but the token count drops sharply.

**Measured impact:** 353,788 tokens saved cumulatively across the production deployment. Per-session input reduction varies by conversation length — long multi-turn sessions see the largest gains.

### Dual-Mode — Output constraint

Injects a structural contract into the request that encourages the model to emit concise, structured output rather than verbose explanations. The constraint is transparent to the model — it still produces the same quality of code, just with less preamble.

**Measured impact:** KV cache hit rate of 91.7% on constrained output. Output token reduction is significant for coding tasks where verbose JSON blocks or explanations are common.

### Cache — Provider cache control

Explicitly marks stable message prefixes with `cache_control` hints (for providers that support it, like Anthropic). Pre-warms caches on startup and on schema changes so the first request of a session hits cache instead of miss.

**Measured impact:** Moves tokens from "billed" to "cached" pricing tiers. On Anthropic, cached tokens cost 90% less than regular input tokens.

### Tool-call fix

Removes empty or malformed `tool_calls` frames from the request stream. These frames force the model into expensive retry loops that double the bill for zero value. Also normalizes streaming `tool_calls` delta chunks to prevent client-side hangs.

**Measured impact:** Eliminates entire wasted turns. Difficult to quantify as a percentage because it depends on the client's behavior, but sessions with buggy tool-call formatting see the largest absolute savings.

---

## Honest limitations

Following the spirit of full transparency (inspired by [caveman's HONEST-NUMBERS.md](https://github.com/JuliusBrussee/caveman)):

1. **L2 folding only helps long conversations.** Single-turn requests or short 2-3 turn sessions see minimal benefit from folding. The savings scale with conversation length.

2. **Cache hit rates depend on prefix stability.** If your system prompt or tool schemas change every request, cache hit rates drop. The more stable your prefix, the higher the savings.

3. **Dual-Mode output constraint changes output format.** Some agents may need adjustment to parse the more compact output. ASG falls back to passthrough on error.

4. **The ~86% overall figure is an aggregate.** Individual sessions vary widely: a 50-turn refactoring session might save 90%+, while a quick one-shot completion might save 20-30%. The weighted average across production traffic is ~86%.

5. **Latency adds ~1.1s overhead.** The optimization pipeline adds processing time. For streaming responses, this is mostly hidden (first token arrives slightly later, but the rest streams normally). For non-streaming requests, the total round-trip is ~1.1s longer.

6. **These numbers are from a single deployment.** They reflect ASG running on 2C/4G infrastructure with DeepSeek as the primary provider. Results on other providers or hardware may differ.

---

## How to verify for yourself

1. Register at [154.12.86.206:8888](http://154.12.86.206:8888/)
2. Add your API key in the dashboard
3. Point your coding agent at `http://154.12.86.206:8888/v1`
4. Run a coding session and check the dashboard for real-time savings metrics

The dashboard shows per-request token counts, cache hit rates, and cost savings. You don't have to take our word for it.
