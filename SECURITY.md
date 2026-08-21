# Security Policy — VBK-AI Agent Gateway

## Reporting Vulnerabilities

If you discover a security vulnerability, please report it responsibly:

- **Email**: security@agentgwapi.online
- **Do NOT** open a public GitHub issue for security vulnerabilities
- **Response time**: We aim to acknowledge within 48 hours

## Security Architecture

### API Key Storage

| Layer | Protection |
|-------|-----------|
| **At rest** | AES-256 encrypted in database |
| **In transit** | TLS 1.2+ for all API communication |
| **In memory** | Keys loaded on-demand, not cached in plaintext |
| **Access control** | Owner-isolated; each user sees only their own keys |

### Data Handling

| Data Type | Retention | Notes |
|-----------|-----------|-------|
| **Chat content** | Not stored | Proxied in real-time only |
| **Provider API keys** | Encrypted at rest | User-managed, rotate anytime |
| **Usage metrics** | Aggregate only | Token counts, cost estimates |
| **Request logs** | 7-day rolling | For debugging, no message content |

### Network Security

- All endpoints served over TLS
- Rate limiting per user/IP (configurable)
- CORS restricted to registered domains
- Admin endpoints require separate authentication

## Threat Model

| Threat | Mitigation |
|--------|-----------|
| API key leakage | Encrypted storage, last-4 display only |
| Unauthorized access | Per-user token isolation, session auth |
| Replay attacks | Request signing, nonce validation |
| Provider outage | Multi-provider failover, circuit breaker |
| DDoS | Rate limiting, nginx layer, IP-based throttling |
| Man-in-the-middle | TLS everywhere, certificate pinning |

## Compliance

- No PII collection beyond registration email
- No conversation content stored
- GDPR-compatible data handling (EU users can request data deletion)
- SOC 2 Type I compliance in progress

## Security Audits

- **Internal**: Quarterly code review and dependency audit
- **External**: Annual penetration testing (results available on request)
- **Dependencies**: Automated vulnerability scanning via GitHub Dependabot

## Supported Versions

| Version | Supported |
|---------|-----------|
| Current (latest) | ✅ |
| Previous release | ✅ (security fixes only) |
| Older versions | ❌ |

---

*Last updated: August 2026*
