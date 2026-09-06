# INTENT-7-SECURE: Mutual TLS and Edge Security Specification (v1.0.0-RFC-4)
> © 2026 CommonIntents. Licensed under CC BY-ND 4.0 (https://creativecommons.org/licenses/by-nd/4.0/).

## 1. Introduction and Objectives

This specification defines **INTENT-7-SECURE**, the cryptographic identity, transport-layer confidentiality, and edge network security standard within the **CommonIntents-144 (CI-144)** suite.

INTENT-7-SECURE is dedicated exclusively to **network-level and cryptographic link protection**. To maintain the strict decoupling of the protocol stack:
- It does NOT manage frame boundaries, fragmentation, or sequence-based anti-reply (which are fully delegated to **BIND-19**) [BIND-19/spec/BIND-19.md].
- It does NOT manage logical permission or task authorization mapping (which are fully delegated to **CAPABILITY-13**) [CAPABILITY-13/spec/CAPABILITY-13.md].
- It acts as the secure, encrypted physical pipe for public networks, and defines the zero-trust local verification and credential-injection boundaries for edge environments.

---

## 2. Mutual TLS 1.3 Cryptographic Profile (Public Networks)

For all remote connections (such as WebSocket, TCP, or internet-facing endpoints), INTENT-7-SECURE mandates **Mutual TLS 1.3 (mTLS)** conforming to RFC 8446.

### 2.1 Cipher Suites and Constraints
To prevent cryptanalytic degradation, implementations MUST disable TLS 1.2 and all previous versions. Only the following cipher suites are permitted:
- `TLS_AES_256_GCM_SHA384`
- `TLS_CHACHA20_POLY1305_SHA256`

### 2.2 Mutual Certificate Verification (mTLS)
- Both client and server MUST exchange and verify X.509 v3 certificates.
- Self-signed certificates are rejected unless they are **explicitly pinned** in the L0 Gene Lock (via SHA-256 fingerprint matching).
- Certificates signed by a private CA are accepted only if the CA's root fingerprint is declared in the L0 Gene Lock.
- Certificate lifecycle management (rotation, revocation) follows the **CAPABILITY-13 Key Rotation Protocol** (Section 5) if the certificate is bound to the L0 Gene Lock identity. Otherwise, it follows standard X.509 CRL/OCSP mechanisms.
- The `traceparent` (W3C Trace Context) from the INTENT-7 metadata MUST be bound to the TLS session ID or TLS Exported Keying Material (EKM) to prevent session-hijacking and connection multiplexing attacks.

---

## 3. Local Transport Security & Peer Verification (0-Overhead)

When BIND-19 binds to local **Unix Domain Sockets (UDS, `unix://`)**, running CPU-heavy asymmetric cryptography (mTLS) is a waste of resource [Mind v3.4 §Law 01, §3.2, 12.5]. 

Instead, INTENT-7-SECURE enforces **OS-level Peer verification**, achieving absolute zero-trust security with **0.00ms cryptographic overhead**:

### 3.1 Socket File Permissions
The physical UDS socket files (e.g., `/var/run/helix.sock`) MUST be chmodded to **`0600`** immediately upon creation:
- Read/Write access is strictly limited to the running process owner.
- Parent directories MUST be restricted to prevent path-traversal attacks.

### 3.2 Peer Credential Verification (`SO_PEERCRED`)
When a client (such as `Anaphase`) connects to the local database daemon (`Helix-Mind`) over UDS, the server MUST intercept the socket and perform a **system-level Peer Credential check** [Anaphase v9.1 §3]:
- On Linux: Query socket options using **`SO_PEERCRED`** to retrieve the connecting client's `ucred` (UID, GID, PID).
- On macOS: Query socket options using **`LOCAL_PEERCRED`** to retrieve peer credentials.
- **Strict UID/GID Matching**: The server MUST verify that the client's UID/GID matches the running server daemon's process owner. If a mismatch is detected, the server MUST immediately drop the connection and log a security audit event [1.4.3].

---

## 4. Tuck Proxy and KMS Isolation Boundary

To prevent the active LLM or sandboxed WASM tools from exposing private credentials, INTENT-7-SECURE defines a strict **Zero-Credential Isolation Boundary** [4.3].

### 4.1 Credential Label Routing (No-Secret Body)
- No component inside the Anaphase executive loop or Tentacle tool sandbox is permitted to hold, view, or parse cleartext cookies or passwords [Anaphase v9.1 §2.2, 7].
- Requests outbound MUST carry the **`X-Identity-Label`** header.
- **Label Namespace Format**: The Label format MUST follow: `<namespace>/<identifier>@<instance-id>`
  - `<namespace>`: The credential provider scope (e.g., `weibo`, `github`, `openai`).
  - `<identifier>`: The specific credential identifier within the namespace.
  - `<instance-id>`: (Optional) The unique Anaphase instance ID to prevent session collisions.
  - *Example*: `X-Identity-Label: weibo/session_1@anaphase_01`
- `Tuck` intercepts the connection at the edge, parses the label, matches it against its internal hardware-locked Key Management Service (KMS), and injects the actual `Cookie: SUB=xxxx;` or `Authorization: Bearer xxxx` into the HTTP header before egressing to the public network.

### 4.2 KMS Chroot Jail Isolation and Standards
- The KMS storage file and the decryption private keys MUST reside in a read-only, hardware-protected, or strictly chrooted directory owned exclusively by the `Tuck` daemon.
- The KMS **SHOULD** support standard interfaces such as **PKCS#11** or integrate with hardware security modules (HSM) / Trusted Platform Module (TPM) for key storage and decryption.
- Even if the `Anaphase` process or the `Tentacle` WASM sandbox is compromised, the attacker can only steal meaningless `Identity Labels`, leaving the real keys secure inside `Tuck`'s physical boundary.

---

## 5. Security Error Codes

When a cryptographic or transport-level security breach is detected, the error **MUST** be propagated to the upper layer via the BIND-19 Error Frame (FrameType `0x06`) [3.2]. The Error Frame payload carries the INTENT-7-SECURE error code in its first 2 bytes:

| Error Code | Hexadecimal | Name | Description |
|:---|:---|:---|:---|
| `2000` | `0x07D0` | `MTLS_HANDSHAKE_FAILED` | Remote TLS 1.3 or client certificate verification failed. |
| `2010` | `0x07DA` | `UNTRUSTED_PEER_UID` | Local UDS peer credential (UID/GID) mismatches the process owner. |
| `2020` | `0x07E4` | `KMS_KEY_INJECTION_FAILED` | Tuck failed to decrypt or inject the credential matching the Label. |
| `2030` | `0x07EE` | `CERTIFICATE_EXPIRED` | X.509 certificate has expired. |
| `2040` | `0x07F8` | `CERTIFICATE_REVOKED` | X.509 certificate has been revoked (via CRL/OCSP). |

