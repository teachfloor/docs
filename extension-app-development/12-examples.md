# Examples

Practical examples and code snippets for common Teachfloor app patterns.

## Table of Contents

- [Backend Integration (No UI)](#backend-integration-no-ui)
- [Simple Widget](#simple-widget)
- [Note-Taking App](#note-taking-app)
- [Progress Tracker](#progress-tracker)
- [Settings Page](#settings-page)
- [Data Visualization](#data-visualization)
- [External API Integration](#external-api-integration)

---

## Backend Integration (No UI)

Apps can run without any UI, listening to platform events and integrating with external services.

### Use Case: Intercom Event Tracking

Track all user learning events to Intercom for customer engagement and analytics.

### Manifest

```json
{
  "id": "intercom-integration",
  "version": "1.0.0",
  "name": "Intercom Integration",
  "description": "Track user learning events in Intercom",
  "distribution_type": "private",
  "permissions": [
    {
      "permission": "user_read",
      "purpose": "Send user profile to Intercom"
    },
    {
      "permission": "user_events_read",
      "purpose": "Track learning activities in Intercom"
    },
    {
      "permission": "course_read",
      "purpose": "Include course context in event tracking"
    }
  ]
}
```

**Note**: No `ui_extension` field - this app runs entirely in the background.

### App Code

```javascript
// src/index.js
import { initialize, subscribeToEvent, useExtensionContext } from '@teachfloor/extension-kit'

// Configuration
const INTERCOM_API_URL = 'https://api.intercom.io/events'
const INTERCOM_TOKEN = 'YOUR_INTERCOM_TOKEN' // In production, store securely

// Initialize the app
initialize()

// Listen to all user events
subscribeToEvent('auth.user.event', async (eventData, objectContext) => {
  try {
    // Map Teachfloor events to Intercom events
    await fetch(INTERCOM_API_URL, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${INTERCOM_TOKEN}`,
        'Content-Type': 'application/json',
        'Accept': 'application/json'
      },
      body: JSON.stringify({
        event_name: `teachfloor_${eventData.event}`,
        created_at: eventData.timestamp,
        user_id: eventData.id,
        email: eventData.email,
        metadata: {
          course_id: objectContext.course?.id,
          course_title: objectContext.course?.title,
          module_id: objectContext.module?.id,
          element_id: objectContext.element?.id,
          viewport: eventData.viewport,
          // Include event-specific data
          ...(eventData.status && { status: eventData.status }),
          ...(eventData.score && { score: eventData.score })
        }
      })
    })

    console.log(`Event ${eventData.event} sent to Intercom`)
  } catch (error) {
    console.error('Failed to send event to Intercom:', error)
  }
})

// Track viewport changes for navigation analytics
subscribeToEvent('environment.viewport.changed', async (viewport, objectContext) => {
  try {
    await fetch(INTERCOM_API_URL, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${INTERCOM_TOKEN}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        event_name: 'teachfloor_page_view',
        created_at: Math.floor(Date.now() / 1000),
        metadata: {
          viewport,
          course_id: objectContext.course?.id,
          module_id: objectContext.module?.id
        }
      })
    })
  } catch (error) {
    console.error('Failed to track page view:', error)
  }
})

console.log('Intercom integration initialized - listening for events')
```

### How It Works

1. **No UI**: The app has no visual components - it runs silently in the background
2. **Event Listening**: Subscribes to `auth.user.event` to track all user activities
3. **Data Forwarding**: Sends events to Intercom API with context (course, module, element)
4. **Always Active**: Once installed, it continuously tracks events for all users

### Similar Patterns

Use this pattern for:
- **Segment/Mixpanel**: Analytics event tracking
- **HubSpot/Salesforce**: CRM activity sync
- **Slack/Discord**: Event notifications
- **Webhooks**: Custom integrations
- **Data Warehouses**: Learning data export

---

## Simple Widget

A minimal widget that displays user information.

### Manifest

```json
{
  "id": "simple-widget",
  "version": "1.0.0",
  "name": "Simple Widget",
  "description": "Display user greeting",
  "distribution_type": "private",
  "ui_extension": {
    "views": [
      {
        "viewport": "teachfloor.dashboard.course.detail",
        "component": "Widget"
      }
    ]
  }
}
```

### Component

```javascript
// src/views/Widget.jsx
import React from 'react'
import {
  Container,
  Text,
  Avatar,
  Group,
  useExtensionContext
} from '@teachfloor/extension-kit'

