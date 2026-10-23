# Environment and Toolchains

## Toolchain Defaults
- **Python**: Default to `uv` (`uv run ...`, `uv pip ...`) for any Python components (e.g. CLI tools, simulation scripts).
- Host agent, firmware, and mobile toolchains follow the pending decisions in `plan.md`.

## Environment Variables
- `KHORA_AGENT_SOCKET`: Overrides the `khora-agent` socket path (default and permissions in `docs/architecture_design.md`).
- `KHORA_DEVICE_PORT`: Optional override for USB virtual serial / device path.
- `KHORA_LOG_LEVEL`: Log verbosity for `khora-agent` (`debug`, `info`, `warn`, `error`).
