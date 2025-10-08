# Security

Verify that webhook events are genuinely sent by Teachfloor.

## Why Verify?

Signature verification ensures:
- Events are genuinely from Teachfloor
- Payloads haven't been tampered with
- Protection against malicious requests

**Always verify signatures in production.**

## How It Works

Teachfloor signs each webhook using HMAC-SHA256:

```
signature = hash_hmac('sha256', json_payload, secret)
```

The signature is sent in the `Teachfloor-Signature` header.

## Getting Your Secret

1. Go to **Developers** → **Webhooks**
2. Select your endpoint
3. Click **Reveal Signing Secret**
4. Store securely (never commit to version control)

## Implementation

### PHP

```php
function verifyWebhookSignature(Request $request, string $secret): bool
{
    $payload = $request->getContent();
    $signature = $request->headers->get('Teachfloor-Signature');
    $generated = hash_hmac('sha256', $payload, $secret);

    return hash_equals($generated, $signature);
}

Route::post('/webhooks/teachfloor', function (Request $request) {
    $secret = config('services.teachfloor.webhook_secret');

    if (!verifyWebhookSignature($request, $secret)) {
        return response('Invalid signature', 401);
    }

    ProcessWebhookJob::dispatch($request->json()->all());
    return response('OK', 200);
});
```

### Node.js

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secret) {
  const generated = crypto
    .createHmac('sha256', secret)
    .update(payload, 'utf8')
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(generated),
    Buffer.from(signature)
  );
}

app.post('/webhooks/teachfloor',
  express.raw({ type: 'application/json' }),
  (req, res) => {
    const secret = process.env.TEACHFLOOR_WEBHOOK_SECRET;
    const signature = req.headers['teachfloor-signature'];
    const payload = req.body.toString('utf8');

    if (!verifyWebhookSignature(payload, signature, secret)) {
      return res.status(401).send('Invalid signature');
    }

    const event = JSON.parse(payload);
    processWebhookEvent(event);
    res.status(200).send('OK');
  }
);
```

## Important Notes

- Use raw request body (not parsed JSON)
- Use timing-safe comparison (`hash_equals` or `timingSafeEqual`)
- Store secrets in environment variables
- Verify before processing

## Next Steps

- [Delivery & Retries](./delivery-retries)
- [Event Reference](./event-reference)
- [Troubleshooting](./troubleshooting)