const Widget = () => {
  const { userContext } = useExtensionContext()

  return (
    <Container p="md">
      <Group spacing="md">
        <Avatar src={userContext.avatar} alt={userContext.full_name} />
        <div>
          <Text fw={600}>Hello, {userContext.full_name}!</Text>
          <Text size="sm" c="dimmed">{userContext.email}</Text>
        </div>
      </Group>
    </Container>
  )
}

export default Widget
```

---

## Note-Taking App

A complete note-taking application using browser storage.

### Manifest

```json
{
  "id": "notes-app",
  "version": "1.0.0",
  "name": "Notes",
  "description": "Take notes while learning",
  "distribution_type": "private",
  "ui_extension": {
    "views": [
      {
        "viewport": "teachfloor.dashboard.course.detail",
        "component": "NotesApp"
      },
      {
        "viewport": "teachfloor.dashboard.course.module.detail",
        "component": "NotesApp"
      }
    ]
  },
  "permissions": [
    {
      "permission": "userdata_write",
      "purpose": "Save and load your notes"
    }
  ]
}
```

### Custom Hook

```javascript
// src/hooks/useNotes.js
import { useState, useEffect } from 'react'
import { showToast, store, retrieve } from '@teachfloor/extension-kit'

export function useNotes() {
  const [notes, setNotes] = useState([])
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    loadNotes()
  }, [])

  async function loadNotes() {
    try {
      const data = await retrieve('notes', 'userdata')
      setNotes(data || [])
    } catch (error) {
      showToast('Failed to load notes', { type: 'error' })
    } finally {
      setLoading(false)
    }
  }

  async function addNote(note) {
    try {
      const newNote = {
        id: Date.now(),
        title: note.title,
        content: note.content,
        createdAt: new Date().toISOString()
      }

      const updated = [...notes, newNote]
      await store('notes', updated, 'userdata')
      setNotes(updated)
      showToast('Note saved!', { type: 'success' })
    } catch (error) {
      showToast('Failed to save note', { type: 'error' })
    }
  }

  async function updateNote(id, updates) {
    try {
      const updated = notes.map(note =>
        note.id === id ? { ...note, ...updates } : note
      )
      await store('notes', updated, 'userdata')
      setNotes(updated)
      showToast('Note updated!', { type: 'success' })
    } catch (error) {
      showToast('Failed to update note', { type: 'error' })
    }
  }

  async function deleteNote(id) {
    try {
      const updated = notes.filter(note => note.id !== id)
      await store('notes', updated, 'userdata')
      setNotes(updated)
      showToast('Note deleted!', { type: 'success' })
    } catch (error) {
      showToast('Failed to delete note', { type: 'error' })
    }
  }

  return { notes, loading, addNote, updateNote, deleteNote }
}
```

### Components

```javascript
// src/views/NotesApp.jsx
import React, { useState } from 'react'
import {
  Container,
  SimpleGrid,
  Button,
  Text,
  Loader,
  showDrawer,
  hideDrawer
} from '@teachfloor/extension-kit'
import { useNotes } from '../hooks/useNotes'
import NotesList from '../components/NotesList'
import NoteForm from '../components/NoteForm'

