# VBK-AI Agent Gateway

**Same budget. Ship more code.**

[Website](http://www.agentgwapi.online:8888/) · [Documentation](docs/) · [Security](SECURITY.md) · [Contributing](CONTRIBUTING.md)

VBK-AI Agent Gateway (agent-gateway) is a server-side API gateway that optimizes LLM traffic for AI coding agents. It sits between your agent and LLM providers, applying caching, request normalization, and routing strategies to reduce token costs by **50-95%** — with zero client-side changes.

> **Zero install.** Register at [www.agentgwapi.online/register](http://www.agentgwapi.online:8888/register), add your provider API key, and point your agent's `base_url` to your gateway endpoint.

---

## Quick Start (3 Steps)

1. **Register** — Visit [www.agentgwapi.online/register](http://www.agentgwapi.online:8888/register) and create a free account
2. **Add Provider Key** — In the dashboard, add your DeepSeek / Anthropic / OpenAI / Qwen / Gemini API key
3. **Configure Agent** — Change your agent's `base_url` to your gateway endpoint:

```
https://your-gateway-url/v1
```

That's it. Your existing workflow is unchanged — same agent, same models, lower cost.

---

---

## Founders

**Tristan Qin** · **Tomcom Shu** · **Lewis**

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

> **Supports all model families from each provider.** Full model names can be specified via the provider API. Below are the available provider families and their representative models.

| Provider | Available Models / Families |
|----------|---------------------------|
| **DeepSeek** | deepseek-v4, deepseek-v4-flash, deepseek-chat, deepseek-r1, deepseek-v3, deepseek-coder-v2, deepseek-prover-v2, deepseek-janus-pro |
| **Anthropic** | claude-opus-4, claude-sonnet-4, claude-sonnet-4-20250514, claude-haiku-4-5, claude-3.5-sonnet, claude-3.5-haiku, claude-3-opus, claude-3-sonnet, claude-3-haiku |
| **OpenAI** | gpt-4o, gpt-4o-mini, gpt-4.1, gpt-4.1-mini, gpt-4.1-nano, o1, o1-pro, o3, o3-mini, o3-pro, o4-mini, gpt-4-turbo, gpt-4, gpt-3.5-turbo |
| **Google Gemini** | gemini-2.5-flash, gemini-2.5-pro, gemini-2.0-flash, gemini-2.0-flash-lite, gemini-1.5-flash, gemini-1.5-pro, gemini-exp-1206 |
| **Qwen (Alibaba)** | qwen3.5-plus, qwen3.5-max, qwen3-coder, qwen3-plus, qwen3-max, qwen3-moe (all sizes), qwen-vl-max, qwen-audio-turbo, qwen-long |
| **GLM (Zhipu AI)** | glm-5.3, glm-5.2, glm-5.1, glm-4-plus, glm-4-long, glm-4-flash, glm-4-air, glm-4-airx, glm-4v-plus, glm-4v, cogview-4, cogvideox |
| **MiniMax** | MiniMax-M3, MiniMax-Text-01, MiniMax-01, abab7, abab6.5s, abab6.5-chat, hailuo-video |
| **xAI** | grok-4, grok-4.5, grok-3, grok-3-mini, grok-2, grok-2-vision |
| **Xiaomi (MiMo)** | mimo-v2.5, mimo-v2-pro, mimo-v2 (all sizes: 7B, 32B, 72B) |
| **Silicon Flow** | deepseek-v3, deepseek-r1, qwen3-235b, glm-4-plus, internlm3, yi-lightning (open-source model hub) |

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

- 🌐 **Website**: [www.agentgwapi.online](http://www.agentgwapi.online:8888/)
- **Register**: [www.agentgwapi.online/register](http://www.agentgwapi.online:8888/register)
- **Login**: [www.agentgwapi.online/login](http://www.agentgwapi.online:8888/login)
- 📧 **Email**: support@agentgwapi.online
- 🐛 **Issues**: [GitHub Issues](https://github.com/430024299qt-cpu/VBK-AI-Agent-Gateway/issues)
