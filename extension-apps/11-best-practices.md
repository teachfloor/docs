# Best Practices

A curated collection of best practices, patterns, and recommendations for building high-quality Teachfloor apps.

## Development

### Project Structure

Organize your code logically:

```
my-app/
├── src/
│   ├── index.js                # Entry point
│   ├── views/                  # Viewport components
│   │   ├── App.jsx
│   │   ├── CourseView.jsx
│   │   └── SettingsView.jsx
│   ├── components/             # Reusable components
│   │   ├── Header.jsx
│   │   └── shared/
│   ├── hooks/                  # Custom hooks
│   │   ├── useAppData.js
│   │   └── usePreferences.js
│   ├── utils/                  # Utilities
│   │   ├── api.js
│   │   └── formatting.js
│   └── constants/              # Constants
│       └── config.js
├── public/
└── teachfloor-app.json
```

### Component Design

**Do**: Create small, focused components
```javascript
// ✅ Good
function NotesList({ notes }) {
  return notes.map(note => <NoteItem key={note.id} note={note} />)
}

function NoteItem({ note }) {
  return <div>{note.title}</div>
}
```

**Don't**: Create monolithic components
```javascript
// ❌ Bad
function NotesApp() {
  // 500 lines of code...
}
```

### Custom Hooks

Extract logic into reusable hooks:

```javascript
// ✅ Good
function useNotes() {
  const [notes, setNotes] = useState([])
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    loadNotes()
  }, [])

  async function loadNotes() {
    const data = await retrieve('notes', 'userdata')
    setNotes(data || [])
    setLoading(false)
  }

  async function addNote(note) {
    const updated = [...notes, note]
    await store('notes', updated, 'userdata')
    setNotes(updated)
  }

  return { notes, loading, addNote }
}

// Usage
function MyComponent() {
  const { notes, loading, addNote } = useNotes()
  // ...
}
```

### Error Boundaries

Implement error boundaries:

```javascript
class ErrorBoundary extends React.Component {
  state = { hasError: false }

  static getDerivedStateFromError(error) {
    return { hasError: true }
  }

  componentDidCatch(error, errorInfo) {
    console.error('App error:', error, errorInfo)
  }

  render() {
    if (this.state.hasError) {
      return (
        <Container>
          <Text>Something went wrong. Please refresh.</Text>
        </Container>
      )
    }

    return this.props.children
  }
}

// Usage
<ErrorBoundary>
  <App />
</ErrorBoundary>
```

---

## Performance

### Lazy Loading

Load components only when needed:

```javascript
import { lazy, Suspense } from 'react'

const HeavyComponent = lazy(() => import('./HeavyComponent'))

function App() {
  return (
    <Suspense fallback={<Loader />}>
      <HeavyComponent />
    </Suspense>
  )
}
```

### Memoization

Memoize expensive calculations:

```javascript
import { useMemo } from 'react'

function AnalyticsDashboard({ data }) {
  const statistics = useMemo(() => {
    // Expensive calculation
    return calculateStatistics(data)
  }, [data])

  return <div>{statistics}</div>
}
```

### Debouncing

Debounce frequent operations:

```javascript
import { useCallback, useRef } from 'react'

function SearchInput() {
  const timeoutRef = useRef(null)

  const debouncedSearch = useCallback((query) => {
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current)
    }

    timeoutRef.current = setTimeout(() => {
      performSearch(query)
    }, 500)
  }, [])

  return (
    <TextInput
      placeholder="Search..."
      onChange={(e) => debouncedSearch(e.target.value)}
    />
  )
}
```

### Bundle Size

Minimize bundle size:

```javascript
// ✅ Good: Import only what you need
import { Button, Text } from '@teachfloor/extension-kit'

// ❌ Bad: Import everything
import * as ExtensionKit from '@teachfloor/extension-kit'
```

### Caching

Implement data caching:

```javascript
const cache = new Map()

async function getCachedData(key, fetchFn, ttl = 60000) {
  const cached = cache.get(key)

  if (cached && Date.now() - cached.timestamp < ttl) {
    return cached.data
  }

  const data = await fetchFn()
  cache.set(key, { data, timestamp: Date.now() })

  return data
}
```

---

## Security

### Input Validation

Always validate user input:

```javascript
function validateEmail(email) {
  const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  return re.test(email)
}

function saveUserData(email, name) {
  if (!validateEmail(email)) {
    throw new Error('Invalid email format')
  }

  if (!name || name.length < 2) {
    throw new Error('Name must be at least 2 characters')
  }

  // Save data
}
```

### Sanitization

Sanitize data before storage:

