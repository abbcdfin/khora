# Architecture Design

## Overview
Khora is a secure hardware-anchored credential bridge system. It decouples long-lived API keys and secrets from untrusted host filesystems, delegating storage to a tamper-resistant hardware device (`KhoraVault`) and requiring out-of-band authorization via a mobile device (`Khora app`).

```
+-------------------+          +-------------------+          +-------------------+
|     Khora App     |   BLE    |    KhoraVault     |   USB    |    khora-agent    |
| (Mobile Auth/MFA) | <------> | (Hardware Enclave)| <------> |   (Host Daemon)   |
+-------------------+          +-------------------+          +-------------------+
                                                                        ^
                                                                        | IPC (Socket)
                                                                        v
                                                              +-------------------+
                                                              | Client Agent      |
                                                              | (openclaw, CLI)   |
                                                              +-------------------+
```

## System Components

### 1. KhoraVault (Hardware Appliance)
- **Role**: Secure physical storage and cryptographic engine.
- **Physical interfaces**: USB-C (host communication), Bluetooth Low Energy (mobile interaction).
- **Core capabilities**:
  - Secure key storage in hardware enclave / secure element.
  - Device-locked state machine: denies credential retrieval unless an active BLE authorization token is validated.
  - Monotonic retry counter, rate limiting, and brute-force protection.

### 2. khora-agent (Host Service)
- **Role**: Lightweight driver/service running on host machines.
- **Interfaces**:
  - Downstream: Communicates with KhoraVault over USB (Virtual Serial or HID).
  - Upstream: Exposes a standardized local IPC seam (Unix domain socket / named pipe) to agent runtimes.
- **Socket location**: `$XDG_RUNTIME_DIR/khora/agent.sock`, falling back to `~/.khora/agent.sock`, inside an owner-only (0700) directory; overridable with `KHORA_AGENT_SOCKET`. Never a shared directory such as `/tmp`, where another user could create the socket first and impersonate the agent.
- **Client identification**: Identifies each connecting process from the kernel (`SO_PEERCRED` on Linux, `LOCAL_PEERPID` on macOS) and attaches it to the approval request.
- **Security**: Never caches master encryption keys or unreleased credentials.

### 3. Khora App (Mobile Authenticator)
- **Role**: Out-of-band user approval interface.
- **Connection**: Direct BLE link to KhoraVault.
- **Workflow**: Displays incoming credential requests (originating process, credential identifier), requiring biometric confirmation (FaceID / Fingerprint) before signaling approval to KhoraVault.

## Data & Communication Seams
1. **Agent to Host Daemon**: Local IPC via structured JSON-RPC or protocol buffers over Unix domain socket.
2. **Host Daemon to KhoraVault**: Encrypted serial/HID request-response protocol. How host and device first establish trust is an open question (see Security).
3. **KhoraVault to Mobile App**: Authenticated, encrypted BLE connection; the scheme is a pending decision in `vdesign/plan.md`.

## Security

### Invariants
- **Master key stays on the device.** `khora-agent` receives only the individual credential released for an approved request.
- **Every release is approved out of band** on the paired Khora app over BLE.
- **Brute-force resistance.** KhoraVault enforces a monotonic failure counter, rate limiting, and an optional cryptographic wipe after repeated failed attempts.
- **Every seam is authenticated and encrypted**: Khora app ↔ KhoraVault over BLE, KhoraVault ↔ host over USB.

### Threat model
Khora protects against:
- Secrets at rest on the host (`.env` files, configuration, local databases) being read or exfiltrated.
- Credential use the user did not approve.
- Theft of the device alone: it stays locked without the paired phone, and brute force is bounded by the failure counter.

Khora does not protect against:
- A compromised host. The request details the app displays (originating process, credential identifier) come from the host and can be forged, and a released credential is plaintext in host memory, readable by anything running with the client's privileges.
- A mistaken approval. Approval is only as reliable as what the user is shown.

### Open questions
- **Host ↔ device pairing**: how KhoraVault and `khora-agent` first establish trust, so the USB channel can be authenticated.
- **Release scope**: one approval per request, or a time-boxed grant per credential and process.
