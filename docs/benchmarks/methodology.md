# Benchmark methodology

How ASG's savings numbers are measured. We report numbers that are **directly
derived from the gateway's own accounting**, not synthetic benchmarks.

## Metrics

### Prefix-cache hit rate

The authoritative definition used throughout this repo:

```
cache hit rate = cached_tokens / (prompt_tokens + cached_tokens)
```

That is: of all input tokens processed, what fraction were served from the
provider's prefix cache (`cached_tokens`) vs. billed as fresh prompt tokens
(`prompt_tokens`). This is the metric that maps directly to input-token cost.

### Tokens saved

`tokens_saved` is computed per request as the input tokens that would have been
billed at full price had ASG not optimized the request. It is recorded by the
gateway's accounting layer for every request.

### Cost reduction

Cost is computed by applying the provider's published pricing to the *actual*
token flows:

```
cost = prompt_tokens × prompt_price + cached_tokens × cached_price
     + output_tokens × output_price
```

Because cached tokens are billed at a deep discount, a high hit rate produces a
**cost reduction larger than the token reduction**. We always report both, and
we prefer the cost figure when discussing "savings".

## Fold measurement

A fold's saving is measured by capturing the message size **before** and
**after** folding on the same request:

```
before = token estimate of the full message
after  = token estimate of the folded message
saved  = before - after
```

A representative production fold: `before=12,104` → `after=11,705` → `saved=399`
tokens, on a real session, with the model continuing to respond correctly.

## Session vs. aggregate

- **Per-session hit rate** — rolling over a single agent session; this is what a
  user experiences and is typically higher (75–94%) because a session has one
  stable prefix.
- **Aggregate hit rate** — rolling over all traffic; lower because it mixes many
  short sessions and different models.

We report both and always label which one is being cited.

## Caveats

- Provider caching behavior changes over time; numbers should be re-measured
  against the current provider pricing and cache semantics.
- "Up to 86%" is the **cost** reduction observed on production traffic. Yours
  will vary with session length, model mix, and how noisy your clients' requests
  are.

Next: [Results](results.md)
