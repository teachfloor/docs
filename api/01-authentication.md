# Authentication

All API requests require authentication using an API key.

## How It Works

Include your API key in the `Authorization` header using Bearer authentication:

```
Authorization: Bearer YOUR_API_KEY
```

## Getting Your API Key

1. Log in to your Teachfloor account
2. Navigate to **Settings** → **Integrations**
3. Click **Create API Key** and give it a name, such as the service that will use it
4. Copy the key from the confirmation dialog and store it securely

The key is shown **once**. Afterwards the Integrations page lists only its name, a masked form (`tf_key_abc123…wxyz`) and when it was last used. You can create as many keys as you need, one per integration, and revoke any of them without affecting the others.

Keys created before named keys were introduced keep working unchanged.

**Security**: Never share your API key publicly or commit it to version control.

A request with a missing or unknown key is answered with `403 Forbidden` and the standard [error envelope](./errors). A valid key on a plan without the Public API is answered with `402 Payment Required`.

## Example Request

```bash
curl https://api.teachfloor.com/v0/members \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json"
```

```javascript
const response = await fetch('https://api.teachfloor.com/v0/members', {
  headers: {
    'Authorization': 'Bearer YOUR_API_KEY',
    'Content-Type': 'application/json'
  }
});
```

```php
$curl = curl_init();
curl_setopt_array($curl, [
    CURLOPT_URL => "https://api.teachfloor.com/v0/members",
    CURLOPT_HTTPHEADER => [
        "Authorization: Bearer YOUR_API_KEY",
        "Content-Type: application/json"
    ],
]);
$response = curl_exec($curl);
```

## Rotating a Key

To replace a key without touching the others:

1. Go to **Settings** → **Integrations**
2. Open the key's menu and click **Rotate**
3. Copy the new key from the confirmation dialog

**Warning**: Rotating revokes the previous secret immediately. Update the application using it before or right after rotating. If you need an overlap window, create a second key, move the application to it, then revoke the first one.

To stop a key for good, choose **Revoke** from the same menu. Revoked keys are refused with `403 Forbidden` on their next request.

## Best Practices

- Store API keys in environment variables
- Never hardcode keys in your application
- Rotate immediately if compromised
- Use one key per integration so a leak is contained and `last used` tells you which key is live

## Next Steps

- [Rate Limiting](./rate-limiting) - Understand usage limits
- [Errors](./errors) - Handle API errors
