# Data Storage

The Extension Kit provides data storage capabilities for persisting app data on the Teachfloor platform. Data is automatically scoped by organization and app, with optional user-level scoping.

## Overview

Three types of storage are available:

| Storage Type | Scope | Use Case |
|-------------|-------|----------|
| **App Data** | Organization + App | Shared settings, configurations |
| **User Data** | Organization + App + User | User-specific preferences, state |
| **User Collection** | Organization + App + User | Lists, activity logs, history |

<scalar-callout type="warning">Each storage type requires appropriate read/write permissions. Write permissions automatically include read access. See [Permissions Reference](./permissions) for details.</scalar-callout>

**Security**: All data is automatically encrypted at rest on the Teachfloor platform.

## App Data Storage

Store data shared across all users in your organization.

### Permissions Required

**Read and Write**:
```json
{
  "permissions": [
    {
      "permission": "appdata_write",
      "purpose": "Save and load app configuration and settings"
    }
  ]
}
```

<scalar-callout type="info">`appdata_write` automatically includes read access.</scalar-callout>

**Read-Only** (if you only need to read):
```json
{
  "permissions": [
    {
      "permission": "appdata_read",
      "purpose": "Load app configuration and settings"
    }
  ]
}
```

### Usage

```javascript
import { store, retrieve } from '@teachfloor/extension-kit'

// Store app-wide data
await store('settings', {
  theme: 'dark',
  language: 'en',
  notifications: true
}, 'appdata')

// Retrieve app-wide data
const settings = await retrieve('settings', 'appdata')
console.log(settings.theme) // 'dark'
```

### Use Cases

- Global app configuration
- Organization-wide settings
- Shared templates or presets
- Feature flags
- API keys (encrypted)

### Example: App Configuration

```javascript
import { store, retrieve, showToast } from '@teachfloor/extension-kit'

// Save configuration
async function saveAppConfig(config) {
  try {
    await store('app-config', config, 'appdata')
    showToast('Configuration saved', { color: 'green' })
  } catch (error) {
    console.error('Failed to save:', error)
    showToast('Failed to save configuration', { color: 'red' })
  }
}

// Load configuration
async function loadAppConfig() {
  try {
    const config = await retrieve('app-config', 'appdata')
    return config || getDefaultConfig()
  } catch (error) {
    console.error('Failed to load:', error)
    return getDefaultConfig()
  }
}

// Usage
await saveAppConfig({
  apiEndpoint: 'https://api.example.com',
  maxRetries: 3,
  timeout: 5000
})

const config = await loadAppConfig()
```

## User Data Storage

Store data specific to individual users.

### Permissions Required

**Read and Write**:
```json
{
  "permissions": [
    {
      "permission": "userdata_write",
      "purpose": "Save and load your personal preferences and app data"
    }
  ]
}
```

<scalar-callout type="info">`userdata_write` includes read access.</scalar-callout>

**Read-Only**:
```json
{
  "permissions": [
    {
      "permission": "userdata_read",
      "purpose": "Load your personal preferences and app data"
    }
  ]
}
```

### Usage

```javascript
import { store, retrieve } from '@teachfloor/extension-kit'

// Store user-specific data
await store('preferences', {
  theme: 'light',
  fontSize: 14,
  sidebarCollapsed: false
}, 'userdata')

// Retrieve user-specific data
const prefs = await retrieve('preferences', 'userdata')
console.log(prefs.theme) // 'light'
```

### Use Cases

- User preferences
- Personal settings
- User state (last viewed page, filters)
- User-specific configurations
- Draft content

### Example: User Preferences

```javascript
import { store, retrieve } from '@teachfloor/extension-kit'

class PreferencesManager {
  constructor() {
    this.defaults = {
      theme: 'light',
      fontSize: 14,
      notifications: true,
      autoSave: true
    }
  }

  async load() {
    try {
      const prefs = await retrieve('user-preferences', 'userdata')
      return { ...this.defaults, ...prefs }
    } catch (error) {
      console.error('Failed to load preferences:', error)
      return this.defaults
    }
  }

  async save(preferences) {
    try {
      await store('user-preferences', preferences, 'userdata')
      return true
    } catch (error) {
      console.error('Failed to save preferences:', error)
      return false
    }
  }

  async update(key, value) {
    const prefs = await this.load()
    prefs[key] = value
    return this.save(prefs)
  }
}

// Usage
const prefsManager = new PreferencesManager()

// Load preferences
const prefs = await prefsManager.load()

// Update a preference
await prefsManager.update('theme', 'dark')

// Save all preferences
await prefsManager.save({
  theme: 'dark',
  fontSize: 16,
  notifications: false
})
```

## User Collection Storage

Store lists of data items for a user, with pagination support.

### Permissions Required

**Read and Write**:
```json
{
  "permissions": [
    {
      "permission": "usercollection_write",
      "purpose": "Save and load your activity history and saved items"
    }
  ]
}
```

**Note**: `usercollection_write` includes read access.

**Read-Only**:
```json
{
  "permissions": [
    {
      "permission": "usercollection_read",
      "purpose": "Load your activity history and saved items"
    }
  ]
}
```

