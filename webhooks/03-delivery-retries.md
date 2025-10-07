# Delivery & Retries

Understanding webhook reliability and retry behavior.

## How Webhook Delivery Works

When a subscribed event occurs, Teachfloor attempts to deliver the webhook to your endpoint:

1. **Event occurs** - A subscribed action happens in Teachfloor
2. **Webhook sent** - HTTP POST request sent to your endpoint
3. **Response checked** - Your endpoint's response status is evaluated
4. **Delivery confirmed** - 2xx status = success, otherwise retry

## Successful Delivery

A webhook delivery is considered successful when your endpoint responds with any `2xx` status code:

- `200 OK` - Standard success response
- `201 Created` - Accepted and processed
- `202 Accepted` - Accepted for async processing
- `204 No Content` - Accepted without response body

**Best Practice**: Return `200 OK` immediately and process events asynchronously.

## Retry Mechanism

Teachfloor automatically retries failed webhook deliveries to ensure reliability.

### When Retries Occur

Retries are triggered when your endpoint returns:

- **3xx** - Redirection responses
- **4xx** - Client errors (400, 401, 404, etc.)
- **5xx** - Server errors (500, 502, 503, etc.)
- **No response** - Connection timeout or network error

### Retry Schedule

| Attempt | Wait Time | Total Elapsed |
|---------|-----------|---------------|
| Initial | 0s | 0s |
| 1st retry | 10s | 10s |
| 2nd retry | 100s | 110s |

After 3 total attempts (initial + 2 retries), the webhook is marked as failed.

### Example Timeline

```
00:00:00 - Initial attempt fails (503 Service Unavailable)
00:00:10 - First retry fails (timeout)
00:01:50 - Second retry succeeds (200 OK)
         - Webhook delivery complete
```

## Why Retries Matter

The retry mechanism helps handle:

- **Temporary outages** - Brief service interruptions
- **Network issues** - Transient connectivity problems
- **Server restarts** - Application deployments
- **Rate limiting** - Temporary request throttling

## Handling Retries in Your Application

### Idempotency

Since webhooks can be delivered multiple times, implement idempotent processing:

```javascript
// Store processed event IDs to prevent duplicate processing
const processedEvents = new Map();

async function processWebhook(event) {
  const eventId = `${event.event}_${event.timestamp}`;

  // Check if already processed
  if (processedEvents.has(eventId)) {
    console.log('Event already processed, skipping');
    return;
  }

  // Process the event
  await handleEvent(event);

  // Mark as processed
  processedEvents.set(eventId, Date.now());
}
```

### Database-Based Deduplication

```javascript
async function processWebhook(event) {
  const eventId = `${event.event}_${event.timestamp}_${event.data.id}`;

  // Check if event exists in database
  const existing = await db.webhookEvents.findOne({ eventId });

  if (existing) {
    console.log('Duplicate event, skipping');
    return;
  }

  // Store event record
  await db.webhookEvents.create({
    eventId,
    eventType: event.event,
    processedAt: new Date(),
    data: event.data
  });

  // Process the event
  await handleEvent(event);
}
```

## Best Practices for Reliability

### 1. Respond Quickly

Always acknowledge receipt immediately:

```javascript
app.post('/webhooks/teachfloor', async (req, res) => {
  // Respond immediately
  res.status(200).send('OK');

  // Process asynchronously
  processWebhookAsync(req.body).catch(console.error);
});
```

### 2. Use a Job Queue

For complex processing, use a background job system:

```javascript
// Node.js with Bull
const webhookQueue = new Queue('webhooks');

app.post('/webhooks/teachfloor', async (req, res) => {
  // Add to queue
  await webhookQueue.add(req.body);

  // Respond immediately
  res.status(200).send('OK');
});

// Process jobs separately
webhookQueue.process(async (job) => {
  await processWebhookEvent(job.data);
});
```

```php
// PHP with Laravel
Route::post('/webhooks/teachfloor', function (Request $request) {
    // Dispatch job
    ProcessWebhookJob::dispatch($request->all());

    // Respond immediately
    return response('OK', 200);
});
```

### 3. Implement Exponential Backoff

If your processing depends on external services, use exponential backoff:

```javascript
async function processWithRetry(event, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      await processEvent(event);
      return; // Success
    } catch (error) {
      if (i === maxRetries - 1) throw error;

      // Exponential backoff: 1s, 2s, 4s
      const delay = Math.pow(2, i) * 1000;
      await sleep(delay);
    }
  }
}
```

### 4. Monitor Webhook Health

Track webhook delivery metrics:

- Delivery success rate
- Average response time
- Retry frequency
- Failed deliveries

