# Khora Project Plan

## Roadmap

No phase accepted yet. The thin slice below is proposed as the first Decision Check Point.

## Candidates

<!-- Holding pen for unnumbered DRAFT ideas awaiting Decision Check Points -->

- **Thin end-to-end slice (proposed first).** A CLI client (stand-in for `openclaw`) requests one credential from `khora-agent`, which fetches it from a simulated KhoraVault; approval is a terminal prompt standing in for the Khora app. Defines the agent IPC and the agent ↔ device protocol only as far as this path needs.
- **Real device.** Replace the simulated KhoraVault with a dev board over USB (virtual serial or HID), with secure storage behind an enclave abstraction.
- **Mobile approval.** Khora app over BLE: pairing, request notification, biometric approval. Replaces the terminal prompt.
- **openclaw integration.** Native `openclaw` client for the agent IPC.
- **Generic environment shim.** Drop-in injector that exports a released credential into a child process's environment.

### Pending decisions

Each needs a product-owner choice before the candidate that depends on it can be accepted; the verdict is recorded in `docs/`.

- Host agent language (Rust / Go / other) — blocks the thin slice.
- Agent IPC encoding (JSON-RPC / protobuf) — blocks the thin slice.
- Target host platforms (Linux first, or Linux + macOS + Windows).
- Channel encryption scheme (Noise / ECDH + AES-GCM / other).
- Device MCU and secure element (ESP32-S3 / nRF52840 / other).
- Mobile stack (Flutter / native Swift + Kotlin).
