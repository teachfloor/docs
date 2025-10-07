# Webhooks

Real-time event notifications from the Teachfloor platform.

## What Are Webhooks?

Teachfloor uses webhooks to notify your application in real-time whenever specific events occur within your account. Instead of continuously polling the API for updates, webhooks push notifications directly to your system as events happen, ensuring timely and efficient processing.

## How Webhooks Work

When a subscribed event occurs, Teachfloor sends an HTTP POST request to your designated webhook endpoint with a JSON payload containing detailed information about the event.

### Common Use Cases

- **Update your database** - Keep records in sync with real-time changes
- **Send notifications** - Alert users or administrators about important events
- **Trigger workflows** - Initiate automated processes like member onboarding or content updates

## Quick Links

- [Getting Started](./01-getting-started.md) - Set up your first webhook
- [Security](./02-security.md) - Verify webhook signatures
- [Delivery & Retries](./03-delivery-retries.md) - Understanding webhook reliability
- [Event Reference](./04-event-reference.md) - Available webhook events

## Documentation Structure

This documentation is organized into focused sections:

1. **Getting Started** - Setup and configuration
2. **Security** - Signature verification and best practices
3. **Delivery & Retries** - Reliability and error handling
4. **Event Reference** - Complete list of available events

## Need Help?

If you have questions or need assistance:
- Check our [troubleshooting guide](./03-delivery-retries.md#troubleshooting)
- Contact support through your Teachfloor dashboard
