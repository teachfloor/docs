# Delivery & Retries

Understanding webhook reliability and retry behavior.

## How Delivery Works

1. Event occurs in Teachfloor
2. HTTP POST sent to your endpoint (10 second timeout)
3. Response checked
4. 2xx status = success, otherwise retry

## Successful Delivery

Your endpoint must respond with 2xx status within 10 seconds:
- `200 OK` (recommended)
- `201 Created`
- `202 Accepted`
- `204 No Content`

## Retry Mechanism

- **Total attempts**: 3 (initial + 2 retries)
- **Timeout**: 10 seconds per attempt
- **Backoff**: Exponential (10^attempt seconds)

### Retry Schedule

| Attempt | Wait Time | Total Elapsed |
|---------|-----------|---------------|
| Initial | 0s | 0s |
| 1st retry | 10s | 10s |
| 2nd retry | 100s | 110s |

### When Retries Occur

- Non-2xx status code
- Timeout (>10 seconds)
- Connection failure

## Handling Duplicates

Every delivery carries a `Teachfloor-Idempotency-Key` header whose value is stable across retry attempts (it equals the envelope `id` field). Use it as your dedupe key — checking the header lets you short-circuit before parsing the body:

```javascript
const processed = new Set();

function processWebhook(req) {
  const key = req.headers['teachfloor-idempotency-key']; // or event.id from the body
  if (processed.has(key)) return;
  processed.add(key);
  // Process event
}
```

Retries carry the SAME key, so any 2xx you send after a retry-triggering timeout will still be treated as duplicate work on your side.

## Monitoring

View delivery logs in your Teachfloor dashboard:
1. Go to **Developers** → **Webhooks**
2. Select your endpoint
3. Check delivery history

## Next Steps

- [Event Reference](./event-reference)
- [Security](./security)
- [Troubleshooting](./troubleshooting)
