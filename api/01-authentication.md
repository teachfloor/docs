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
3. Click **Generate API Key**
4. Copy and store the key securely

**Security**: Never share your API key publicly or commit it to version control.

## Example Request

<scalar-tabs nested>
  <scalar-tab title="curl">
```bash
curl https://api.teachfloor.com/v0/members \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json"
```
  </scalar-tab>
  <scalar-tab title="JavaScript">
```javascript
const response = await fetch('https://api.teachfloor.com/v0/members', {
  headers: {
    'Authorization': 'Bearer YOUR_API_KEY',
    'Content-Type': 'application/json'
  }
});
```
  </scalar-tab>
  <scalar-tab title="PHP">
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
  </scalar-tab>
</scalar-tabs>

## Regenerating Your Key

You can regenerate your API key at any time:

1. Go to **Settings** → **Integrations**
2. Click **Regenerate API Key**

**Warning**: Regenerating invalidates the previous key. Update all applications using the old key.

## Best Practices

- Store API keys in environment variables
- Never hardcode keys in your application
- Regenerate immediately if compromised

## Next Steps

- [Rate Limiting](./rate-limiting) - Understand usage limits
- [Errors](./errors) - Handle API errors
