# Introduction to Teachfloor Apps

## Overview

Teachfloor Apps are powerful extensions that allow you to customize and enhance the Teachfloor learning platform. Built using modern web technologies, apps integrate seamlessly into the platform's interface while maintaining security and performance.

## What Can You Build?

### UI Extensions
Add custom interface components throughout the Teachfloor dashboard:
- Sidebar widgets for quick access
- Course-specific tools
- Custom settings pages
- Analytics dashboards
- Productivity tools

### Data Applications
Create apps that store and manage data:
- Note-taking apps
- Task managers
- Progress trackers
- Custom reports

### Backend Integrations (No UI Required)
Build apps that run in the background without any user interface:
- Event tracking to analytics platforms (Segment, Mixpanel)
- User activity sync to CRM systems (Intercom, HubSpot)
- Learning data export to data warehouses
- Automated notifications and webhooks
- Third-party API integrations

### Full-Stack Integrations
Combine UI and backend capabilities:
- Communication tools (Slack, Discord)
- AI assistants with chat interfaces
- Custom analytics with dashboards
- Content recommendation engines

## Architecture

### Three Core Components

```
┌─────────────────────────────────────────────────────┐
│                  Teachfloor CLI                      │
│  Command-line tool for app development & deployment │
└─────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────┐
│               Extension Kit Library                 │
│    React components & utilities for building UIs    │
└─────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────┐
│               Teachfloor Platform                   │
│    Hosts apps, manages permissions, serves SDK      │
└─────────────────────────────────────────────────────┘
```

### How Apps Run

Apps are rendered as sandboxed iframes within the Teachfloor dashboard:

1. **Isolation**: Each app runs in its own iframe for security
2. **Communication**: Apps communicate with the platform via PostMessage RPC
3. **Context**: Platform provides user, organization, and environment data
4. **Storage**: Secure data persistence through the Extension Kit

```
┌──────────────────────────────────────────────┐
│       Teachfloor Dashboard (Parent)          │
│  ┌────────────────────────────────────────┐  │
│  │     Your App (Sandboxed iframe)        │  │
│  │                                        │  │
│  │  ┌──────────────────────────────┐      │  │
│  │  │   React Components           │      │  │
│  │  │   (Extension Kit)            │      │  │
│  │  └──────────────────────────────┘      │  │
│  │              ↕                         │  │
│  │  ┌──────────────────────────────┐      │  │
│  │  │   Teachfloor SDK             │      │  │
│  │  │   (PostMessage RPC)          │      │  │
│  │  └──────────────────────────────┘      │  │
│  └────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
```

## Key Concepts

### App Manifest

The manifest is a JSON configuration file that defines your app:

```json
{
  "id": "unique-app-id",
  "version": "1.0.0",
  "name": "My Awesome App",
  "description": "Enhance your workflow",
  "distribution_type": "public",
  "ui_extension": {
    "views": [
      {
        "viewport": "teachfloor.dashboard.course.list",
        "component": "CourseListView"
      }
    ]
  },
  "permissions": [
    {
      "permission": "course_read",
      "purpose": "Display course information"
    }
  ]
}
```

### Viewports

Viewports define where your app can display within the platform. Each viewport corresponds to a specific page or section, such as course pages, settings, community areas, and more.

See [Viewports System](./04-viewports.md) for the complete list of available viewports and their usage.

### Extension Context

Apps receive real-time context from the platform:

```javascript
{
  userContext: {
    id: "user-123",
    full_name: "John Doe",
    email: "john@example.com",
    avatar: "https://...",
    language: "en",
    timezone: "America/New_York"
  },
  appContext: {
    id: "app-456",
    name: "My Awesome App"
  },
  environment: {
    initialized: true,
    viewport: "teachfloor.dashboard.course.list",
    path: "/org/courses"
  }
}
```

### Permissions

Apps must request permissions to access platform resources:

- `user_read`: Read user information
- `course_read`: Access course data
- `module_read`: Access module content
- `element_read`: Access learning elements
- `user_events_read`: Track user activity

## Development Workflow

```
1. Install CLI
   ↓
2. Authenticate
   ↓
3. Create App
   ↓
4. Add Views & Components
   ↓
5. Request Permissions
   ↓
6. Test Locally
   ↓
7. Build & Upload
   ↓
8. Submit for Review
   ↓
9. Publish to Marketplace
```

## Distribution Models

### Private Apps
- Only visible to your organization
- Perfect for internal tools
- No review required for use
- Instant deployment

### Public Apps
- Listed in the Teachfloor Marketplace
- Available to all organizations
- Requires review and approval
- Potential for monetization

## Technology Stack

### Required
- **React 18+**: UI framework
- **JavaScript/TypeScript**: Programming languages
- **Webpack**: Module bundler (configured by CLI)

### Provided
- **Extension Kit**: Pre-built components library
- **Teachfloor SDK**: Platform integration API
- **CLI Tools**: Development and deployment utilities

### Optional
- Any React-compatible libraries
- State management (Redux, Zustand, etc.)
- Styling solutions (Emotion, Tailwind, etc.)
- Additional build tools

## Security & Privacy

### Sandboxing
- Apps run in isolated iframes
- Cannot access parent page directly
- All communication through SDK

### Permissions
- Explicit permission declarations in manifest
- Granular access control
- Transparent to users during installation

### Data Storage
- Scoped to app and organization
- Encrypted at rest
- GDPR compliant

### Review Process
- Code quality checks
- Security vulnerability scanning
- Privacy policy requirements
- Terms of service compliance

## Limitations & Constraints

### Technical
- Maximum 6 apps per organization
- 10 MB bundle size limit (recommended)
- 1000 API calls per minute (rate limiting)
- 10 MB data storage per user

### Platform
- Apps cannot modify core platform UI
- Limited to defined viewports
- Cannot execute server-side code
- No direct database access

## Next Steps

Now that you understand the basics, let's build your first app!

→ Continue to [Quickstart Guide](./02-quickstart.md)

## Additional Resources

- [Extension Kit Integration](./06-integration.md)
