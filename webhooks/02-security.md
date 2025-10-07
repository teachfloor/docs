# Security

Verify that webhook events are genuinely sent by Teachfloor.

## Why Verify Webhooks?

Webhook signature verification ensures:

- **Authenticity** - Events are genuinely from Teachfloor
- **Integrity** - Payloads haven't been tampered with
- **Security** - Protection against malicious requests

**Important**: Always verify webhook signatures in production environments.

## How Signature Verification Works

Teachfloor signs each webhook event using HMAC (Hash-based Message Authentication Code) with SHA-256. The signature is included in the `Teachfloor-Signature` header of each request.

### Verification Process

1. Retrieve your endpoint's signing secret
2. Generate a signature from the payload using HMAC-SHA256
3. Compare your generated signature with the header signature
4. Accept the event only if signatures match

## Getting Your Signing Secret

Each webhook endpoint has a unique signing secret:

1. Go to **Developers** → **Webhooks** in your Teachfloor dashboard
2. Select your webhook endpoint
3. Click **Reveal Signing Secret**
4. Copy and securely store the secret

**Security Note**: Keep your signing secret secure and never commit it to version control.

## Implementing Verification

### PHP Example

```php
<?php

/**
 * Verify Teachfloor webhook signature
 */
function verifyWebhookSignature(Request $request, string $signingSecret): bool
{
    // Get the raw payload
    $payload = $request->getContent();

    // Get the signature from the header
    $receivedSignature = $request->headers->get('Teachfloor-Signature');

    // Generate signature using your signing secret
    $generatedSignature = hash_hmac('sha256', $payload, $signingSecret);

    // Compare signatures using timing-safe comparison
    return hash_equals($generatedSignature, $receivedSignature);
}

// Usage in your webhook endpoint
Route::post('/webhooks/teachfloor', function (Request $request) {
    $signingSecret = config('services.teachfloor.webhook_secret');

    // Verify signature
    if (!verifyWebhookSignature($request, $signingSecret)) {
        return response('Invalid signature', 401);
    }

    // Signature verified, process the webhook
    $event = $request->json()->all();
    ProcessWebhookJob::dispatch($event);

    return response('OK', 200);
});
```

### Node.js Example

```javascript
const crypto = require('crypto');

/**
 * Verify Teachfloor webhook signature
 */
function verifyWebhookSignature(payload, receivedSignature, signingSecret) {
  // Generate signature using your signing secret
  const generatedSignature = crypto
    .createHmac('sha256', signingSecret)
    .update(payload, 'utf8')
    .digest('hex');

  // Compare signatures using timing-safe comparison
  return crypto.timingSafeEqual(
    Buffer.from(generatedSignature),
    Buffer.from(receivedSignature)
  );
}

// Usage in your webhook endpoint
app.post('/webhooks/teachfloor', express.raw({ type: 'application/json' }), (req, res) => {
  const signingSecret = process.env.TEACHFLOOR_WEBHOOK_SECRET;
  const receivedSignature = req.headers['teachfloor-signature'];
  const payload = req.body.toString('utf8');

  // Verify signature
  if (!verifyWebhookSignature(payload, receivedSignature, signingSecret)) {
    return res.status(401).send('Invalid signature');
  }

  // Signature verified, process the webhook
  const event = JSON.parse(payload);
  processWebhookEvent(event);

  res.status(200).send('OK');
});
```

### Python Example

```python
import hmac
import hashlib

def verify_webhook_signature(payload: bytes, received_signature: str, signing_secret: str) -> bool:
    """Verify Teachfloor webhook signature"""
    # Generate signature using your signing secret
    generated_signature = hmac.new(
        signing_secret.encode('utf-8'),
        payload,
        hashlib.sha256
    ).hexdigest()

    # Compare signatures using timing-safe comparison
    return hmac.compare_digest(generated_signature, received_signature)

# Usage in your webhook endpoint (Flask example)
@app.route('/webhooks/teachfloor', methods=['POST'])
def handle_webhook():
    signing_secret = os.environ['TEACHFLOOR_WEBHOOK_SECRET']
    received_signature = request.headers.get('Teachfloor-Signature')
    payload = request.get_data()

    # Verify signature
    if not verify_webhook_signature(payload, received_signature, signing_secret):
        return 'Invalid signature', 401

    # Signature verified, process the webhook
    event = request.get_json()
    process_webhook_event(event)

    return 'OK', 200
```

### Ruby Example

