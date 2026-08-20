# Client configuration examples

The entire client change is: **point `base_url` at ASG.** Model names and keys
stay the same. Below are examples for common clients.

Assume the ASG endpoint is `https://asg.example.com` and your key is
`sk-asg-...`.

## Claude Code

```bash
export ANTHROPIC_BASE_URL="https://asg.example.com"
export ANTHROPIC_AUTH_TOKEN="sk-asg-..."
# keep ANTHROPIC_MODEL as-is, or set the model you want
export ANTHROPIC_MODEL="deepseek-chat"
```

> If ASG exposes the Anthropic passthrough path, Claude Code can talk to it
> natively and get provider-native caching.

## Cline (VS Code)

In the provider settings, add an OpenAI-compatible provider:

- **Base URL:** `https://asg.example.com/v1`
- **API Key:** `sk-asg-...`
- **Model:** the upstream model name (e.g. `deepseek-chat`)

Use `/v1/chat/completions` — Cline's `/models` probe is answered by the edge
too, so model listing works.

## Cursor

Settings → Models → API Base URL:

```
https://asg.example.com/v1
```

Use an OpenAI-compatible override with your ASG key.

## opencode / NextChat / similar OpenAI-compatible clients

Set `base_url` (or `endpoint`) to:

```
https://asg.example.com/v1
```

## General rule

- **OpenAI-compatible clients:** `https://asg.example.com/v1`
- **Anthropic-native clients:** `https://asg.example.com` (passthrough path)
- **`/v1` normalization:** ASG accepts `base_url` with or without `/v1` and
  normalizes it, so both `https://asg.example.com/v1` and
  `https://asg.example.com` work on the OpenAI path.

## Verify

```bash
curl -s https://asg.example.com/v1/models \
  -H "Authorization: Bearer sk-asg-..." | head -40
```

Then run one chat request and check the response headers for cache accounting
(e.g. cache-hit indicators) to confirm the optimized path is active.
