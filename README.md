# INTENT-7-SECURE — INTENT-7 Secure Transport (Optional Reference Implementation) [![Org](https://img.shields.io/badge/Org-CommonIntents--144-darkgray.svg)](https://github.com/CommonIntents)

**One possible implementation of BIND-19, not a mandatory protocol component.**

INTENT-7-SECURE provides mTLS-based encrypted transport as a reference implementation of the BIND-19 transport binding specification.

INTENT-7-SECURE is NOT the "skeleton" of the protocol family — transport security is infrastructure, and trust is a multi-dimensional collaboration across the ecosystem (L0 gene-lock hash, Cellrix view hash, Helix-Mind experience verification, Tuck physical isolation).

## Identity Model
The Agent's private key is its local trust anchor. Identity is cryptographically proven at mTLS handshake.

## Protocol Stack
```
INTENT-7 (intent syntax)
  ↑
BIND-19 (transport binding)
  ↑
INTENT-7-SECURE ← You are here (optional mTLS reference implementation)
  ↑
CAPABILITY-13 (consensus confirmation)
```

## Read the Spec
- [INTENT-7-SECURE v1.0.0-RFC-4](spec/INTENT-7-SECURE.md)
- [中文版](spec/INTENT-7-SECURE.zh-CN.md)

## Related
| Protocol | Repository |
|:---|:---|
| INTENT-7 | [CommonIntents/INTENT-7](https://github.com/CommonIntents/INTENT-7) |
| CAPABILITY-13 | [CommonIntents/CAPABILITY-13](https://github.com/CommonIntents/CAPABILITY-13) |
| BIND-19 | [CommonIntents/BIND-19](https://github.com/CommonIntents/BIND-19) |

## License
Apache 2.0 — see [LICENSE](LICENSE).
