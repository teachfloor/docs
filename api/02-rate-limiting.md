# Rate Limiting

The API enforces rate limits to ensure platform stability and fair usage.

## Rate Limit

**50 requests per minute** per organization by default. The limit is counted across every API key of the organization, not per key. Plans can carry a higher limit; the headers below always tell you the one that applies to you.

## Rate Limit Headers

Each API response includes headers showing your current rate limit status:

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Maximum requests per minute for your organization |
| `X-RateLimit-Remaining` | Requests remaining in the current minute |
| `Retry-After` | Only on a `429`: seconds to wait before retrying |

## Example Response Headers

```
X-RateLimit-Limit: 50
X-RateLimit-Remaining: 42
```

## Exceeding the Limit

If you exceed the rate limit, you'll receive:

**Status Code:** `429 Too Many Requests`

**Response:**
```json
{
  "error": {
    "code": 429,
    "message": "Too many attempts, retry in a minute"
  },
  "payload": null
}
```

Requests that hit the limit still count towards it. Stop sending until the window resets rather than retrying in a tight loop.

## Best Practices

### Monitor Remaining Requests

```javascript
const response = await fetch('https://api.teachfloor.com/v0/members', {
  headers: { 'Authorization': 'Bearer YOUR_API_KEY' }
});

const remaining = Number(response.headers.get('X-RateLimit-Remaining'));

if (remaining < 5) {
  // Slow down requests
}
```

### Implement Backoff

```javascript
async function makeRequest(url) {
  const response = await fetch(url, {
    headers: { 'Authorization': 'Bearer YOUR_API_KEY' }
  });

  if (response.status === 429) {
    const retryAfter = Number(response.headers.get('Retry-After') || 60);
    await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
    return makeRequest(url);
  }

  return response;
}
```

### Reduce Calls

- Cache responses when possible
- Use the `search` endpoints instead of listing and filtering client-side
- Use [webhooks](/docs/webhooks) for real-time updates instead of polling

## Need Higher Limits?

Contact support through your Teachfloor dashboard to discuss your use case.

## Next Steps

- [Errors](./errors) - Handle API errors
- [Authentication](./authentication) - API key setup