const NotesApp = () => {
  const { notes, loading, addNote, updateNote, deleteNote } = useNotes()
  const [showForm, setShowForm] = useState(false)
  const [editingNote, setEditingNote] = useState(null)

  React.useEffect(() => {
    showDrawer()
    return () => hideDrawer()
  }, [])

  function handleNew() {
    setEditingNote(null)
    setShowForm(true)
  }

  function handleEdit(note) {
    setEditingNote(note)
    setShowForm(true)
  }

  async function handleSave(note) {
    if (editingNote) {
      await updateNote(editingNote.id, note)
    } else {
      await addNote(note)
    }
    setShowForm(false)
    setEditingNote(null)
  }

  if (loading) {
    return <Loader />
  }

  if (showForm) {
    return (
      <NoteForm
        note={editingNote}
        onSave={handleSave}
        onCancel={() => setShowForm(false)}
      />
    )
  }

  return (
    <Container p="md">
      <SimpleGrid verticalSpacing="md">
        <Text size="xl" fw={700}>My Notes</Text>

        {notes.length === 0 ? (
          <div>
            <Text c="dimmed">No notes yet. Create your first note!</Text>
            <Button onClick={handleNew} mt="md">Create Note</Button>
          </div>
        ) : (
          <>
            <Button onClick={handleNew}>New Note</Button>
            <NotesList
              notes={notes}
              onEdit={handleEdit}
              onDelete={deleteNote}
            />
          </>
        )}
      </SimpleGrid>
    </Container>
  )
}

export default NotesApp
```

```javascript
// src/components/NotesList.jsx
import React from 'react'
import { SimpleGrid, Group, Text, Button } from '@teachfloor/extension-kit'

const NotesList = ({ notes, onEdit, onDelete }) => {
  return (
    <SimpleGrid verticalSpacing="sm">
      {notes.map(note => (
        <div key={note.id} style={{ border: '1px solid #e0e0e0', padding: '12px', borderRadius: '4px' }}>
          <Text fw={600}>{note.title}</Text>
          <Text size="sm" c="dimmed">{note.content}</Text>
          <Group spacing="sm" mt="sm">
            <Button size="xs" variant="subtle" onClick={() => onEdit(note)}>Edit</Button>
            <Button size="xs" variant="subtle" color="red" onClick={() => onDelete(note.id)}>Delete</Button>
          </Group>
        </div>
      ))}
    </SimpleGrid>
  )
}

export default NotesList
```

```javascript
// src/components/NoteForm.jsx
import React, { useState } from 'react'
import {
  Container,
  SimpleGrid,
  TextInput,
  Textarea,
  Button,
  Group
} from '@teachfloor/extension-kit'

const NoteForm = ({ note, onSave, onCancel }) => {
  const [title, setTitle] = useState(note?.title || '')
  const [content, setContent] = useState(note?.content || '')

  function handleSubmit(e) {
    e.preventDefault()
    onSave({ title, content })
  }

  return (
    <Container p="md">
      <form onSubmit={handleSubmit}>
        <SimpleGrid verticalSpacing="md">
          <TextInput
            label="Title"
            value={title}
            onChange={(e) => setTitle(e.target.value)}
            required
          />
          <Textarea
            label="Content"
            value={content}
            onChange={(e) => setContent(e.target.value)}
            minRows={6}
            required
          />
          <Group spacing="sm">
            <Button type="submit">Save</Button>
            <Button variant="subtle" onClick={onCancel}>Cancel</Button>
          </Group>
        </SimpleGrid>
      </form>
    </Container>
  )
}

export default NoteForm
```

---

## Progress Tracker

Track user's learning progress.

### Manifest

```json
{
  "id": "progress-tracker",
  "version": "1.0.0",
  "name": "Progress Tracker",
  "description": "Track your learning progress",
  "ui_extension": {
    "views": [
      {
        "viewport": "teachfloor.dashboard.course.detail",
        "component": "ProgressTracker"
      }
    ]
  },
  "permissions": [
    {
      "permission": "user_events_read",
      "purpose": "Track your learning activities"
    },
    {
      "permission": "course_read",
      "purpose": "Display course information"
    }
  ]
}
```

### Component

```javascript
// src/views/ProgressTracker.jsx
import React, { useState, useEffect } from 'react'
import {
  Container,
  SimpleGrid,
  Text,
  Progress,
  Badge,
  Loader
} from '@teachfloor/extension-kit'