### Usage

Collections allow you to store multiple items under the same key and retrieve them with pagination.

```javascript
import { createCollection } from '@teachfloor/extension-kit'

// Create a collection manager
const notes = createCollection('user-notes', { limit: 15 })

// Add items to the collection
await notes.add({
  title: 'My Note',
  content: 'Note content',
  createdAt: Date.now()
})

await notes.add({
  title: 'Another Note',
  content: 'More content',
  createdAt: Date.now()
})

// List items (first page)
const page1 = await notes.list()
console.log(page1.items)      // Array of collection records
console.log(page1.items[0].value) // Your actual data
console.log(page1.items[0].id)    // Database record ID
console.log(page1.hasMore)    // true if more pages exist
console.log(page1.nextCursor) // Cursor for next page

// Load next page
if (page1.hasMore) {
  const page2 = await notes.list({ cursor: page1.nextCursor })
}

// Update an existing item
const itemId = page1.items[0].id
await notes.update(itemId, {
  title: 'Updated Title',
  content: 'Updated content',
  updatedAt: Date.now()
})

// Remove an item
await notes.remove(itemId)

// Get all items (auto-pagination)
const allNotes = await notes.getAll()
```

### Pagination

```javascript
import { createCollection } from '@teachfloor/extension-kit'

const messages = createCollection('chat-messages', { limit: 20 })

// Manual pagination
const page1 = await messages.list()
console.log(page1.items)      // First 20 collection records
console.log(page1.items[0].value) // First item's data
console.log(page1.hasMore)    // true if more exist

// Load next page
if (page1.hasMore) {
  const page2 = await messages.list({ cursor: page1.nextCursor })
}

// Auto-pagination (get all items)
const allMessages = await messages.getAll()
```


### Use Cases

- Activity logs
- User notes or annotations
- Saved items or bookmarks
- History or timeline data
- Multi-entry forms

### Example: Chat Messages

Complete example using the Collection Manager API:

```javascript
import { createCollection, showToast } from '@teachfloor/extension-kit'
import { useState, useEffect } from 'react'

function ChatApp() {
  const [messages, setMessages] = useState([])
  const [isLoading, setIsLoading] = useState(false)
  const [nextCursor, setNextCursor] = useState(null)

  // Create collection manager
  const chatMessages = createCollection('chat-messages', { limit: 15 })

  // Load initial messages
  useEffect(() => {
    loadMessages()
  }, [])

  const loadMessages = async (cursor = null) => {
    setIsLoading(true)

    try {
      const page = await chatMessages.list({ cursor })

      // Extract the .value from each item
      const items = page.items.map(item => item.value)
      setMessages(prev => cursor ? [...prev, ...items] : items)
      setNextCursor(page.hasMore ? page.nextCursor : null)
    } catch (error) {
      console.error('Failed to load messages:', error)
      showToast('Failed to load messages', { type: 'error' })
    } finally {
      setIsLoading(false)
    }
  }

  const sendMessage = async (text) => {
    try {
      const message = {
        role: 'user',
        text,
        timestamp: Date.now()
      }

      await chatMessages.add(message)

      // Optimistically add to UI
      setMessages(prev => [message, ...prev])

      showToast('Message sent', { type: 'success' })
    } catch (error) {
      console.error('Failed to send message:', error)
      showToast('Failed to send message', { type: 'error' })
    }
  }

  const editMessage = async (itemId, newText) => {
    try {
      const page = await chatMessages.list()
      const item = page.items.find(i => i.id === itemId)

      if (item) {
        await chatMessages.update(itemId, {
          ...item.value,
          text: newText,
          edited: true,
          editedAt: Date.now()
        })

        // Update in UI
        setMessages(prev => prev.map(m =>
          m.id === itemId ? { ...item.value, text: newText, edited: true } : m
        ))

        showToast('Message updated', { type: 'success' })
      }
    } catch (error) {
      console.error('Failed to update message:', error)
      showToast('Failed to update message', { type: 'error' })
    }
  }

  const deleteMessage = async (itemId) => {
    try {
      await chatMessages.remove(itemId)

      // Remove from UI
      setMessages(prev => prev.filter(m => m.id !== itemId))

      showToast('Message deleted', { type: 'success' })
    } catch (error) {
      console.error('Failed to delete message:', error)
      showToast('Failed to delete message', { type: 'error' })
    }
  }

  return (
    <div>
      {messages.map((msg, i) => (
        <div key={i}>{msg.text}</div>
      ))}

      {nextCursor && (
        <button onClick={() => loadMessages(nextCursor)}>
          Load More
        </button>
      )}
    </div>
  )
}
```

### Updating Collection Items

You can update existing collection items by their ID:

```javascript
import { createCollection, showToast } from '@teachfloor/extension-kit'

const notes = createCollection('user-notes')

async function updateNote(noteId, updates) {
  try {
    // Get the current item
    const page = await notes.list()
    const note = page.items.find(item => item.id === noteId)

    if (!note) {
      showToast('Note not found', { type: 'error' })
      return
    }

    // Update with merged data
    await notes.update(noteId, {
      ...note.value,
      ...updates,
      updatedAt: Date.now()
    })

    showToast('Note updated successfully', { type: 'success' })
  } catch (error) {
    console.error('Update failed:', error)
    showToast('Failed to update note', { type: 'error' })
  }
}

// Usage
await updateNote('123', {
  title: 'Updated Title',
  content: 'Updated content'
})
```

