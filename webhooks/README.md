# Teachfloor Webhooks

Real-time event notifications from the Teachfloor platform.

## What Are Webhooks?

Teachfloor uses webhooks to notify your application in real-time whenever specific events occur within your account. Instead of continuously polling the API for updates, webhooks push notifications directly to your system as events happen.

## How Webhooks Work

When a subscribed event occurs, Teachfloor sends an HTTP POST request to your designated webhook endpoint with a JSON payload containing detailed information about the event.

### Common Use Cases

- **Sync your database** - Keep records updated with course enrollments, completions
- **Send notifications** - Alert users about important events
- **Trigger workflows** - Automate processes based on platform events

## Quick Links

- [Getting Started](./getting-started) - Set up your first webhook
- [Security](./security) - Verify webhook signatures
- [Delivery & Retries](./delivery-retries) - Understanding webhook reliability
- [Event Reference](./event-reference) - Available webhook events
- [Troubleshooting](./troubleshooting) - Common issues and solutions

## Need Help?

If you have questions or need assistance, contact support through your Teachfloor dashboard.
