# OAuth

Extension apps get two independent OAuth capabilities on a single app registration — pick either or both depending on what your app needs:

- **`install`** — install-integrated OAuth. Every install auto-mints an org-scoped access token + refresh token, delivered to your backend inside the `app.installed` webhook. No browser step; the Install click IS the grant. Use this for background integrations that call the Teachfloor API without any specific user in the loop (nightly sync, event-triggered updates, admin dashboards).
- **`authorize`** — classic OAuth 2.0 authorization-code flow. A user signs in via Teachfloor's `/oauth/authorize`, your app receives a code, exchanges it for tokens. Add the `openid` scope and it becomes full OpenID Connect (id_token identifying the user) — perfect for "Sign in with Teachfloor" flows in your app's UI.

Both flows share the SAME `client_id` + `client_secret`. Configure one, both, or none — the manifest tells Teachfloor which behaviors to enable.

This chapter covers both capabilities: opting in, receiving credentials, making API calls, refreshing tokens, silent sign-in, and revocation.

## What you can build

**With `install`:**
- **Backend integrations that read platform data** — sync course completions to your HR system, mirror members to your CRM, ingest activity streams into your data warehouse.
- **Automated workflows** — poll the API from your backend on your own schedule, augmenting the push-based webhook data.

**With `authorize`:**
- **Sign in with Teachfloor** on your own web UI — users click a button, get redirected to Teachfloor, land back on your app already signed in.
- **Silent sign-in on post-install redirect** — after a user installs your app, drop them onto your setup page already signed in (no re-authentication).

