# Quickstart

Pointing a coding agent at ASG takes one change: **set its API base URL and a
key**. No agent-side install, no wrapper, no SDK change.

## 1. Get access

ASG is a managed or self-hosted commercial gateway. Obtain from the operator:

- **Base URL** — e.g. `https://gateway.example.com/v1`
- **API key** — a per-user token issued by the gateway

## 2. Point your client at the gateway

Every OpenAI-compatible client has a `base_url` (or equivalent) setting:

| Client | Where to change | Value |
|---|---|---|
| Claude Code | `ANTHROPIC_BASE_URL` | `https://gateway.example.com/v1` |
| Cline | Provider → OpenAI-compatible → base URL | `https://gateway.example.com/v1` |
| Cursor | Settings → API base URL | `https://gateway.example.com/v1` |
| NextChat | Settings → API host | `https://gateway.example.com` |
| opencode | `~/.config/opencode/` provider config | `https://gateway.example.com/v1` |
| pi | `PI_AGENT_BASE_URL` | `https://gateway.example.com/v1` |

Set the API key to the token the operator gave you. Use a **model name the
gateway recognizes** (e.g. `deepseek-v4-flash`) — the gateway maps it to the
upstream route.

> If your client appends `/v1` itself, use the host root (`https://gateway.example.com`)
> to avoid a doubled `/v1/v1` path. ASG's ingress normalizes both forms.

## 3. Verify

Send a chat completion and confirm the response:

```bash
curl https://gateway.example.com/v1/chat/completions \
  -H "Authorization: Bearer <your-key>" \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4-flash","messages":[{"role":"user","content":"Say OK"}],"stream":false}'
```

You should get a `200` with a normal completion — and the gateway will start
recording per-request savings for your session.

## 4. Watch the savings

Ask the operator for a session-level breakdown, or check the gateway dashboard.
You should see:

- prefix-cache hit rates climbing as your session stabilizes (target 75%+),
- `[FOLD]` reductions on long multi-turn sessions,
- and a per-request `tokens_saved` figure.

## Next

- [Supported clients](supported-clients.md)
- [Configuration reference](configuration.md)
- [Concepts: how the layers work](../concepts/)
