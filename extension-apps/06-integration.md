# Extension Kit Integration

This guide covers platform integration functions provided by the Teachfloor Extension Kit (`@teachfloor/extension-kit`).

## What This Document Covers

This document focuses exclusively on **API functions** for platform integration:

### Core Functions
- `initialize()` - Signal that your app is ready
- `subscribeToEvent()` - Listen to platform events
- `store()`, `retrieve()`, `createCollection()` - Data storage and retrieval
- `showToast()` - Display notifications
- `showDrawer()`, `hideDrawer()`, `toggleDrawer()` - Control app drawer
- `goToViewport()` - Navigate to different platform areas

### React Hooks
- `useExtensionContext()` - Access user and platform data reactively

### Quick Reference

| What you need | Where to find it |
|---------------|------------------|
| Events, Storage, Navigation, Toasts | **This document** (06-integration.md) |
| Buttons, Forms, Layouts, Charts | [Extension Kit Components](./core-concepts/extension-kit/components) |

<scalar-callout type="info">**Need UI components?** For buttons, inputs, layouts, and charts, see the [Extension Kit Components](./core-concepts/extension-kit/components) guide.</scalar-callout>

<scalar-callout type="info">All imports come from `@teachfloor/extension-kit`, which is automatically installed when you create an app.</scalar-callout>

## Signaling App Ready

### Basic Usage

Signal to the platform that your app is ready:

```javascript
import { initialize } from '@teachfloor/extension-kit'

function App() {
  useEffect(() => {
    initialize() // Signal app is ready
  }, [])

  return <div>My App</div>
}
```

### With Context Provider

```javascript
import { ExtensionContextProvider } from '@teachfloor/extension-kit'

// The provider signals readiness automatically
<ExtensionContextProvider autoInit={true}>
  <App />
</ExtensionContextProvider>
```

## API Reference

### Events

#### Subscribe to Events

Listen to platform events using `subscribeToEvent()`. Events receive two parameters: the event data and an `objectContext` containing contextual information based on your app's permissions.

```javascript
import { subscribeToEvent } from '@teachfloor/extension-kit'

// Viewport changes - objectContext includes course/module/element based on permissions
subscribeToEvent('environment.viewport.changed', (viewport, objectContext) => {
  console.log('User navigated to:', viewport)

  if (objectContext.course) {
    console.log('Course:', objectContext.course.name)
  }
  if (objectContext.module) {
    console.log('Module:', objectContext.module.name)
  }
  if (objectContext.element) {
    console.log('Element:', objectContext.element.name)
  }
})

// Path changes - single parameter
subscribeToEvent('environment.path.changed', (path) => {
  console.log('Path changed:', path)
})

// User events - objectContext includes contextual data
subscribeToEvent('auth.user.event', (eventData, objectContext) => {
  console.log('User event:', eventData.type)

  if (objectContext.course) {
    console.log('Happened in course:', objectContext.course.name)
  }
})
```

**Available Events:**

| Event | Parameters | Description |
|-------|-----------|-------------|
| `environment.viewport.changed` | `(viewport, objectContext)` | User navigated to a different viewport. `objectContext` contains course/module/element data based on permissions. |
| `environment.path.changed` | `(path)` | URL path changed |
| `auth.user.event` | `(eventData, objectContext)` | User activity event (login, element_completed, quiz_submitted, etc.). `objectContext` contains contextual data. |

<scalar-callout type="warning">The `objectContext` parameter only includes data for permissions declared in your app manifest.</scalar-callout>

**objectContext Structure:**
```typescript
interface ObjectContext {
  course?: {
    id: string
    object: 'course'
    name: string
    created_at: string
    cover: string | null
    availability: string
    visibility: string
    currency: string
    price: number | null
    metadata: object
  }
  module?: {
    id: string
    object: 'module'
    name: string
    created_at: string
    cover: string | null
    type: string
    position: number
    metadata: object
  }
  element?: {
    id: string
    object: 'element'
    name: string
    created_at: string
    cover: string | null
    type: string
    position: number
    metadata: object
  }
}
```

