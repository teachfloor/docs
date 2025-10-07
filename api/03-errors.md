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

## Next Steps

- [Authentication](./01-authentication.md) - API key setup
- [Rate Limiting](./02-rate-limiting.md) - Usage limits
