# Errors

How the API reports failures, and what to do about each one.

## Response Envelope

Every response, success or failure, has the same two top-level keys:

```json
{
  "error": null,
  "payload": { ... }
}
```

On failure `error` is an object and `payload` is normally `null`:

```json
{
  "error": {
    "code": 404,
    "message": "Not Found"
  },
  "payload": null
}
```

`error.code` always matches the HTTP status code. Check `error` first: a `null` error means the request succeeded.

## HTTP Status Codes

| Code | Meaning | When |
|------|---------|------|
| `200` | OK | The request succeeded. |
| `400` | Bad Request | The request could not be processed as sent (for example a missing file on an upload, or a storage limit reached). |
| `402` | Payment Required | Your plan does not include the Public API, or the feature the endpoint needs (for example custom fields). |
| `403` | Forbidden | Missing or invalid API key, insufficient scope, or an action that needs explicit consent. |
| `404` | Not Found | The resource does not exist in your organization, or the endpoint does not exist. |
| `405` | Method Not Allowed | The endpoint exists but not for this HTTP method. |
| `422` | Unprocessable Entity | Validation failed; `payload` lists the errors per field. |
| `429` | Too Many Requests | Rate limit exceeded. See [Rate Limiting](./rate-limiting). |
| `500` | Internal Server Error | Something failed on our side. Retry with backoff; contact support if it persists. |

## Common Errors

### Missing or invalid API key

**Status:** `403 Forbidden`

```json
{
  "error": {
    "code": 403,
    "message": "Unauthorized"
  },
  "payload": null
}
```

The API answers `403`, not `401`, for a missing or unknown key. Check that the key is sent as `Authorization: Bearer YOUR_API_KEY` and has not been regenerated.

### Plan does not include the API

**Status:** `402 Payment Required`

```json
{
  "error": {
    "code": 402,
    "message": "Unauthorized - Please upgrade to a valid subscription plan to access this resource"
  },
  "payload": null
}
```

The key is valid but your subscription plan does not include the Public API, or does not include the feature the endpoint depends on (custom fields endpoints and the `custom_fields` parameter). Upgrade the plan or drop the parameter.

### Insufficient scope

**Status:** `403 Forbidden`

```json
{
  "error": {
    "code": 403,
    "message": "Insufficient scope. Required: members:read"
  },
  "payload": null
}
```

Only extension apps authenticating with an OAuth token see this. Organization API keys carry every scope.

### Explicit consent required

**Status:** `403 Forbidden`

```json
{
  "error": {
    "code": 403,
    "message": "Explicit consent required to delete courses via API. Please visit Teachfloor Developers page to provide your consent."
  },
  "payload": null
}
```

Deleting courses, modules and elements, and creating members, are gated behind consents an organization owner grants once in **Developers** on the dashboard.

### Resource not found

**Status:** `404 Not Found`

```json
{
  "error": {
    "code": 404,
    "message": "Not Found"
  },
  "payload": null
}
```

Returned both for an unknown endpoint and for a resource id that does not exist in your organization. Ids belonging to another organization are reported as not found, never as forbidden.

### Validation failed

**Status:** `422 Unprocessable Entity`

```json
{
  "error": {
    "code": 422,
    "message": "Unprocessable Entity"
  },
  "payload": {
    "availability": [
      "The selected availability is invalid."
    ]
  }
}
```

This is the one failure where `payload` is set: an object keyed by parameter name, each holding the list of messages for that parameter. Nested parameters use dot notation (`custom_fields.field_7`).

### Rate limit exceeded

**Status:** `429 Too Many Requests`

```json
{
  "error": {
    "code": 429,
    "message": "Too many attempts, retry in a minute"
  },
  "payload": null
}
```

Wait for the window to reset, then retry. See [Rate Limiting](./rate-limiting).

## Handling Errors

```javascript
const response = await fetch('https://api.teachfloor.com/v0/members', {
  headers: { 'Authorization': 'Bearer YOUR_API_KEY' }
});

const body = await response.json();

if (body.error) {
  switch (body.error.code) {
    case 422:
      console.error('Validation failed', body.payload);
      break;
    case 429:
      // back off and retry
      break;
    default:
      console.error(`${body.error.code}: ${body.error.message}`);
  }
} else {
  console.log(body.payload);
}
```

```php
$body = json_decode($response, true);

if ($body['error'] !== null) {
    throw new RuntimeException($body['error']['message'], $body['error']['code']);
}

$data = $body['payload'];
```

## Next Steps

- [Authentication](./authentication) - API key setup
- [Rate Limiting](./rate-limiting) - Usage limits
