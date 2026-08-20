# Prefix-cache optimization

The provider's prefix cache is the single biggest discount on the bill. ASG makes
sure the gateway actually uses it — explicitly and on purpose.

## The mechanism

ASG combines three techniques:

### 1. `cache_control` injection
For providers that honor the Anthropic `cache_control` convention (and for
OpenAI-style providers via the gateway's normalization), ASG explicitly marks the
**stable prefix** — the first system block and the first user message — with a
cache TTL directive:

```
cache_control: { "type": "ephemeral", "ttl": "1h" }
```

Marking it explicitly is different from merely *hoping* the provider caches: it
tells the provider this prefix is intentionally stable, so it is stored and reused
across turns in the session.

### 2. Warmup
Because a cache entry is only created when a request with that prefix actually
arrives, ASG runs an asynchronous **warmup**: after a session's request is
processed, a background task re-sends the stabilized prefix once so the provider
materializes the cache entry *before* the next turn — instead of paying the full
cost of the first uncached turn.

### 3. Session-aware evaluation
The gateway tracks per-session cache statistics (cached vs. missed tokens) and
uses them to decide when optimization is paying off. If a session shows a poor
hit rate, the gateway can re-normalize or rotate the session state rather than
persist a broken prefix.

## Why caching beats every other lever

| Technique | Typical saving |
|---|---|
| Input caching (hit vs. miss) | up to 90% off input tokens on hit |
| Output constraint (Dual-Mode) | reduces output volume |
| History folding (Layer 2) | reduces input volume |

Cache hits are the *multiplier*: they reduce the price of the largest fixed cost
(the prefix) by an order of magnitude. A stable, warm prefix converts a session's
repeated ~4,000–5,000 token prefix from **full price every turn** to **~free after
the first turn**.

## Measured effect

Production sessions show per-session prefix-cache hit rates of **75–94%**, with
individual hours peaking above 96%. See [Benchmarks](../benchmarks/results.md).

Next: [Tool-call robustness](tool-call-robustness.md)
