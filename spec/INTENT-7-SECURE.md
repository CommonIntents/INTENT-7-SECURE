# INTENT-7-SECURE: Secure Intent & Control Protocol

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) [![Version](https://img.shields.io/badge/Version-0.1.0--draft-orange.svg)]() [![Status](https://img.shields.io/badge/Status-RFC%20Draft-yellow.svg)]() [![Org](https://img.shields.io/badge/Org-CommonIntents-144-darkgray.svg)](https://github.com/CommonIntents)

**Version**: 0.1.0-draft  
**Status**: Working Group Internal Draft  
**Date**: 2026-05-22  
**License**: Apache 2.0  

---

## 1. Core Positioning

INTENT-7-SECURE (INTENT-7 Secure) is the **transport security implementation based on mTLS**.

It establishes an end-to-end encrypted channel between the Agent and the tool, and completes cryptographic identity proof within the first millisecond of connection establishment.

INTENT-7-SECURE is the **skeleton** of the protocol stack — providing structural security guarantees.

---

## 2. Relationship with Other Layers

BIND-19 is the **specification**; INTENT-7-SECURE is the **implementation**.

BIND-19 defines: "Intent data is carried over a secure transport channel in a negotiated format."  
INTENT-7-SECURE implements: "That secure channel is mTLS over HTTPS."

INTENT-7-SECURE provides **long-term identity proof**. CAPABILITY-13 issues **short-lived operational credentials** (JWT) on top of this. Long-term identity is not directly used for operational authorization; operational authorization must pass through a time-bound credential.

---

## 3. Identity Model

### 3.1 Trust Anchor

The Agent's private key is its sole trust anchor for identity. It does not rely on tokens issued by any centralized authority.

The Agent's identity is determined by the public key of the certificate corresponding to the private key it holds. The certificate may be a standard X.509 certificate issued by a CA, or a self-signed certificate. Regardless of the certificate's origin, the Agent's identity is cryptographically proven upon completion of the mTLS handshake.

### 3.2 Identity Verification Flow

```
Agent                         Server
  │                              │
  ├──── Establish TCP Connection ─►
  ├──── TLS ClientHello ─────────►
  │                              │
  │◄─── TLS ServerHello ────────┤
  │      + Server Certificate    │
  │◄─── CertificateRequest ─────┤
  │      (client cert required)  │
  │                              │
  ├──── Client Certificate ──────►
  ├──── CertificateVerify ───────►
  │      (signs handshake data   │
  │       with private key)      │
  │                              │
  │◄─── Handshake Complete ─────┤
  │      Agent identity          │
  │      cryptographically proven│
  │                              │
  ├──── Encrypted INTENT-7/CAPABILITY-13 Data ──►
```

Identity binding is completed within the first millisecond of connection establishment. Any subsequent operations are bound to this already-proven identity.

---

## 4. mTLS Handshake Specification

### 4.1 Certificate Requirements

Both server and client MUST hold certificates. The certificate public key serves as the identity identifier.

### 4.2 Certificate Verification

The server verifies the validity of the client certificate: the certificate chain is complete, the certificate is within its validity period, and the certificate has not been revoked.

The Agent verifies the validity of the server certificate: to prevent man-in-the-middle attacks.

### 4.3 Session Management

TLS sessions MAY be cached to accelerate subsequent connections. The management and validity period of session caches are determined by the implementation.

---

## 5. Protocol Boundaries

INTENT-7-SECURE **is responsible for**:
- Defining the mTLS handshake specification
- Defining the private-key-based identity model
- Defining certificate verification requirements

INTENT-7-SECURE **is not responsible for**:
- Mandating the specific TLS version (determined by operational configuration)
- Mandating whether to enable 0-RTT (determined by operational configuration)
- Mandating certificate issuance and revocation processes (provided by the PKI ecosystem)
- Defining transport format negotiation (BIND-19's responsibility)
- Defining identity-based authorization logic (CAPABILITY-13's responsibility)

---

## 6. Security Properties

| Property | Implementation |
|:---|:---|
| **Encryption** | TLS end-to-end encryption; data cannot be eavesdropped during transmission |
| **Identity** | Client certificate verification; Agent identity established at handshake |
| **Integrity** | TLS record layer MAC; tampering results in immediate connection drop |
| **Forward Secrecy** | Guaranteed by TLS key exchange algorithm (depending on operational configuration) |

---

## 7. Certificate Management

### 7.1 Certificate Issuance

Certificates are provided by the PKI ecosystem. Enterprise internal CAs, public CAs, or self-signed certificates may be used. The protocol does not mandate the issuance method.

### 7.2 Certificate Revocation

Certificate revocation is handled through standard mechanisms of the PKI ecosystem (CRL or OCSP). The protocol does not mandate the revocation method.

### 7.3 Key Protection

The storage and protection of the Agent's private key is the responsibility of the implementation. The use of secure hardware modules or operating system keychains is RECOMMENDED. The protocol does not mandate the protection method.

---

## 8. Relationship with BIND-19

BIND-19 binds INTENT-7 to INTENT-7-SECURE. After BIND-19 negotiates the transport format and integrity check, the actual data transmission is completed through INTENT-7-SECURE's mTLS channel.

INTENT-7-SECURE is the transport implementation to which BIND-19 is currently bound. In the future, alternative implementations such as INTENT-7-SECURE-QUIC or INTENT-7-SECURE-PQC may emerge. All alternative implementations must provide the same security properties: end-to-end encryption, identity proof at handshake, and transport integrity protection.

---

## 9. Protocol Boundaries Reaffirmed

INTENT-7-SECURE is the layer responsible for **transport security** in the protocol stack. It does not define intent semantics (INTENT-7's responsibility), does not define authorization logic (CAPABILITY-13's responsibility), and does not define format negotiation (BIND-19's responsibility).

**INTENT-7-SECURE exists so that CAPABILITY-13 and INTENT-7 do not need to worry about whether the channel is secure — INTENT-7-SECURE guarantees it.**

---

*This white paper is maintained by the INTENT-7/CAPABILITY-13 Protocol Working Group.*