```ruby
require 'openssl'

# Verify Teachfloor webhook signature
def verify_webhook_signature(payload, received_signature, signing_secret)
  # Generate signature using your signing secret
  generated_signature = OpenSSL::HMAC.hexdigest(
    OpenSSL::Digest.new('sha256'),
    signing_secret,
    payload
  )

  # Compare signatures using timing-safe comparison
  ActiveSupport::SecurityUtils.secure_compare(generated_signature, received_signature)
end

# Usage in your webhook endpoint (Rails example)
class WebhooksController < ApplicationController
  skip_before_action :verify_authenticity_token

  def teachfloor
    signing_secret = ENV['TEACHFLOOR_WEBHOOK_SECRET']
    received_signature = request.headers['Teachfloor-Signature']
    payload = request.raw_post

    # Verify signature
    unless verify_webhook_signature(payload, received_signature, signing_secret)
      return render plain: 'Invalid signature', status: :unauthorized
    end

    # Signature verified, process the webhook
    event = JSON.parse(payload)
    ProcessWebhookJob.perform_later(event)

    render plain: 'OK', status: :ok
  end
end
```

## Best Practices

### 1. Use Timing-Safe Comparison

Always use timing-safe comparison functions to prevent timing attacks:

```javascript
// Good: Timing-safe comparison
crypto.timingSafeEqual(Buffer.from(sig1), Buffer.from(sig2));

// Bad: Regular string comparison
sig1 === sig2;
```

### 2. Store Secrets Securely

- Use environment variables or secret management systems
- Never hardcode secrets in your application
- Rotate secrets periodically
- Use different secrets for different environments

```javascript
// Good: Environment variable
const secret = process.env.TEACHFLOOR_WEBHOOK_SECRET;

// Bad: Hardcoded secret
const secret = 'whsec_abc123...';
```

### 3. Verify Before Processing

Always verify the signature before processing the webhook payload:

```javascript
// Good: Verify first
if (!verifySignature(payload, signature, secret)) {
  return res.status(401).send('Invalid signature');
}
processEvent(payload);

// Bad: Process first
processEvent(payload);
if (!verifySignature(payload, signature, secret)) {
  // Too late!
}
```

### 4. Use Raw Request Body

Verification requires the raw request body exactly as received:

```javascript
// Good: Use raw body parser
app.use('/webhooks', express.raw({ type: 'application/json' }));

// Bad: JSON parsed body won't match signature
app.use(express.json());
```

### 5. Log Verification Failures

Monitor and log signature verification failures:

```javascript
if (!verifySignature(payload, signature, secret)) {
  logger.warn('Webhook signature verification failed', {
    ip: req.ip,
    endpoint: req.path,
    timestamp: new Date().toISOString()
  });
  return res.status(401).send('Invalid signature');
}
```

## Troubleshooting

### Signature Verification Always Fails

**Common causes:**

1. **Wrong signing secret**
   - Verify you're using the correct secret for the endpoint
   - Check if secret was recently rotated

2. **Modified payload**
   - Ensure you're using the raw request body
   - Don't parse or modify the body before verification
   - Check middleware isn't transforming the payload

3. **Character encoding issues**
   - Use UTF-8 encoding for the payload
   - Ensure no character encoding transformations

4. **Trailing whitespace**
   - Use the exact raw payload with no trimming
   - Verify no middleware is modifying the body

### Debug Verification

```javascript
function debugSignatureVerification(payload, receivedSignature, secret) {
  console.log('Payload length:', payload.length);
  console.log('Payload (first 100 chars):', payload.substring(0, 100));
  console.log('Received signature:', receivedSignature);

  const generated = crypto
    .createHmac('sha256', secret)
    .update(payload, 'utf8')
    .digest('hex');

  console.log('Generated signature:', generated);
  console.log('Signatures match:', generated === receivedSignature);
}
```

## Rotating Signing Secrets

If you need to rotate your signing secret:

1. Generate a new secret in the Teachfloor dashboard
2. Update your application configuration with the new secret
3. Deploy your application
4. Delete the old secret from the dashboard

**Tip**: During rotation, temporarily accept both old and new secrets to prevent downtime.

## Security Checklist

- [ ] Signing secret stored securely (environment variables/secrets manager)
- [ ] Signature verification implemented and tested
- [ ] Using timing-safe comparison for signature checking
- [ ] Verification happens before processing
- [ ] Using raw request body for verification
- [ ] Logging verification failures
- [ ] HTTPS endpoint with valid SSL certificate
- [ ] Secrets not committed to version control
- [ ] Different secrets for different environments

## Next Steps

- Learn about [delivery and retries](./03-delivery-retries.md)
- Explore the [event reference](./04-event-reference.md)
