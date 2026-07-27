# Webhooks

Webhooks deliver platform events to your app's **own backend** — HTTP POSTs signed with your app's secret, fired whenever something the app cares about happens on the platform. Complements [Realtime Channels](/docs/apps/advanced-topics/realtime): realtime pushes events into the browser iframe while a learner is active; webhooks reach your server even when no learner is online.

## What you can build

- **Data sync** — mirror course completions, quiz submissions, or member joins into an HR system, LMS, analytics warehouse, or CRM.
- **Backend integrations** — post activity notifications to Slack, Discord, Microsoft Teams, or a shared inbox.
- **Provisioning + teardown** — bootstrap per-tenant state when your app is installed on a new organization; clean up when it's uninstalled.
- **Third-party mirroring** — keep a system-of-record (e.g. a marketing tool or student-info system) in step with events happening on the platform.

## How it works

Every extension app can optionally declare a single **webhook endpoint URL** in its manifest. When an organization installs the app, Teachfloor auto-provisions the delivery pipeline against that URL — no per-tenant configuration required. The app's backend receives POSTs for every event it subscribed to, plus two lifecycle events (`app.installed`, `app.uninstalled`) that fire regardless of subscription.

One URL per app is delivered to for **every** installing organization. The payload carries the organization identifier so your backend can attribute the event to the right tenant.

## Manifest schema

Declare the block at the top level of your manifest alongside `views` and `permissions`:

```json
{
  "id": "my-app",
  "name": "My App",
  "version": "1.0.0",
  "webhook": {
    "url": "https://my-app.com/teachfloor/webhook",
    "events": [
      "course.completed",
      "element.completed",
      "course.join"
    ]
  }
}
```

**Fields:**

