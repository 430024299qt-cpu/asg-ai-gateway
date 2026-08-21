# VBK-AI Agent Gateway — Benchmark Results

> **Transparency first.** These numbers come from production traffic on our gateway infrastructure. We encourage independent verification.

---

## Aggregate Savings

| Metric | Value | Source |
|--------|-------|--------|
| Average token-cost reduction | **~86%** | Production 7-day average |
| Peak cache hit rate | **>96%** | Hourly peak measurement |
| Cache efficiency | **91.7%** | Production measurement |
| Response latency improvement | **-60%** | Production optimization |

---

## What You'll See

### Short Sessions (Single Task)

- Cache warms up within the first few requests
- Typical savings: **50-70%** on token costs
- Latency reduced due to optimized request handling

### Long Sessions (Full Coding Session)

- Cache efficiency increases as session progresses
- Typical savings: **80-95%** on token costs
- Context compression reduces overhead on extended conversations

### Peak Performance

- During active coding sessions, cache hit rates exceed **96%**
- Near-elimination of redundant token processing

---

## How to Verify

1. **Register** at [www.agentgwapi.online](http://www.agentgwapi.online:8888/)
2. **Configure** your agent with the gateway endpoint
3. **Run** a typical coding session for 1+ hours
4. **Compare** token usage in your provider dashboard vs. gateway dashboard

---

## Methodology

- **Traffic source**: Production requests from real users
- **Measurement period**: 7-day rolling average (Aug 2026)
- **Savings formula**: `1 - (optimized_cost / raw_cost)`

---

## Limitations

- **Provider-dependent**: Savings vary by provider's caching policy
- **Workload-dependent**: Highly repetitive prompts benefit more from caching; novel prompts benefit less
- **Cold start**: First request in a session has no cache benefit; savings increase with session length
- **Not applicable to**: Image generation, audio, or non-text modalities

---

## Comparison with Alternatives

| Approach | Where Savings Happen | Install Required | Typical Savings |
|----------|---------------------|------------------|-----------------|
| **VBK-AI Agent Gateway** | Server-side, zero client change | No (SaaS) | 50-95% |
| Client-side context truncation | Client code changes | Yes | 20-40% |
| Provider native caching only | Provider-side | No | 30-60% |
| Manual prompt optimization | Per-prompt | No | 10-30% |

---

*Last updated: August 2026*
