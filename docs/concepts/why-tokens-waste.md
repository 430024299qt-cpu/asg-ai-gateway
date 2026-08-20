# Why AI coding burns tokens

Understanding where the waste comes from is the first step to understanding why
ASG exists.

## The anatomy of a coding-agent API call

A typical multi-turn coding session sends, on every turn:

```
system prompt            ~2,000–4,000 tokens   (identical every turn)
tool schemas             ~500–1,500 tokens     (identical every turn)
conversation history     ~N,000 tokens         (grows every turn)
recent tool results      ~1,000–20,000 tokens  (varies widely)
```

Each of these is **input** token — billed every time it is sent.

## Four sources of waste

### 1. Repetitive prefixes
The `system + tools` prefix is identical across turns, but many clients and
providers re-price it in full. Providers offer **prefix caching** — if you send
the exact same prefix, later occurrences are billed at a steep discount (often
~10% of the original price, or even free). But cache hits only happen if the
prefix is *byte-stable*. Tiny, avoidable variations (tool-schema order, trailing
whitespace, timestamps, UUIDs) **invalidate the cache every turn**.

→ ASG **Layer 1** canonicalizes the request so the prefix stays stable.

### 2. Exploding histories
Multi-turn sessions with tool calls grow fast. Every tool result, every retry,
every failed parse gets appended to the history. The model needs the *recent*
context, but the *cold* history (old turns) is re-sent and re-billed every turn
for almost no benefit.

→ ASG **Layer 2** compresses cold history into a compact marker.

### 3. Unconstrained outputs
Many calls don't need a verbose model answer — they need a compact, structured
result (a short status, a small JSON, a patch). Without a structural contract,
the model pads output with prose, JSON boilerplate, and apologies. Output tokens
are often **more expensive per token** than input tokens.

→ ASG **Dual-Mode** injects a structural output contract.

### 4. Broken tool calls
Agents rely on function calling. When a model emits an **empty or malformed
`tool_calls`** frame, or when a streaming proxy mangles the delta, the client
either retries (doubling cost) or hangs (wasting a whole session). Some agents
even emit *orphan* tool messages that violate the provider's schema and get a
`400`.

→ ASG **Tool-fix** removes garbage frames and normalizes streaming deltas.

## Where client-side tools fit (and where they don't)

Client-side proxies (e.g. Rust CLI wrappers) filter command *output* before it
enters the context — they reduce how much noise the agent *sees*. That is
valuable and complementary. But it does nothing for the four sources above,
which live **on the wire between the client and the provider**. That is exactly
where ASG operates.

Next: [Layer 1 — request normalization](layer-1-request-normalization.md)
