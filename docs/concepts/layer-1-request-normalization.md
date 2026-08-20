# Layer 1 — Request normalization

The cheapest token is the one that hits the provider's **prefix cache**.

## The mechanism

Provider prefix caches are byte-sensitive: a request matches a cached prefix only
if the leading portion is *identical* to what was cached. Layer 1 canonicalizes
the parts of the request that are under the client's or the gateway's control, so
that identical semantic content produces identical bytes.

Concretely, Layer 1:

- **Canonicalizes message content** — strips or normalizes volatile fields
  (timestamps, nonces, empty fields, inconsistent whitespace) that would
  otherwise break prefix continuity.
- **Sorts tool schemas deterministically** — clients often emit tool
  definitions in nondeterministic order; sorting makes the schema block stable
  across turns.
- **Normalizes the system/roles structure** — maps role variants (e.g.
  `developer` → `system`) to a single canonical form so the prefix doesn't
  change when the client library version changes.

## Why it matters

Without normalization, a prefix that is *semantically* the same every turn can be
*byte-wise* different — and the provider re-prices the whole prefix at full cost.
With normalization, the same prefix hits the cache from turn two onward, moving
most of the fixed input cost (system + tools) from **billed** to **cached**.

## Design notes

- **Stateless per request** — Layer 1 only rewrites the current request; it does
  not keep per-session mutable state in the hot path.
- **Conservative** — normalization never drops semantic content. It only removes
  or reorders what is already unstable or duplicated.
- **Measured** — Layer 1 is the primary driver of the 75–94% per-session cache
  hit rates reported in [Benchmarks](../benchmarks/results.md).

Next: [Layer 2 — context folding](layer-2-context-folding.md)
