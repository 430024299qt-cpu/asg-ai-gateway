# VBK-AI Agent Gateway — Benchmark Results

> **Transparency first.** These numbers come from production traffic on our Hong Kong gateway node. We encourage independent verification.

---

## Aggregate Savings

| Metric | Value | Source |
|--------|-------|--------|
| Average token-cost reduction | **~86%** | Production 7-day average |
| Peak prefix-cache hit rate | **>96%** | Hourly peak measurement |
| Dual-Mode KV cache hit rate | **91.7%** | Production measurement |
| Non-stream latency improvement | **2800ms → 1100ms (-60%)** | P0-1 optimization deployment |

---

## Per-Layer Measurements

### L1: Request Normalization

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Prefix-cache hit rate (cold start) | ~30% | 75% | +45pp |
| Prefix-cache hit rate (warm session) | ~50% | 94% | +44pp |
| Prefix-cache hit rate (peak hour) | ~60% | 96.9% | +37pp |

**How it works**: System prompt deduplication, tool schema compression, and cache-aware prefix ordering ensure maximum reuse of cached prompt prefixes across requests.

### L2: Context Folding

| Metric | Value |
|--------|-------|
| Tokens saved per session (typical) | ~350k |
| Trigger rate | Production active |
| Semantic preservation | Structure-aware truncation |

**How it works**: Compresses conversation history by folding redundant tool outputs and earlier context while preserving semantic structure. K=3 round boundary guard with 2048 token threshold.

### Dual-Mode: Output Constraint

| Metric | Value |
|--------|-------|
| KV cache hit rate | 91.7% |
| Output token reduction | ~50% on tool-call responses |

**How it works**: Enforces structured output format for tool-call responses, reducing output verbosity while maintaining correctness.

### Cache (cache_control + warmup)

| Metric | Value |
|--------|-------|
| Cost reduction on cached tokens | 90% (Anthropic native) |
| Cache TTL | 1 hour (Anthropic native) |
| Cross-session warmup | Enabled via prefix fingerprinting |

### P0-1: Non-stream Latency Optimization

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Non-stream response latency | 2800ms | 1100ms | -60% |

**Residual 1.1s** attributed to DeepSeek reasoning model thinking time (inherent to provider).

---

## Methodology

### Measurement Approach

- **Traffic source**: Production requests from real users on HK gateway node
- **Measurement period**: 7-day rolling average (Aug 2026)
- **Cache hit rate formula**: `cache_read / (prompt_tokens + cache_read_tokens)`
- **Cost reduction formula**: `1 - (optimized_cost / raw_cost)`

### How to Verify

1. **Self-test**: Run your own agent through the gateway for 1 hour, compare token usage in your provider dashboard
2. **A/B test**: Run the same prompt sequence with and without gateway, compare costs
3. **Independent audit**: Use [LLM Price Check](https://llmpricecheck.com/) or similar tools to verify

---

## Limitations

- **Provider-dependent**: Savings vary by provider's caching policy (Anthropic 1h cache > DeepSeek auto-cache > OpenAI auto-cache)
- **Workload-dependent**: Highly repetitive prompts benefit more from caching; novel prompts benefit less
- **Cold start**: First request in a session has no cache benefit; savings increase with session length
- **Not applicable to**: Image generation, audio, or non-text modalities

---

## Comparison with Alternatives

| Approach | Where Savings Happen | Install Required | Typical Savings |
|----------|---------------------|------------------|-----------------|
| **VBK-AI Agent Gateway** | Server-side, zero client change | ❌ No (SaaS) | 50-95% |
| Client-side context truncation | Client code changes | ✅ Yes | 20-40% |
| Provider native caching only | Provider-side | ❌ No | 30-60% |
| Manual prompt optimization | Per-prompt | ❌ No | 10-30% |

---

## Reproducing Results

To reproduce these measurements:

1. Register at [www.agentgwapi.online](http://www.agentgwapi.online/)
2. Configure your agent with the gateway endpoint
3. Run a typical coding session for 1+ hours
4. Compare token usage in your provider dashboard vs. gateway dashboard
5. Cache hit rates visible in gateway dashboard metrics

---

*Last updated: August 2026*
