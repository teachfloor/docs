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

- [Authentication](./api/authentication) - Get your API key
- [Rate Limiting](./api/rate-limiting) - Usage limits
- [Errors](./api/errors) - Error handling
- API Reference - Interactive documentation at [docs.teachfloor.com](https://docs.teachfloor.com/api-reference)

## Getting Started

1. Generate an API key from your Teachfloor dashboard
2. Include it in the `Authorization` header
3. Make requests to the API endpoints

```bash
curl https://api.teachfloor.com/v0/members \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## Available Resources

- **Courses**, **Modules** and **Elements** - Create, read, update, delete and search course content; enroll and revoke course members
- **Members** - Create, update, search and revoke organization members; send invites
- **Activities** - Retrieve completion and submission data per member, course, module or element
- **Custom Fields** - Read the field definitions and write values for members and course members
- **Uploads** - Upload files to use in course content

See the full [API Reference](https://docs.teachfloor.com/api-reference) for complete endpoint documentation.

## Need Help?

Contact support through your Teachfloor dashboard.
