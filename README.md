# VBK-AI Agent Gateway

**Same budget. Ship more code.**

[Website](http://www.agentgwapi.online/) · [Documentation](docs/) · [Security](SECURITY.md) · [Contributing](CONTRIBUTING.md)

VBK-AI Agent Gateway (agent-gateway) is a server-side API gateway that optimizes LLM traffic for AI coding agents. It sits between your agent and LLM providers, applying caching, request normalization, and routing strategies to reduce token costs by **50-95%** — with zero client-side changes.

> **Zero install.** Register at [www.agentgwapi.online/register](http://www.agentgwapi.online/register), add your provider API key, and point your agent's `base_url` to your gateway endpoint.

---

## Quick Start (3 Steps)

1. **Register** — Visit [www.agentgwapi.online/register](http://www.agentgwapi.online/register) and create a free account
2. **Add Provider Key** — In the dashboard, add your DeepSeek / Anthropic / OpenAI / Qwen / Gemini API key
3. **Configure Agent** — Change your agent's `base_url` to your gateway endpoint:

```
https://your-gateway-url/v1
```

That's it. Your existing workflow is unchanged — same agent, same models, lower cost.

---

## Supported Agents

| Agent | Provider | Status |
|-------|----------|--------|
| **Claude Code** | Anthropic | ✅ Production |
| **Cursor** | OpenAI-compatible | ✅ Production |
| **Windsurf** | OpenAI-compatible | ✅ Production |
| **Cline** | OpenAI-compatible | ✅ Production |
| **Codex CLI** | OpenAI-compatible | ✅ Production |
| **Continue** | OpenAI-compatible | ✅ Production |
| **Aider** | OpenAI-compatible | ✅ Production |
| **OpenCode** | OpenAI-compatible | ✅ Production |

---

## Supported Providers

| Provider | Models |
|----------|--------|
| **DeepSeek** | deepseek-v4-flash, deepseek-chat |
| **Anthropic** | claude-sonnet-4, haiku-4-5, claude-opus-4 |
| **OpenAI** | gpt-4o, gpt-4o-mini, o1, o3-mini |
| **Google Gemini** | gemini-2.5-flash, gemini-2.5-pro |
| **Qwen (Alibaba)** | qwen3.5-plus |
| **GLM (Zhipu)** | glm-5.2, glm-5.3 |
| **MiniMax** | MiniMax-M3 |
| **xAI** | grok-4.5 |
| **Xiaomi** | mimo-v2.5 |

---

## Optimization Layers

| Layer | What It Does | Savings |
|-------|-------------|---------|
| **L1 Request Normalization** | System prompt dedup, tool schema compression, cache-aware prefix ordering | Prefix-cache hits 75-94% |
| **L2 Context Folding** | Compress conversation history while preserving semantic structure | ~350k tokens saved per session |
| **Dual-Mode Output Constraint** | Structured output enforcement for tool-call responses | 91.7% KV cache hit rate |
| **Cache (cache_control + warmup)** | Anthropic-native 1h cache, cross-session prefix warmup | 90% cost reduction on cached tokens |
| **Tool-call Fix** | Streaming tool-call normalization across all providers | Eliminates stuck/hallucinated tool calls |
| **Intent Routing** | Lightweight classifier routes simple queries to cheaper models | Route-based cost reduction |

**Combined effect: ~86% average token-cost reduction** on production workloads.

See [BENCHMARK.md](docs/benchmarks/BENCHMARK.md) for detailed per-layer measurements.

---

## How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                     Your Machine                                │
│  ┌──────────┐     base_url     ┌──────────────────────────┐    │
│  │ Agent    │ ──────────────►  │  VBK-AI Agent Gateway    │    │
│  │ (Cursor, │   /v1/chat/      │                          │    │
│  │  Cline,  │   completions    │  L1 Normalize            │    │
│  │  Aider,  │                  │  L2 Context Fold         │    │
│  │  ...)    │ ◄────────────── │  Dual-Mode Output        │    │
│  │          │   optimized      │  Cache + Warmup          │    │
│  │          │   response       │  Intent Router            │    │
│  └──────────┘                  └──────────┬───────────────┘    │
│                                           │                    │
└───────────────────────────────────────────┼────────────────────┘
                                            │ optimized request
                                            ▼
                                   ┌────────────────┐
                                   │  LLM Provider   │
                                   │  (DeepSeek,     │
                                   │   Anthropic,    │
                                   │   OpenAI, ...)  │
                                   └────────────────┘
```

---

## Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `GATEWAY_PORT` | Gateway listen port | `18788` |
| `AI_GATEWAY_PORT` | ai-gateway (VaalaCat) port | `8140` |
| `NEWAPI_ENABLED` | Enable NewAPI compatibility layer | `true` |
| `DEEPSEEK_API_KEY` | DeepSeek provider key | — |
| `ANTHROPIC_API_KEY` | Anthropic provider key | — |
| `OPENAI_API_KEY` | OpenAI provider key | — |

### .env Configuration

```bash
# Copy from .env.example
cp .env.example .env

# Edit with your API keys
vim .env

# Start the gateway
python gateway_merged.py
```

---

## Privacy & Data Policy

- **No chat content stored**: All requests are proxied in real-time; no conversation logs are retained
- **API keys encrypted at rest**: Provider keys are stored encrypted in the database
- **Metrics only**: The dashboard shows aggregate cost/token metrics, not message content
- **Open audit**: Security practices documented in [SECURITY.md](SECURITY.md)

---

## Documentation

| Document | Description |
|----------|-------------|
| [docs/getting-started/](docs/getting-started/) | Installation and configuration guides |
| [docs/concepts/](docs/concepts/) | Architecture and optimization layer explanations |
| [docs/benchmarks/BENCHMARK.md](docs/benchmarks/BENCHMARK.md) | Detailed performance measurements |
| [SECURITY.md](SECURITY.md) | Security policy and vulnerability reporting |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contribution guidelines |

---

## Contributing

We welcome documentation contributions, bug reports, and feature requests. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

> **Note**: This repository contains documentation only. The gateway source code is proprietary and not included.

---

## License

This repository is licensed under the [MIT License](LICENSE) for documentation content.

The VBK-AI Agent Gateway software itself is proprietary. See [LICENSE](LICENSE) for details.

---

## Links

- 🌐 **Website**: [www.agentgwapi.online](http://www.agentgwapi.online/)
- **Register**: [www.agentgwapi.online/register](http://www.agentgwapi.online/register)
- **Login**: [www.agentgwapi.online/login](http://www.agentgwapi.online/login)
- 📧 **Email**: support@agentgwapi.online
- 🐛 **Issues**: [GitHub Issues](https://github.com/430024299qt-cpu/Agent-Gateway/issues)
