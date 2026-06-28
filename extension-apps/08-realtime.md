# Realtime Channels

Live, server-driven event streams scoped to a course or user. Extension apps subscribe to a channel and receive any events that other clients publish on it; they can also publish their own.

:::info
**Requires `@teachfloor/extension-kit` ≥ 1.22.0.** Earlier versions of the kit don't ship the `realtime` namespace. Bump your app's dependency before adding the `realtime` permission to your manifest.
:::

## What you can build

- Live cohort presence — *"3 learners are also reviewing this lesson"*.
- Multiplayer review — flashcards / quizzes / polls with synchronized state.
- Live admin dashboards — *"new submission received"*, *"X just completed the module"*.
- Cross-iframe coordination — one app emits, another app reacts.

## Channel model

A channel is identified by a **scope** and a **resource id**. v1 supports two scopes:

| Scope | Resource id | Who can subscribe |
|---|---|---|
| `course` | The id of a course in the user's organization. | Any member of that course's organization who has the app installed with the `realtime` permission. |
| `user` | The current user's id. | Only the user themselves. |

The resource id is the same id you receive from `useExtensionContext()` — for example `environment.context.course.id` on a course viewport. You never construct the channel string yourself; the SDK does it for you.

Channels are **private**: Teachfloor authenticates every subscription server-side and enforces the rule in the table above before accepting it.

## Permissions

Add a single permission to your manifest:

```json
{
  "permissions": [
    {
      "permission": "realtime",
      "purpose": "Show live cohort presence and broadcast quiz answers to peers in real time"
    }
  ]
}
```

The `realtime` permission grants:

- Subscribing to `course`-scoped channels for any course the user belongs to.
- Subscribing to your own `user`-scoped channel (`scope: 'user'`, `id: <auth user>`).
- Publishing to channels you're subscribed to.

It does **not** grant cross-app, cross-org, or other-user channel access.

## SDK API

```js
import { realtime } from '@teachfloor/extension-kit'

// Subscribe
const channel = realtime.subscribe({
  scope: 'course',
  id: courseId,
})

// Listen to a specific event
const off = channel.on('card_added', (payload) => {
  console.log('new card from a peer:', payload.data)
  console.log('sent by user:', payload.fromUserId)
  console.log('at:', payload.at)
})

// Or listen to every event on the channel
channel.onAny((eventName, payload) => { /* ... */ })

// Publish
channel.publish('card_added', { cardId: 'abc', front: '...', back: '...' })

// Clean up
off()                  // remove a single listener
channel.unsubscribe()  // tear down the entire subscription
```

### Event payload shape

What your `on` handler receives:

```js
{
  data:        { /* what the publisher passed to .publish() */ },
  fromUserId:  42,                          // publisher's user id
  at:          '2026-06-28T03:24:11+02:00', // server timestamp
}
```

### Notes

- Publishers do **not** receive their own events back. Update your own UI optimistically when you call `.publish()`.
- Subscribing to a channel you don't have access to (wrong scope / wrong user / wrong org) fails silently. Register a `channel.onError(…)` handler to surface those failures.
- There's no message history. If a learner joins after an event was published, they won't see it. Cache state in `appdata` / `userdata` if you need persistence.

## Limits (v1)

| Limit | Value |
|---|---|
| Event name | `^[a-z][a-z0-9_]*$`, max 64 chars |
| Payload size | 4 KB per published message |
| Per user, per app | 60 messages / minute |
| Per app (org-wide) | 600 messages / minute |
| Concurrent subscriptions per iframe | unlimited (but bounded by the host's single WebSocket) |

Rate limits return `429` with a `retry_after_seconds` field; the SDK surfaces them through `channel.onError(…)` with `code: 'rate_limited'`.

## Example: live "currently reviewing" counter

```jsx
import { useEffect, useState } from 'react'
import { realtime, useExtensionContext } from '@teachfloor/extension-kit'

const LiveReviewCounter = () => {
  const { environment, userContext } = useExtensionContext()
  const [activeUserIds, setActiveUserIds] = useState(new Set())

  useEffect(() => {
    const courseId = environment?.context?.course?.id
    if (!courseId) return

    const channel = realtime.subscribe({ scope: 'course', id: courseId })

    // Announce ourselves
    channel.publish('joined', {})
    // Periodically heartbeat so stale clients drop off
    const heartbeat = setInterval(() => channel.publish('joined', {}), 30000)

    channel.on('joined', (payload) => {
      setActiveUserIds((s) => new Set(s).add(payload.fromUserId))
    })

    return () => {
      clearInterval(heartbeat)
      channel.publish('left', {})
      channel.unsubscribe()
    }
  }, [environment])

  // +1 for the current user (publisher never receives their own events)
  return <div>{activeUserIds.size + 1} reviewing right now</div>
}
```

## Security model

Permission and scope checks run server-side on **both** the subscribe and publish paths. Even if a client bypassed the SDK and called the underlying API directly, it would still hit the same checks:

- The app must be installed in the resource's organization and declare the `realtime` permission.
- The user must have access to the underlying resource (member of the course's org for `course` channels; only their own user id for `user` channels).
- Publish payloads are validated against the size and rate limits above.

There is no way for an app to read a channel it doesn't subscribe to, or to publish on behalf of another user.

## Roadmap (v2 candidates)

- Presence channels with `members()` and `joining` / `leaving` events.
- Org-scoped channels (`scope: 'org'`).
- Server-originated broadcasts (Teachfloor pushes events into app channels).
- Optional message history (last N events delivered on subscribe).
- Per-app usage analytics in the developer dashboard.

## Next Steps

→ Continue to [Permissions](/docs/apps/advanced-topics/permissions)

## Additional Resources

- [Permissions](/docs/apps/advanced-topics/permissions) - The full `realtime` permission listing alongside the rest of the platform's permissions.
- [Data Storage](/docs/apps/advanced-topics/data-storage) - Pair realtime with `appdata` / `userdata` for state that survives reloads.
