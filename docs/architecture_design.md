# Architecture Design

## Overview
Khora is a secure hardware-anchored credential bridge system. It decouples long-lived API keys and secrets from untrusted host filesystems, delegating storage to a tamper-resistant hardware device (`KhoraVault`) and requiring out-of-band authorization via a mobile device (`Khora app`). What is stored, and the policy under which it may leave the device, is defined in `credential_model.md`.

```
+-------------------+          +-------------------+          +-------------------+
|     Khora App     |   BLE    |    KhoraVault     |   USB    |      khorad       |
| (Mobile Auth/MFA) | <------> | (Hardware Enclave)| <------> |   (Host Service)  |
+-------------------+          +-------------------+          +-------------------+
                                                                        ^
                                                                        | Unix socket
                                                                        v
                                                              +-------------------+
                                                              |      Clients      |
                                                              | (agents, khora)   |
                                                              +-------------------+
```

## System Components

### 1. KhoraVault (Hardware Appliance)
- **Role**: Secure physical storage and cryptographic engine.
- **Physical interfaces**: USB-C (host communication), Bluetooth Low Energy (mobile interaction).
- **Core capabilities**:
  - Secure key storage in hardware enclave / secure element.
  - Device-locked state machine: denies credential retrieval unless an active BLE authorization token is validated.
  - Release-policy evaluation for every request (`credential_model.md`).
  - Monotonic retry counter, rate limiting, and brute-force protection.

### 2. khorad (Host Service)
- **Role**: Lightweight service on the host: the only process that talks to KhoraVault, and the endpoint every client connects to.
- **Implementation**: Rust (see Decisions). The `host/` crate builds two binaries: `khorad`, the service, and `khora`, a command-line client for shells and testing.
- **Device seam**: KhoraVault is reached through a `Device` interface that owns approval and release-policy evaluation as well as storage, so the trust boundary sits on the device side. Phase 1 implements it with an in-process simulator whose approval is a terminal prompt; the USB device replaces it later without changing `khorad`.
- **Interfaces**:
  - Downstream: Communicates with KhoraVault over USB (Virtual Serial or HID).
  - Upstream: The client protocol (`client_protocol.md`) on a Unix domain socket.
- **Socket location**: `$XDG_RUNTIME_DIR/khora/khorad.sock`, falling back to `~/.khora/khorad.sock`, inside an owner-only (0700) directory; overridable with `KHORA_SOCKET`. Never a shared directory such as `/tmp`, where another user could create the socket first and impersonate `khorad`.
- **Client identification**: Identifies each connecting process from the kernel (`SO_PEERCRED` on Linux, `LOCAL_PEERPID` on macOS), with its parent chain, and passes it to the device for policy evaluation and the approval prompt.
- **Security**: Never writes credentials to disk or caches master keys; credential values in memory are wiped (`zeroize`) once delivered.

### 3. Khora App (Mobile Authenticator)
- **Role**: Out-of-band user approval interface.
- **Connection**: Direct BLE link to KhoraVault.
- **Workflow**: Displays incoming requests (credential, delivery channel, requesting client, origin or host), requiring biometric confirmation (FaceID / Fingerprint) before signaling approval to KhoraVault.

## Data & Communication Seams
1. **Clients to khorad**: Newline-delimited JSON-RPC 2.0 over the local socket; contract in `client_protocol.md`.
2. **khorad to KhoraVault**: Encrypted serial/HID request-response protocol. How host and device first establish trust is an open question (see Security).
3. **KhoraVault to Khora App**: Authenticated, encrypted BLE connection; the scheme is a pending decision in `vdesign/plan.md`.

## Security

### Invariants
- **Master key stays on the device.** `khorad` receives only the individual credential released for an approved request, and `passkey` credentials never leave the device at all.
- **No credential value reaches a language model.** Models handle credential names only; values go to the narrowest component that can use them (`credential_model.md`).
- **Every release is governed by the credential's policy** and covered by an out-of-band approval on the Khora app, per request or for a time-boxed grant.
- **Brute-force resistance.** KhoraVault enforces a monotonic failure counter, rate limiting, and an optional cryptographic wipe after repeated failed attempts.
- **Every seam is authenticated and encrypted**: Khora app ↔ KhoraVault over BLE, KhoraVault ↔ host over USB.

### Threat model
Khora protects against:
- Secrets at rest on the host (`.env` files, configuration, local databases) being read or exfiltrated.
- Credential use the user did not approve.
- An agent misled by prompt injection: policy confines each credential to its listed clients, origins, and hosts, and with `fill`, `inject`, and `sign` delivery the value never reaches the agent at all.
- Theft of the device alone: it stays locked without the paired phone, and brute force is bounded by the failure counter.

Khora does not protect against:
- A compromised host. The request details the app displays (requesting client, origin or host) come from the host and can be forged, and a released credential is plaintext in host memory, readable by anything running with the client's privileges.
- Misuse by a process that received a value by `value` delivery: anything it runs or prints can see the value.
- What an agent does with access once granted: a logged-in browser session or an access token permits whatever the account or the token's scopes permit.
- A mistaken approval. Approval is only as reliable as what the user is shown.

### Open questions
- **Host ↔ device pairing**: how KhoraVault and `khorad` first establish trust, so the USB channel can be authenticated.

## Decisions

### 2026-10-23: `khorad` is written in Rust
Chosen over Go. The service holds plaintext credentials, so it must wipe them from memory reliably (`zeroize`); Go's garbage collector can leave copies behind. Rust also lets the host and KhoraVault firmware share one protocol crate, so the two ends cannot drift. Accepted cost: slower iteration.

### 2026-10-23: Clients talk to `khorad` in newline-delimited JSON-RPC 2.0
Chosen over protobuf. Any runtime can integrate with a socket and a JSON parser, shell scripts included, which is what the universal agent interface requires; protobuf would need code generation in every client. Every request waits on a human approval, so encoding speed is irrelevant. The method schema lives in `docs/`. This covers the client ↔ host seam only; the host ↔ device encoding is decided with the real-device work.

### 2026-10-23: Credentials are typed and released under a per-credential policy
A single "return the value" release would hand every secret to whichever process asks, including an agent steered by its model. Typing credentials lets each be delivered by the narrowest channel that works (`fill`, `inject`, `sign` keep the value from the agent entirely), and a policy stored on KhoraVault, changed only with phone approval, keeps the host and agents from widening a credential's reach. The device evaluates the policy so the simulator and the real hardware share one enforcement point.

### 2026-10-23: Phase 1 is a thin end-to-end slice on a simulated device
The `khora` CLI, `khorad`, and an in-process simulated KhoraVault with terminal approval, on Linux only, releasing `secret` credentials by `value` delivery. It exercises the client protocol, client identification, release policy, and the device seam before any hardware exists, so the contract is proven by a working path rather than written up front. Simulating the device in-process (rather than over a pty) defers the host ↔ device encoding to the real-device work, where it belongs.
