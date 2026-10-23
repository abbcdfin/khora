# Environment and Toolchains

## Toolchain Defaults
- **Python**: Default to `uv` (`uv run ...`, `uv pip ...`) for any Python components (e.g. CLI tools, simulation scripts).
- **Host service**: Rust stable via `rustup`; build and test with `cargo` from the repository root.
- Firmware and mobile toolchains follow the pending decisions in `plan.md`.

## Environment Variables
- `KHORA_SOCKET`: Overrides the `khorad` socket path (default and permissions in `docs/architecture_design.md`).
- `KHORA_DEVICE_PORT`: Optional override for USB virtual serial / device path.
- `KHORA_SIM_CREDENTIALS`: Path to the simulated KhoraVault's credential file: a JSON object mapping each name to a credential with the fields of `docs/credential_model.md`. Development only; holds test secrets in plaintext.
- `KHORA_LOG_LEVEL`: Log verbosity for `khorad` (`debug`, `info`, `warn`, `error`).
