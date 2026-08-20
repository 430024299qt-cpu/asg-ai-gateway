# Tool-call robustness

Nothing wastes tokens like a **failed turn**. ASG's tool-call layer removes the
garbage that causes retries and hangs.

## The problem

Coding agents drive tool calls through the API. Real-world traffic surfaces a
family of broken frames:

- **Empty `tool_calls`** — an assistant turn that declares a tool call with no
  name or no arguments. The provider rejects it, or the client retries, doubling
  cost.
- **Orphan tool messages** — a `tool` role message with no matching assistant
  `tool_calls` frame ahead of it. Strict providers return a `400` and the whole
  session can hang.
- **Streaming delta mangling** — in SSE streams, tool-call arguments arrive as a
  sequence of deltas. Proxies and client SDKs often lose the `name` on the first
  chunk or drop the `id`, and the assembled call is malformed.

## What the layer does

- **Cleanup** — scans the outgoing message sequence and drops empty or invalid
  `tool_calls` frames and orphan `tool` messages, producing a sequence that
  satisfies the provider's schema.
- **Delta normalization** — rewrites streaming tool-call deltas so the first
  chunk preserves the tool call `id` and `name`, and later chunks only carry
  argument fragments. This makes the client SDK assemble a valid call.
- **Non-stream aggregation** — for non-streaming fallback paths, aggregates the
  upstream response into a single well-formed JSON object so downstream parsing
  never splits on a boundary.

## Why it matters

A single malformed tool frame can cost an entire session: the client hangs or
enters a retry loop, burning tokens and developer time. The fix is invisible when
it works — traffic just flows — but it is the difference between a session that
completes and one that dies.

## Design notes

- **Fail-open** — if the cleanup cannot confidently repair a sequence, the
  request passes through untouched rather than risking a wrong rewrite.
- **Streaming-safe** — the normalization runs on the SSE stream without breaking
  real-time delivery (ingress buffering is disabled for streaming paths).

Next: [Benchmarks](../benchmarks/)
