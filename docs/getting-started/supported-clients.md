# Supported clients

ASG exposes a standard **OpenAI-compatible** API (`/v1/chat/completions`,
`/v1/models`) and an **Anthropic Messages** endpoint (`/v1/messages`). Any client
that can be pointed at a custom base URL works.

## Verified clients

| Client | Protocol | Notes |
|---|---|---|
| **Claude Code** | Anthropic Messages | Set `ANTHROPIC_BASE_URL`. Benefits from `cache_control` injection and tool-call normalization. |
| **Cline** | OpenAI-compatible | Uses `/v1/chat/completions`. Use a recognized model name. |
| **Cursor** | OpenAI-compatible | Custom API base URL. |
| **NextChat** | OpenAI-compatible | Custom API host. |
| **opencode** | OpenAI-compatible | Provider config in `~/.config/opencode/`. |
| **pi** | OpenAI-compatible | `PI_AGENT_BASE_URL`. |
| **Codex CLI** | OpenAI-compatible | Custom base URL. |
| **Any OpenAI SDK** | OpenAI-compatible | Point `base_url` at the gateway. |

## Compatibility notes

- **Doubled `/v1`** — clients that append `/v1` to a base URL that already
  contains it will produce `/v1/v1/...`. ASG's ingress normalizes this.
- **Model lists** — `/v1/models` returns the models the gateway routes, filtered
  by the caller's token.
- **Streaming** — SSE streaming is supported and buffering is disabled at the
  ingress so tool-call deltas arrive in real time.
- **Responses API** — Anthropic Responses-style paths are normalized to Messages.

## Not supported

- Protocol features the upstream provider doesn't offer (e.g. a model that cannot
  do structured output will not suddenly do so).
- Clients that bypass the gateway by calling the upstream directly.

Next: [Configuration reference](configuration.md)
