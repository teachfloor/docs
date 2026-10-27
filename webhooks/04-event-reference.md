# Event Reference

Complete reference of the webhook events Teachfloor delivers, with the payload each one carries.

## Event Structure

Every delivery is a JSON document with the same envelope. The `data` block is the object the event is about, in the same shape the [Public API](/docs/api) returns it.

```json
{
  "id": "WqX3nKp7Lm2vRt9Z",
  "type": "course.join",
  "created_at": "2026-10-07T15:30:00.000000Z",
  "data": {
    // Event-specific object
  }
}
```

### Root Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier for this delivery. Stable across retries; mirrored in the `Teachfloor-Idempotency-Key` header, so you can deduplicate on either. |
| `type` | string | The event type (e.g. `course.join`, `element.completed`). |
| `created_at` | string | ISO 8601 timestamp of when the event was recorded. |
| `installation_id` | string | Present only on deliveries to an endpoint provisioned by an extension app: the id of the installation the event belongs to. See [Extension app webhooks](/docs/apps/advanced-topics/webhooks). |
| `data` | object | The event's object. Its `object` field names the shape (`course`, `module`, `element`, `member`, `course_member`, `organization_member`, `activity`). |

### Update events

Every `*.updated` event (`course.updated`, `module.updated`, `element.updated`, `member.updated`) carries the full current object in `data` plus a `data.previous_attributes` object holding only the keys that changed, with the value each had before the write. Keys that did not change are not listed. The previous value of a key is its complete old value: for `custom_fields` that is the whole previous map, not the changed field alone.

```json
{
  "id": "WqX3nKp7Lm2vRt9Z",
  "type": "module.updated",
  "created_at": "2026-10-07T15:30:00.000000Z",
  "data": {
    "id": "no0aApWNyRMD68rv",
    "object": "module",
    "name": "Week 2 - Group formation",
    "availability": "SCHEDULED",
    "start_date": "2026-02-20T17:30:00.000000Z",
    "end_date": "2026-02-20T17:50:00.000000Z",
    "previous_attributes": {
      "name": "Week 2",
      "availability": "CONTINUOUS",
      "start_date": null,
      "end_date": null
    }
  }
}
```

Enumerated values (`status`, `availability`, `visibility`, element `type`) are upper-case, exactly as the Public API returns them.

## Available Events

