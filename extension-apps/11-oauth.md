# OAuth

Extension apps can call the Teachfloor public API on behalf of the organization that installed them. Every install of an OAuth-enabled app auto-mints an access token + refresh token, delivered to your app's backend through the `app.installed` webhook. No browser authorize step — clicking Install in the marketplace IS the consent grant.

This chapter covers how to opt in, how to receive and store credentials, how to make API calls, how to refresh tokens, and how revocation works.

## What you can build

- **Backend integrations that read platform data** — sync course completions to your HR system, mirror members to your CRM, ingest activity streams into your data warehouse.
- **Third-party dashboards** — build a Teachfloor-authenticated view of the org's courses in your own product's admin panel.
- **Automated workflows** — poll the API from your backend on your own schedule, augmenting the push-based webhook data.

If you only need in-iframe data access (inside your app's dashboard view), the SDK's [`teachfloor.get(...)`](/docs/apps/core-concepts/extension-kit/integration) helpers already do that. OAuth is for when your **backend** — outside the iframe, outside the browser — needs to reach platform data.

## How it works

When an organization installs your app, Teachfloor mints an access token + refresh token bound to that installation and delivers them inside the `app.installed` webhook payload. Your backend stores the tokens per-installation and uses them as bearer tokens against `https://api.teachfloor.com/v0/*`.

One `client_id` + `client_secret` pair identifies your app across every organization that installs it; each install then gets its own `access_token` and `refresh_token`. The `client_id` + `client_secret` are shown to you once at app creation (via the CLI and the Developer dashboard) — store them in your app config; they never appear in webhook payloads.

## Opt in

To receive credentials on install, declare both of these in your manifest:

1. An `oauth` block with `type: "install"`.
2. At least one permission that maps to an OAuth scope.

```json
{
  "id": "my-app",
  "name": "My App",
  "version": "1.0.0",
  "oauth": { "type": "install" },
  "permissions": [
    { "permission": "courses:read", "purpose": "Sync course completions to our HR tool" },
    { "permission": "modules:read", "purpose": "Attribute completions to the right module" }
  ],
  "webhook": {
    "url": "https://my-app.com/teachfloor/webhook",
    "events": ["course.completed"]
  }
}
```

Both signals are required. If either is missing, no credentials are minted — the webhook still fires (with the `installer` block), just without `credentials`.

**Fields:**

| Field | Required | Rules |
|---|---|---|
| `oauth.type` | yes (when block present) | Must be `"install"`. |

Also required: a `webhook` block. Credentials arrive inside the `app.installed` webhook, so if there's no webhook endpoint declared, there's nowhere to deliver them to.

## Permission → scope mapping

Adding a permission to the manifest grants the matching OAuth scope on the issued token — no separate scope declaration needed.

| Manifest permission | OAuth scope | Grants |
|---|---|---|
| `courses:read` | `courses:read` | GET `/v0/courses/*` |
| `modules:read` | `modules:read` | GET `/v0/modules/*` |
| `elements:read` | `elements:read` | GET `/v0/elements/*` |

Other manifest permissions (data storage, realtime, etc.) don't have a public-API equivalent and don't appear on issued tokens.

## Receiving credentials

Credentials arrive in the `data.credentials` block of the `app.installed` webhook. See the [Webhooks chapter](/docs/apps/advanced-topics/webhooks#lifecycle-events) for the full envelope.

```json
{
  "type": "app.installed",
  "data": {
    "organization": { "id": "acme-corp", "name": "Acme Corp" },
    "installer":    { "id": "usr_...", "email": "alice@acme.com", "full_name": "Alice Doe" },
    "credentials": {
      "client_id":     "9f8e7d6c-...",
      "access_token":  "eyJhbGc...",
      "refresh_token": "def50200...",
      "expires_at":    "2026-08-25T14:32:01+00:00",
      "token_type":    "Bearer",
      "scope":         "courses:read modules:read"
    }
  }
}
```

Store `access_token` and `refresh_token` per-install (keyed by `organization.id`) — you'll need both for API calls and for the refresh flow. Combine them at refresh time with the `client_id` + `client_secret` you already have in your app config from when you created the app.

The `client_secret` is **not** in this payload. Fetch it once from the Teachfloor developer dashboard (Developers → Apps → your app → **OAuth Client Secret**) or from the `teachfloor apps create` CLI response, and store it in your app config — it's the same across every install of your app, so per-install storage isn't required.

**Recommended per-install storage shape:**

```sql
CREATE TABLE teachfloor_installations (
  organization_id  VARCHAR(255) PRIMARY KEY,   -- from data.organization.id
  installer_email  VARCHAR(255),               -- from data.installer.email
  client_id        VARCHAR(255) NOT NULL,      -- from data.credentials.client_id
  access_token     TEXT NOT NULL,
  refresh_token    TEXT NOT NULL,
  expires_at       TIMESTAMP NOT NULL,
  scopes           TEXT NOT NULL,
  installed_at     TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Keep `client_secret` in your app-level config (env var, secrets manager), NOT per-install.

Treat all four values as sensitive. Don't log them to any observability tool that might retain them, and redact them from error reports.

## Auto-provisioning users on install

The `installer` block identifies the user who clicked Install — always present, whether or not the app opted into OAuth. Use it to auto-provision an account on your side BEFORE the user's browser arrives at your post-install landing page:

```javascript
// Inside your webhook handler for app.installed
async function handleAppInstalled(event) {
  const { organization, installer, credentials } = event.data

  // Provision or match an account for the installer.
  const user = await findOrCreateUser({
    email:    installer.email,
    name:     installer.full_name,
    externalId: installer.id,
  })

  // Store the OAuth credentials keyed by org.
  if (credentials) {
    await saveInstallation({
      organizationId: organization.id,
      installerEmail: installer.email,
      ...credentials,
    })
  }

  // Optionally sign the user in via a magic link so their post-install
  // redirect lands in a signed-in state.
  await sendMagicLink(user, { redirectUrl: '/dashboard' })
}
```

By the time the user's browser arrives at your post-install redirect URL, their account exists on your side. Match them to the installation by email and drop them into a signed-in experience — no sign-up form.

## Install delivery timing

The `app.installed` webhook lands at your backend **before** the browser follows the post-install redirect. Your landing page can rely on the installation already being provisioned on your side (credentials stored, user auto-provisioned).

To make that guarantee your endpoint has a **3-second window** to ack. If you respond in time, the redirect fires only after your ack. If you don't (slow, down, error), the install still succeeds and the webhook is retried on the standard schedule — the redirect proceeds and your landing page falls back to your normal sign-up flow.

**Practical implication:** keep your `app.installed` handler fast. Store the credentials, provision a user, ack with 200. Anything heavy (email dispatch, external API calls, analytics) should happen in a background job after the ack.

## Making API calls

Bearer-authenticate with the `access_token`:

```bash
curl https://api.teachfloor.com/v0/courses \
  -H "Authorization: Bearer eyJhbGc..."
```

Every request must include the token. The token grants access scoped to:

- **The organization that installed the app** — you can only reach that org's data.
- **The scopes declared in the token** — a token with only `courses:read` can call `GET /v0/courses/*` but gets 403 on `GET /v0/members`.

If a request returns 401, the token has expired or been revoked — refresh it. If it returns 403 `Insufficient scope`, the token doesn't have permission for that endpoint (your manifest didn't declare a mapping permission).

**Node.js:**

```javascript
const fetch = require('node-fetch')

async function getCourses(installation) {
  let accessToken = installation.access_token
  if (Date.parse(installation.expires_at) < Date.now() + 60_000) {
    // Refresh proactively if we're within 60s of expiry.
    ({ access_token: accessToken } = await refreshToken(installation))
  }

  const res = await fetch('https://api.teachfloor.com/v0/courses', {
    headers: { 'Authorization': `Bearer ${accessToken}` },
  })

  if (res.status === 401) {
    // Expired between our check and the request — refresh once and retry.
    const { access_token: fresh } = await refreshToken(installation)
    return fetch('https://api.teachfloor.com/v0/courses', {
      headers: { 'Authorization': `Bearer ${fresh}` },
    }).then(r => r.json())
  }

  return res.json()
}
```

**Python:**

```python
import requests
from datetime import datetime, timezone, timedelta

def get_courses(installation):
    access_token = installation["access_token"]
    expires_at = datetime.fromisoformat(installation["expires_at"])
    if expires_at < datetime.now(timezone.utc) + timedelta(seconds=60):
        access_token = refresh_token(installation)["access_token"]

    r = requests.get(
        "https://api.teachfloor.com/v0/courses",
        headers={"Authorization": f"Bearer {access_token}"},
    )

    if r.status_code == 401:
        access_token = refresh_token(installation)["access_token"]
        r = requests.get(
            "https://api.teachfloor.com/v0/courses",
            headers={"Authorization": f"Bearer {access_token}"},
        )

    return r.json()
```

**PHP:**

```php
function getCourses(array $installation): array {
    $accessToken = $installation['access_token'];
    $expiresAt   = strtotime($installation['expires_at']);
    if ($expiresAt < time() + 60) {
        $accessToken = refreshToken($installation)['access_token'];
    }

    $ch = curl_init('https://api.teachfloor.com/v0/courses');
    curl_setopt($ch, CURLOPT_HTTPHEADER, ["Authorization: Bearer {$accessToken}"]);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    $body = curl_exec($ch);
    $code = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);

    if ($code === 401) {
        $accessToken = refreshToken($installation)['access_token'];
        // …retry…
    }

    return json_decode($body, true);
}
```

## Refreshing tokens

Access tokens expire after **1 hour**. Refresh tokens are valid for **30 days** and rotate on every use — the response contains a fresh `refresh_token` that supersedes the previous one; store it immediately.

```
POST https://api.teachfloor.com/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&refresh_token=<the-refresh-token>
&client_id=<the-client-id>
&client_secret=<the-client-secret>
```

Response:

```json
{
  "token_type": "Bearer",
  "expires_in": 3600,
  "access_token": "eyJhbGc...(new)...",
  "refresh_token": "def50200...(new)..."
}
```

Store the new `access_token`, `refresh_token`, and the new expiry (`now() + expires_in` seconds). The old refresh token is invalidated the moment this request succeeds — using it again returns `invalid_grant`.

**Node.js:**

```javascript
async function refreshToken(installation) {
  const body = new URLSearchParams({
    grant_type:    'refresh_token',
    refresh_token: installation.refresh_token,
    client_id:     installation.client_id,
    client_secret: process.env.TEACHFLOOR_CLIENT_SECRET,   // from your app config
  })

  const res = await fetch('https://api.teachfloor.com/oauth/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body,
  })

  if (!res.ok) throw new Error(`refresh failed: ${res.status}`)

  const { access_token, refresh_token, expires_in } = await res.json()
  const expires_at = new Date(Date.now() + expires_in * 1000).toISOString()

  await updateInstallation(installation.organization_id, {
    access_token, refresh_token, expires_at,
  })

  return { access_token, refresh_token, expires_at }
}
```

**Python:**

```python
def refresh_token(installation):
    r = requests.post(
        "https://api.teachfloor.com/oauth/token",
        data={
            "grant_type":    "refresh_token",
            "refresh_token": installation["refresh_token"],
            "client_id":     installation["client_id"],
            "client_secret": os.environ["TEACHFLOOR_CLIENT_SECRET"],   # from your app config
        },
    )
    r.raise_for_status()
    body = r.json()
    body["expires_at"] = (
        datetime.now(timezone.utc) + timedelta(seconds=body["expires_in"])
    ).isoformat()
    update_installation(installation["organization_id"], body)
    return body
```

**PHP:**

```php
function refreshToken(array $installation): array {
    $ch = curl_init('https://api.teachfloor.com/oauth/token');
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query([
        'grant_type'    => 'refresh_token',
        'refresh_token' => $installation['refresh_token'],
        'client_id'     => $installation['client_id'],
        'client_secret' => getenv('TEACHFLOOR_CLIENT_SECRET'),   // from your app config
    ]));
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    $body = json_decode(curl_exec($ch), true);
    curl_close($ch);

    $body['expires_at'] = gmdate('c', time() + $body['expires_in']);
    updateInstallation($installation['organization_id'], $body);
    return $body;
}
```

**Retry pattern:** on any 401 from a public API call, refresh once and retry. Don't refresh preemptively on every call — every unnecessary refresh rotates the refresh token, which can race with concurrent workers of your own.

## Revocation

**On uninstall** — every access token and refresh token issued for that installation is revoked automatically. The `app.uninstalled` webhook fires just before revocation; use it to purge the installation from your side:

```javascript
async function handleAppUninstalled(event) {
  await deleteInstallation(event.data.organization.id)
}
```

After the uninstall completes, any API call with the old `access_token` returns 401 and any refresh with the old `refresh_token` returns `invalid_grant`. Neither can be recovered — the org would need to reinstall the app to get new tokens.

**On version upgrade** — the old tokens are revoked and the new version's install fires a fresh `app.installed` webhook with new tokens. If the new version's permissions changed, the new tokens carry the new scope set. Treat this the same as any other install event.

**On manifest re-push of the same version** — existing tokens keep working.

## Security

- **Never log or transmit `client_secret`, `access_token`, or `refresh_token` outside your controlled backend.** Redact them from error reports and observability tools.
- **Store tokens encrypted at rest.** If your database is compromised, plaintext tokens are as good as full account access.
- **Verify the webhook signature before trusting the payload.** The `credentials` block only appears in webhooks that pass HMAC-SHA256 signature verification with your app's signing secret — see [Signature verification](/docs/apps/advanced-topics/webhooks#signature-verification).
- **HTTPS only.** Every OAuth call is HTTPS-only. Any redirect URL you use for `post_install_action.url` must be HTTPS.
- **Scope-limit your tokens.** Only declare the permissions your app actually needs. A `courses:read` token can't reach `/v0/members` or any other endpoint outside its declared scopes.

## Next Steps

→ Continue to [Permissions](/docs/apps/advanced-topics/permissions)

## Additional Resources

- [Webhooks](/docs/apps/advanced-topics/webhooks) - The delivery mechanism for the `credentials` block on `app.installed`.
- [Permissions](/docs/apps/advanced-topics/permissions) - The manifest permission catalog; OAuth scopes are derived from a subset of it.
- [App Manifest](/docs/apps/core-concepts/app-manifest) - The full manifest schema, including the `oauth` block.