<scalar-callout type="warning">Objects are only present when the permission is granted and the viewport context matches.</scalar-callout>

**How it works:**

Objects are included based on two conditions:
1. **Permission granted**: Your app must have the corresponding permission declared in the manifest
2. **Relevant context**: The current viewport/path must be within that context

When both conditions are met, the **complete object** is included. Otherwise, the key is **not present** in the object.

**Example:**
```javascript
// With course_read permission on a course detail page:
{
  course: { id: "abc123", name: "Introduction to React", /* ...other course fields */ }
  // module and element keys not present (not in their context)
}

// Without course_read permission on a course detail page:
{
  // No keys present (no permissions granted)
}

// With course_read and module_read on a module detail page:
{
  course: { id: "abc123", name: "Introduction to React", /* ...other course fields */ },
  module: { id: "def456", name: "Getting Started", /* ...other module fields */ }
  // element key not present (not in element context)
}
```

<scalar-callout type="info">See [Permissions](./advanced-topics/permissions) for details on requesting access to course, module, and element data.</scalar-callout>

### Data Storage

The Extension Kit provides `store()`, `retrieve()`, and `createCollection()` functions for persisting data. Three types of storage are available: app data (organization-wide), user data (user-specific), and user collections (paginated lists).

<scalar-callout type="info">See [Data Storage](./advanced-topics/data-storage) for complete documentation, API reference, and usage examples.</scalar-callout>

## Context API

### Accessing Context

```javascript
import { useExtensionContext } from '@teachfloor/extension-kit'

function MyComponent() {
  const { userContext, appContext, environment } = useExtensionContext()

  return (
    <div>
      <p>User: {userContext.full_name}</p>
      <p>App: {appContext.name}</p>
      <p>Viewport: {environment.viewport}</p>
    </div>
  )
}
```

### Context Structure

```typescript
interface Context {
  userContext: {
    id: string
    created_at: string
    full_name: string
    email: string
    avatar: string
    language: string
    timezone: string
    identity_provider: {       // SSO/Identity provider details (null if not configured)
      provider: string         // Provider name (e.g., 'auth0', 'saml', 'neoncrm')
      user_id: string | null   // User ID in the identity provider
      user_metadata: object    // Additional user metadata from provider
    } | null
  }

  appContext: {
    id: string
    name: string
    version: string        // Current installed version
    permissions: string[]  // Array of granted permission scopes
    views: array           // Array of app view configurations from manifest
  }

  environment: {
    initialized: boolean
    viewport: string
    path: string
  }
}
```

### Reactive Context

Context updates automatically when user data changes:

```javascript
function UserGreeting() {
  const { userContext } = useExtensionContext()

  // Re-renders when userContext changes
  return <h1>Hello, {userContext.full_name}!</h1>
}
```

## UI Integration

### Toast Notifications

```javascript
import { showToast } from '@teachfloor/extension-kit'

// Success message
showToast('Changes saved successfully', { type: 'success' })

// Error message
showToast('Failed to save changes', { type: 'error' })

// Info message
showToast('Processing your request', { type: 'info' })

// Warning message
showToast('Please review your input', { type: 'warning' })
```

**Options:**
```typescript
interface ToastOptions {
  type?: 'success' | 'error' | 'info' | 'warning'
  duration?: number  // milliseconds
}
```

### Drawer Control

```javascript
import { showDrawer, hideDrawer, toggleDrawer } from '@teachfloor/extension-kit'

// Show drawer
function handleOpen() {
  showDrawer()
}

// Hide drawer
function handleClose() {
  hideDrawer()
}

// Toggle drawer
function handleToggle() {
  toggleDrawer()
}

// Auto-show on mount
useEffect(() => {
  showDrawer()
  return () => hideDrawer()
}, [])
```

