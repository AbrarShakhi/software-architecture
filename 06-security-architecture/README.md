# 06 — Security Architecture

> Design systems that protect data, authenticate users, and resist attack — not as an afterthought, but as a core architectural concern.

Security is not a feature to bolt on at the end. Decisions made early — how sessions are stored, where trust is established, which inputs are validated — determine whether a system can be compromised.

## Topics

| # | Topic | Core Idea |
|---|-------|-----------|
| 1 | [Auth Patterns](./01-auth-patterns.md) | Session, JWT, OAuth 2.0, OIDC |
| 2 | [Zero Trust](./02-zero-trust.md) | Never trust, always verify |
| 3 | [OWASP Top 10](./03-owasp-top10.md) | The ten most critical web vulnerabilities |

## Security is Layered

```
┌──────────────────────────────────────────────────┐
│  Authentication  Who are you?                    │
│  Authorization   What are you allowed to do?     │
│  Input Validation  Is the data safe?             │
│  Transport Security  Is the channel encrypted?   │
│  Data Security  Is data protected at rest?       │
│  Audit Logging  What happened and when?          │
└──────────────────────────────────────────────────┘
```

All layers must hold. A system that authenticates perfectly but trusts all input is still exploitable.
