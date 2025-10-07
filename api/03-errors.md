# Errors

Understanding and handling API errors.

## Error Response Format

All errors return JSON with an `error` field:

```json
{
  "error": "Error message describing what went wrong"
}
```

## HTTP Status Codes

| Code | Meaning | Description |
|------|---------|-------------|
| `200` | OK | Request successful |
| `400` | Bad Request | Invalid request parameters |
| `401` | Unauthorized | Invalid or missing API key |
| `403` | Forbidden | Authenticated but not authorized |
| `404` | Not Found | Resource doesn't exist |
| `429` | Too Many Requests | Rate limit exceeded |
| `500` | Internal Server Error | Server error |

## Common Errors

### Invalid API Key

**Status:** `401 Unauthorized`

```json
{
  "error": "Unauthorized"
}
```

**Solution:** Check your API key is correct and included in the `Authorization` header.

### Resource Not Found

**Status:** `404 Not Found`

```json
{
  "error": "Resource not found"
}
```

**Solution:** Verify the resource ID exists and you have access to it.

### Rate Limit Exceeded

**Status:** `429 Too Many Requests`

```json
{
  "error": "Rate limit exceeded. Please try again later."
}
```

**Solution:** Wait 60 seconds before retrying. See [Rate Limiting](./02-rate-limiting.md).

### Invalid Parameters

**Status:** `400 Bad Request`

```json
{
  "error": "Invalid parameters provided"
}
```

**Solution:** Check your request parameters match the API documentation.

## Error Handling Example

### JavaScript

```javascript
async function fetchMembers() {
  try {
    const response = await fetch('https://api.teachfloor.com/v0/members', {
      headers: {
        'Authorization': 'Bearer YOUR_API_KEY',
        'Content-Type': 'application/json'
      }
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || 'API request failed');
    }

    return await response.json();
  } catch (error) {
    console.error('API Error:', error.message);
    throw error;
  }
}
```

### PHP

```php
function fetchMembers($apiKey) {
    $curl = curl_init();

    curl_setopt_array($curl, [
        CURLOPT_URL => "https://api.teachfloor.com/v0/members",
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_HTTPHEADER => [
            "Authorization: Bearer " . $apiKey,
            "Content-Type: application/json"
        ],
    ]);

    $response = curl_exec($curl);
    $httpCode = curl_getinfo($curl, CURLINFO_HTTP_CODE);
    curl_close($curl);

    if ($httpCode !== 200) {
        $error = json_decode($response, true);
        throw new Exception($error['error'] ?? 'API request failed');
    }

    return json_decode($response, true);
}
```

## Retry Logic

For transient errors (500, 503), implement retry logic:

```javascript
async function fetchWithRetry(url, options, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url, options);

      if (response.status >= 500) {
        throw new Error('Server error');
      }

      return response;
    } catch (error) {
      if (i === maxRetries - 1) throw error;

      // Exponential backoff
      await new Promise(resolve =>
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
}
```

## Next Steps

- [Authentication](./01-authentication.md) - API key setup
- [Rate Limiting](./02-rate-limiting.md) - Usage limits