### Navigation

```javascript
import { goToViewport } from '@teachfloor/extension-kit'

// Navigate to courses
function goToCourses() {
  goToViewport('teachfloor.dashboard.course.list')
}

// Navigate to settings
function goToSettings() {
  goToViewport('teachfloor.dashboard.settings.general.detail')
}

// Navigate to account
function goToAccount() {
  goToViewport('teachfloor.dashboard.account.detail')
}
```

## AI Generation

### Text Generation

Generate text using AI models with the platform's built-in AI capabilities.

```javascript
import { generate } from '@teachfloor/extension-kit'

// Generate text
async function generateContent() {
  try {
    const result = await generate(
      'Write a summary of this course',
      'ai/text-generate'
    )

    console.log(result) // Generated text response
    return result
  } catch (error) {
    console.error('Generation failed:', error)
  }
}
```

**Parameters:**
- `prompt` (string, required): The prompt to send to the AI model. Can include placeholders like `{{course.content}}`
- `generationType` (string, optional): Type of generation (default: `'ai/text-generate'`)

**Available Generation Types:**
- `'ai/text-generate'`: General text generation

**Permissions Required:**
- `ai_text_generate`: Always required to use AI generation
- `ai_context_external_send`: Only required when using placeholders
- Contextual permissions: Required for corresponding placeholders (`course_read` for course placeholders, etc.)

### Using Placeholders

Include platform data directly in prompts using placeholders:

```javascript
import { generate } from '@teachfloor/extension-kit'

// Course placeholders (requires: ai_text_generate + course_read + ai_context_external_send)
const courseSummary = await generate(
  'Summarize this course: {{course.content}}'
)

// Module placeholders (requires: ai_text_generate + module_read + ai_context_external_send)
const moduleQuiz = await generate(
  'Create a 5-question quiz about {{module.name}}: {{module.content}}'
)

// Element placeholders (requires: ai_text_generate + element_read + ai_context_external_send)
const studyNotes = await generate(
  'Generate study notes for {{element.name}}: {{element.content}}'
)
```

**Supported Placeholders:**
- `{{course.name}}` - Course title
- `{{course.content}}` - Course content (text format)
- `{{module.name}}` - Module title
- `{{module.content}}` - Module content (text format)
- `{{element.name}}` - Element title
- `{{element.content}}` - Element content (text format)

### Without Placeholders

You can also manually include context data (only requires `ai_text_generate`):

```javascript
import { generate, useExtensionContext } from '@teachfloor/extension-kit'

function SummarizeButton() {
  const { objectContext } = useExtensionContext()

  const handleSummarize = async () => {
    if (!objectContext.course) return

    // Manually including context - no placeholders
    const prompt = `Summarize this course:
      Name: ${objectContext.course.name}
      ID: ${objectContext.course.id}
    `

    const summary = await generate(prompt)
    console.log(summary)
  }

  return <button onClick={handleSummarize}>Generate Summary</button>
}
```

**Error Handling:**
```javascript
async function safeGenerate(prompt) {
  try {
    const result = await generate(prompt)
    return { success: true, data: result }
  } catch (error) {
    if (error.message === 'Teachfloor is not available') {
      return { success: false, error: 'Platform unavailable' }
    }
    return { success: false, error: error.message }
  }
}
```

<scalar-callout type="info">See [Permissions](./advanced-topics/permissions#ai-permissions) for more details on AI permissions.</scalar-callout>

## Next Steps

→ Continue to [Data Storage](./advanced-topics/data-storage)

## Additional Resources

- [Extension Kit Components](./core-concepts/extension-kit/components) - UI components reference
- [Permissions](./advanced-topics/permissions) - Permission scopes and usage
- [Best Practices](./references/best-practices) - Integration patterns and error handling
- [Examples](./references/examples) - Integration examples
