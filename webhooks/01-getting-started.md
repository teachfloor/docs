# Getting Started

Set up webhooks to receive real-time event notifications from Teachfloor.

## Prerequisites

Before setting up webhooks, ensure you have:

- A Teachfloor account with access to the Developers page
- A publicly accessible HTTPS endpoint to receive webhook events
- The ability to process incoming HTTP POST requests

## Setting Up Your First Webhook

### Step 1: Access Webhook Settings

1. Log in to your [Teachfloor account](https://app.teachfloor.com)
2. Navigate to the **Developers** page
3. Click on the **Webhooks** tab

### Step 2: Add a Webhook Endpoint

1. Click the **Add Endpoint** button
2. Enter your webhook endpoint URL
   - Must be a valid HTTPS URL
   - Should be publicly accessible
   - Example: `https://api.yourdomain.com/webhooks/teachfloor`

### Step 3: Select Events

Choose which events you want to receive notifications for:

- Member events (joins, updates, completions)
- Course events (created, updated, deleted)
- Element events (completed, graded)
- Custom events specific to your use case

**Tip**: Start with a few essential events and add more as needed.

### Step 4: Save Configuration

Click **Save** to finalize your webhook setup. Teachfloor will immediately start sending notifications to your endpoint when subscribed events occur.

## Testing Your Webhook

After setup, test your webhook endpoint:

1. Trigger a test event in your Teachfloor account (e.g., join a course)
2. Check your endpoint logs to confirm receipt
3. Verify the payload structure matches your expectations

## Webhook Payload Structure

All webhook events follow a consistent JSON structure:

```json
{
  "event": "member.joined",
  "timestamp": "2025-10-07T15:30:00Z",
  "data": {
    "member": {
      "id": "mem_123",
      "email": "user@example.com",
      "name": "John Doe"
    },
    "course": {
      "id": "course_456",
      "name": "Introduction to Development"
    }
  }
}
```

### Payload Fields

| Field | Type | Description |
|-------|------|-------------|
| `event` | string | The event type that triggered the webhook |
| `timestamp` | string | ISO 8601 timestamp of when the event occurred |
| `data` | object | Event-specific data (structure varies by event type) |

## Endpoint Requirements

Your webhook endpoint must:

- **Accept POST requests** - Webhooks are sent via HTTP POST
- **Respond with 2xx status** - Return `200 OK` or similar to acknowledge receipt
- **Respond quickly** - Process events asynchronously to avoid timeouts
- **Use HTTPS** - Secure endpoints are required for production

### Example Endpoint (Node.js/Express)

```javascript
app.post('/webhooks/teachfloor', async (req, res) => {
  // Acknowledge receipt immediately
  res.status(200).send('OK');

  // Process event asynchronously
  const event = req.body;
  processWebhookEvent(event).catch(console.error);
});
```

### Example Endpoint (PHP/Laravel)

```php
Route::post('/webhooks/teachfloor', function (Request $request) {
    // Acknowledge receipt immediately
    response('OK', 200)->send();

    // Process event asynchronously
    $event = $request->all();
    ProcessWebhookJob::dispatch($event);
});
```

## Best Practices

### 1. Acknowledge Receipt Immediately

Return a `2xx` status code as quickly as possible. Process the event asynchronously to avoid timeouts.

```javascript
// Good: Immediate acknowledgment
res.status(200).send('OK');
processEventAsync(event);

// Bad: Processing before acknowledgment
await processEvent(event);
res.status(200).send('OK');
```

### 2. Handle Duplicate Events

Webhooks may be delivered more than once. Use idempotency keys to prevent duplicate processing:

```javascript
const processedEvents = new Set();

function processWebhook(event) {
  const eventId = `${event.event}_${event.timestamp}`;

  if (processedEvents.has(eventId)) {
    console.log('Duplicate event, skipping');
    return;
  }

  processedEvents.add(eventId);
  // Process event...
}
```

### 3. Implement Error Handling

Gracefully handle errors to ensure your endpoint remains reliable:

```javascript
app.post('/webhooks/teachfloor', async (req, res) => {
  try {
    res.status(200).send('OK');
    await processWebhookEvent(req.body);
  } catch (error) {
    console.error('Webhook processing error:', error);
    // Log error but don't fail the request
  }
});
```

### 4. Monitor Your Endpoint

- Log all incoming webhooks
- Track processing failures
- Set up alerts for unusual patterns
- Monitor response times

## Next Steps

- [Secure your webhooks](./02-security.md) with signature verification
- Learn about [delivery and retries](./03-delivery-retries.md)
- Explore the [event reference](./04-event-reference.md)

## Troubleshooting

### Webhook Not Receiving Events

**Check:**
- Endpoint URL is correct and publicly accessible
- Firewall/security groups allow incoming HTTPS traffic
- SSL certificate is valid and not expired
- Events are properly selected in webhook settings

### Events Being Retried

**Possible causes:**
- Endpoint not responding with 2xx status code
- Response taking too long (timeout)
- Network connectivity issues

See [Delivery & Retries](./03-delivery-retries.md) for more details.
