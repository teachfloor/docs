# Data Storage

The Extension Kit provides data storage capabilities for persisting app data on the Teachfloor platform. Data is automatically scoped by organization and app, with optional user-level scoping.

## Overview

Three types of storage are available:

| Storage Type | Scope | Use Case |
|-------------|-------|----------|
| **App Data** | Organization + App | Shared settings, configurations |
| **User Data** | Organization + App + User | User-specific preferences, state |
| **User Collection** | Organization + App + User | Lists, activity logs, history |

Two API styles are available on top of App Data and User Data:

- **Raw primitives** (`store` / `retrieve`) — one row per key, described in the App Data / User Data sections below.
- **Storage Manager** (`createStorage`, **recommended**) — a namespaced wrapper adding TTL and `query()` for paged filter/sort iteration across many rows. See [Storage Manager](#storage-manager-recommended).

For per-key append semantics (many rows sharing a key, id-based CRUD), use [User Collection Storage](#user-collection-storage) instead.

:::caution
Each storage type requires appropriate read/write permissions. Write permissions automatically include read access. See [Permissions Reference](./permissions) for details.
:::

**Security**: All data is automatically encrypted at rest on the Teachfloor platform.

## App Data Storage

Store data shared across all users in your organization.

### Permissions Required

**Read and Write**:
```json
{
  "permissions": [
    {
      "permission": "appdata:write",
      "purpose": "Save and load app configuration and settings"
    }
  ]
}
```

:::info
`appdata:write` automatically includes read access.
:::

**Read-Only** (if you only need to read):
```json
{
  "permissions": [
    {
      "permission": "appdata:read",
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
      "permission": "userdata:write",
      "purpose": "Save and load your personal preferences and app data"
    }
  ]
}
```

:::info
`userdata:write` includes read access.
:::

**Read-Only**:
```json
{
  "permissions": [
    {
      "permission": "userdata:read",
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

## Storage Manager (recommended)

The Storage Manager (`createStorage`) is the recommended way to work with App Data and User Data. It's a thin wrapper over `store` / `retrieve` that adds three things:

1. **Namespaced keys** — every operation is scoped under a `baseKey` prefix so different features of your app can hold their own storage instances without key collisions.
2. **TTL / expiry** — pass `{ ttl: seconds }` on `set()` to auto-expire values.
3. **`query()`** — paged iteration across many rows in the namespace with a small filter + sort DSL. Available in kit `1.29.0+`.

### Permissions Required

Same as App Data / User Data — the Storage Manager doesn't add its own permissions. Pass `{ source: 'appdata' }` for org-shared storage or `{ source: 'userdata' }` for per-user storage.

```json
{
  "permissions": [
    {
      "permission": "userdata:write",
      "purpose": "Save and load user notes"
    }
  ]
}
```

### Basic Usage

```javascript
import { createStorage } from '@teachfloor/extension-kit'

// Per-user lesson notes, all keys automatically prefixed with 'lesson-notes:'
const notes = createStorage('lesson-notes', { source: 'userdata' })

// set → row key: 'lesson-notes:lesson-42-note-1'
await notes.set('lesson-42-note-1', {
  content: 'Mitochondria produce ATP via oxidative phosphorylation',
  lesson_id: 'lesson-42',
  tag: 'lecture',
})

// get by sub-key (namespace stripped from returned key)
const note = await notes.get('lesson-42-note-1')
// → { content: 'Mitochondria produce ATP via oxidative phosphorylation',
//     lesson_id: 'lesson-42', tag: 'lecture' }

// TTL — auto-expires after 3600 seconds
// (e.g. flag a lesson as "recently viewed" for one hour)
await notes.set('recently-viewed:lesson-42', true, { ttl: 3600 })

// remove
await notes.remove('lesson-42-note-1')
```

The kit prepends the `baseKey` to every operation, so callers only ever see un-namespaced sub-keys. Different `createStorage(...)` instances can't accidentally reach each other's data.

### Query — paged filter + sort

`query({ where, sort, limit, after })` returns `{ items, nextCursor }` where each item is `{ key, value, created_at, updated_at }`. `key` is the sub-key (namespace stripped).

```javascript
// Simplest — first page of the namespace, newest first
const page = await notes.query()
// → {
//     items: [
//       { key: 'lesson-73-note-2', value: {...}, created_at: '...', updated_at: '...' },
//       { key: 'lesson-73-note-1', value: {...}, created_at: '...', updated_at: '...' },
//       { key: 'lesson-42-note-1', value: {...}, created_at: '...', updated_at: '...' },
//       ...
//     ],
//     nextCursor: '...' | null
//   }

// Pagination — resume with `after: nextCursor`
let cursor = null
do {
  const p = await notes.query({ limit: 20, after: cursor })
  render(p.items)
  cursor = p.nextCursor
} while (cursor)
```

#### DSL

Filters run against **metadata columns only** — the `value` column is encrypted at rest and can't be predicated on. Expired rows are always excluded (no escape).

**Field / op matrix:**

| Field | Operators |
|---|---|
| `key` | `=`, `!=`, `in`, `not in`, `contains`, `not contains` |
| `created_at`, `updated_at` | `>`, `>=`, `<`, `<=` |

**Sort fields:** `updated_at`, `created_at`. **Sort directions:** `asc`, `desc`. Default when omitted: `[['updated_at', 'desc']]`.

**Predicates** are `[field, op, value]` tuples. Multiple tuples in a `where` array AND together implicitly. For OR, wrap in a `{ or: [...] }` group; for explicit AND groups, use `{ and: [...] }`. Groups nest arbitrarily.

```javascript
// All notes for lesson 42 (key sub-namespace)
await notes.query({
  where: [['key', 'contains', 'lesson-42-']],
})

// Batch fetch by known ids (max 100 values per in / not in)
await notes.query({
  where: [['key', 'in', ['lesson-42-note-1', 'lesson-73-note-2']]],
})

// Recent notes only, oldest first — e.g. review what you took this week
await notes.query({
  where: [['updated_at', '>=', '2026-07-01']],
  sort: [['updated_at', 'asc']],
})

// Combined AND — notes for lesson 42, updated since July 1
await notes.query({
  where: [
    ['key', 'contains', 'lesson-42-'],
    ['updated_at', '>=', '2026-07-01'],
  ],
})

// Nested OR — recent notes from module 3 OR any pinned exam-prep card
await notes.query({
  where: [
    { or: [
      { and: [
        ['key', 'contains', 'module-3-'],
        ['updated_at', '>=', '2026-07-01'],
      ]},
      ['key', 'in', ['exam-prep:cell-biology', 'exam-prep:genetics']],
    ]},
  ],
  sort: [['updated_at', 'desc']],
  limit: 20,
})
```

For `key` exact-match ops (`=`, `!=`, `in`, `not in`), values are treated as sub-keys and namespaced automatically — you write un-namespaced sub-keys, matching `get()` / `set()` semantics. `contains` / `not contains` values pass through as raw substrings and search only within the current namespace.

#### Constraints and errors

- **`in` / `not in` cap** — max 100 values per predicate. Larger arrays throw `storage.query: "in" cannot accept more than 100 values (got N)`. Split into multiple pages instead.
- **Result size cap** — server hard cap is 200 rows per page regardless of `limit`.
- **Fail-loud validation** — unsupported fields or operators throw at the call site before any RPC. Example: `where: [['value', '=', 'x']]` throws `storage.query: unsupported field "value"`.
- **Cursor is opaque** — pass the exact string back in `after`. Don't decode / hand-craft.
- **Cursor is tied to the sort order it was minted with** — if you change `sort` between pages, the cursor becomes semantically wrong (no error, but rows may be skipped or duplicated).

### Example: Lesson notes with load-more search

A learner's notes browser — filter by lesson (`lesson-42-`, `lesson-73-`, …), paginate through everything.

```jsx
import React, { useEffect, useState } from 'react'
import { createStorage } from '@teachfloor/extension-kit'

const notes = createStorage('lesson-notes', { source: 'userdata' })

function LessonNotesList() {
  const [items, setItems] = useState([])
  const [cursor, setCursor] = useState(null)
  const [lessonFilter, setLessonFilter] = useState('')  // e.g. 'lesson-42-'

  const loadMore = async (reset = false) => {
    const page = await notes.query({
      where: lessonFilter ? [['key', 'contains', lessonFilter]] : [],
      limit: 20,
      after: reset ? null : cursor,
    })
    setItems(reset ? page.items : [...items, ...page.items])
    setCursor(page.nextCursor)
  }

  useEffect(() => { loadMore(true) }, [lessonFilter])

  return (
    <>
      <input
        value={lessonFilter}
        placeholder="Filter by lesson id (e.g. lesson-42-)"
        onChange={(e) => setLessonFilter(e.target.value)}
      />
      {items.map((r) => <NoteCard key={r.key} note={r.value} />)}
      {cursor && <button onClick={() => loadMore()}>Load more</button>}
    </>
  )
}
```

Storage Manager vs raw `store` / `retrieve`: use raw primitives only for a small, well-known set of keys (like a single `'config'` blob). If you're storing multiple items and might want to enumerate or filter them, use Storage Manager from the start.

## User Collection Storage

Store lists of data items for a user, with pagination support.

### Permissions Required

**Read and Write**:
```json
{
  "permissions": [
    {
      "permission": "usercollection:write",
      "purpose": "Save and load your activity history and saved items"
    }
  ]
}
```

**Note**: `usercollection:write` includes read access.

**Read-Only**:
```json
{
  "permissions": [
    {
      "permission": "usercollection:read",
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
- Requires `usercollection:write` permission
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
- Requires `usercollection:write` permission
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

→ Continue to [Realtime Channels](/docs/apps/advanced-topics/realtime)

## Additional Resources

- [Best Practices](/docs/apps/references/best-practices) - Storage patterns, error handling, caching, and performance
- [Permissions](/docs/apps/advanced-topics/permissions) - Storage permission requirements
- [Extension Kit Integration](/docs/apps/core-concepts/extension-kit/integration)
