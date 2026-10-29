# Getting Started

What every MCP client needs to connect to your Teachfloor organization. The per-client pages — [Claude](./clients/claude), [ChatGPT](./clients/chatgpt), [custom clients](./clients/custom) — cover the steps specific to each.

**Endpoint:** `https://mcp.teachfloor.com`

**Requires:** a plan that includes MCP. Requests from an organization without it are answered with `403 Forbidden`.

## Authentication

The server accepts two kinds of credential in the `Authorization: Bearer` header.

### OAuth 2.0

The path interactive clients use. The server publishes its metadata at `/.well-known/oauth-authorization-server` and `/.well-known/oauth-protected-resource`, and supports dynamic client registration, so a client can connect without anyone registering an application by hand.

Access is granted per organization — the consent screen asks which one the token should act on — and carries the permissions of the person who approved it.

### API Key

A long-lived key from **Settings → Integrations**, the same credential the [REST API](https://docs.teachfloor.com/api/authentication) uses. Convenient for scripts and backend automation. It carries full access rather than scoped access, so prefer OAuth where the client supports it.

See [Custom Clients](./clients/custom) for how to use one.

### Failures

A request with a missing, expired or revoked credential is answered with `401 Unauthorized` and a `WWW-Authenticate` header pointing at the server's metadata. That header is what tells an MCP client to start the OAuth flow, so a client that has never connected before will see it once on its first request.

## Scopes

OAuth tokens carry one or both of:

| Scope | Grants |
|---|---|
| `mcp:read` | The 29 tools that only read data |
| `mcp:write` | The 22 tools that create, modify or delete data |

A tool called without its scope is refused. API keys are not scoped and can call everything.

## Confirmation Prompts

Every tool declares whether it reads or writes. Clients that honour these annotations run read-only tools without asking each time, and ask before anything that creates, modifies or deletes. [Tools Reference](./tools-reference) marks which is which.

## Protocol Notes

- The server negotiates its protocol revision on `initialize`, and speaks `2025-03-26` and `2024-11-05`. A client asking for anything else is answered with `2025-03-26`.
- Transport is JSON-RPC over HTTP `POST`. The optional server-to-client SSE stream is not implemented: a `GET` on the endpoint answers `405` with `Allow: POST`.
- Batched requests are accepted: send a JSON array and you get one response per entry, with notifications omitted. A batch of only notifications answers `204`.
- A failed tool call comes back as a tool result with `isError: true` and a message the assistant can act on, rather than as a JSON-RPC error.

## Limits

The MCP endpoint shares the [rate limits](https://docs.teachfloor.com/api/rate-limiting) of the REST API it calls.