| Event | Fires when | `data` object |
|-------|-----------|---------------|
| `organization.join` | A member joins the organization | [Organization member](#organization-member-object) |
| `member.created` | A member joins the organization (same moment as `organization.join`) | [Member](#member-object) |
| `member.updated` | A member's name, avatar, email or organization-level custom fields change | [Member](#member-object) with `previous_attributes` |
| `member.revoked` | A member is removed from the organization | [Member](#member-object) as it was just before removal |
| `member.login` | A member logs in | [Login activity](#memberlogin) |
| `course.created` | A course is created | [Course](#course-object) |
| `course.updated` | A course is updated | [Course](#course-object) with `previous_attributes` |
| `course.join` | A member joins a course | [Course member](#course-member-object) |
| `course.completed` | A member completes a course | [Activity](#activity-object) with a course `context` |
| `module.created` | A module is created | [Module](#module-object) |
| `module.updated` | A module is updated | [Module](#module-object) with `previous_attributes` |
| `element.created` | An element is created | [Element](#element-object) |
| `element.updated` | An element is updated | [Element](#element-object) with `previous_attributes` |
| `element.deleted` | An element is deleted | [Deleted element](#elementdeleted) |
| `element.completed` | A member completes an element | [Activity](#activity-object) with an element `context` |
| `xapi.statement.created` | An xAPI statement is recorded | [xAPI statement](#xapistatementcreated) |
| `app.installed` | An organization installs your extension app | See [Extension app webhooks](/docs/apps/advanced-topics/webhooks) |
| `app.uninstalled` | An organization uninstalls your extension app | See [Extension app webhooks](/docs/apps/advanced-topics/webhooks) |

Events fire for changes made from the dashboard, through the Public API, by automations and by the platform itself. A change your own integration makes through the API is reported like any other.

### Organization Events

#### `organization.join`

Fires when a user becomes a member of the organization: signup, invite acceptance, SSO provisioning, or the Members API. The `data` block is the membership, with the member and the organization nested.

```json
{
  "id": "WqX3nKp7Lm2vRt9Z",
  "type": "organization.join",
  "created_at": "2026-10-07T15:30:00.000000Z",
  "data": {
    "id": "gMANlEd0yb8BwazP",
    "object": "organization_member",
    "joined_at": "2026-10-07T15:30:00.000000Z",
    "member": {
      "id": "e7W6kP8wvjd3GRBz",
      "object": "member",
      "first_name": "John",
      "last_name": "Doe",
      "full_name": "John Doe",
      "avatar": "https://cdn.teachfloor.com/public/1/avatar.jpg",
      "email": "john@example.com",
      "is_email_verified": true,
      "last_seen": null
    },
    "organization": {
      "id": "7YWJgrMnJMybawx1",
      "object": "organization",
      "created_at": "2024-01-01T00:00:00.000000Z",
      "name": "Acme Academy",
      "domain": "learn.acme.com",
      "url": "https://app.teachfloor.com/acme"
    },
    "custom_fields": {
      "field_7": "Head of Product"
    }
  }
}
```

### Member Events

#### `member.created`

Fires at the same moment as `organization.join`, carrying the plain member object instead of the membership envelope. Subscribe to whichever shape suits your integration.

```json
{
  "id": "WqX3nKp7Lm2vRt9Z",
  "type": "member.created",
  "created_at": "2026-10-07T15:30:00.000000Z",
  "data": {
    "id": "e7W6kP8wvjd3GRBz",
    "object": "member",
    "first_name": "John",
    "last_name": "Doe",
    "full_name": "John Doe",
    "avatar": null,
    "email": "john@example.com",
    "is_email_verified": true,
    "last_seen": null,
    "custom_fields": {
      "field_7": "Head of Product"
    }
  }
}
```

#### `member.updated`

Fires when a member's first name, last name, avatar or email changes, or when their organization-level custom field values change, from any surface: the member's own account page, the members or team pages, the profile editor, the Public API, or a confirmed email change. Course-level custom fields are not part of this event.

```json
{
  "id": "WqX3nKp7Lm2vRt9Z",
  "type": "member.updated",
  "created_at": "2026-10-07T15:30:00.000000Z",
  "data": {
    "id": "e7W6kP8wvjd3GRBz",
    "object": "member",
    "first_name": "John",
    "last_name": "Doe-Smith",
    "full_name": "John Doe-Smith",
    "avatar": "https://cdn.teachfloor.com/public/1/avatar.jpg",
    "email": "john@example.com",
    "is_email_verified": true,
    "last_seen": "2026-10-07T15:29:40.000000Z",
    "custom_fields": {
      "field_7": "VP Product",
      "field_9": ["Option 1", "Option 2"]
    },
    "previous_attributes": {
      "last_name": "Doe",
      "custom_fields": {
        "field_7": "Head of Product",
        "field_9": ["Option 1", "Option 2"]
      }
    }
  }
}
```

An email change is confirmed by the member through a signed link and applies to their account as a whole, so every organization the member belongs to receives its own `member.updated`.

#### `member.revoked`

Fires when a member is removed from the organization: from the members page, the team page, the Organization API `revoke` endpoint, an automation, or the member deleting their own account. The `data` block is the member object as it was just before removal, custom fields included.

```json
{
  "id": "WqX3nKp7Lm2vRt9Z",
  "type": "member.revoked",
  "created_at": "2026-10-07T15:30:00.000000Z",
  "data": {
    "id": "e7W6kP8wvjd3GRBz",
    "object": "member",
    "first_name": "John",
    "last_name": "Doe",
    "full_name": "John Doe",
    "avatar": null,
    "email": "john@example.com",
    "is_email_verified": true,
    "last_seen": "2026-09-10T23:05:58.000000Z",
    "custom_fields": {}
  }
}
```

#### `member.login`

Fires when a member logs in. The `data` block is a small login activity rather than the full member.

```json
{
  "id": "WqX3nKp7Lm2vRt9Z",
  "type": "member.login",
  "created_at": "2026-10-07T15:30:00.000000Z",
  "data": {
    "id": "B4peAvmZ1L9OY3Xr",
    "object": "activity",
    "member": "e7W6kP8wvjd3GRBz",
    "logged_in_at": "2026-10-07T15:30:00.000000Z"
  }
}
```

### Course Events

#### `course.created`

Fires when a course is created. The `data` block is the [Course object](#course-object).

#### `course.updated`

Fires when a course is updated. The `data` block is the [Course object](#course-object) with `previous_attributes`.

```json
{
  "id": "WqX3nKp7Lm2vRt9Z",
  "type": "course.updated",
  "created_at": "2026-10-07T15:30:00.000000Z",
  "data": {
    "id": "p5v87ERWKR3dkjyD",
    "object": "course",
    "created_at": "2026-04-27T19:24:11.000000Z",
    "name": "Introduction to Development (2026)",
    "status": "PUBLISHED",
    "cover": "https://cdn.teachfloor.com/public/1/cover.jpg",
    "availability": "CONTINUOUS",
    "visibility": "PRIVATE",
    "start_date": null,
    "end_date": null,
    "currency": "usd",
    "price": null,
    "free_label": "Free",
    "url": "https://app.teachfloor.com/acme/courses/187",
    "public_url": "https://app.teachfloor.com/acme/c/187",
    "join_url": "https://app.teachfloor.com/acme/c/187/join",
    "content": "# Introduction\nWelcome to the course.",
    "metadata": {
      "category": "engineering"
    },
    "previous_attributes": {
      "name": "Introduction to Development"
    }
  }
}
```

#### `course.join`

Fires when a member joins a course. The `data` block is the course membership, with the member and the course nested, plus the member's course-level custom fields and tags.

```json
{
  "id": "WqX3nKp7Lm2vRt9Z",
  "type": "course.join",
  "created_at": "2026-10-07T15:30:00.000000Z",
  "data": {
    "id": "Vzo2KDjObalAXmMa",
    "object": "course_member",
    "joined_at": "2026-10-07T15:30:00.000000Z",
    "member": {
      "id": "e7W6kP8wvjd3GRBz",
      "object": "member",
      "first_name": "John",
      "last_name": "Doe",
      "full_name": "John Doe",
      "avatar": null,
      "email": "john@example.com",
      "is_email_verified": true,
      "last_seen": "2026-10-07T14:00:00.000000Z"
    },
    "course": {
      "id": "p5v87ERWKR3dkjyD",
      "object": "course",
      "created_at": "2026-01-01T00:00:00.000000Z",
      "name": "Introduction to Development",
      "status": "PUBLISHED",
      "cover": "https://cdn.teachfloor.com/public/1/cover.jpg",
      "availability": "CONTINUOUS",
      "visibility": "PUBLIC",
      "start_date": null,
      "end_date": null,
      "currency": "usd",
      "price": 99,
      "free_label": null,
      "url": "https://app.teachfloor.com/acme/courses/456",
      "public_url": "https://app.teachfloor.com/acme/c/456",
      "join_url": "https://app.teachfloor.com/acme/c/456/join",
      "content": null,
      "metadata": {}
    },
    "custom_fields": {
      "field_12": "Vegetarian"
    },
    "tags": ["vip", "beta-cohort"]
  }
}
```

#### `course.completed`

Fires when a member completes a course. The `data` block is an [Activity](#activity-object) whose `context` is the [Course object](#course-object).

```json
{
  "id": "WqX3nKp7Lm2vRt9Z",
  "type": "course.completed",
  "created_at": "2026-10-07T15:30:00.000000Z",
  "data": {
    "id": "m2Kd7RzQ4pLw9XtB",
    "object": "activity",
    "timestamp": "2026-10-07T15:30:00.000000Z",
    "member": {
      "id": "e7W6kP8wvjd3GRBz",
      "object": "member",
      "first_name": "John",
      "last_name": "Doe",
      "full_name": "John Doe",
      "avatar": null,
      "email": "john@example.com",
      "is_email_verified": true,
      "last_seen": "2026-10-07T15:30:00.000000Z"
    },
    "context": {
      "id": "p5v87ERWKR3dkjyD",
      "object": "course",
      "name": "Introduction to Development",
      "status": "PUBLISHED"
    }
  }
}
```

### Module Events

#### `module.created`

Fires when a module is created. The `data` block is the [Module object](#module-object).

#### `module.updated`

Fires when a module is updated. The `data` block is the [Module object](#module-object) with `previous_attributes`; see the [update events example](#update-events).

### Element Events

#### `element.created`

Fires when an element is created. The `data` block is the [Element object](#element-object).

#### `element.updated`

Fires when an element is updated. The `data` block is the [Element object](#element-object) with `previous_attributes`.

```json
{
  "id": "WqX3nKp7Lm2vRt9Z",
  "type": "element.updated",
  "created_at": "2026-10-07T15:30:00.000000Z",
  "data": {
    "id": "QB8nmEJjqdbLA2K5",
    "object": "element",
    "created_at": "2026-10-15T18:58:58.000000Z",
    "name": "Course recording",
    "cover": null,
    "type": "FILE",
    "content": null,
    "position": 0,
    "course": "p5v87ERWKR3dkjyD",
    "module": "no0aApWNyRMD68rv",
    "metadata": {},
    "properties": {
      "file_url": "https://cdn.teachfloor.com/public/1/recording.mp4"
    },
    "previous_attributes": {
      "name": "Recording"
    }
  }
}
```

#### `element.deleted`

Fires when an element is deleted. Because the element no longer exists, the `data` block is reduced to its id and a `deleted` flag.

```json
{
  "id": "WqX3nKp7Lm2vRt9Z",
  "type": "element.deleted",
  "created_at": "2026-10-07T15:30:00.000000Z",
  "data": {
    "id": "QB8nmEJjqdbLA2K5",
    "object": "element",
    "deleted": true
  }
}
```

#### `element.completed`

Fires when a member completes an element. The `data` block is an [Activity](#activity-object) whose `context` is the [Element object](#element-object). For group work the `member` field is replaced by a `members` array.

```json
{
  "id": "Hs8dFk2mNp4qTv6Y",
  "type": "element.completed",
  "created_at": "2026-10-07T15:30:00.000000Z",
  "data": {
    "id": "B4peAvmZ1L9OY3Xr",
    "object": "activity",
    "timestamp": "2026-10-07T15:30:00.000000Z",
    "status": "COMPLETED",
    "passed": true,
    "score": 95,
    "completed_by": "e7W6kP8wvjd3GRBz",
    "member": {
      "id": "e7W6kP8wvjd3GRBz",
      "object": "member",
      "first_name": "John",
      "last_name": "Doe",
      "full_name": "John Doe",
      "avatar": null,
      "email": "john@example.com",
      "is_email_verified": true,
      "last_seen": "2026-10-07T15:30:00.000000Z"
    },
    "context": {
      "id": "QB8nmEJjqdbLA2K5",
      "object": "element",
      "created_at": "2026-01-01T00:00:00.000000Z",
      "name": "Final quiz",
      "cover": null,
      "type": "QUIZ",
      "content": null,
      "position": 4,
      "course": "p5v87ERWKR3dkjyD",
      "module": "no0aApWNyRMD68rv",
      "metadata": {},
      "properties": {
        "passing_score": 80
      }
    }
  }
}
```

The fields after `timestamp` (`status`, `passed`, `score`, `completed_by`, and any type-specific values) depend on the element type.

### xAPI Events

#### `xapi.statement.created`

Fires once per xAPI statement the platform records. The `data` block is the complete statement, so you can forward it to a Learning Record Store unchanged.

```json
{
  "id": "WqX3nKp7Lm2vRt9Z",
  "type": "xapi.statement.created",
  "created_at": "2026-10-07T15:30:00.000000Z",
  "data": {
    "id": "6f1a2b3c-4d5e-4f60-8a9b-0c1d2e3f4a5b",
    "actor": {
      "objectType": "Agent",
      "account": {
        "homePage": "https://teachfloor.com/xapi",
        "name": "e7W6kP8wvjd3GRBz"
      }
    },
    "verb": {
      "id": "http://adlnet.gov/expapi/verbs/completed",
      "display": { "en-US": "completed" }
    },
    "object": {
      "objectType": "Activity",
      "id": "https://teachfloor.com/xapi/activities/element/QB8nmEJjqdbLA2K5",
      "definition": {
        "type": "https://teachfloor.com/xapi/activity-types/element",
        "name": { "en-US": "Final quiz" }
      }
    },
    "result": {
      "success": true,
      "completion": true
    },
    "timestamp": "2026-10-07T15:30:00+00:00",
    "version": "1.0.3"
  }
}
```

### Extension app events

`app.installed` and `app.uninstalled` are delivered only to endpoints declared in an extension app's manifest, and always, regardless of the events the manifest subscribes to. Their payloads carry the installing organization, the installer and, when the app opts in, OAuth credentials. See [Extension app webhooks](/docs/apps/advanced-topics/webhooks).

## Object Types

### Member Object

```json
{
  "id": "e7W6kP8wvjd3GRBz",
  "object": "member",
  "first_name": "John",
  "last_name": "Doe",
  "full_name": "John Doe",
  "avatar": "https://cdn.teachfloor.com/public/1/avatar.jpg",
  "email": "john@example.com",
  "is_email_verified": true,
  "last_seen": "2026-10-07T15:30:00.000000Z",
  "custom_fields": {
    "field_7": "Head of Product"
  }
}
```

`custom_fields` holds the member's organization-level custom field values keyed by field key, and is present on the `member.created`, `member.updated` and `member.revoked` events. The member nested inside `organization.join`, `course.join` and the activity events carries no `custom_fields` of its own; `organization.join` and `course.join` carry them one level up on the membership instead.

### Organization Member Object

The membership of a member in the organization, used by `organization.join`.

| Field | Description |
|-------|-------------|
| `id` | Membership id |
| `object` | `organization_member` |
| `joined_at` | When the member joined |
| `member` | [Member object](#member-object) without `custom_fields` |
| `organization` | `id`, `object` (`organization`), `created_at`, `name`, `domain`, `url` |
| `custom_fields` | The member's organization-level custom field values |

### Course Object

```json
{
  "id": "p5v87ERWKR3dkjyD",
  "object": "course",
  "created_at": "2026-01-01T00:00:00.000000Z",
  "name": "Course Name",
  "status": "PUBLISHED",
  "cover": "https://cdn.teachfloor.com/public/1/cover.jpg",
  "availability": "CONTINUOUS",
  "visibility": "PUBLIC",
  "start_date": null,
  "end_date": null,
  "currency": "usd",
  "price": 99,
  "free_label": null,
  "url": "https://app.teachfloor.com/acme/courses/456",
  "public_url": "https://app.teachfloor.com/acme/c/456",
  "join_url": "https://app.teachfloor.com/acme/c/456/join",
  "content": "# Introduction\nCourse description in Markdown.",
  "metadata": {}
}
```

`status` is `DRAFT` or `PUBLISHED`; `availability` is `CONTINUOUS` or `SCHEDULED` (with `start_date` and `end_date` set); `visibility` is `PUBLIC` or `PRIVATE`. `metadata` is the key-value store you write through the Public API.

### Course Member Object

The membership of a member in a course, used by `course.join`.

| Field | Description |
|-------|-------------|
| `id` | Course membership id |
| `object` | `course_member` |
| `joined_at` | When the member joined the course |
| `member` | [Member object](#member-object) without `custom_fields` |
| `course` | [Course object](#course-object) |
| `custom_fields` | The member's course-level custom field values |
| `tags` | The member's tags in this course, as an array of names |

### Module Object

```json
{
  "id": "no0aApWNyRMD68rv",
  "object": "module",
  "created_at": "2026-05-04T18:37:55.000000Z",
  "name": "Week 1",
  "type": "LESSON",
  "availability": "CONTINUOUS",
  "start_date": null,
  "end_date": null,
  "content": null,
  "position": 0,
  "course": "p5v87ERWKR3dkjyD",
  "metadata": {}
}
```

### Element Object

```json
{
  "id": "QB8nmEJjqdbLA2K5",
  "object": "element",
  "created_at": "2026-01-01T00:00:00.000000Z",
  "name": "Element Name",
  "cover": null,
  "type": "VIDEO",
  "content": null,
  "position": 1,
  "course": "p5v87ERWKR3dkjyD",
  "module": "no0aApWNyRMD68rv",
  "metadata": {},
  "properties": {
    "video_url": "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
    "video_type": "URL"
  }
}
```

`type` is the upper-case element type (`CONTENT`, `VIDEO`, `QUIZ`, `FORM`, `SUBMISSION`, and so on). `properties` holds the type-specific settings, with the same keys the Public API accepts when creating or updating the element.

### Activity Object

A member's activity on a course or element, used by `course.completed` and `element.completed`.

| Field | Description |
|-------|-------------|
| `id` | Activity id |
| `object` | `activity` |
| `timestamp` | When the activity was recorded |
| `member` | [Member object](#member-object) without `custom_fields`. Replaced by `members`, an array, for group activities |
| `context` | The [Course](#course-object) or [Element object](#element-object) the activity belongs to |
| `status`, `passed`, `score`, `completed_by` | Element activities only; further keys depend on the element type |

### Deleted objects

When the object an event is about no longer exists (`element.deleted`, or a nested course or module that was removed), it is reduced to `{ "id", "object", "deleted": true }`.

## Related Documentation

- [Getting Started](./getting-started) - Set up your first webhook
- [Security](./security) - Verify webhook signatures
- [Delivery & Retries](./delivery-retries) - Understand reliability
