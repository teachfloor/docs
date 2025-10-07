# Troubleshooting

Common issues and solutions for webhooks.

## Not Receiving Webhooks

**Check:**
- Endpoint URL is correct and accessible
- SSL certificate is valid
- Events are selected in webhook settings
- Firewall allows incoming HTTPS traffic
- Endpoint returns 2xx status code

**Test your endpoint:**
```bash
curl -X POST https://your-endpoint.com/webhooks/teachfloor \
  -H "Content-Type: application/json" \
  -d '{"test": true}'
```

## High Retry Rate

**Causes:**
- Endpoint not responding with 2xx status
- Response taking longer than 3 seconds
- Application errors during processing
- Infrastructure issues

**Solutions:**
- Respond with 2xx immediately (before processing)
- Move processing to background jobs
- Fix application errors
- Check server logs for issues

## Signature Verification Fails

**Check:**
1. Using correct secret for this endpoint
2. Using raw request body (not parsed JSON)
3. No middleware modifying the body
4. Correct character encoding (UTF-8)

**Debug:**
```javascript
console.log('Received signature:', req.headers['teachfloor-signature']);
console.log('Payload length:', payload.length);
console.log('Generated signature:', generatedSignature);
```

## Timeouts

**Causes:**
- Synchronous processing before responding
- Slow database queries
- External API calls before sending response

**Solution:**
```javascript
// Correct - respond immediately
app.post('/webhooks', (req, res) => {
  res.status(200).send('OK');
  processAsync(req.body);
});

// Wrong - may timeout
app.post('/webhooks', async (req, res) => {
  await processEvent(req.body);
  res.status(200).send('OK');
});
```

## Duplicate Events

**Cause:**
Webhooks may be delivered more than once due to retries.

**Solution:**
Use `event.id` for deduplication:

```javascript
const processed = new Set();

function processWebhook(event) {
  if (processed.has(event.id)) return;
  processed.add(event.id);
  // Process event
}
```

## Viewing Delivery Logs

Check webhook delivery history:
1. Go to **Developers** → **Webhooks**
2. Select your endpoint
3. View delivery log

The log shows:
- Timestamp of each attempt
- Response status code
- Response time
- Error details

## Need Help?

Contact support through your Teachfloor dashboard.
