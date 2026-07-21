# Realtime Channels

Live event streams scoped to a course or user. Extension apps subscribe to a channel, receive any events that other clients publish on it, and can publish their own.

:::info
**Requires `@teachfloor/extension-kit` ≥ 1.22.0.** Earlier versions of the kit don't ship the `realtime` namespace. Bump your app's dependency before adding the `realtime` permission to your manifest.
:::

## What you can build

- Live cohort presence — *"3 learners are also on this course right now"*.
- Multiplayer review — flashcards / quizzes / polls with synchronized state.
- Live admin dashboards — *"new submission received"*, *"X just completed the module"*.

## Channel model

A channel is identified by a **scope** and a **resource id**. Two scopes are available:

| Scope | Resource id | Who can subscribe |
|---|---|---|
| `course` | The id of a course in the user's organization. | Any member of that course's organization who has the app installed with the `realtime` permission. |
| `user` | The current user's id. | Only the user themselves. |

The resource id is the same id you receive from `useExtensionContext()` — for example `environment.context.course.id` on a course viewport. You never construct the channel string yourself; the SDK does it for you.

Channels are **private**: Teachfloor authenticates every subscription server-side and enforces the rule in the table above before accepting it.

## Permissions

Add the `realtime` permission to your manifest:

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

- Subscribing to `course`-scoped channels for any course in an organization the user is a member of.
- Subscribing to your own `user`-scoped channel (`scope: 'user'`, `id: <auth user>`).
- Publishing to channels you're subscribed to.

It does **not** grant cross-app, cross-org, or other-user channel access.

:::info
**Resource ids are gated by their own read permissions.** Subscribing to a `course`-scoped channel needs `course_read` in addition to `realtime` — without it the host strips `course` from the viewport payload, so `environment.context.course.id` is `undefined` and there's no id to subscribe with. The same applies for `module_read` and `element_read` if you ever derive an id from those contexts.
:::

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
  fromUserId:  'QnX4ogW9QbyLpEz9',          // publisher's id (same shape as userContext.id)
  at:          '2026-06-28T03:24:11+02:00', // server timestamp
}
```

`fromUserId` is the publisher's id — the same value you get from `useExtensionContext().userContext.id`. You can compare it against your own user id to filter your own activity, or use it in a navigation helper like `goToPath('/${slug}/users/${fromUserId}')` to deep-link to the peer's profile.

### Notes

- Publishers do **not** receive their own events back. Update your own UI optimistically when you call `.publish()`.
- Subscription and publish failures (wrong scope, missing permission, rate limit hit, network blip) are delivered to the channel's `onError(…)` handler — register one to log or surface them. Without it, failures are dropped.
- There's no message history. If a learner joins after an event was published, they won't see it. Cache state in `appdata` / `userdata` if you need persistence.

## Limits

| Limit | Value |
|---|---|
| Event name | `^[a-z][a-z0-9_]*$`, max 64 chars |
| Payload size | 4 KB per published message |
| Per user, per app | 60 messages / minute |
| Per app (org-wide) | 600 messages / minute |
| Concurrent subscriptions per app instance | unlimited (but bounded by the host's single WebSocket) |

Rate limits return `429` with a `retry_after_seconds` field; the SDK surfaces them through `channel.onError(…)` with `code: 'rate_limited'`.

## Example: live "currently here" counter

Realtime channels don't keep a presence roster for you — building one is a pattern of three pieces: announce yourself, refresh the announcement on a heartbeat, and locally drop peers you haven't heard from in a while. The example below shows all three.

```jsx
import { useEffect, useState } from 'react'
import { realtime, useExtensionContext } from '@teachfloor/extension-kit'

const HEARTBEAT_MS  = 30_000   // re-announce every 30s
const STALE_AFTER   = 60_000   // drop peers we haven't heard from in 60s
const PRUNE_TICK_MS = 10_000   // re-check the stale window every 10s

const LiveHereCounter = () => {
  const { environment } = useExtensionContext()
  const [peers, setPeers] = useState({}) // { [userId]: lastSeen timestamp }

  useEffect(() => {
    const courseId = environment?.context?.course?.id
    if (!courseId) return

    const channel = realtime.subscribe({ scope: 'course', id: courseId })

    // Add a peer or refresh their lastSeen on every heartbeat.
    channel.on('joined', ({ fromUserId }) => {
      setPeers((prev) => ({ ...prev, [fromUserId]: Date.now() }))
    })

    // Drop peers who explicitly leave (covers tab close on unmount).
    channel.on('left', ({ fromUserId }) => {
      setPeers((prev) => {
        const next = { ...prev }
        delete next[fromUserId]
        return next
      })
    })

    // Announce ourselves immediately, then on every heartbeat.
    const announce = () => channel.publish('joined', {})
    announce()
    const heartbeat = setInterval(announce, HEARTBEAT_MS)

    // Locally prune peers who've gone silent — covers closed laptops
    // and network drops where no `left` is fired.
    const prune = setInterval(() => {
      const cutoff = Date.now() - STALE_AFTER
      setPeers((prev) => Object.fromEntries(
        Object.entries(prev).filter(([, lastSeen]) => lastSeen >= cutoff)
      ))
    }, PRUNE_TICK_MS)

    return () => {
      clearInterval(heartbeat)
      clearInterval(prune)
      try { channel.publish('left', {}) } catch (_) { /* unsubscribed already */ }
      channel.unsubscribe()
    }
  }, [environment])

  // +1 for the current user — the publisher never receives their own events.
  return <div>{Object.keys(peers).length + 1} here right now</div>
}
```

The numbers come from a few design choices worth thinking through for your own app:

- **Heartbeat interval** trades freshness against publish budget. At 30 seconds, each learner uses 2 of their 60-events-per-minute budget — leaves plenty of headroom for other event types.
- **Stale window** is set to two missed heartbeats so a transient network blip doesn't drop someone for a beat.
- **Prune tick** controls how smoothly peers fade out in the UI; smaller values feel snappier but cost a tiny bit more re-rendering.

## Security model

Permission and scope checks run server-side on **both** the subscribe and publish paths. Even if a client bypassed the SDK and called the underlying API directly, it would still hit the same checks:

- The app must be installed in the resource's organization and declare the `realtime` permission.
- The user must have access to the underlying resource (member of the course's org for `course` channels; only their own user id for `user` channels).
- Publish payloads are validated against the size and rate limits above.

There is no way for an app to read a channel it doesn't subscribe to, or to publish on behalf of another user.

## Next Steps

→ Continue to [Permissions](/docs/apps/advanced-topics/permissions)

## Additional Resources

- [Permissions](/docs/apps/advanced-topics/permissions) - The full `realtime` permission listing alongside the rest of the platform's permissions.
- [Data Storage](/docs/apps/advanced-topics/data-storage) - Pair realtime with `appdata` / `userdata` for state that survives reloads.
