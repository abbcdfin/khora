# Active Plan: Phase 1 — Thin end-to-end slice

**Done when:** with `khorad` running, `khora get <name>` prints a `secret` credential after the user approves at `khorad`'s terminal, gets `Denied` if they reject it, and gets `NotFound` for an unknown name or a client the policy does not list. An example agent runtime, connected to the socket directly, uses a credential inside a tool whose result never contains it.

## Board
- [x] Client protocol contract for `credential.get` (`docs/client_protocol.md`)
- [x] Credential types and release policy (`docs/credential_model.md`)
- [ ] Cargo workspace at the repository root with the `host/` crate (binaries `khorad` and `khora`)
- [ ] `Device` interface; simulated KhoraVault loading `KHORA_SIM_CREDENTIALS`, evaluating release policy, approving by terminal prompt
- [ ] `khorad`: socket path resolution, owner-only directory, `SO_PEERCRED` client identity with parent chain, JSON-RPC loop, `credential.get`
- [ ] `khora get`: one request, value to stdout, error to stderr with non-zero exit
- [ ] Credentials held in `zeroize` types from device to socket write
- [ ] Example runtime client (`examples/runtime_client.py`, run with `uv`): connects to the socket directly and uses the value inside one tool, returning only the outcome
- [ ] End-to-end tests: approve, deny, unknown name, unlisted client, malformed line, unknown method, example runtime client

## Out of scope (fail loudly)
- Credential types other than `secret`, delivery other than `value`, and `grant` approval: the simulator rejects them when loading.
- Concurrent requests: one connection is served at a time.
- macOS and Windows: Linux only until target platforms are decided.
- Approval timeout: the prompt waits until answered.
