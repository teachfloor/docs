# Custom Clients

Connect your own MCP client, or call the server directly from a script.

## Any MCP Client

The server is a standard remote MCP server over HTTP, with OAuth 2.0 and dynamic client registration. Any client that supports remote servers connects the same way a packaged integration does: point it at the endpoint and let it run discovery.

**Endpoint:** `https://mcp.teachfloor.com`

No application has to be registered with Teachfloor first — the client registers itself.

## From a Script

For backend automation, use a Teachfloor API key instead of the OAuth flow. The same key the [REST API](https://docs.teachfloor.com/api/authentication) uses works here.

List the available tools:

```bash
curl https://mcp.teachfloor.com \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/list"
  }'
```

Call one:

```bash
curl https://mcp.teachfloor.com \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "tools/call",
    "params": {
      "name": "teachfloor_list_courses",
      "arguments": {}
    }
  }'
```

Tool names are prefixed `teachfloor_` on the wire — see [Tools Reference](../tools-reference).

## Batching

Send a JSON array to issue several requests at once. You get one response per entry:

```bash
curl https://mcp.teachfloor.com \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '[
    { "jsonrpc": "2.0", "id": 1, "method": "initialize", "params": { "protocolVersion": "2025-03-26" } },
    { "jsonrpc": "2.0", "id": 2, "method": "tools/list" }
  ]'
```

Notifications — entries with no `id` — are processed but produce no response. A batch of only notifications answers `204` with no body.

## Reading Results

A tool result comes back as content, with the payload as JSON text:

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "content": [
      { "type": "text", "text": "{ \"payload\": [ ... ] }" }
    ]
  }
}
```

A failed call is a result with `isError: true` and a message explaining what went wrong, rather than a JSON-RPC error. Validation failures say which argument was wrong; everything else is reported generically and logged server-side.
