# VBK-AI Agent Gateway

**Same budget. Ship more code.**

[Website](http://www.agentgwapi.online:8888/) · [Documentation](docs/) · [Security](SECURITY.md) · [Contributing](CONTRIBUTING.md)

VBK-AI Agent Gateway is a server-side API gateway that optimizes LLM traffic for AI coding agents. It sits between your agent and LLM providers, applying proprietary caching, normalization, and routing strategies to reduce token costs by **50-95%** — with zero client-side changes.

> **Zero install.** Register at [www.agentgwapi.online/register](http://www.agentgwapi.online:8888/register), add your provider API key, and point your agent's `base_url` to your gateway endpoint.

---

## Quick Start (3 Steps)

1. **Register** — Visit [www.agentgwapi.online/register](http://www.agentgwapi.online:8888/register) and create a free account
2. **Add Provider Key** — In the dashboard, add your DeepSeek / Anthropic / OpenAI / Qwen / Gemini API key
3. **Configure Agent** — Change your agent's `base_url` to your gateway endpoint and start coding

That's it. Your existing workflow is unchanged — same agent, same models, lower cost.

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

## How It Works

```
Your Agent (Cursor, Cline, Aider, ...)
        │
        ▼
  VBK-AI Agent Gateway
  ┌─────────────────────────────┐
  │  Proprietary Optimization   │
  │  • Request normalization    │
  │  • Context compression      │
  │  • Smart caching & routing  │
  └──────────┬──────────────────┘
             │ optimized request
             ▼
       LLM Provider
  (DeepSeek, Anthropic,
   OpenAI, Gemini, ...)
```

Your agent sends requests to the gateway instead of directly to the LLM provider. The gateway applies multiple layers of proprietary optimization transparently — your coding workflow stays exactly the same.

See [BENCHMARK.md](docs/benchmarks/BENCHMARK.md) for detailed performance measurements.

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
| [docs/getting-started/](docs/getting-started/) | Quick start and client configuration guides |
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

- **Website**: [www.agentgwapi.online](http://www.agentgwapi.online:8888/)
- **Register**: [www.agentgwapi.online/register](http://www.agentgwapi.online:8888/register)
- **Login**: [www.agentgwapi.online/login](http://www.agentgwapi.online:8888/login)
- **Email**: support@agentgwapi.online
- **Issues**: [GitHub Issues](https://github.com/VBK-AI-Agent-Gateway/VBK-AI-Agent-Gateway/issues)
