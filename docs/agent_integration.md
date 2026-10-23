# Agent Integration

How an AI agent uses Khora. An agent has two parts, and Khora presents itself differently to each. The **runtime** (the agent's own code: its process, tool implementations, and model-provider client) holds a credential value for the moment it uses it. The **model** reasons and proposes tool calls, and handles credential names only. Khora is part of the harness: it executes and enforces; the model decides.

## The runtime

The runtime connects to `khorad` over the client protocol (`client_protocol.md`), usually through a small per-runtime integration (an SDK or a plugin), and uses each value inside its own code:

1. The model calls a tool the runtime implements, e.g. `list_pull_requests(repo)`.
2. The tool requests a credential whose name is fixed in the tool's configuration, never chosen by the model.
3. `khorad` identifies the runtime from the kernel, the device evaluates the policy, and the user approves.
4. The tool uses the value in its own request and returns only the outcome to the model.

Rules:
- **Connect to the socket directly**, not through the `khora` CLI. Through the CLI, the requesting client is `khora`, not the runtime, and policy cannot tell them apart.
- **Never place a value in anything the model sees**: prompts, tool results, error messages, or logs fed back to it. Tool code redacts secrets from any error text it returns.
- **Sandbox the model's command execution away from the socket.** A shell tool running as the user can run `khora get` or connect to the socket itself, and a script run with the runtime's interpreter (e.g. `node`) is indistinguishable from the runtime by executable identity. The agent's command execution must run where `$XDG_RUNTIME_DIR/khora/` is unreachable: a container or a separate user.

## The model

Khora appears to the model as capabilities, never as a secret store. From least to most visible:

1. **Invisible (default).** Authenticated tools look like any other tool; the model meets Khora only through the error results below.
2. **Discovery.** `khora_list` returns the credentials this client may use, as name, type, and where usable (`hotmail · login · login.live.com`), never values. It lets the model go to a login page instead of asking the user for a password.
3. **Authenticated requests** (needs the token proxy). `khora_request(credential, method, url, body)` sends the request through the proxy, which attaches the credential under its `inject` rules (`credential_model.md`); the response comes back with any copy of the credential removed.

Both tools are served by `khora mcp`, an MCP server the runtime launches. It is a client of `khorad` like any other, and no value crosses it.

Never offered to the model:
- A tool that returns a credential value.
- A tool that runs a model-chosen command with credentials in its environment: the model would run `env`.

### Error results
The runtime, or `khora mcp`, turns protocol errors into results with one meaning and one next step for the model:

| Result | From | Meaning for the model |
|---|---|---|
| `denied` | `Denied` | The user declined. Stop, tell the user, do not retry. |
| `not_available` | `NotFound` | No such credential is usable here. Do not guess other names; ask the user to add one. |
| `approval_timeout` | Not yet defined | The user did not respond. Report it and wait for the user. |

### Standing instruction
Every model-facing surface (tool descriptions, or the runtime's system prompt) carries: *Credentials are held by Khora. Never ask the user to type a password or key into the conversation.*