```javascript
function sanitizeString(str) {
  return str
    .replace(/<script[^>]*>.*?<\/script>/gi, '')
    .replace(/<[^>]+>/g, '')
    .trim()
}

async function saveNote(content) {
  const sanitized = sanitizeString(content)
  await store('note', sanitized, 'userdata')
}
```

### Sensitive Data

Never expose secrets:

```javascript
// ❌ Bad
const API_KEY = 'secret-key-123'

// ✅ Good: Let users provide their own keys
function Settings() {
  const [apiKey, setApiKey] = useState('')

  async function saveKey() {
    await store('api-key', apiKey, 'userdata')
  }

  return (
    <div>
      <PasswordInput
        value={apiKey}
        onChange={(e) => setApiKey(e.target.value)}
      />
      <Button onClick={saveKey}>Save</Button>
    </div>
  )
}
```

### HTTPS Only

Always use HTTPS for external APIs:

```javascript
// ✅ Good
fetch('https://api.example.com/data')

// ❌ Bad
fetch('http://api.example.com/data')
```

---

## User Experience

### Loading States

Show loading indicators:

```javascript
function DataDisplay() {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    loadData()
  }, [])

  if (loading) {
    return <Loader />
  }

  return <DisplayData data={data} />
}
```

### Empty States

Handle empty data gracefully:

```javascript
function NotesList({ notes }) {
  if (notes.length === 0) {
    return (
      <Container>
        <Text c="dimmed">No notes yet. Create your first note!</Text>
        <Button onClick={createNote}>Create Note</Button>
      </Container>
    )
  }

  return notes.map(note => <NoteItem key={note.id} note={note} />)
}
```

### Error Handling

Provide helpful error messages:

```javascript
function DataLoader() {
  const [error, setError] = useState(null)

  async function loadData() {
    try {
      const data = await fetchData()
      setData(data)
    } catch (error) {
      setError('Failed to load data. Please try again.')
      showToast(error.message, { type: 'error' })
    }
  }

  if (error) {
    return (
      <Container>
        <Text c="red">{error}</Text>
        <Button onClick={loadData}>Retry</Button>
      </Container>
    )
  }

  return <div>Content</div>
}
```

### Feedback

Provide immediate feedback:

```javascript
async function saveData(data) {
  try {
    showToast('Saving...', { type: 'info' })
    await store('data', data, 'userdata')
    showToast('Saved successfully!', { type: 'success' })
  } catch (error) {
    showToast('Failed to save', { type: 'error' })
  }
}
```

### Accessibility

Make your app accessible:

```javascript
// ✅ Good: Proper labels and ARIA attributes
<Button
  aria-label="Save note"
  onClick={saveNote}
>
  Save
</Button>

<TextInput
  label="Note title"
  aria-required="true"
  aria-invalid={hasError}
/>
```

---

## Code Quality

### Naming Conventions

Use clear, descriptive names:

```javascript
// ✅ Good
const userNotes = []
function calculateTotalScore() {}
const isAuthenticated = true

// ❌ Bad
const arr = []
function calc() {}
const flag = true
```

### Comments

Write meaningful comments:

```javascript
// ✅ Good
// Calculate the total score by summing all module scores
// and applying a weighted average based on difficulty
function calculateTotalScore(modules) {
  // ...
}

// ❌ Bad
// This function calculates score
function calculateTotalScore(modules) {
  // Loop through modules
  // ...
}
```

### Constants

Use constants for magic numbers:

```javascript
// ✅ Good
const MAX_NOTES = 100
const CACHE_TTL = 60000 // 1 minute

if (notes.length >= MAX_NOTES) {
  // Handle limit
}

// ❌ Bad
if (notes.length >= 100) {
  // Handle limit
}
```

### DRY Principle

Don't repeat yourself:

```javascript
// ✅ Good
function formatDate(date) {
  return new Intl.DateTimeFormat('en-US').format(date)
}

const created = formatDate(note.createdAt)
const updated = formatDate(note.updatedAt)

// ❌ Bad
const created = new Intl.DateTimeFormat('en-US').format(note.createdAt)
const updated = new Intl.DateTimeFormat('en-US').format(note.updatedAt)
```

---

## Testing

### Unit Tests

Write tests for utility functions:

```javascript
// utils/formatting.test.js
import { formatDate } from './formatting'

test('formats date correctly', () => {
  const date = new Date('2024-01-01')
  expect(formatDate(date)).toBe('1/1/2024')
})
```

### Component Tests

Test component rendering:

