# Authentication
To access the Teachfloor API, authentication is required for all endpoints. Authentication is handled via the Authorization header, where you must include your unique API Key. The format for this header is:

```plaintext
Authorization: Bearer {YOUR_API_KEY}
```

This API Key acts as a secure token that grants your application access to the API. It’s essential to keep your API Key secure and not share it publicly, as it allows full access to your Teachfloor account through the API.

To generate and retrieve your API Key, follow these steps:

- Log in to your Teachfloor account
- Navigate to the Settings page
- Go to the Integrations section
- Click on "Generate API Key"
  - A new API Key will be created specifically for your account
  - Copy this key and store it securely

You can regenerate your API Key at any time if you believe it has been compromised or if you simply need to refresh it. Remember that regenerating the key will invalidate the previous one, so update any applications or services that use the API with the new key.