**Important**:
- Requires `usercollection_write` permission
- Item ID comes from `item.id` when listing items
- Update replaces the entire value - merge with existing data if needed
- Returns the updated value

### Removing Collection Items

You can delete collection items by their ID:

```javascript
import { createCollection, showToast } from '@teachfloor/extension-kit'

const bookmarks = createCollection('saved-bookmarks')

async function removeBookmark(bookmarkId) {
  try {
    await bookmarks.remove(bookmarkId)
    showToast('Bookmark removed', { type: 'success' })
    return true
  } catch (error) {
    console.error('Delete failed:', error)
    showToast('Failed to remove bookmark', { type: 'error' })
    return false
  }
}

// Usage
const page = await bookmarks.list()
const itemToDelete = page.items[0]
await removeBookmark(itemToDelete.id)
```

**Important**:
- Requires `usercollection_write` permission
- Item ID comes from `item.id` when listing items
- Delete operations are permanent
- Returns `null` on success

### Complete CRUD Example

Here's a complete example showing create, read, update, and delete operations:

```javascript
import { createCollection, showToast } from '@teachfloor/extension-kit'
import { useState, useEffect } from 'react'

function NotesManager() {
  const [notes, setNotes] = useState([])
  const notesCollection = createCollection('user-notes', { limit: 20 })

  // Create
  const addNote = async (title, content) => {
    try {
      await notesCollection.add({
        title,
        content,
        createdAt: Date.now()
      })

      await loadNotes() // Refresh list
      showToast('Note added', { type: 'success' })
    } catch (error) {
      showToast('Failed to add note', { type: 'error' })
    }
  }

  // Read
  const loadNotes = async () => {
    try {
      const page = await notesCollection.list()
      setNotes(page.items)
    } catch (error) {
      showToast('Failed to load notes', { type: 'error' })
    }
  }

  // Update
  const updateNote = async (noteId, updates) => {
    try {
      const note = notes.find(n => n.id === noteId)

      await notesCollection.update(noteId, {
        ...note.value,
        ...updates,
        updatedAt: Date.now()
      })

      await loadNotes() // Refresh list
      showToast('Note updated', { type: 'success' })
    } catch (error) {
      showToast('Failed to update note', { type: 'error' })
    }
  }

  // Delete
  const deleteNote = async (noteId) => {
    try {
      await notesCollection.remove(noteId)
      setNotes(prev => prev.filter(n => n.id !== noteId))
      showToast('Note deleted', { type: 'success' })
    } catch (error) {
      showToast('Failed to delete note', { type: 'error' })
    }
  }

  useEffect(() => {
    loadNotes()
  }, [])

  return (
    <div>
      {notes.map(note => (
        <div key={note.id}>
          <h3>{note.value.title}</h3>
          <p>{note.value.content}</p>
          <button onClick={() => updateNote(note.id, { title: 'New Title' })}>
            Edit
          </button>
          <button onClick={() => deleteNote(note.id)}>
            Delete
          </button>
        </div>
      ))}
      <button onClick={() => addNote('New Note', 'Content')}>
        Add Note
      </button>
    </div>
  )
}
```

## Data Types

All storage methods automatically handle serialization:

### Supported Types

```javascript
// String
await store('name', 'John Doe', 'userdata')

// Number
await store('count', 42, 'userdata')

// Boolean
await store('enabled', true, 'userdata')

// Object
await store('settings', { theme: 'dark', lang: 'en' }, 'userdata')

// Array
await store('items', [1, 2, 3, 4, 5], 'userdata')

// Nested Objects
await store('config', {
  ui: { theme: 'dark' },
  features: { beta: true },
  limits: { max: 100 }
}, 'appdata')
```

### Type Handling

```javascript
// Data is automatically serialized and deserialized
const settings = await retrieve('settings', 'userdata')

// No need to JSON.parse - objects are returned as objects
console.log(settings.theme) // Direct property access

// Arrays remain arrays
const items = await retrieve('items', 'userdata')
items.forEach(item => console.log(item))
```

## Security

All data stored through the Extension Kit is **automatically encrypted at rest** on the Teachfloor platform.

**Safe to store**:
- User preferences and settings
- App configurations
- UI state and draft content
- Non-sensitive user data
- Cached public data
- API keys for third-party services

**Do not store**:
- User passwords
- Credit card numbers or payment information
- Social security numbers or national IDs
- Private encryption keys
- Data belonging to other users

## Next Steps

→ Continue to [Permissions](/apps/advanced-topics/permissions)

## Additional Resources

- [Best Practices](/apps/references/best-practices) - Storage patterns, error handling, caching, and performance
- [Permissions](/apps/advanced-topics/permissions) - Storage permission requirements
- [Extension Kit Integration](/apps/core-concepts/extension-kit/integration)
