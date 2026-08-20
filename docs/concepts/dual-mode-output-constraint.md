# Dual-Mode — output constraint

Output tokens often cost **more per token** than input tokens. Dual-Mode makes
the model spend them only where they earn their keep.

## The mechanism

For request shapes where the caller does not need a free-form essay — a tool call,
a short status, a structured value — ASG injects a **structural contract** into
the request that tells the model the expected output shape:

- **Mode detection** — the gateway inspects the request (does it carry tool
  definitions? a `response_format`? a terse user message?) and decides which
  output mode applies.
- **Constraint injection** — a compact instruction block is appended to the
  system context, telling the model to emit a minimal, structured, no-prose
  result in the detected mode.
- **No client change** — the client never asked for this; the gateway adds it.

## Why it works

Modern models comply well with explicit output contracts. Left unconstrained,
they pad: they restate the question, emit JSON with extra keys, add explanatory
prose around tool arguments, and apologize. A constraint that says "output only
the result" removes most of that padding. In tool-calling-heavy sessions the
output token savings compound because *every turn* produces a constrained output.

## Relation to client-side `response_format`

Some clients already send `response_format`. Dual-Mode is complementary:

- If the client already constrains the output, Dual-Mode detects that and adds
  nothing (no double-billing of the instruction, no conflict).
- If the client does not, Dual-Mode supplies the constraint for the cases where
  it is safe and beneficial.

## Measured effect

On the production deployment, output tokens are a meaningful share of *cost*
(disproportionate to their token share) because of their higher unit price;
Dual-Mode is what keeps that share small. Numbers are in
[Benchmarks](../benchmarks/results.md).

Next: [Prefix-cache optimization](prefix-cache-optimization.md)
