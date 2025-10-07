# Teachfloor API

REST API for integrating with the Teachfloor platform.

## Overview

The Teachfloor Public API provides programmatic access to your organization's data, enabling you to build integrations, automate workflows, and extend the platform's functionality.

**Base URL:** `https://api.teachfloor.com`

## Key Features

- **RESTful design** - Standard HTTP methods (GET, POST, PUT, DELETE)
- **JSON responses** - Easy to parse and work with
- **Resource-oriented** - Predictable URL structure
- **Comprehensive** - Access courses, members, activities, and more

## Quick Links

- [Authentication](./01-authentication.md) - Get your API key
- [Rate Limiting](./02-rate-limiting.md) - Usage limits
- [Errors](./03-errors.md) - Error handling
- API Reference - Interactive documentation at [docs.teachfloor.com](https://docs.teachfloor.com)

## Getting Started

1. Generate an API key from your Teachfloor dashboard
2. Include it in the `Authorization` header
3. Make requests to the API endpoints

```bash
curl https://api.teachfloor.com/v0/members \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## Available Resources

- **Activities** - Retrieve activity completion data
- **Courses** - Manage courses and enrollments
- **Members** - Access member information
- **Custom Fields** - Work with custom field data

See the full [API Reference](https://docs.teachfloor.com) for complete endpoint documentation.

## Need Help?

Contact support through your Teachfloor dashboard.
