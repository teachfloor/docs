# Event Reference

Complete reference of available webhook events.

## Event Structure

All webhook events follow a consistent JSON structure:

```json
{
  "event": "event.type",
  "timestamp": "2025-10-07T15:30:00Z",
  "data": {
    // Event-specific data
  }
}
```

### Common Fields

| Field | Type | Description |
|-------|------|-------------|
| `event` | string | The event type identifier |
| `timestamp` | string | ISO 8601 timestamp when the event occurred |
| `data` | object | Event-specific payload (structure varies by event) |

## Available Events

Documentation for specific event types is coming soon. Check back or contact support for information about specific events you need to handle.

### Event Categories

Events are organized into the following categories:

- **Member Events** - User actions and lifecycle
- **Course Events** - Course creation, updates, and deletions
- **Element Events** - Learning element interactions
- **Progress Events** - Completion and progress tracking
- **Custom Events** - Organization-specific events

## Event Naming Convention

Event names follow the pattern: `resource.action`

Examples:
- `member.joined` - A member joined a course
- `course.updated` - A course was updated
- `element.completed` - A learning element was completed

## Payload Examples

### Member Event

```json
{
  "event": "member.joined",
  "timestamp": "2025-10-07T15:30:00Z",
  "data": {
    "member": {
      "id": "mem_abc123",
      "email": "user@example.com",
      "name": "John Doe",
      "avatar": "https://example.com/avatar.jpg"
    },
    "course": {
      "id": "course_xyz789",
      "name": "Introduction to Development",
      "slug": "intro-dev"
    },
    "organization": {
      "id": "org_123",
      "name": "Acme Academy"
    }
  }
}
```

### Course Event

```json
{
  "event": "course.updated",
  "timestamp": "2025-10-07T15:30:00Z",
  "data": {
    "course": {
      "id": "course_xyz789",
      "name": "Introduction to Development",
      "slug": "intro-dev",
      "status": "published",
      "updated_at": "2025-10-07T15:30:00Z"
    },
    "changes": {
      "name": {
        "old": "Intro to Dev",
        "new": "Introduction to Development"
      }
    },
    "organization": {
      "id": "org_123",
      "name": "Acme Academy"
    }
  }
}
```

### Element Completion Event

```json
{
  "event": "element.completed",
  "timestamp": "2025-10-07T15:30:00Z",
  "data": {
    "member": {
      "id": "mem_abc123",
      "email": "user@example.com",
      "name": "John Doe"
    },
    "element": {
      "id": "elem_456",
      "title": "Introduction Video",
      "type": "video"
    },
    "module": {
      "id": "mod_789",
      "title": "Getting Started"
    },
    "course": {
      "id": "course_xyz789",
      "name": "Introduction to Development"
    },
    "completion": {
      "completed_at": "2025-10-07T15:30:00Z",
      "score": 100,
      "status": "completed"
    }
  }
}
```

## Testing Events

### Test Webhook

You can trigger a test event from your Teachfloor dashboard:

1. Go to **Developers** → **Webhooks**
2. Select your endpoint
3. Click **Send Test Event**

Test events have the same structure as production events but include a `test: true` flag:

```json
{
  "event": "test.event",
  "timestamp": "2025-10-07T15:30:00Z",
  "test": true,
  "data": {
    "message": "This is a test webhook event"
  }
}
```

### Handling Test Events

```javascript
function handleWebhook(event) {
  // Skip test events in production
  if (event.test && process.env.NODE_ENV === 'production') {
    console.log('Test event received, skipping');
    return;
  }

  // Process real events
  processEvent(event);
}
```

## Event Filtering

When setting up your webhook, you can select which events to receive. This helps:

- Reduce unnecessary traffic
- Process only relevant events
- Simplify your webhook handler logic

**Tip**: Start with essential events and add more as needed.

## Event Versioning

As the Teachfloor platform evolves, event payloads may be updated. We follow these principles:

- **Additive changes** - New fields added without breaking existing integrations
- **Deprecation notices** - Advance warning before removing fields
- **Version headers** - Future events may include version identifiers

### Handling Unknown Fields

Design your webhook handler to ignore unknown fields:

```javascript
function processEvent(event) {
  // Extract only the fields you need
  const { event: eventType, timestamp, data } = event;

  // Ignore any additional fields
  // Your integration won't break if new fields are added
}
```

## Rate Limits

Webhook delivery is not rate-limited by default. However, your endpoint should:

- Handle bursts of events gracefully
- Implement your own rate limiting if needed
- Use queuing for high-volume scenarios

## Webhook Ordering

Events are delivered in near real-time, but:

- **No guaranteed order** - Events may arrive out of sequence
- **Timestamps included** - Use `timestamp` field to determine actual order
- **Design for idempotency** - Handle events in any order

### Example: Handling Out-of-Order Events

```javascript
function processEvent(event) {
  const lastProcessed = await getLastProcessedTimestamp(event.data.resource_id);

  // Skip if we've already processed a newer event
  if (lastProcessed && new Date(event.timestamp) < new Date(lastProcessed)) {
    console.log('Skipping older event');
    return;
  }

  // Process event and update timestamp
  await handleEvent(event);
  await updateLastProcessedTimestamp(event.data.resource_id, event.timestamp);
}
```

## Need More Information?

For detailed information about specific events:

- Check the [Teachfloor API documentation](https://docs.teachfloor.com)
- Contact support through your Teachfloor dashboard
- Request documentation for specific events you need

## Related Documentation

- [Getting Started](./01-getting-started.md) - Set up your first webhook
- [Security](./02-security.md) - Verify webhook signatures
- [Delivery & Retries](./03-delivery-retries.md) - Understand reliability