const ProgressTracker = () => {
  const [stats, setStats] = useState(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    loadStats()
  }, [])

  async function loadStats() {
    // Simulated data - replace with actual API calls
    const mockStats = {
      totalModules: 10,
      completedModules: 6,
      totalElements: 50,
      completedElements: 32,
      timeSpent: 1200 // minutes
    }

    setStats(mockStats)
    setLoading(false)
  }

  if (loading) {
    return <Loader />
  }

  const moduleProgress = (stats.completedModules / stats.totalModules) * 100
  const elementProgress = (stats.completedElements / stats.totalElements) * 100

  return (
    <Container p="md">
      <SimpleGrid verticalSpacing="lg">
        <Text size="xl" fw={700}>Your Progress</Text>

        <div>
          <Text size="sm" fw={500} mb="xs">Modules</Text>
          <Progress value={moduleProgress} color="blue" />
          <Text size="sm" c="dimmed" mt="xs">
            {stats.completedModules} of {stats.totalModules} completed
          </Text>
        </div>

        <div>
          <Text size="sm" fw={500} mb="xs">Elements</Text>
          <Progress value={elementProgress} color="green" />
          <Text size="sm" c="dimmed" mt="xs">
            {stats.completedElements} of {stats.totalElements} completed
          </Text>
        </div>

        <div>
          <Text size="sm" fw={500} mb="xs">Time Spent</Text>
          <Badge size="lg">{Math.floor(stats.timeSpent / 60)} hours</Badge>
        </div>
      </SimpleGrid>
    </Container>
  )
}

export default ProgressTracker
```

---

## Settings Page

App configuration interface.

### Component

```javascript
// src/views/AppSettings.jsx
import React, { useState, useEffect } from 'react'
import {
  SettingsView,
  SimpleGrid,
  TextInput,
  Select,
  Switch,
  showToast,
  store,
  retrieve
} from '@teachfloor/extension-kit'

const AppSettings = () => {
  const [status, setStatus] = useState('')
  const [settings, setSettings] = useState(null)

  useEffect(() => {
    loadSettings()
  }, [])

  async function loadSettings() {
    const saved = await retrieve('settings', 'appdata')
    setSettings(saved || {
      apiKey: '',
      language: 'en',
      notifications: true
    })
  }

  async function saveSettings(values) {
    setStatus('Saving...')

    try {
      await store('settings', values, 'appdata')
      setStatus('Saved successfully!')
      showToast('Settings saved!', { type: 'success' })
    } catch (error) {
      setStatus('Failed to save settings')
      showToast('Failed to save', { type: 'error' })
    }
  }

  if (!settings) {
    return <div>Loading...</div>
  }

  return (
    <SettingsView
      onSave={saveSettings}
      statusMessage={status}
      initialValues={settings}
    >
      <SimpleGrid verticalSpacing="md">
        <TextInput
          name="apiKey"
          label="API Key"
          placeholder="Enter your API key"
          type="password"
        />

        <Select
          name="language"
          label="Language"
          placeholder="Select language"
          data={[
            { value: 'en', label: 'English' },
            { value: 'es', label: 'Spanish' },
            { value: 'fr', label: 'French' }
          ]}
        />

        <Switch
          name="notifications"
          label="Enable notifications"
        />
      </SimpleGrid>
    </SettingsView>
  )
}

export default AppSettings
```

---

## Data Visualization

Display analytics with charts.

### Component

```javascript
// src/views/AnalyticsDashboard.jsx
import React, { useState, useEffect } from 'react'
import {
  Container,
  SimpleGrid,
  Text,
  Select,
  BarChart,
  LineChart,
  Loader
} from '@teachfloor/extension-kit'

