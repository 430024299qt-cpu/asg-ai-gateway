# Layer 2 — Context folding

The second-biggest input-token cost in a long session is the **history that keeps
growing but is no longer needed**.

## The mechanism

ASG tracks a session's conversation and classifies it into two zones:

- **Active zone** — the most recent turns, including the current one and the
  immediately relevant tool context. Kept in full.
- **Cold zone** — everything older. Compressed into a compact marker.

A folded turn is replaced by a placeholder such as:

```
[FOLD|tool_name1,tool_name2]
```

and the whole cold region is wrapped in a `[FOLDED_HISTORY] ... [/FOLDED_HISTORY]`
block. The model still sees *that* older turns existed and what tools they used —
but not their full payloads.

## When it triggers

Folding is not a blunt "always fold everything". It is gated by:

- **Session length** — folding only pays off after a session is long enough that
  the cold zone is substantial relative to the active zone.
- **Ratio threshold** — a guard ratio decides when the cold region is worth
  compressing (a configurable constant, ~15% of the budget).
- **Per-session state** — fold decisions are cached per session so the same
  session is folded consistently, not re-decided every turn.

## Why it works

In a multi-turn tool session, most of the history is *prelude*: early turns set
up context the model no longer needs word-for-word. The model is far better served
by a concise summary of "these turns happened, these tools were used" than by the
raw kilobytes of old tool output. Folding converts a growing per-turn input cost
into a roughly constant one.

## Measured effect

On the production deployment a single fold reduced a message from ~12,100 to
~11,700 tokens (≈400 tokens saved) in a real session, with the same model
continuing to respond correctly. Aggregate savings across sessions are reported
in [Benchmarks](../benchmarks/results.md).

## Design notes

- **Deterministic markers** — folded blocks are rendered with a stable format so
  they do not themselves break prefix-cache continuity.
- **No-op when cold history is empty** — short sessions are never folded, so the
  layer adds zero overhead to ordinary requests.
- **Safety valve** — if the fold cannot be computed safely, the request passes
  through unmodified.

Next: [Dual-Mode — output constraint](dual-mode-output-constraint.md)
