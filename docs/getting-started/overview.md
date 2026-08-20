# Overview

ASG (AI Savings Gateway) is a **server-side LLM API gateway** whose job is to make
AI coding agents cheaper to run by optimizing every request and response that
passes through it.

## The problem it solves

When an AI coding agent works on a task, it makes many API calls. Each call
re-sends:

- the **system prompt** (thousands of tokens),
- the **tool schemas** (hundreds to thousands of tokens),
- the **conversation history** (which grows every turn),
- and **recent tool results** (which can be huge).

Providers bill input tokens at full price on the first occurrence and, depending
on the provider, at a discounted rate on cache hits. But most clients and
providers do a poor job of *maximizing* cache hits and *minimizing* what gets
re-sent. The result: a session that should cost $1 costs $7.

## The design philosophy

ASG is built on four principles:

1. **Server-side, not client-side.** The client keeps using the exact same API
   it already uses. Optimization happens between the client and the provider.
2. **Transparent.** If any optimization layer fails, the request still goes
   through, unmodified (fail-closed / fail-open design chosen per layer).
3. **Protocol-compatible.** The public surface is OpenAI-compatible
   (`/v1/chat/completions`) and Anthropic Messages (`/v1/messages`). Clients do
   not need new SDKs.
4. **Provider-agnostic.** One gateway fronting DeepSeek, GLM, Qwen, MiniMax,
   OpenAI, Anthropic, xAI, Gemini, and more.

## The optimization layers

| Layer | Mechanism | Goal |
|---|---|---|
| L1 | Request normalization | Increase prefix-cache hit rate by canonicalizing the request shape |
| L2 | Context folding | Compress cold history so each turn re-sends fewer tokens |
| Dual-Mode | Output constraint | Make the model emit a compact, structural result |
| Cache | `cache_control` injection + warmup | Explicitly promote stable prefixes to cached tokens |
| Tool-fix | Empty-call cleanup + stream delta normalization | Eliminate garbage frames that force retries |
| Routing | Intent classification + rules | Send each request to the cheapest capable model |

## Deployment shape

ASG runs as a small set of systemd services behind nginx:

```
nginx :8888  →  ASG Gateway :18788  →  Provider Relay :8140  →  upstream providers
```

- **nginx** — public ingress, TLS, rate limiting, path routing.
- **ASG Gateway** — the optimization brain. A single Python service (~100 KB)
  implementing all six layers.
- **Provider Relay** — a lightweight Go relay handling auth, key management, and
  upstream protocol adaptation.

The gateway is **multi-tenant**: users have their own tokens, their own model
routes, and their own metrics.

## What this repository does *not* contain

The implementation. ASG is closed-source. This repo documents the architecture,
the mechanisms, and the measured results so that the product is understandable,
evaluable, and discoverable — without exposing the source.

Next: [Quickstart](quickstart.md)
