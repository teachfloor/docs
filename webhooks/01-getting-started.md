# Getting Started

Set up webhooks to receive real-time event notifications from Teachfloor.

## Prerequisites

- Teachfloor account with access to Developers page
- Publicly accessible HTTPS endpoint
- Ability to process HTTP POST requests

## Setup Steps

<scalar-steps>
  <scalar-step id="step-1" title="Access Webhook Settings">
1. Log in to your Teachfloor account
2. Navigate to **Developers** → **Webhooks**
  </scalar-step>
  <scalar-step id="step-2" title="Add Endpoint">
1. Click **Add Endpoint**
2. Enter your HTTPS endpoint URL
3. Select which events to receive
4. Click **Save**
  </scalar-step>
  <scalar-step id="step-3" title="Get Signing Secret">
1. Click **Reveal Signing Secret** on your endpoint
2. Copy and store the secret securely
3. Use this to verify webhook signatures
  </scalar-step>
</scalar-steps>

## Webhook Payload

All webhooks have this structure:

```json
{
  "id": "evt_abc123",
  "type": "course.join",
  "created_at": "2025-10-07T15:30:00.000000Z",
  "data": {
    // Event-specific data
  }
}
```

## Basic Implementation

### Node.js Example

```javascript
app.post('/webhooks/teachfloor', express.json(), (req, res) => {
  // Respond immediately
  res.status(200).send('OK');

  // Process async
  processWebhook(req.body).catch(console.error);
});
```

### PHP Example

```php
Route::post('/webhooks/teachfloor', function (Request $request) {
    // Respond immediately
    response('OK', 200)->send();

    // Process async
    ProcessWebhookJob::dispatch($request->json()->all());
});
```

## Requirements

Your endpoint must:
- Accept POST requests
- Return 2xx status within 3 seconds
- Verify signatures (see [Security](./02-security.md))

## Next Steps

- [Security](./02-security.md) - Verify signatures
- [Delivery & Retries](./03-delivery-retries.md) - Understand reliability
- [Event Reference](./04-event-reference.md) - View available events
- [Troubleshooting](./05-troubleshooting.md) - Common issues
