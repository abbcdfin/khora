# Khora Project Plan

## Roadmap

- [/] **Phase 1: Thin end-to-end slice.** `khora get <name>` returns a `secret` credential from a simulated KhoraVault, under its release policy, after a terminal approval standing in for the Khora app. Board in `active_plan.md`.

## Candidates

<!-- Holding pen for unnumbered DRAFT ideas awaiting Decision Check Points -->

- **Board spike.** Throwaway, in `vdesign/utils/`. Read the P4's chip revision (`esptool.py --chip esp32p4 chip_id`; Zephyr's board targets v1.3). On the P4 board and an ESP32-S3-DevKitC: boot Zephyr, enumerate USB as a data interface plus HID, advertise BLE (through the C6 on the P4), round-trip a `zcbor`-generated message.
- **Real device.** Replace the simulated KhoraVault with Zephyr firmware on the dev boards over USB, messages in CBOR per the CDDL schema, keys in the secure element.
- **Device hardening.** MCUboot secure boot and flash encryption on the ESP32-S3, and on the P4 once Zephyr supports it. Burns eFuses irreversibly: done deliberately, on a dedicated board.
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
- Production MCU (ESP32-S3 / ESP32-P4 v3 silicon), once Zephyr's P4 secure boot support lands or not.
- Secure element part (ATECC608B / NXP SE050 / Infineon OPTIGA Trust M) — blocks the real device.
- Mobile stack (Flutter / native Swift + Kotlin).
- OAuth refresh exchange and client registration (open questions in `docs/credential_model.md`) — block the token proxy.
