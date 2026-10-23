# Khora Project Plan

## Roadmap

- [/] **Phase 1: Thin end-to-end slice.** `khora get <name>` returns a `secret` credential from a simulated KhoraVault, under its release policy, after a terminal approval standing in for the Khora app. Board in `active_plan.md`.

## Candidates

<!-- Holding pen for unnumbered DRAFT ideas awaiting Decision Check Points -->

- **Real device.** Replace the simulated KhoraVault with a dev board over USB (virtual serial or HID), with secure storage behind an enclave abstraction.
- **Mobile approval.** Khora app over BLE: pairing, request notification, biometric approval. Replaces the terminal prompt.
- **openclaw integration.** Native `openclaw` client for the client protocol.
- **Environment shim.** `khora exec` runs a command with `value`-delivered credentials in its environment, for programs the agent does not steer.
- **Browser fill (`fill`).** Khora browser extension and native-messaging host: fills and submits `login` credentials on bound origins, with TOTP codes computed on the device.
- **Token proxy (`inject`).** Local reverse proxy attaching `secret` and `oauth` tokens to requests for bound hosts, where their scheme defines, including OAuth enrollment (`khora oauth add`) and refresh.
- **Passkeys (`sign`).** KhoraVault as a FIDO2 security key over USB HID (CTAP2), approved on the phone. The browser reaches the device directly, not through `khorad`.
- **SSH agent (`sign`).** `khorad` serves the ssh-agent protocol for `ssh_key` credentials, so `ssh` and `git` authenticate with keys that never leave KhoraVault.
- **Model-facing tools (`khora mcp`).** MCP server with `khora_list` (discovery), then `khora_request` once the token proxy exists (`docs/agent_integration.md`).

### Pending decisions

Each needs a product-owner choice before the candidate that depends on it can be accepted; the verdict is recorded in `docs/`.

- Target host platforms (Linux first, or Linux + macOS + Windows).
- Channel encryption scheme (Noise / ECDH + AES-GCM / other).
- Device MCU and secure element (ESP32-S3 / nRF52840 / other).
- Mobile stack (Flutter / native Swift + Kotlin).
- OAuth refresh exchange and client registration (open questions in `docs/credential_model.md`) — block the token proxy.
