# Client Protocol

The interface between clients and `khorad`: agent runtimes, the `khora` CLI, and Khora's own delivery components. It covers what Phase 1 implements; methods are added when a phase needs them.

## Transport
- Unix domain socket at the location given in `architecture_design.md` (Socket location).
- Newline-delimited JSON-RPC 2.0: each request and each response is one UTF-8 JSON object on one line, terminated by `\n`.
- A connection carries one request at a time; `khorad` answers it before reading the next line. Batches and notifications are rejected with `-32600`.

## Client identity
Taken from the kernel, never from the request. On connect, `khorad` reads the peer's PID and UID (`SO_PEERCRED`), resolves its executable from `/proc/<pid>/exe`, and records its parent chain. The device evaluates release policy against this identity (`credential_model.md`), and the approval prompt shows it.

## Methods

### `credential.get`
Releases a credential by `value` delivery, if its release policy admits this client, after the user approves.

| Params | Type | Meaning |
|---|---|---|
| `name` | string | Credential identifier |

| Result | Type | Meaning |
|---|---|---|
| `value` | string | The credential, in plaintext |

```
→ {"jsonrpc":"2.0","id":1,"method":"credential.get","params":{"name":"openai_api_key"}}
← {"jsonrpc":"2.0","id":1,"result":{"value":"sk-..."}}
```

## Errors
Standard JSON-RPC codes: `-32700` parse error, `-32600` invalid request, `-32601` method not found, `-32602` invalid params.

| Code | Name | When |
|---|---|---|
| `-32001` | `Denied` | The user rejected the approval. |
| `-32002` | `NotFound` | No credential has that name, or its release policy does not admit this request. Returned without prompting. |
