# Delivery & Retries

Understanding webhook reliability and retry behavior.

## How Delivery Works

1. Event occurs in Teachfloor
2. HTTP POST sent to your endpoint (3 second timeout)
3. Response checked
4. 2xx status = success, otherwise retry

## Successful Delivery

Your endpoint must respond with 2xx status within 3 seconds:
- `200 OK` (recommended)
- `201 Created`
- `202 Accepted`
- `204 No Content`

## Retry Mechanism

- **Total attempts**: 3 (initial + 2 retries)
- **Timeout**: 3 seconds per attempt
- **Backoff**: Exponential (10^attempt seconds)

### Retry Schedule

| Attempt | Wait Time | Total Elapsed |
|---------|-----------|---------------|
| Initial | 0s | 0s |
| 1st retry | 10s | 10s |
| 2nd retry | 100s | 110s |

### When Retries Occur

- Non-2xx status code
- Timeout (>3 seconds)
- Connection failure

## Handling Duplicates

Use `event.id` to prevent duplicate processing:

```javascript
const processed = new Set();

function processWebhook(event) {
  if (processed.has(event.id)) return;
  processed.add(event.id);
  // Process event
}
```

## Monitoring

View delivery logs in your Teachfloor dashboard:
1. Go to **Developers** → **Webhooks**
2. Select your endpoint
3. Check delivery history

## Next Steps

- [Event Reference](./04-event-reference.md)
- [Security](./02-security.md)
- [Troubleshooting](./05-troubleshooting.md)