| Field | Required | Rules |
|---|---|---|
| `webhook.url` | yes (when block present) | `https://` only. HTTP is rejected — signed payloads over plaintext give no meaningful integrity guarantee. Max 2048 chars. |
| `webhook.events` | no | Array of event names drawn from the [event catalog](#event-catalog). Empty array is valid — you still receive the lifecycle events. Unknown event names are rejected at manifest push time. |

Omit the block entirely if your app doesn't need backend deliveries.

## The signing secret

Every app has a single **master signing secret** used to sign every webhook delivery to your endpoint, across every installing organization. You manage **one secret** per app, not one per install.

**Where to find it:** Developers → Apps → your app → the header metadata strip includes a "Signing Secret" cell with a reveal button. Click to show the raw value.

The secret is generated automatically when your app is created.

## Signature verification

Every delivery includes a `Teachfloor-Signature` header. The signature is `HMAC-SHA256(secret, request_body)`. Verify it before trusting the payload.

**Node.js / Express:**

```javascript
const crypto = require('crypto')
const express = require('express')

const app = express()

// Capture the raw body — Express's default JSON middleware discards it.
app.use('/teachfloor/webhook', express.raw({ type: 'application/json' }))

const TEACHFLOOR_SECRET = process.env.TEACHFLOOR_WEBHOOK_SECRET

app.post('/teachfloor/webhook', (req, res) => {
  const signature = req.header('Teachfloor-Signature')
  const expected = crypto
    .createHmac('sha256', TEACHFLOOR_SECRET)
    .update(req.body)
    .digest('hex')

  if (!signature || !crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(expected))) {
    return res.status(401).send('invalid signature')
  }

  const event = JSON.parse(req.body.toString())
  console.log('received', event.type, 'for', event.data.organization.id)
  res.status(200).end()
})
```

**Python / Flask:**

```python
import hmac
import hashlib
import json
import os
from flask import Flask, request, abort

app = Flask(__name__)
TEACHFLOOR_SECRET = os.environ["TEACHFLOOR_WEBHOOK_SECRET"].encode()

@app.route("/teachfloor/webhook", methods=["POST"])
def webhook():
    signature = request.headers.get("Teachfloor-Signature", "")
    expected = hmac.new(TEACHFLOOR_SECRET, request.data, hashlib.sha256).hexdigest()

    if not hmac.compare_digest(signature, expected):
        abort(401)

    event = json.loads(request.data)
    print(f"received {event['type']} for {event['data']['organization']['id']}")
    return "", 200
```

**PHP:**

```php
$secret    = getenv('TEACHFLOOR_WEBHOOK_SECRET');
$body      = file_get_contents('php://input');
$signature = $_SERVER['HTTP_TEACHFLOOR_SIGNATURE'] ?? '';
$expected  = hash_hmac('sha256', $body, $secret);

if (!hash_equals($expected, $signature)) {
    http_response_code(401);
    exit;
}

$event = json_decode($body, true);
error_log("received {$event['type']} for {$event['data']['organization']['id']}");
http_response_code(200);
```

Always use a constant-time comparison (`timingSafeEqual` / `hmac.compare_digest` / `hash_equals`). A plain `==` opens a timing side-channel on the signature check.

## Payload shape

Every delivery has the same envelope, with an event-specific `data` block inside:

```json
{
  "id": "evt_abc123",
  "type": "course.completed",
  "created_at": "2026-07-25T14:32:01.000000Z",
  "data": {
    ...event-specific fields
  }
}
```

`id` is stable and unique — safe to use as an idempotency key. `type` matches one of the [subscribable event names](#event-catalog).

### Lifecycle events

Two events are always delivered to your endpoint regardless of your `events` subscription:

**`app.installed`** — fired when an organization installs your app.

```json
{
  "id": "evt_...",
  "type": "app.installed",
  "created_at": "2026-07-25T14:32:01.000000Z",
  "data": {
    "installation_id": "7YWJgrMnJMybawx1",
    "organization": {
      "id": "acme-corp",
      "name": "Acme Corp"
    },
    "app": {
      "id": "6a41091c53b30",
      "name": "My App",
      "version": "1.0.0"
    },
    "installer": {
      "id": "usr_abc123",
      "email": "alice@acme.com",
      "full_name": "Alice Doe"
    },
    "credentials": {
      "client_id": "9f8e7d6c-...",
      "access_token": "eyJhbGc...",
      "refresh_token": "def50200...",
      "expires_at": "2026-08-25T14:32:01+00:00",
      "token_type": "Bearer",
      "scope": "courses:read modules:read"
    }
  }
}
```

The `installer` block identifies the user who clicked Install — always present on `app.installed`, regardless of whether the app declares any additional integrations. Use it to auto-provision or match an account on your landing page before the user arrives, so the post-install redirect can drop them straight into a signed-in experience rather than a sign-up form.

`installer` fields:
- `id` — the installer's user hashid; same format the SDK surfaces elsewhere.
- `email` — key your account lookup on this.
- `full_name` — for greetings / display.

The `credentials` block is **opt-in** and only appears when both of these are true in the app manifest:

1. An explicit `oauth: { type: "install" }` block declares that the app wants install-integrated OAuth.
2. At least one declared permission maps to an OAuth scope (currently: `courses:read`, `modules:read`, `elements:read`).

See the [OAuth chapter](/docs/apps/advanced-topics/oauth) for the full end-to-end recipe — opt-in, storage shape, calling the API, refresh flow, and code samples in Node / Python / PHP.

```json
{
  "id": "my-app",
  "oauth": { "type": "install" },
  "permissions": [
    { "permission": "courses:read", "purpose": "Sync course completions to our HR tool" }
  ],
  "webhook": { "url": "https://…", "events": ["course.completed"] }
}
```

Without the `oauth` block, no tokens are minted even if the manifest declares OAuth-mappable permissions — an iframe-only app that adds `courses:read` for in-iframe use won't unexpectedly start receiving credentials in its webhook receiver logs. Adding `oauth: { type: "install" }` is the explicit opt-in.

Iframe-only apps and apps with only SDK-scoped storage permissions receive no `credentials` — they can still deliver a signed webhook, they just don't get an access token for the public API. When `credentials` is present, use the tokens to call the public API on behalf of the installing organization — bearer-authenticated with `access_token`, refreshable via the standard OAuth 2.0 `/oauth/token` refresh flow using `client_id` + `client_secret` + `refresh_token`. Store `access_token` and `refresh_token` per-install (keyed by `organization.id`).

The `client_secret` is **not** in the webhook payload — you fetch it once from the Teachfloor developer dashboard (Developers → Apps → your app → OAuth Client Secret) or from the `teachfloor apps create` CLI response, and store it in your app config. It's the same across every install of your app.

`credentials` fields:
- `client_id` — the app's OAuth client identifier. Same across every install; not sensitive. Delivered here for correlation.
- `access_token` — bearer token for public API calls. Expires per `expires_at`.
- `refresh_token` — POST to `/oauth/token` with `grant_type=refresh_token` to mint a new access token; refresh tokens rotate (the old one is invalidated).
- `expires_at` — ISO 8601 timestamp for the access token; refresh before this.
- `token_type` — always `Bearer`.
- `scope` — space-separated scopes granted to this token.

**`app.uninstalled`** — fired just before an organization uninstalls your app.

Same payload shape as `app.installed` **minus the `installer` block**. `app.version` reflects the version being uninstalled (relevant during upgrades — see below). Use this event to purge tenant state.

### Version upgrades

When an organization upgrades your app to a new version, the platform tears down the old install and creates a new one. Your backend receives, in order:

1. `app.uninstalled` for the old version (fired against the old endpoint URL if that changed between versions).
2. `app.installed` for the new version (fired against the new endpoint URL).

Design your handlers to be idempotent. The `installation_id` changes between the old and new install; correlate by `organization.id` if you need to detect an upgrade rather than a fresh install.

If the new manifest **drops** the `webhook` block, only `app.uninstalled` fires — the app.installed event has nowhere to go. Treat the `app.uninstalled` as your last signal from that org.

## Event catalog

You can subscribe to any of the following:

| Event | Fires when |
|---|---|
| `course.created` | A course is created in the org |
| `course.updated` | Course metadata changes |
| `course.completed` | A learner marks a course as completed |
| `course.join` | A learner joins a course |
| `module.created` | A module is added to a course |
| `module.updated` | Module metadata changes |
| `element.created` | An element is added to a module |
| `element.updated` | Element metadata changes |
| `element.deleted` | An element is removed |
| `element.completed` | A learner completes an element |
| `member.login` | A member logs into the org |
| `app.installed` | (always delivered) |
| `app.uninstalled` | (always delivered) |

Subscribing to an unknown event name is rejected at manifest push time.

## Delivery guarantees

- **Retry** — up to 3 attempts on any non-2xx response, with exponential backoff between attempts.
- **Timeout** — 3 seconds per attempt. Endpoints that take longer are considered failed and retried.
- **Order** — deliveries are queued and processed asynchronously. Events for the same organization may be delivered out of order if one takes multiple attempts to succeed. Use the envelope's `created_at` if strict ordering matters.
- **At-least-once** — a single event may be delivered more than once if your endpoint responds 200 after our timeout window. Use the envelope `id` as an idempotency key to dedupe.
- **HTTPS required** — HTTP endpoints are rejected at manifest validation. TLS 1.2+ is expected on your endpoint.

Respond with any 2xx status code to acknowledge receipt. Any other response (4xx, 5xx, timeout) triggers a retry.

## Security model

- **Signature** verification is your responsibility. A delivery that doesn't verify against your master secret is forged — reject it.
- **The delivery URL and secret cross the wire only over HTTPS**, both from admin to browser (when revealing the secret) and from Teachfloor to your endpoint.
- **No inbound path is opened by declaring webhooks.** Webhooks are strictly outbound — Teachfloor never calls into your app's iframe or your platform-side runtime as a result of a webhook declaration.
- **Uninstall removes the endpoint.** When an org uninstalls your app, the webhook endpoint row is deleted (after the final `app.uninstalled` delivery). Further platform events do not attempt to reach your endpoint for that org.

## Next Steps

→ Continue to [OAuth](/docs/apps/advanced-topics/oauth)

## Additional Resources

- [Realtime Channels](/docs/apps/advanced-topics/realtime) - Sub-second event delivery into your app's iframe while the learner is active. Complementary to webhooks (which reach your backend regardless).
- [Permissions](/docs/apps/advanced-topics/permissions) - The permission scopes your app can declare in its manifest.
- [App Manifest](/docs/apps/core-concepts/app-manifest) - The full manifest schema, including the `webhook` block.