const AnalyticsDashboard = () => {
  const [data, setData] = useState(null)
  const [timeframe, setTimeframe] = useState('week')
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    loadData(timeframe)
  }, [timeframe])

  async function loadData(period) {
    setLoading(true)

    // Simulated data - replace with actual API calls
    const mockData = {
      week: [
        { name: 'Mon', completions: 4 },
        { name: 'Tue', completions: 3 },
        { name: 'Wed', completions: 5 },
        { name: 'Thu', completions: 2 },
        { name: 'Fri', completions: 6 }
      ],
      month: [
        { name: 'Week 1', completions: 15 },
        { name: 'Week 2', completions: 22 },
        { name: 'Week 3', completions: 18 },
        { name: 'Week 4', completions: 25 }
      ]
    }

    setData(mockData[period])
    setLoading(false)
  }

  if (loading) {
    return <Loader />
  }

  return (
    <Container p="md">
      <SimpleGrid verticalSpacing="lg">
        <div>
          <Text size="xl" fw={700}>Analytics</Text>
          <Select
            value={timeframe}
            onChange={setTimeframe}
            data={[
              { value: 'week', label: 'This Week' },
              { value: 'month', label: 'This Month' }
            ]}
            style={{ maxWidth: 200, marginTop: 12 }}
          />
        </div>

        <div>
          <Text fw={600} mb="md">Completions Over Time</Text>
          <BarChart
            data={data}
            dataKey="completions"
            height={300}
          />
        </div>

        <div>
          <Text fw={600} mb="md">Trend</Text>
          <LineChart
            data={data}
            dataKey="completions"
            height={300}
          />
        </div>
      </SimpleGrid>
    </Container>
  )
}

export default AnalyticsDashboard
```

---

## External API Integration

Integrate with external service.

### API Client

```javascript
// src/utils/api.js
class ExternalAPI {
  constructor(apiKey) {
    this.apiKey = apiKey
    this.baseURL = 'https://api.example.com'
  }

  async request(endpoint, options = {}) {
    const response = await fetch(`${this.baseURL}${endpoint}`, {
      ...options,
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json',
        ...options.headers
      }
    })

    if (!response.ok) {
      throw new Error(`API error: ${response.statusText}`)
    }

    return response.json()
  }

  async getData() {
    return this.request('/data')
  }

  async postData(data) {
    return this.request('/data', {
      method: 'POST',
      body: JSON.stringify(data)
    })
  }
}

export default ExternalAPI
```

### Component

```javascript
// src/views/IntegrationView.jsx
import React, { useState, useEffect } from 'react'
import {
  Container,
  SimpleGrid,
  Button,
  Text,
  Loader,
  showToast
} from '@teachfloor/extension-kit'
import ExternalAPI from '../utils/api'

const IntegrationView = () => {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(false)
  const [api, setApi] = useState(null)

  useEffect(() => {
    initializeAPI()
  }, [])

  async function initializeAPI() {
    try {
      const saved = localStorage.getItem('teachfloor-settings')
      const settings = saved ? JSON.parse(saved) : null

      if (settings?.apiKey) {
        setApi(new ExternalAPI(settings.apiKey))
      }
    } catch (error) {
      showToast('Failed to load settings', { type: 'error' })
    }
  }

  async function loadData() {
    if (!api) {
      showToast('Please configure API key in settings', { type: 'warning' })
      return
    }

    setLoading(true)

    try {
      const result = await api.getData()
      setData(result)
      showToast('Data loaded successfully', { type: 'success' })
    } catch (error) {
      showToast('Failed to load data', { type: 'error' })
    } finally {
      setLoading(false)
    }
  }

  if (!api) {
    return (
      <Container p="md">
        <Text>Please configure your API key in settings</Text>
      </Container>
    )
  }

  return (
    <Container p="md">
      <SimpleGrid verticalSpacing="md">
        <Text size="xl" fw={700}>External Data</Text>

        <Button onClick={loadData} loading={loading}>
          Load Data
        </Button>

        {data && (
          <div>
            <Text>Data loaded!</Text>
            <pre>{JSON.stringify(data, null, 2)}</pre>
          </div>
        )}
      </SimpleGrid>
    </Container>
  )
}

export default IntegrationView
```

---

## Next Steps

See troubleshooting guide for common issues:

→ Continue to [Troubleshooting](./13-troubleshooting.md)

## Additional Resources

- [Extension Kit](./05-components.md)
- [Extension Kit Integration](./06-integration.md)
- [Best Practices](./11-best-practices.md)
