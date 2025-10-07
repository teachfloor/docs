# Rate Limiting

The API enforces rate limits to ensure platform stability and fair usage.

## Rate Limit

**50 requests per minute** per API key.

## Rate Limit Headers

Each API response includes headers showing your current rate limit status:

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Maximum requests per minute (50) |
| `X-RateLimit-Remaining` | Requests remaining in current minute |

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
  "error": "Rate limit exceeded. Please try again later."
}
```

## Best Practices

### Monitor Remaining Requests

```javascript
const response = await fetch('https://api.teachfloor.com/v0/members', {
  headers: { 'Authorization': 'Bearer YOUR_API_KEY' }
});

const remaining = response.headers.get('X-RateLimit-Remaining');
console.log(`Requests remaining: ${remaining}`);

if (remaining < 5) {
  // Slow down requests
}
```

### Implement Backoff

```javascript
async function makeRequest(url) {
  try {
    const response = await fetch(url, {
      headers: { 'Authorization': 'Bearer YOUR_API_KEY' }
    });

    if (response.status === 429) {
      // Wait 60 seconds and retry
      await new Promise(resolve => setTimeout(resolve, 60000));
      return makeRequest(url);
    }

    return response;
  } catch (error) {
    console.error('Request failed:', error);
  }
}
```

### Batch Requests

Optimize your API usage:
- Cache responses when possible
- Combine related data fetches
- Use webhooks for real-time updates instead of polling

## Need Higher Limits?

Contact support through your Teachfloor dashboard to discuss your use case.

## Next Steps

- [Errors](./03-errors.md) - Handle API errors
- [Authentication](./01-authentication.md) - API key setup
