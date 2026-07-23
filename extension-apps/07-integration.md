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
- `openModal()`, `closeModal()` - Promote a widget into a modal container (widget surface only)
- `goToViewport()` - Navigate to different platform areas using viewports
- `goToPath()` - Navigate to different platform areas using paths

### React Hooks
- `useExtensionContext()` - Access user and platform data reactively
- `useLaunchState()` - Read the app-provided launch state passed by `openModal({ state })`

### Quick Reference

| What you need | Where to find it |
|---------------|------------------|
| Events, Storage, Navigation, Toasts | **This document** (06-integration.md) |
| Buttons, Forms, Layouts, Charts | [Extension Kit Components](./components) |

:::info
**Need UI components?** For buttons, inputs, layouts, and charts, see the [Extension Kit Components](./components) guide.
:::

:::info
All imports come from `@teachfloor/extension-kit`, which is automatically installed when you create an app.
:::

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

:::caution
The `objectContext` parameter only includes data for permissions declared in your app manifest.
:::

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

:::caution
Objects are only present when the permission is granted and the viewport context matches.
:::

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

:::info
See [Permissions](/docs/apps/advanced-topics/permissions) for details on requesting access to course, module, and element data.
:::

### Data Storage

The Extension Kit provides `store()`, `retrieve()`, and `createCollection()` functions for persisting data. Three types of storage are available: app data (organization-wide), user data (user-specific), and user collections (paginated lists).

:::info
See [Data Storage](/docs/apps/advanced-topics/data-storage) for complete documentation, API reference, and usage examples.
:::

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
    surface: 'drawer' | 'page' | 'widget'
    presentation: 'default' | 'modal'  // 'modal' when the widget was opened via openModal()
  }

  // App-provided payload passed to openModal({ state }). Set once at
  // mount and does not update.
  // null when no launch state was provided.
  state: object | null
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
showToast('Changes saved successfully', { color: 'green' })

// Error message
showToast('Failed to save changes', { color: 'red' })

// Info message
showToast('Processing your request', { color: 'blue' })

// Warning message
showToast('Please review your input', { color: 'orange' })
```

**Options:**
```typescript
interface ToastOptions {
  color?: 'green' | 'red' | 'blue' | 'orange'
  autoClose?: number  // milliseconds
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

### Modal Control

Widget-surface views can promote themselves into a modal for more room — useful for detail views, forms, or wizards triggered from a compact dashboard widget. The widget always opens ITSELF in the modal (never a different widget), and the modal iframe runs the same component with a fresh mount.

```javascript
import { openModal, closeModal, useExtensionContext } from '@teachfloor/extension-kit'

function MyWidget() {
  const { environment } = useExtensionContext()
  const isModal = environment.presentation === 'modal'

  // Compact rendering in the widget slot, full rendering in the modal
  if (!isModal) {
    return <button onClick={() => openModal({ size: 'lg' })}>Expand</button>
  }

  return (
    <div>
      <FullDetailView />
      <button onClick={() => closeModal()}>Done</button>
    </div>
  )
}
```

#### `openModal(options)`

Available on the **widget surface only**. Calls from drawer or page surfaces are silently ignored.

```typescript
interface ModalOptions {
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl' | '100%'  // default 'md'
  closeOnClickOutside?: boolean                       // default true
  closeOnEscape?: boolean                             // default true
  state?: object                                      // arbitrary launch state (see below)
}
```

Anything outside the allowed values is dropped by the host — you can't pass arbitrary strings through to the underlying modal.

#### `closeModal()`

Dismiss the modal from within its own iframe. Only meaningful when `environment.presentation === 'modal'`. Typical use: form submit success, wizard finish. Calls from a normally-placed widget/drawer/page are no-ops.

#### Launch state — `openModal({ state })` + `useLaunchState()`

Passing `state` in `openModal` hands an arbitrary object to the modal-hosted iframe. The child reads it via `useLaunchState()` (or `useExtensionContext().state`). Set once at mount, does not update. Use it for deep-linking, initial form values, or opener context.

```javascript
import { openModal, useLaunchState } from '@teachfloor/extension-kit'

// In the widget slot:
function CompactWidget() {
  return (
    <button onClick={() => openModal({
      size: 'lg',
      state: { noteId: 'abc-123', mode: 'edit' },
    })}>
      Edit note
    </button>
  )
}

// In the same widget rendered in the modal:
function ModalView() {
  const launchState = useLaunchState()  // { noteId: 'abc-123', mode: 'edit' } — or null

  if (!launchState?.noteId) return <NewNoteForm />
  return <EditNoteForm noteId={launchState.noteId} mode={launchState.mode} />
}
```

`useLaunchState()` returns `null` when the view wasn't launched with state (i.e. rendered directly in its widget slot, not opened via `openModal`).

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

### Deeplinking

Use `goToPath` when you already have a fully-resolved in-app path — typically captured from `environment.path` — and want to deeplink the user back to it (e.g. a saved bookmark or a "back to where you were" button).

```javascript
import { goToPath, useExtensionContext } from '@teachfloor/extension-kit'

// Save the current path...
const { environment } = useExtensionContext()
const savedPath = environment.path  // e.g. "/org-slug/courses/123/modules/456"

// ...and deeplink back later
goToPath(savedPath)
```

**Rules and guards:**

- `path` must be a relative path beginning with a single `/` (no protocol-relative `//…`, no absolute URLs). Anything else is ignored.
- The path must belong to the **current organization** — deeplinks whose first segment doesn't match the user's org slug are rejected, so an app installed in one org can't redirect the user into another.
- On custom domains, the org slug is stripped from the URL automatically — paths captured from `environment.path` work on both URL shapes.

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

:::info
See [Permissions](/docs/apps/advanced-topics/permissions#ai-permissions) for more details on AI permissions.
:::

## Next Steps

→ Continue to [Data Storage](/docs/apps/advanced-topics/data-storage)

## Additional Resources

- [Extension Kit Components](./components) - UI components reference
- [Permissions](/docs/apps/advanced-topics/permissions) - Permission scopes and usage
- [Best Practices](/docs/apps/references/best-practices) - Integration patterns and error handling
- [Examples](/docs/apps/references/examples) - Integration examples
