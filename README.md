<p align="center">
  <strong>ASG — AI Savings Gateway</strong>
</p>

<h3 align="center">Same budget. Ship more code.</h3>

<p align="center">
  A <strong>hosted API gateway</strong> that sits between your AI coding tools and model providers.<br>
  Bring your own API keys. Token costs drop <strong>50%–95%</strong>. Zero installation.
</p>

<p align="center">
  <a href="http://www.agentgwapi.online/">Website</a> · <a href="http://154.12.86.206:8888/">Dashboard</a> · <a href="docs/getting-started/quickstart.md">Quick Start</a> · <a href="docs/faq.md">FAQ</a>
</p>

---

## Why ASG exists

AI coding agents are token-hungry. A single multi-turn session with tool calls can burn tens of thousands of input tokens per request — most of it **repetition**: the same system prompt, the same tool schemas, the same long conversation history, re-sent and re-billed every turn.

Client-side tools (e.g. Rust CLIs) trim command output *before* it reaches the model. That helps — but it only covers one slice of the waste. The biggest slice happens **on the wire**, at the API layer.

ASG attacks all four sources of waste **server-side**, transparently, for every client that points at it.

---

## Quick start — no installation required

ASG is a **hosted SaaS gateway**. There is nothing to install. You keep using your existing tools and API keys — just point them at ASG.

| Step | What to do |
|------|------------|
| **1** | **Register** at [154.12.86.206:8888](http://154.12.86.206:8888/) — create a free account |
| **2** | **Add your API key(s)** in the dashboard — DeepSeek, OpenAI, Anthropic, GLM, Qwen, MiniMax, xAI, Gemini, and more |
| **3** | **Change one URL** in your client — point `base_url` to `http://154.12.86.206:8888/v1` |

That's it. Your client talks the same OpenAI-compatible API. ASG intercepts, optimizes, and forwards — you see the savings in the dashboard.

```text
BEFORE:  Claude Code ──────────────► api.deepseek.com   (full price)
AFTER:   Claude Code ──► ASG :8888 ──► api.deepseek.com  (50-95% less)
```

> **Works with any OpenAI-compatible or Anthropic-compatible client.**
> Claude Code, Cursor, Windsurf, Cline, Codex CLI, Continue, Aider, OpenCode, and more — no agent-side changes needed.

---

## What you save

ASG stacks multiple optimization layers. Each one addresses a different source of waste:

| Layer | Mechanism | What it does | Where it saves |
|-------|-----------|--------------|----------------|
| **L1** | Request normalization | Canonicalizes messages, sorts tool schemas, strips volatile fields | Raises provider prefix-cache hit rate |
| **L2** | Context folding | Compresses cold conversation history into compact `[FOLD]` markers | Shrinks per-turn input tokens |
| **Dual-Mode** | Output constraint | Injects a structural contract that makes the model emit compact output | Cuts output tokens |
| **Cache** | `cache_control` injection + warmup | Explicitly marks stable prefixes for provider caching and pre-warms them | Moves tokens from billed to cached |
| **Tool-call fix** | Empty-call cleanup + stream delta normalization | Removes malformed `tool_calls` frames that force expensive retries | Eliminates whole wasted turns |
| **Routing** | Intent classification + rules | (Optional) Routes requests to the cheapest capable model | Cuts unit price |

**Measured on the production deployment: ~86% overall token-cost reduction**, with per-session prefix-cache hit rates of 75–94% (peak above 96%).

> For detailed methodology, per-layer breakdown, and honest limitations, see [docs/benchmarks/](docs/benchmarks/).

---

## Supported models & platforms

One gateway, **all major providers**:

| Provider | Models |
|----------|--------|
| DeepSeek | deepseek-v4-flash, deepseek-chat, deepseek-r1, … |
| Anthropic | claude-haiku-4-5, claude-sonnet-4, claude-opus-4, … |
| OpenAI | gpt-4o, gpt-4.1, o3, o4-mini, … |
| GLM (Zhipu) | glm-5.2, glm-5.3, … |
| Qwen (Alibaba) | qwen3.5-plus, qwen-plus, … |
| MiniMax | MiniMax-M3, … |
| xAI | grok-4.5, … |
| Google Gemini | gemini-2.5-pro, gemini-2.5-flash, … |
| Xiaomi | mimo-v2.5, … |

You can bring your own keys for any of these, or use ASG's managed keys.

**Compatible with any coding agent** that speaks the OpenAI Chat Completions API or Anthropic Messages API:

Claude Code · Cursor · Windsurf · Cline · Codex CLI · Continue · Aider · OpenCode · Cline · Roo Code · Augment · And more

---

## Architecture at a glance

```
coding agents
  (Claude Code / Cursor / Cline / Windsurf / ...)
        │   OpenAI-compatible API
        ▼
  nginx :8888          public ingress, TLS, rate limiting, path routing
        │
        ▼
  ASG Gateway :18788   the optimization brain (L1 + L2 + Dual-Mode + Cache + Tool-fix)
        │
        ▼
  Provider Relay :8140  auth, key management, upstream adapters
        │
        ▼
  Model providers       DeepSeek / GLM / Qwen / MiniMax / OpenAI / Anthropic / xAI / ...
```

The optimization layers are **protocol-transparent**: clients talk a standard OpenAI-compatible or Anthropic Messages API, and upstreams receive a cleaned, compressed, cache-optimized request. Nothing about the client's request format has to change.

---

## Privacy & security

| Concern | What ASG does |
|---------|---------------|
| API keys | Stored encrypted at rest. Never shared, never logged in plaintext. |
| Chat content | Processed in-memory for optimization. Not retained after the request completes. |
| Model outputs | Forwarded to you in real time. Not stored, not used for training. |
| Usage data | Aggregate metrics only (token counts, cache hits, latency). No per-message logging. |

> ASG runs on infrastructure you can audit. See [SECURITY.md](SECURITY.md) for the full security policy.

---

## Documentation

- [Overview & design goals](docs/getting-started/overview.md)
- [Quick Start — point a client at the gateway](docs/getting-started/quickstart.md)
- [Supported clients](docs/getting-started/supported-clients.md)
- [Configuration reference](docs/getting-started/configuration.md)
- [Concepts: how each optimization layer works](docs/concepts/)
- [Benchmarks & methodology](docs/benchmarks/)
- [Architecture (no code)](docs/contributing/ARCHITECTURE.md)
- [Technical deep dive](docs/contributing/TECHNICAL.md)
- [FAQ](docs/faq.md)
- [Examples](examples/): [nginx routing](examples/nginx-routing.conf.example), [client configs](examples/client-configs.md)

---

## License & source

- **This repository:** Documentation only. © 2026 ASG. See [LICENSE](LICENSE).
- **Source code:** Closed-source, proprietary. Not published.
- **Contact:** Open an issue or discussion — or reach the team for commercial licensing.

> ⚠️ This is a **public documentation repository for a commercial product**. The mechanisms described here are the product's intellectual property. If you are evaluating whether to build this yourself, note that the value is in the *implementation*, not the concept.
