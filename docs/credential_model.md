# Credential Model

What KhoraVault stores, and the rules under which any of it leaves the device.

## Principle
A language model handles credential names only, never values. Each value goes to the narrowest component that can use it: ideally nowhere (the device uses it itself), next to one of Khora's own delivery components, and to an agent's process only where the credential's policy allows it.

## Credential types

| Type | Stored on KhoraVault | Delivery | What leaves the device |
|---|---|---|---|
| `secret` | Opaque value: API key, personal access token | `value`, `inject` | The value |
| `login` | Username, password, optional TOTP seed, origins | `fill` | Username, password, and the current TOTP code (from the device's phone-supplied time); never the seed |
| `oauth` | Provider (issuer, client ID), scopes, refresh token, hosts | `inject` | A short-lived access token |
| `passkey` | FIDO2 private key, relying-party ID | `sign` | Nothing; only signatures |
| `ssh_key` | SSH private key | `sign` | Nothing; only signatures |

## Delivery channels
- **`value`**: returned to the requesting client over the client protocol (`client_protocol.md`). The plaintext then lives in that process, and anything the process runs or prints can see it. Use it only for programs the agent cannot steer, never for a process that executes model-chosen commands.
- **`fill`**: sent only to the Khora browser extension, which fills and submits the login form when the browser reports the page's origin as one of the credential's `origins`. The model only navigates to the page; it cannot direct the credential to another site.
- **`inject`**: held by the Khora proxy, which attaches it to requests bound for the credential's `hosts`, only where its authentication scheme puts it (the `Authorization` header, a query parameter). The agent sends requests through the proxy and never sees the token.
  - Clients reach the proxy as a reverse proxy: they point their API base URL at it, and it opens HTTPS upstream. Intercepting TLS with a local CA is reserved for clients that cannot change their base URL, since it breaks certificate pinning and makes the CA key a secret of its own.
  - The proxy never substitutes placeholders in request bodies: the model could place one in content sent to an allowed host (an issue body on `api.github.com`) and publish the secret.
  - Schemes that sign each request (AWS SigV4, HMAC) need a per-scheme signer in the proxy, added when a credential needs one.
- **`sign`**: the key never leaves KhoraVault; the device answers challenges itself (WebAuthn for `passkey`, the ssh-agent protocol for `ssh_key`).

## Choosing a channel

| How the secret is used | Channel |
|---|---|
| A token in an HTTP header or query parameter | `inject` |
| A signature computed over each request (AWS SigV4, HMAC) | `inject`, with a per-scheme signer |
| Proof of a private key (SSH; later TLS client certificates, commit signing) | `sign` |
| An interactive web login | `fill`, or `sign` with a passkey |
| A password login over a non-HTTP protocol (Postgres, IMAP/SMTP) | `value`, into runtime-controlled code; a protocol-aware proxy only if one protocol becomes important enough |

## Release policy
Each credential carries a policy, stored with it on KhoraVault. Creating or changing a policy needs phone approval, so neither the host nor an agent can widen a credential's reach. The device evaluates the policy on every request, against facts `khorad` supplies: the requesting client, the delivery channel, and the origin or host.

| Field | Meaning |
|---|---|
| `clients` | Executables allowed to request the credential, matched against the kernel-reported identity. Required for `value`, optional for `ssh_key`. For `fill` and `inject` the client is Khora's own extension or proxy; `passkey` requests reach the device from the browser over USB, without `khorad`. |
| `origins` | `login` only: origins where `fill` may use it. |
| `hosts` | `secret` and `oauth` with `inject`: hosts the proxy may send it to. |
| `approval` | `each` (default): every release is approved. `grant <duration>`: one approval covers releases of this credential to the same client for that period. |

- A request the policy does not admit is answered `NotFound` without prompting, so a client can neither enumerate credentials it is not allowed nor flood the user with approvals.
- The approval prompt shows the credential, delivery channel, requesting client (executable, PID, parent chain), origin or host, and any grant duration.
- Model-facing interfaces (`agent_integration.md`) expose names and actions, never values.

## OAuth lifecycle
- **Enrollment**: `khora oauth add <name> --provider <p> --scope <s>` runs the authorization-code flow with PKCE in the user's own browser (never the agent's), redirecting to a loopback address served by `khorad`. `khorad` exchanges the code, sends the refresh token to KhoraVault under phone approval, and wipes its copy; nothing is written to host disk.
- **Refresh**: access tokens expire within about an hour and refresh tokens rotate on use, so every refresh writes a new refresh token back to the device. A `grant` approval lets refreshes within its period proceed without prompting.

## Open questions
- **Relays**: the `khora` CLI prints a value to whoever runs it, so listing it in `clients` admits any process that can run it. Agent runtimes avoid this by connecting to the socket directly (`agent_integration.md`); for scripts using the CLI, policy may need to match the CLI's caller, from the parent chain, instead of the CLI itself.
- **Action scope for `inject`**: whether policy also binds HTTP methods and paths, and whether approval shows the request (`POST …/issues`) rather than the credential. Host binding alone does not stop misuse on a multi-tenant host: `api.github.com` also serves an attacker's repositories.
- **OAuth refresh exchange**: KhoraVault has no network. Either `khorad` performs the exchange (simple, but the refresh token passes through host memory on every refresh, and a stolen one keeps working off-host until it expires or is revoked), or KhoraVault performs it over a TLS session that `khorad` only relays (the refresh token never leaves the device; needs a TLS stack, CA roots, and trusted time on the MCU).
- **OAuth client registration**: Khora ships a client ID per provider, or each user registers their own. Provider rules for shared clients differ; Google, for example, restricts Gmail scopes for unverified apps.
