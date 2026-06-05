# Contributing to INTENT-7-SECURE

INTENT-7-SECURE (Secure Intent & Control Protocol) is the skeleton of the protocol family.

## Specification Files

- English: [spec/INTENT-7-SECURE.md](spec/INTENT-7-SECURE.md)
- Chinese: [spec/INTENT-7-SECURE.zh-CN.md](spec/INTENT-7-SECURE.zh-CN.md)

## How to Propose Changes

1. Read the [organization-level contribution guide](https://github.com/CommonIntents/.github/blob/main/CONTRIBUTING.md)
2. Open an Issue describing the problem or improvement you've identified
3. INTENT-7-SECURE is a transport security implementation; proposals should remain within its scope
4. Update both the English and Chinese versions in your PR

## Scope Constraint

INTENT-7-SECURE is responsible for transport security only. It does not define intent semantics (INTENT-7),
authorization logic (CAPABILITY-13), or format negotiation (BIND-19). Proposals crossing these boundaries
will be redirected to the appropriate protocol.