If you only need in-app data access (from within your app's dashboard view), the SDK's [`teachfloor.get(...)`](/docs/apps/core-concepts/extension-kit/integration) helpers already do that. OAuth is for anything OUTSIDE the dashboard — your backend, your own web UI, external integrations.

## How it works

**`install` flow** — When an organization installs your app, Teachfloor mints an access token + refresh token bound to that installation and delivers them inside the `app.installed` webhook payload. Your backend stores the tokens per-installation and uses them as bearer tokens against `https://api.teachfloor.com/v0/*`.

**`authorize` flow** — Your app redirects the user's browser to `https://app.teachfloor.com/oauth/authorize?client_id=...&scope=openid+profile+email&redirect_uri=...`. The user consents (or silent-approves if you use `prompt=none`), Teachfloor redirects to your callback with a code, your backend exchanges the code at `/oauth/token` for tokens including an id_token identifying the user.

One `client_id` + `client_secret` pair identifies your app across every organization that installs it; each install then gets its own `access_token` and `refresh_token`. The `client_id` + `client_secret` are shown to you once at app creation (via the CLI and the Developer dashboard) — store them in your app config; they never appear in webhook payloads.

## Opt in

The `oauth` manifest block declares which capabilities your app wants. Pick either or both:

**Install-integrated (background API access):**

```json
"oauth": { "install": true }
```

**Authorize flow with OIDC (user sign-in):**

```json
"oauth": {
  "authorize": {
    "redirect_uris": [
      "https://my-app.com/oauth/callback",
      "http://localhost:3000/oauth/callback"
    ]
  }
}
```

**Hybrid (both — same app, same client credentials):**

```json
"oauth": {
  "install": true,
  "authorize": {
    "redirect_uris": ["https://my-app.com/oauth/callback"]
  }
}
```

Full manifest example (hybrid):

```json
{
  "id": "my-app",
  "name": "My App",
  "version": "1.0.0",
  "oauth": {
    "install": true,
    "authorize": {
      "redirect_uris": ["https://my-app.com/oauth/callback"]
    }
  },
  "permissions": [
    { "permission": "courses:read", "purpose": "Sync course completions to our HR tool" },
    { "permission": "modules:read", "purpose": "Attribute completions to the right module" },
    { "permission": "openid",       "purpose": "Sign you in to My App using your Teachfloor account" },
    { "permission": "profile",      "purpose": "Show your name on your My App dashboard" },
    { "permission": "email",        "purpose": "Contact you about integration issues" }
  ],
  "webhook": {
    "url": "https://my-app.com/teachfloor/webhook",
    "events": ["course.completed"]
  }
}
```

**Fields:**

| Field | Required | Rules |
|---|---|---|
| `oauth.install` | no | Must be the literal boolean `true` when present. Enables install-integrated tokens delivered via webhook. |
| `oauth.authorize` | no | Object with `redirect_uris` when present. Enables browser code flow at `/oauth/authorize`. |
| `oauth.authorize.redirect_uris` | yes (when `authorize` block present) | Array of one or more `https://` callback URLs. `http://localhost` / `http://127.0.0.1` / `http://[::1]` allowed for local dev per RFC 8252. |

At least one of `install` or `authorize` must be declared when the `oauth` block is present.

### Additional requirements per capability

**For `install`:**
- A `webhook` block — credentials arrive inside the `app.installed` webhook, so if there's no webhook endpoint, there's nowhere to deliver them.
- At least one permission that maps to an OAuth scope (see the [permission mapping table](#permission-scope-mapping)).

**For `authorize`:**
- OIDC identity permissions in your `permissions` array — declare which claims your app is allowed to request. Requests for scopes not in your manifest are rejected with `invalid_scope`. Minimum for basic sign-in: `openid`. Add `profile` for the user's name, `email` for their email address.

## Permission → scope mapping

Adding a permission to the manifest allows your app to request the matching OAuth scope — no separate scope declaration needed. Permissions split into two categories by flow:

**Grantable on `install` tokens** (delivered via `app.installed` webhook, org-scoped, hit the public API):

| Manifest permission | OAuth scope | Grants |
|---|---|---|
| `courses:read` | `courses:read` | GET `/v0/courses/*` |
| `modules:read` | `modules:read` | GET `/v0/modules/*` |
| `elements:read` | `elements:read` | GET `/v0/elements/*` |
| `members:read` | `members:read` | GET `/v0/members/*`, GET `/v0/courses/{id}/members/*` |
| `activities:read` | `activities:read` | GET `/v0/activities/*`, GET `/v0/elements/{id}/activities` |

**Grantable on `authorize` tokens** (delivered via `/oauth/token` code exchange, user-scoped, populate id_token claims):

| Manifest permission | OAuth scope | Delivered claim in id_token |
|---|---|---|
| `openid` | `openid` | `sub` (stable user id), `iss`, `aud`, `iat`, `exp` (base claims — always present when openid is granted) |
| `profile` | `profile` | `name` |
| `email` | `email` | `email`, `email_verified` |

These two sets are mutually exclusive per token flow: install tokens can't carry identity claims; authorize tokens can't reach the public API. If your app needs both, declare `install: true` AND `authorize: {...}` in the manifest and use each flow for its intended purpose.

Other manifest permissions (data storage, realtime, AI, etc.) are SDK-only — they don't map to any OAuth scope and don't appear on issued tokens.

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

---

## Sign in with Teachfloor (OpenID Connect)

If your app declares `oauth.authorize` in its manifest, users can sign in to your app via Teachfloor — same UX as "Sign in with Google" or "Sign in with GitHub." Adding the `openid` scope on the authorize request unlocks OpenID Connect: your callback receives an `id_token` (a signed JWT) identifying the user, in addition to the standard OAuth access + refresh tokens.

The primary use case: users installing your app via Teachfloor's marketplace get dropped onto your setup page already signed in — no re-authentication, no separate account creation.

### Discovery

Teachfloor publishes standard OIDC discovery so any conformant OIDC client library (Node `openid-client`, Python `authlib`, Go `go-oidc`, PHP `league/oauth2-client`, etc.) auto-configures with one URL:

```
https://app.teachfloor.com/.well-known/openid-configuration
```

Public keys for id_token signature verification:

```
https://app.teachfloor.com/.well-known/jwks.json
```

Both endpoints are cacheable (1h TTL); your OIDC client library handles caching automatically.

### The authorize URL

Redirect the user's browser to:

```
https://app.teachfloor.com/oauth/authorize
  ?client_id=<YOUR_CLIENT_ID>
  &response_type=code
  &scope=openid+profile+email
  &redirect_uri=<YOUR_CALLBACK_URL>
  &state=<CSRF_TOKEN>
```

Required params:
- `client_id` — your app's client_id (from `teachfloor apps create` or the Developer dashboard)
- `response_type=code` — authorization-code flow
- `scope` — space-separated list; must include `openid` for OIDC; add `profile`/`email` for additional claims. Any scope not declared in your manifest's `permissions` returns `invalid_scope`
- `redirect_uri` — must match one of the `redirect_uris` declared in your manifest's `oauth.authorize` block exactly
- `state` — CSRF token you generate + verify on the callback

Optional:
- `prompt=none` — silent auth (see [below](#silent-sign-in))
- `organization=<slug>` — pin the flow to a specific organization when the user is a member of multiple orgs where your app is installed

### Handling the callback

Teachfloor redirects the user's browser to your `redirect_uri` with `?code=<AUTH_CODE>&state=<YOUR_STATE>`. Exchange the code for tokens:

```javascript
// Node example using stock fetch — most OIDC libraries do this for you
const response = await fetch('https://app.teachfloor.com/oauth/token', {
  method:  'POST',
  headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
  body:    new URLSearchParams({
    grant_type:    'authorization_code',
    code:          authCode,
    client_id:     process.env.TEACHFLOOR_CLIENT_ID,
    client_secret: process.env.TEACHFLOOR_CLIENT_SECRET,
    redirect_uri:  process.env.TEACHFLOOR_REDIRECT_URI,
  }),
})
const tokens = await response.json()
// tokens contains: access_token, refresh_token, expires_in, token_type, scope, id_token
```

### Verifying the id_token

The `id_token` is a JWT signed with RS256. You MUST verify:
1. Signature — against Teachfloor's public key from the JWKS endpoint
2. `iss` claim equals `https://app.teachfloor.com`
3. `aud` claim equals your `client_id`
4. `exp` claim is in the future

Most OIDC client libraries do all four automatically. Example with Node's `openid-client`:

```javascript
import { Issuer } from 'openid-client'

// Once at startup
const teachfloor = await Issuer.discover('https://app.teachfloor.com')
const client = new teachfloor.Client({
  client_id:     process.env.TEACHFLOOR_CLIENT_ID,
  client_secret: process.env.TEACHFLOOR_CLIENT_SECRET,
  redirect_uris: [process.env.TEACHFLOOR_REDIRECT_URI],
  response_types: ['code'],
})

// In your callback handler
const params    = client.callbackParams(req)
const tokenSet  = await client.callback(process.env.TEACHFLOOR_REDIRECT_URI, params, { state: savedState })
const claims    = tokenSet.claims()
// claims: { sub, iss, aud, iat, exp, email, email_verified, name }
```

The `sub` claim is a stable identifier for the user across sessions — use it as your foreign-key when linking a Teachfloor user to a row in your database. `email` is convenient but users can change their email address; `sub` is guaranteed stable.

### Silent sign-in (`prompt=none`)

For zero-UI silent authentication — e.g. after a fresh install, dropping the user onto your setup page already signed in — add `&prompt=none` to the authorize URL. Behavior per OIDC Core §3.1.2.6:

- User is signed in to Teachfloor + your app is installed for their org → immediate redirect to your callback with `?code=...` (no consent screen shown)
- User is NOT signed in → redirect back with `?error=login_required` — fall back to your normal sign-in form
- Your app is not installed for any of the user's orgs → redirect back with `?error=interaction_required` — direct them to install first
- Requested scopes exceed what's in your manifest → redirect back with `?error=invalid_scope` — a real developer bug on your side

**Handling errors:** any `?error=...` on the callback is a normal failure mode, not an exception. Show a helpful message or redirect to a fallback (magic link, "install first" prompt, etc.) — don't crash.

### The `organization` disambiguation param

If a user belongs to multiple organizations that all have your app installed, the consent screen would normally show a picker. When you know which org the flow is for (e.g. post-install redirect from Teachfloor includes `?organization=<slug>`), forward that slug in your authorize URL:

```
https://app.teachfloor.com/oauth/authorize?...&organization=acme
```

Silent auth pins to that specific org. If the user isn't a member of that org (or the app isn't installed there), you get `interaction_required`.

### Recommended end-to-end pattern for post-install silent sign-in

1. In your manifest, `post_install_action.url` = `https://my-app.com/setup?organization=<slug>` (Teachfloor auto-appends the org slug)
2. On landing at your setup page, check if the user has an existing session on your side — if yes, render the setup UI
3. If no session, redirect the user's browser to `/oauth/authorize?...&prompt=none&organization=<slug>` (forward the org slug from the URL)
4. In your OIDC callback, exchange the code, verify the id_token, mint a session cookie keyed to the user's `sub` claim, redirect back to your setup page
5. On subsequent renders, the session cookie exists — go straight to the setup UI

Total user-visible latency for the silent auth: ~500ms-1s (comparable to "Sign in with Google" when you're already logged in).

### Failure fallback

Users who reach your setup page from a shared link or bookmark (no active Teachfloor session in this browser) will hit `error=login_required`. Provide a fallback flow — magic link, password login, or a "Sign in with Teachfloor" button that runs the same authorize URL WITHOUT `prompt=none` (which then shows the interactive consent screen).

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
- **HTTPS only.** Every OAuth call is HTTPS-only. Any redirect URL you use for `post_install_action.url` or `oauth.authorize.redirect_uris` must be HTTPS (loopback `http://localhost` allowed for local dev only).
- **Scope-limit your tokens.** Only declare the permissions your app actually needs. A `courses:read` token can't reach `/v0/members` or any other endpoint outside its declared scopes.

**For `authorize` (OIDC) flows specifically:**

- **Always verify the id_token signature** against Teachfloor's JWKS. Never trust an id_token's claims without signature verification — use a standard OIDC client library that enforces this by default.
- **Always verify `iss`, `aud`, and `exp`** on incoming id_tokens. An attacker who obtained an id_token intended for a DIFFERENT app should not be able to sign in to yours.
- **Always validate the `state` parameter** on the callback. Reject callbacks whose `state` doesn't match what you generated when starting the flow. This is the CSRF defense for the OAuth code flow.
- **Never accept access tokens from the browser as evidence of identity.** Only the id_token, verified server-side, is trusted identity. Access tokens from the browser can be replayed or stolen.
- **Use short-lived session cookies keyed to `sub`.** Don't store the id_token in a cookie or in localStorage — turn it into your app's own session immediately, then discard.

## Next Steps

→ Continue to [Permissions](/docs/apps/advanced-topics/permissions)

## Additional Resources

- [Webhooks](/docs/apps/advanced-topics/webhooks) - The delivery mechanism for the `credentials` block on `app.installed`.
- [Permissions](/docs/apps/advanced-topics/permissions) - The manifest permission catalog; OAuth scopes are derived from a subset of it.
- [App Manifest](/docs/apps/core-concepts/app-manifest) - The full manifest schema, including the `oauth` block.
