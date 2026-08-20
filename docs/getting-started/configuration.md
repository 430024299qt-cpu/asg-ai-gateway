# Configuration reference

ASG is configured through environment variables and a JSON routing file. This
reference documents the *shape* of the configuration. Actual values are set by
the operator at deploy time and are not part of this repository.

## Environment variables

### Core

| Variable | Purpose |
|---|---|
| `UPSTREAM_URL` | Default upstream provider endpoint. |
| `ASG_UPSTREAM_KEY` | Key used by the gateway to authenticate to the upstream. |
| `ASG_UPSTREAM_URL` | Upstream base URL for the optimized path. |
| `MODE` | Operating mode switch (e.g. multi-provider vs. single). |

### Models & routing

| Variable | Purpose |
|---|---|
| `ASG_LARGE_MODEL` | Model name used for "large" tasks. |
| `ASG_SMALL_MODEL` | Model name used for "small"/cheap tasks. |
| `ASG_LOCAL_MODEL` | Optional local model name (if a local endpoint is wired). |
| `ASG_LOCAL_URL` | Optional local model endpoint. |

### Optimization layers

| Variable | Purpose | Default |
|---|---|---|
| `ASG_FOLD_OPTIMIZER_ENABLED` | Master switch for Layer 2 context folding. | off (opt-in) |
| `ASG_FOLD_SHADOW` | Shadow mode: compute fold statistics without rewriting traffic. | off |
| `ASG_STREAM_TOOL_NORMALIZE` | Streaming tool-call delta normalization. | on |
| `ASG_CACHE_TTL` | Cache TTL injected via `cache_control` (e.g. `1h`). | provider default |
| `ASG_LOCAL_KEY` | Key for the local model endpoint, if any. | — |

### NewAPI / token management

| Variable | Purpose |
|---|---|
| `NEWAPI_ENABLED` | Whether token/user management is handled by the bundled relay. |
| `NEWAPI_ADMIN_TOKEN` | Admin token for the token-management service. |
| `NEWAPI_BASE_URL` | Base URL of the token-management service. |

### Provider keys

`DEEPSEEK_API_KEY`, `DEEPSEEK_API_URL`, `DEEPSEEK_WARMUP_MODEL`, and the
corresponding variables for other providers are used for the cache **warmup**
path.

## Routing rules

Routing is driven by a JSON document with intent categories
(`command`, `coding`, `writing`, `analysis`, `other`, …). Each category carries:

- `exact_match` — literal command matches,
- `keywords` — keyword triggers,
- `templates` — pattern triggers,
- `route_to` — the model to route to,
- `enabled` — whether the rule is active.

Routing is **optional** and off by default; when disabled, every request goes to
the configured default model. When enabled, ASG classifies the request intent and
routes it to the cheapest capable model.

## Layering

Configuration is intentionally layered: a base `.env` (core), and an extended
`.env_multi` (model routing + fold toggles). The gateway re-reads the extension
without restart for hot-tunable switches.

> The exact schema may evolve; this document describes the stable contract.
