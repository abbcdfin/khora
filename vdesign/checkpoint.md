# Checkpoint

## Current State (2026-10-23)
- Renamed the project from `clawvault` to Khora (system) and KhoraVault (hardware device) in `README.org` and `docs/`; naming rationale in `docs/product_specification.md`.
- Renamed the GitHub repository to `abbcdfin/khora`; `origin` points at it and `main` is pushed.
- Initialized Vibe project structure (`vdesign/`, `docs/`) and drafted `docs/architecture_design.md`, including the threat model.
- Renamed the host service `khora-agent` to `khorad`, with `khora` as its command-line client.
- Decided: `khorad` in Rust, client protocol in newline-delimited JSON-RPC 2.0 (rationale in `docs/architecture_design.md`).
- Designed credential types, delivery channels (`value`, `fill`, `inject`, `sign`), and per-credential release policy in `docs/credential_model.md`; client protocol in `docs/client_protocol.md`.
- Designed agent integration in `docs/agent_integration.md`: the runtime holds values, the model sees capabilities only (`khora mcp`), model-facing error results, and the sandbox requirement. Narrowed `inject` to scheme-defined locations via a reverse proxy; added the `ssh_key` type.
- Decided KhoraVault platform: Zephyr with C firmware; development on the ESP32-P4-Function-EV-Board (pre-v3), secure boot and flash encryption work on an ESP32-S3; a secure element (part pending) holds the master key; host ↔ device messages are CBOR per one CDDL schema; device time comes from the phone.
- Phase 1 (thin end-to-end slice) accepted and in progress: `secret` credentials by `value` delivery only, plus an example runtime client.

## Next Steps
- When the P4 board arrives: read its chip revision and run the board spike candidate; order an ESP32-S3-DevKitC.
- Build the Phase 1 board in `active_plan.md`, starting with the Cargo workspace and the `Device` interface.