```javascript
async function handleWebhook(req, res) {
  const startTime = Date.now();

  try {
    res.status(200).send('OK');
    await processWebhook(req.body);

    // Log success
    metrics.recordWebhookSuccess(Date.now() - startTime);
  } catch (error) {
    // Log failure
    metrics.recordWebhookFailure(error);
    console.error('Webhook processing failed:', error);
  }
}
```

### 5. Handle Graceful Degradation

If your system is overloaded, still acknowledge receipt:

```javascript
app.post('/webhooks/teachfloor', async (req, res) => {
  // Check system load
  if (isSystemOverloaded()) {
    // Still acknowledge receipt
    res.status(200).send('OK');

    // Queue for later processing
    await deferredQueue.add(req.body, { delay: 30000 }); // Process in 30s
    return;
  }

  // Normal processing
  res.status(200).send('OK');
  await processWebhook(req.body);
});
```

## Monitoring Failed Deliveries

### Check Webhook Logs

View delivery attempts and failures in your Teachfloor dashboard:

1. Go to **Developers** → **Webhooks**
2. Select your endpoint
3. View the **Delivery Log** tab

The log shows:
- Timestamp of each attempt
- Response status code
- Response time
- Error details (if any)

### Set Up Alerts

Monitor your webhook endpoint:

```javascript
// Alert on high failure rate
if (failureRate > 0.1) { // 10% failure rate
  alerting.sendAlert('High webhook failure rate', {
    rate: failureRate,
    endpoint: '/webhooks/teachfloor'
  });
}

// Alert on consecutive failures
if (consecutiveFailures > 5) {
  alerting.sendAlert('Webhook endpoint may be down', {
    failures: consecutiveFailures
  });
}
```

## Troubleshooting

### High Retry Rate

**Symptoms**: Many webhooks being retried

**Common causes**:
- Endpoint returning non-2xx status codes
- Slow response times causing timeouts
- Application errors during processing
- Infrastructure issues (server down, network problems)

**Solutions**:
- Ensure endpoint returns 2xx immediately
- Move processing to background jobs
- Fix application errors
- Scale infrastructure if needed
- Add health checks and monitoring

### Timeouts

**Symptoms**: Webhooks timing out before response

**Common causes**:
- Synchronous processing taking too long
- Blocking operations in request handler
- Database queries in request handler
- External API calls before responding

**Solutions**:
```javascript
// Bad: Synchronous processing
app.post('/webhooks', async (req, res) => {
  await updateDatabase(req.body);  // Slow
  await callExternalAPI(req.body); // Slow
  res.status(200).send('OK');      // Too late!
});

// Good: Immediate response
app.post('/webhooks', async (req, res) => {
  res.status(200).send('OK');      // Immediate
  // Process async
  queue.add(req.body);
});
```

### Missing Webhooks

**Symptoms**: Expected webhooks not received

**Check**:
1. Event is selected in webhook settings
2. Endpoint URL is correct
3. Endpoint is publicly accessible
4. SSL certificate is valid
5. Firewall allows incoming traffic
6. No rate limiting blocking requests

**Debug**:
```bash
# Test endpoint accessibility
curl -X POST https://your-endpoint.com/webhooks/teachfloor \
  -H "Content-Type: application/json" \
  -d '{"test": true}'

# Check SSL certificate
openssl s_client -connect your-endpoint.com:443 -servername your-endpoint.com
```

### Duplicate Events

**Symptoms**: Same event processed multiple times

**Causes**:
- Retries after temporary failures
- Network issues causing duplicate sends
- Application not implementing idempotency

**Solution**: Implement idempotency (see [Handling Retries](#handling-retries-in-your-application))

## Performance Tips

### 1. Optimize Response Time

```javascript
// Good: Sub-100ms response
app.post('/webhooks', (req, res) => {
  res.status(200).send('OK');      // ~1ms
  queue.add(req.body);             // ~5ms
});

// Bad: Multi-second response
app.post('/webhooks', async (req, res) => {
  await database.query();          // 200ms
  await externalAPI.call();        // 1000ms
  await processComplex();          // 500ms
  res.status(200).send('OK');      // Too slow!
});
```

### 2. Use Connection Pooling

```javascript
// Reuse database connections
const pool = new Pool({
  max: 20,
  idleTimeoutMillis: 30000
});
```

### 3. Implement Rate Limiting

Protect your endpoint from excessive load:

```javascript
const rateLimit = require('express-rate-limit');

const webhookLimiter = rateLimit({
  windowMs: 1 * 60 * 1000, // 1 minute
  max: 100, // 100 requests per minute
  message: 'Too many webhook requests'
});

app.post('/webhooks/teachfloor', webhookLimiter, handleWebhook);
```

## Next Steps

- Explore the [event reference](./04-event-reference.md)
- Review [security best practices](./02-security.md)
- Check the [getting started guide](./01-getting-started.md)
