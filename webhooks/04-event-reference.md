# Event Reference

Complete reference of available webhook events.

## Event Structure

All webhook events follow this JSON structure:

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

### Root Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier for this webhook event |
| `type` | string | The event type (e.g., `course.join`, `element.completed`) |
| `created_at` | string | ISO 8601 timestamp when the event was created |
| `data` | object | Event-specific payload (structure varies by event type) |

## Available Events

### Course Events

#### `course.created`
Triggered when a new course is created.

#### `course.updated`
Triggered when a course is updated.

#### `course.completed`
Triggered when a member completes a course.

#### `course.join`
Triggered when a member joins a course.

**Payload example:**
```json
{
  "id": "evt_abc123",
  "type": "course.join",
  "created_at": "2025-10-07T15:30:00.000000Z",
  "data": {
    "id": "cm_xyz789",
    "object": "course_member",
    "joined_at": "2025-10-07T15:30:00.000000Z",
    "member": {
      "id": "usr_123",
      "object": "member",
      "first_name": "John",
      "last_name": "Doe",
      "full_name": "John Doe",
      "avatar": "https://example.com/avatar.jpg",
      "email": "john@example.com",
      "is_email_verified": true,
      "last_seen": "2025-10-07T14:00:00.000000Z"
    },
    "course": {
      "id": "crs_456",
      "object": "course",
      "created_at": "2025-01-01T00:00:00.000000Z",
      "name": "Introduction to Development",
      "status": "published",
      "cover": "https://example.com/cover.jpg",
      "availability": "public",
      "visibility": "listed",
      "start_date": null,
      "end_date": null,
      "currency": "USD",
      "price": 99.00,
      "free_label": null,
      "url": "https://app.teachfloor.com/org/intro-dev",
      "public_url": "https://app.teachfloor.com/org/intro-dev/join",
      "join_url": "https://app.teachfloor.com/org/intro-dev/join",
      "metadata": {},
      "custom_fields": {}
    },
    "custom_fields": {}
  }
}
```

### Module Events

#### `module.created`
Triggered when a new module is created.

#### `module.updated`
Triggered when a module is updated.

### Element Events

#### `element.created`
Triggered when a new element is created.

#### `element.updated`
Triggered when an element is updated.

#### `element.deleted`
Triggered when an element is deleted.

#### `element.completed`
Triggered when a member completes an element.

**Payload example:**
```json
{
  "id": "evt_def456",
  "type": "element.completed",
  "created_at": "2025-10-07T15:30:00.000000Z",
  "data": {
    "id": "act_789",
    "object": "activity",
    "timestamp": "2025-10-07T15:30:00.000000Z",
    "status": "completed",
    "passed": true,
    "score": 95,
    "completed_by": "usr_123",
    "member": {
      "id": "usr_123",
      "object": "member",
      "first_name": "John",
      "last_name": "Doe",
      "full_name": "John Doe",
      "avatar": "https://example.com/avatar.jpg",
      "email": "john@example.com",
      "is_email_verified": true,
      "last_seen": "2025-10-07T15:30:00.000000Z"
    },
    "context": {
      "id": "elm_321",
      "object": "element",
      "created_at": "2025-01-01T00:00:00.000000Z",
      "name": "Introduction Video",
      "cover": "https://example.com/thumb.jpg",
      "type": "video",
      "position": 1,
      "module": "mod_654",
      "metadata": {}
    }
  }
}
```

### Member Events

#### `member.login`
Triggered when a member logs in.

## Object Types

### Member Object

```json
{
  "id": "usr_123",
  "object": "member",
  "first_name": "John",
  "last_name": "Doe",
  "full_name": "John Doe",
  "avatar": "https://example.com/avatar.jpg",
  "email": "john@example.com",
  "is_email_verified": true,
  "last_seen": "2025-10-07T15:30:00.000000Z"
}
```

### Course Object

```json
{
  "id": "crs_456",
  "object": "course",
  "created_at": "2025-01-01T00:00:00.000000Z",
  "name": "Course Name",
  "status": "published",
  "cover": "https://example.com/cover.jpg",
  "availability": "public",
  "visibility": "listed",
  "start_date": null,
  "end_date": null,
  "currency": "USD",
  "price": 99.00,
  "free_label": null,
  "url": "https://app.teachfloor.com/org/course-slug",
  "public_url": "https://app.teachfloor.com/org/course-slug/join",
  "join_url": "https://app.teachfloor.com/org/course-slug/join",
  "metadata": {},
  "custom_fields": {}
}
```

### Element Object

```json
{
  "id": "elm_321",
  "object": "element",
  "created_at": "2025-01-01T00:00:00.000000Z",
  "name": "Element Name",
  "cover": "https://example.com/thumb.jpg",
  "type": "video",
  "position": 1,
  "module": "mod_654",
  "metadata": {}
}
```

## Related Documentation

- [Getting Started](./getting-started) - Set up your first webhook
- [Security](./security) - Verify webhook signatures
- [Delivery & Retries](./delivery-retries) - Understand reliability
