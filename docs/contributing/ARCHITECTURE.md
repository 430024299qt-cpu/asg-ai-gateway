# Architecture

High-level view of how ASG fits into an AI workflow. This is the *logical*
architecture; the implementation is proprietary.

```
┌─────────────┐   ┌─────────────┐   ┌─────────────────────┐   ┌────────────────┐
│  AI client  │──▶│  HTTP edge  │──▶│   ASG gateway       │──▶│  Model provider │
│ (Claude Code│   │  (nginx)    │   │  (optimization      │   │  (DeepSeek,     │
│  Cline, ...)│   │  /v1 8888   │   │   layers 1–6)       │   │   GLM, ...)     │
└─────────────┘   └─────────────┘   └─────────────────────┘   └────────────────┘
                                           │
                                    ┌──────┴─────────┐
                                    │ Token mgmt     │
                                    │ (relay/admin)  │
                                    └────────────────┘
```

The client is unchanged except for `base_url`. Every request/response flows
through the gateway, where the six layers run.

## Layers

| # | Layer | Where it acts | Effect |
|---|---|---|---|
| 1 | Request normalization | outbound request | byte-stable prefix → cacheable |
| 2 | Context folding | outbound request | cold history → `[FOLD]` markers |
| 3 | Dual-Mode output constraint | request + stream | steer output size/verbosity |
| 4 | Prefix-cache management | request + warmup | `cache_control` pinning + async warmup |
| 5 | Tool-call robustness | request + stream | drop empty/orphan frames, fix deltas |
| 6 | Routing (optional) | request | intent → cheapest capable model |

## Request lifecycle

1. Client sends a normal `POST /v1/chat/completions`.
2. Gateway **normalizes** the message sequence (schema sort, role mapping).
3. If the session is long enough, **folding** compresses cold history.
4. **Routing** (if enabled) classifies intent and may switch the target model.
5. **Cache control** pins the stable prefix; the request is sent upstream.
6. Upstream streams the response; the **tool-call** and **output-constraint**
   layers act on the stream in real time.
7. Client receives a standard streaming response.

## Why server-side?

A server-side gateway sees *every* request for *every* client on the team,
including closed-source clients that can't be modified. One deployment point
normalizes all traffic, and the savings compound across users. Client-side
tools like rtk are complementary — they filter *before* the prompt; ASG
optimizes *after*.

## Consistency & safety

- **Shadow mode** — every rewrite layer can run in shadow (measure, don't touch).
- **Fail-open** — uncertain rewrites pass traffic through untouched.
- **Opt-in layers** — folding and routing are off unless enabled.
