# Technical reference

Implementation-level notes on the optimization layers. No source code is
published; this describes *contracts and invariants* so that operators can
reason about behavior and validate their deployment.

## 1. Request normalization

**Goal:** make the prompt prefix byte-stable across turns so the provider's
prefix cache hits.

Invariants:

- Tool/function schemas are serialized in a deterministic order (e.g. sorted),
  independent of the order the client sends them.
- Role mapping is applied uniformly (`system`/`developer` → provider-native
  roles) so the same semantic message always produces the same bytes.
- No client-specific header or timestamp noise enters the cached prefix.

Measurable: for a fixed session, consecutive requests share a prefix that grows
monotonically; prefix diff between turn *k* and *k+1* equals only the new turn's
content.

## 2. Context folding

**Goal:** compress already-answered cold history into markers.

Invariants:

- Only the *cold* zone (history older than the active window) is eligible.
- A fold is applied only when cold history is large relative to the active zone
  (guard ratio, e.g. ~15% threshold) — short sessions are never folded.
- Folded content is replaced with `[FOLD|tool_names]` markers; the model
  continues correctly because the markers carry enough structure to answer.
- Shadow mode computes before/after token estimates without rewriting.

Measurable: per-fold `saved = before − after` (representative production fold:
399 tokens).

## 3. Dual-Mode output constraint

**Goal:** reduce billed output for requests that don't need full verbosity.

Invariants:

- Mode is detected from the request (task type, model, client intent).
- Constraint instructions are injected only when the request indicates reduced
  output is acceptable; full-output requests pass unchanged.
- Streams are terminated promptly when the constraint is satisfied.

Measurable: output-token and output-cost share of total; on production, output
was ~0.6% of tokens but ~8% of cost — the constraint targets the cost share.

## 4. Prefix-cache management

**Goal:** make cache hits reliable and warm.

Invariants:

- A stable prefix is pinned with provider-native `cache_control` (ephemeral
  TTL, e.g. 1 hour) so it is not evicted between turns.
- A warmup path issues lightweight asynchronous requests to refresh the prefix
  before the real request lands.
- Cache accounting is session-aware: hit rate is measured per session and in
  aggregate, never conflated.

Measurable: `cache hit rate = cached_tokens / (prompt_tokens + cached_tokens)`.

## 5. Tool-call robustness

**Goal:** eliminate the failed turns that double cost and hang sessions.

Invariants:

- Empty/invalid `tool_calls` frames and orphan `tool` messages are dropped from
  the outgoing sequence.
- Streaming tool-call deltas are normalized: first chunk preserves `id` + `name`;
  later chunks carry only argument fragments.
- Non-streaming fallback aggregates the upstream response into a single JSON
  object.
- All repairs are **fail-open** — uncertainty passes traffic through untouched.

## 6. Routing (optional)

**Goal:** send each request to the cheapest capable model.

Invariants:

- Intent classification (command / coding / writing / analysis / other) is
  deterministic from message content.
- Rules are JSON, hot-reloadable, and off by default.
- Routing never rewrites the response side.

## Testing philosophy

Every layer ships with:

- **Shadow mode** — measure impact without altering traffic.
- **Contract tests** — assert the invariants above hold on representative
  fixtures (long session, streaming tool-call stream, empty tool_calls, etc.).
- **Fail-open semantics** — a rewrite error degrades to passthrough, never to
  corruption.
