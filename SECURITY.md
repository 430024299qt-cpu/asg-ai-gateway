# Security Policy

## Overview

ASG (AI Savings Gateway) is a hosted API gateway that processes LLM traffic on behalf of authenticated users. This document describes the security boundaries, data handling practices, and how to report vulnerabilities.

---

## Data handling

### API keys

- Stored **encrypted at rest** using application-level encryption.
- Never logged in plaintext. Only the last 4 characters are visible in the dashboard.
- Never shared with third parties or transmitted to any endpoint other than the intended model provider.
- Users can rotate or delete their keys at any time via the dashboard.

### Chat content (messages, tool calls, tool results)

- Processed **in-memory** for the purpose of token optimization (normalization, folding, caching).
- **Not persisted** after the request completes. Optimization state (cache fingerprints, fold markers) is ephemeral.
- Not used for model training, fine-tuning, or any purpose other than optimizing the current request.

### Model outputs

- Forwarded to the client in real time (streaming or non-streaming).
- Not stored on ASG servers after delivery.

### Usage metrics

- Aggregate metrics are collected: token counts, cache hit rates, latency, cost savings.
- **No per-message logging.** Individual prompts, completions, and tool calls are not logged.
- Dashboard users can only see their own metrics. Full admin access is restricted.

---

## Network & infrastructure

- All public traffic enters through **nginx** on port 8888 with TLS termination (when configured).
- Internal services (ASG Gateway :18788, Provider Relay :8140) bind to localhost and are not directly accessible from the internet.
- Rate limiting is applied per-IP at the nginx layer.
- Upstream connections to model providers use HTTPS.

---

## Authentication

- Users authenticate via the web dashboard (port 8888) with username/password.
- API requests are authenticated via bearer tokens issued per-user.
- Admin accounts have elevated privileges for dashboard access only — they do not have access to user API keys in plaintext.

---

## Threat model

| Threat | Mitigation |
|--------|------------|
| API key leakage | Encrypted at rest, never logged in plaintext, user-rotatable |
| Man-in-the-middle (client ↔ ASG) | TLS termination at nginx; users should configure HTTPS where possible |
| Man-in-the-middle (ASG ↔ provider) | HTTPS to upstream providers |
| Unauthorized dashboard access | Password auth + rate limiting on login |
| Denial of service | Per-IP rate limiting, connection limits |
| Insider access | Minimal logging, encrypted key storage, no per-message retention |

---

## Reporting a vulnerability

If you discover a security vulnerability in ASG:

1. **Do not** open a public issue.
2. Open a private discussion on the [GitHub repository](https://github.com/430024299qt-cpu/asg-ai-gateway) or contact the team directly.
3. Include a description of the vulnerability, steps to reproduce, and any potential impact.
4. We aim to acknowledge reports within 48 hours and provide a fix timeline within 7 days.

---

## Scope

This security policy applies to:
- The ASG Gateway software and its deployment at 154.12.86.206
- The web dashboard at port 8888
- The GitHub documentation repository

This policy does **not** cover:
- Third-party model providers (DeepSeek, Anthropic, OpenAI, etc.) — refer to their respective security policies
- User-managed infrastructure (client machines, network configurations)
