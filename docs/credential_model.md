# Credential Model

What KhoraVault stores, and the rules under which any of it leaves the device.

## Principle
A language model handles credential names only, never values. Each value goes to the narrowest component that can use it: ideally nowhere (the device uses it itself), next to one of Khora's own delivery components, and to an agent's process only where the credential's policy allows it.

## Credential types

| Type | Stored on KhoraVault | Delivery | What leaves the device |
|---|---|---|---|
| `secret` | Opaque value: API key, personal access token | `value`, `inject` | The value |
| `login` | Username, password, optional TOTP seed, origins | `fill` | Username, password, and the current TOTP code; never the seed |
| `oauth` | Provider (issuer, client ID), scopes, refresh token, hosts | `inject` | A short-lived access token |
| `passkey` | FIDO2 private key, relying-party ID | `sign` | Nothing; only signatures |

## Delivery channels
- **`value`**: returned to the requesting client over the client protocol (`client_protocol.md`). The plaintext then lives in that process, and anything the process runs or prints can see it. Use it only for programs the agent cannot steer, never for a process that executes model-chosen commands.
- **`fill`**: sent only to the Khora browser extension, which fills and submits the login form when the browser reports the page's origin as one of the credential's `origins`. The model only navigates to the page; it cannot direct the credential to another site.
- **`inject`**: held by the Khora proxy, which attaches it to requests bound for the credential's `hosts`. The agent sends requests through the proxy and never sees the token.
- **`sign`**: the key never leaves KhoraVault; the device answers challenges (WebAuthn) itself.

## Release policy
Each credential carries a policy, stored with it on KhoraVault. Creating or changing a policy needs phone approval, so neither the host nor an agent can widen a credential's reach. The device evaluates the policy on every request, against facts `khorad` supplies: the requesting client, the delivery channel, and the origin or host.

| Field | Meaning |
|---|---|
| `clients` | Executables allowed to request the credential, matched against the kernel-reported identity. Required for `value`; for the other channels the client is Khora's own extension or proxy. |
| `origins` | `login` only: origins where `fill` may use it. |
| `hosts` | `secret` and `oauth` with `inject`: hosts the proxy may send it to. |
| `approval` | `each` (default): every release is approved. `grant <duration>`: one approval covers releases of this credential to the same client for that period. |

- A request the policy does not admit is answered `NotFound` without prompting, so a client can neither enumerate credentials it is not allowed nor flood the user with approvals.
- The approval prompt shows the credential, delivery channel, requesting client (executable, PID, parent chain), origin or host, and any grant duration.
- If an MCP interface is ever added, it exposes names and actions, never values.

## OAuth lifecycle
- **Enrollment**: `khora oauth add <name> --provider <p> --scope <s>` runs the authorization-code flow with PKCE in the user's own browser (never the agent's), redirecting to a loopback address served by `khorad`. `khorad` exchanges the code, sends the refresh token to KhoraVault under phone approval, and wipes its copy; nothing is written to host disk.
- **Refresh**: access tokens expire within about an hour and refresh tokens rotate on use, so every refresh writes a new refresh token back to the device. A `grant` approval lets refreshes within its period proceed without prompting.

## Open questions
- **Relays**: the `khora` CLI prints a value to whoever runs it, so listing it in `clients` admits any process that can run it. Policy may need to match the CLI's caller, from the parent chain, instead of the CLI itself.
- **OAuth refresh exchange**: KhoraVault has no network. Either `khorad` performs the exchange (simple, but the refresh token passes through host memory on every refresh, and a stolen one keeps working off-host until it expires or is revoked), or KhoraVault performs it over a TLS session that `khorad` only relays (the refresh token never leaves the device; needs a TLS stack, CA roots, and trusted time on the MCU).
- **OAuth client registration**: Khora ships a client ID per provider, or each user registers their own. Provider rules for shared clients differ; Google, for example, restricts Gmail scopes for unverified apps.
- **TOTP time**: without its own clock, the device computes codes from host-supplied time, so a host could obtain future codes.