```javascript
import { render, screen } from '@testing-library/react'
import NotesList from './NotesList'

test('renders empty state', () => {
  render(<NotesList notes={[]} />)
  expect(screen.getByText(/no notes yet/i)).toBeInTheDocument()
})

test('renders notes', () => {
  const notes = [{ id: 1, title: 'Test Note' }]
  render(<NotesList notes={notes} />)
  expect(screen.getByText('Test Note')).toBeInTheDocument()
})
```

### Manual Testing

Test in all viewports:

```
□ Tested in course list page
□ Tested in course detail page
□ Tested in settings page
□ Tested on mobile viewport
□ Tested with different user roles
□ Tested with empty data
□ Tested with maximum data
□ Tested error scenarios
```

---

## Deployment

### Pre-Deployment Checklist

Before uploading:

```
□ Version incremented in manifest
□ All console.log() removed
□ Environment variables removed
□ Tests passing
□ No linting errors
□ Build succeeds
□ Bundle size acceptable
□ Permissions list complete
□ README updated
□ Changelog updated
```

### Versioning

Follow semantic versioning:

```
1.0.0 → 1.0.1: Bug fix
1.0.0 → 1.1.0: New feature
1.0.0 → 2.0.0: Breaking change
```

### Changelog

Maintain a changelog:

```markdown
# Changelog

## [1.1.0] - 2024-01-15
### Added
- Export notes to PDF feature
- Dark mode support

### Fixed
- Note saving bug on slow connections

## [1.0.0] - 2024-01-01
### Added
- Initial release
- Basic note-taking functionality
```

### Documentation

Keep documentation updated:

```markdown
# My App

## Features
- Feature 1
- Feature 2

## Installation
1. Install from marketplace
2. Configure settings

## Usage
...

## Support
Email: support@example.com
```

---

## Patterns

### State Management

Use appropriate state management:

```javascript
// Local state
const [count, setCount] = useState(0)

// Context for app-wide state
const AppContext = createContext()

function AppProvider({ children }) {
  const [user, setUser] = useState(null)

  return (
    <AppContext.Provider value={{ user, setUser }}>
      {children}
    </AppContext.Provider>
  )
}

// Custom hooks for complex state
function useAppState() {
  const [state, setState] = useState(initialState)

  function updateState(updates) {
    setState(prev => ({ ...prev, ...updates }))
  }

  return [state, updateState]
}
```

### API Integration

Structure API calls:

```javascript
// api.js
class API {
  constructor(baseURL) {
    this.baseURL = baseURL
  }

  async get(endpoint) {
    const response = await fetch(`${this.baseURL}${endpoint}`)
    if (!response.ok) throw new Error('API error')
    return response.json()
  }

  async post(endpoint, data) {
    const response = await fetch(`${this.baseURL}${endpoint}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    })
    if (!response.ok) throw new Error('API error')
    return response.json()
  }
}

export const api = new API('https://api.example.com')

// Usage
const data = await api.get('/notes')
await api.post('/notes', { title: 'New Note' })
```

### Form Handling

Handle forms effectively:

```javascript
function NoteForm({ onSave }) {
  const [values, setValues] = useState({ title: '', content: '' })
  const [errors, setErrors] = useState({})

  function validate() {
    const errors = {}

    if (!values.title) errors.title = 'Title is required'
    if (!values.content) errors.content = 'Content is required'

    return errors
  }

  async function handleSubmit(e) {
    e.preventDefault()

    const errors = validate()
    if (Object.keys(errors).length > 0) {
      setErrors(errors)
      return
    }

    try {
      await onSave(values)
      showToast('Saved!', { type: 'success' })
    } catch (error) {
      showToast('Failed to save', { type: 'error' })
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <TextInput
        label="Title"
        value={values.title}
        onChange={(e) => setValues({ ...values, title: e.target.value })}
        error={errors.title}
      />
      <Textarea
        label="Content"
        value={values.content}
        onChange={(e) => setValues({ ...values, content: e.target.value })}
        error={errors.content}
      />
      <Button type="submit">Save</Button>
    </form>
  )
}
```

---

## Common Pitfalls

### Avoid

1. **Not handling loading states**
2. **Ignoring errors**
3. **Over-requesting permissions**
4. **Large bundle sizes**
5. **Missing error boundaries**
6. **Not testing edge cases**
7. **Hardcoding values**
8. **Exposing sensitive data**
9. **Not cleaning up effects**
10. **Forgetting accessibility**

---

## Resources

- [React Documentation](https://react.dev/)
- [Extension Kit](./05-components.md)
- [Examples](./12-examples.md)
- [Troubleshooting](./13-troubleshooting.md)

---

**Remember**: Quality over speed. Take time to write maintainable, secure, and user-friendly code.
