# Permissions

Permissions control what your app can access on the Teachfloor platform.

## Overview

Permissions control two types of access:
1. **Contextual Data**: Data the platform includes in event callbacks (course, module, element)
2. **Storage & Features**: Platform features your app can use (data storage, AI generation)

All permissions must be declared in your app manifest with a user-facing explanation.

## How Permissions Work

When your app subscribes to events, the platform includes an `objectContext` parameter containing contextual data based on your granted permissions. For example, with `course_read` permission, `objectContext.course` includes the current course data when in a course viewport.

<scalar-callout type="info">See [Integration Guide - Events](./core-concepts/extension-kit/integration#events) for detailed `objectContext` structure and usage examples.</scalar-callout>

## Available Permissions

Your app can request the following permissions:

### Contextual Data Permissions

| Permission | Description | Access Type |
|-----------|-------------|-------------|
| `user_read` | Access user profile information | User data in objectContext |
| `user_events_read` | Access user activity events | User events |
| `course_read` | Access course contextual data | Course object in objectContext |
| `module_read` | Access module contextual data | Module object in objectContext |
| `element_read` | Access element contextual data | Element object in objectContext |

### Storage Permissions

| Permission | Description | Access Type | Hierarchy |
|-----------|-------------|-------------|-----------|
| `appdata_read` | Read organization-wide app data | Storage API - App Data | Read only |
| `appdata_write` | Write organization-wide app data | Storage API - App Data | **Includes read** |
| `userdata_read` | Read user-specific app data | Storage API - User Data | Read only |
| `userdata_write` | Write user-specific app data | Storage API - User Data | **Includes read** |
| `usercollection_read` | Read user data collections | Storage API - Collections | Read only |
| `usercollection_write` | Write to user data collections | Storage API - Collections | **Includes read** |

<scalar-callout type="info">**Important**: Write permissions (`*_write`) automatically grant read access. Requesting `*_write` is sufficient for both reading and writing.</scalar-callout>

### AI & Feature Permissions

| Permission | Description | Access Type |
|-----------|-------------|-------------|
| `ai_text_generate` | Generate text using AI models | AI Generation API |
| `ai_context_external_send` | Send platform data to AI models | AI Context Sharing |

## Permission Details

### User Permissions

#### `user_read`
Access basic user profile information.

**Data available in objectContext**:
- User ID
- Full name
- Email address
- Avatar URL
- Language preference
- Timezone

**Use cases**:
- Personalization
- User greetings
- Profile displays

**Example**:
```json
{
  "permission": "user_read",
  "purpose": "Display personalized greetings and user information"
}
```

#### `user_events_read`
Access user activity and learning events.

**Data available in objectContext**:
- Course enrollments
- Module completions
- Element interactions
- Login history
- Activity timestamps

**Use cases**:
- Progress tracking
- Analytics dashboards
- Activity feeds
- Engagement metrics

**Example**:
```json
{
  "permission": "user_events_read",
  "purpose": "Track learning progress and generate activity reports"
}
```

### Contextual Permissions

These permissions control what data appears in the `objectContext` parameter based on the current viewport.

#### `course_read`
Access course information when user is in a course viewport.

**Data available in objectContext.course**:
- Course ID
- Course title
- Description
- Status
- Enrollment data
- Course settings

**Available in viewports**:
- `teachfloor.dashboard.course.detail`
- `teachfloor.dashboard.course.module.detail`
- `teachfloor.dashboard.course.element.detail`

**Example**:
```json
{
  "permission": "course_read",
  "purpose": "Display course information in notes and widgets"
}
```

**Usage**:
```javascript
import { subscribeToEvent } from '@teachfloor/extension-kit'

subscribeToEvent('environment.viewport.changed', (viewport, objectContext) => {
  if (objectContext.course) {
    console.log('Course title:', objectContext.course.name)
    console.log('Course ID:', objectContext.course.id)
  }
})
```

#### `module_read`
Access module information when user is viewing a module.

**Data available in objectContext.module**:
- Module ID
- Module title
- Description
- Order/sequence
- Completion status

**Available in viewports**:
- `teachfloor.dashboard.course.module.detail`
- `teachfloor.dashboard.course.element.detail`

**Example**:
```json
{
  "permission": "module_read",
  "purpose": "Show module progress and navigation"
}
```

**Usage**:
```javascript
import { subscribeToEvent } from '@teachfloor/extension-kit'

subscribeToEvent('environment.viewport.changed', (viewport, objectContext) => {
  if (objectContext.module) {
    console.log('Module title:', objectContext.module.name)
    console.log('Module order:', objectContext.module.position)
  }
})
```

#### `element_read`
Access learning element information when user is viewing an element.

**Data available in objectContext.element**:
- Element ID
- Element type (video, assignment, quiz, etc.)
- Title and description
- Content metadata
- Completion status

**Available in viewports**:
- `teachfloor.dashboard.course.element.detail`

**Example**:
```json
{
  "permission": "element_read",
  "purpose": "Display element information and add notes to content"
}
```

**Usage**:
```javascript
import { subscribeToEvent } from '@teachfloor/extension-kit'

subscribeToEvent('environment.viewport.changed', (viewport, objectContext) => {
  if (objectContext.element) {
    console.log('Element type:', objectContext.element.type)
    console.log('Element title:', objectContext.element.name)
    console.log('Completed:', objectContext.element.completed)
  }
})
```

### Storage Permissions

Storage permissions allow your app to persist data on the Teachfloor platform. See [Data Storage](./advanced-topics/data-storage) for detailed usage.

#### `appdata_read` & `appdata_write`

Store and retrieve organization-wide app data shared across all users.

<scalar-callout type="info">**Permission Hierarchy**: `appdata_write` includes `appdata_read` access.</scalar-callout>

**Use cases**:
- App configuration
- Global settings
- Shared templates
- Feature flags

**Example (Read and Write)**:
```json
{
  "permissions": [
    {
      "permission": "appdata_write",
      "purpose": "Save and load app configuration and shared settings"
    }
  ]
}
```

**Example (Read-Only)**:
```json
{
  "permissions": [
    {
      "permission": "appdata_read",
      "purpose": "Load app configuration and shared settings"
    }
  ]
}
```

**Usage**:
```javascript
import { store, retrieve } from '@teachfloor/extension-kit'

// Write app data
await store('config', { theme: 'dark', lang: 'en' }, 'appdata')

// Read app data
const config = await retrieve('config', 'appdata')
```

#### `userdata_read` & `userdata_write`

Store and retrieve user-specific data.

**Permission Hierarchy**: `userdata_write` includes `userdata_read` access.

**Use cases**:
- User preferences
- Personal settings
- User state
- Draft content

**Example (Read and Write)**:
```json
{
  "permissions": [
    {
      "permission": "userdata_write",
      "purpose": "Save and load your personal preferences and settings"
    }
  ]
}
```

**Example (Read-Only)**:
```json
{
  "permissions": [
    {
      "permission": "userdata_read",
      "purpose": "Load your personal preferences and settings"
    }
  ]
}
```

**Usage**:
```javascript
import { store, retrieve } from '@teachfloor/extension-kit'

// Write user data
await store('preferences', { theme: 'light', fontSize: 14 }, 'userdata')

// Read user data
const prefs = await retrieve('preferences', 'userdata')
```

#### `usercollection_read` & `usercollection_write`

Store and retrieve collections of data items for a user, with pagination support.

<scalar-callout type="info">**Permission Hierarchy**: `usercollection_write` includes `usercollection_read` access.</scalar-callout>

**Use cases**:
- Activity logs
- User notes or annotations
- Saved items
- History data

**Example (Read and Write)**:
```json
{
  "permissions": [
    {
      "permission": "usercollection_write",
      "purpose": "Save and load your notes and activity history"
    }
  ]
}
```

**Example (Read-Only)**:
```json
{
  "permissions": [
    {
      "permission": "usercollection_read",
      "purpose": "Load your saved notes and activity history"
    }
  ]
}
```

**Usage**:
```javascript
import { store, retrieve } from '@teachfloor/extension-kit'

// Add item to collection
await store('notes', {
  title: 'My Note',
  content: 'Content...',
  createdAt: Date.now()
}, 'usercollection')

// Retrieve collection with pagination
const result = await retrieve('notes?limit=10', 'usercollection')
console.log(result.data) // Array of items
console.log(result.next) // Next cursor
```

### AI Permissions

#### `ai_text_generate`

Generate text using AI language models.

**Use cases**:
- Content generation
- Text completion
- Summarization
- Translation

**Example**:
```json
{
  "permission": "ai_text_generate",
  "purpose": "Generate content suggestions and summaries"
}
```

**Usage**:
```javascript
import { generate } from '@teachfloor/extension-kit'

// Generate text using AI
const result = await generate(
  'Write a summary of this course',
  'ai/text-generate'
)

console.log(result) // Generated text response
```

#### `ai_context_external_send`

Permission to use platform data placeholders in AI prompts.

**How it works**: This permission is **only checked when you use placeholders** like `{{course.name}}` or `{{module.content}}` in your AI prompts. Without this permission, you can still use `generate()` with regular prompts.

**Supported Placeholders**:
- `{{course.name}}` - Course title
- `{{course.content}}` - Course content (text format)
- `{{module.name}}` - Module title
- `{{module.content}}` - Module content (text format)
- `{{element.name}}` - Element title
- `{{element.content}}` - Element content (text format)

<scalar-callout type="warning">**Important**: When using placeholders, you must also have the corresponding read permission:
- Course placeholders require `course_read`
- Module placeholders require `module_read`
- Element placeholders require `element_read`</scalar-callout>

**Example**:
```json
{
  "permissions": [
    {
      "permission": "ai_text_generate",
      "purpose": "Generate content suggestions"
    },
    {
      "permission": "course_read",
      "purpose": "Access course information"
    },
    {
      "permission": "ai_context_external_send",
      "purpose": "Include course content in AI prompts"
    }
  ]
}
```

**Usage**:
```javascript
import { generate } from '@teachfloor/extension-kit'

// Without placeholders - only needs ai_text_generate
const simpleResult = await generate('Write a motivational message')

// With placeholders - needs ai_text_generate + ai_context_external_send + course_read
const contextResult = await generate(
  'Summarize this course: {{course.content}}'
)

// Multiple placeholders
const detailedResult = await generate(
  'Create a quiz about {{module.name}} covering: {{module.content}}'
)
```

## Permission Management

### Adding Permissions

#### Using CLI

```bash
teachfloor apps grant permission
```

Select permission and enter purpose when prompted.

#### Manual Addition

Edit `teachfloor-app.json`:

```json
{
  "permissions": [
    {
      "permission": "course_read",
      "purpose": "Display course information in widgets"
    },
    {
      "permission": "user_events_read",
      "purpose": "Track your learning progress"
    }
  ]
}
```

### Removing Permissions

#### Using CLI

```bash
teachfloor apps revoke permission
```

#### Manual Removal

Remove from manifest:

```json
{
  "permissions": [
    // Remove the permission object you no longer need
  ]
}
```

### Permission Purposes

Each permission must have a clear, user-facing explanation.

**Good purposes**:
- "Display course information in your notes" ✓
- "Show your current module progress" ✓
- "Track your learning progress for analytics" ✓

**Poor purposes**:
- "Access data" ✗ (too vague)
- "Platform integration" ✗ (not user-facing)
- "Required for functionality" ✗ (not specific)

## Using Permissions

Always check if contextual data exists before accessing it:

```javascript
import { subscribeToEvent } from '@teachfloor/extension-kit'

subscribeToEvent('environment.viewport.changed', (viewport, objectContext) => {
  // Check before accessing
  if (objectContext.course) {
    console.log('Course:', objectContext.course.name)
  }

  // Use optional chaining
  const moduleTitle = objectContext.module?.title || 'No module'
})
```

<scalar-callout type="info">See [Integration Guide - Events](./core-concepts/extension-kit/integration#events) for complete `objectContext` usage examples.</scalar-callout>

## Next Steps

→ Continue to [Deployment](./advanced-topics/deployment)

## Additional Resources

- [Integration Guide](./core-concepts/extension-kit/integration) - Using permissions with events and storage
- [Best Practices](./references/best-practices) - Permission best practices and patterns
- [Examples](./references/examples) - Complete permission usage examples
