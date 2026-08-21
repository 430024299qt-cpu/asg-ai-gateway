# Quickstart

Pointing a coding agent at VBK-AI Agent Gateway takes one change: **set its API base URL and a key**. No agent-side install, no wrapper, no SDK change.

## 1. Get access

Register at [www.agentgwapi.online/register](http://www.agentgwapi.online:8888/register) to get:

- **Base URL** — your gateway endpoint (shown in dashboard after registration)
- **API key** — a per-user token issued by the gateway

## 2. Point your client at the gateway

Every OpenAI-compatible client has a `base_url` (or equivalent) setting. Set it to your gateway endpoint URL provided in the dashboard.

Set the API key to the token the gateway issued you. Use a **model name the gateway recognizes** (e.g. `deepseek-v4-flash`) — the gateway maps it to the upstream provider automatically.

## 3. Verify

Send a chat completion request through your agent and confirm you get a normal response. The gateway will start recording per-request savings for your session.

## 4. Watch the savings

Check the gateway dashboard. You should see:

- Cache hit rates climbing as your session stabilizes
- Context compression on long multi-turn sessions
- Per-request token savings figures

## Next

- [Supported clients](supported-clients.md)
- [Configuration reference](configuration.md)
