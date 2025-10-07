# SDK Methods

The Teachfloor.js SDK provides three core methods for interacting with the platform.

## API.get()

Retrieve data from Teachfloor.

### Usage

```javascript
API.get(objectIdentifier)
```

### Parameters

- `objectIdentifier` (string): Identifier for the object to retrieve

### Available Objects

| Identifier | Description |
|------------|-------------|
| `auth.user` | Authenticated user details (ID, name, email, profile data) |

### Returns

Promise that resolves with the requested data or rejects on error.

### Example

```javascript
API.get('auth.user')
  .then(user => console.log('User:', user))
  .catch(error => console.error('Error:', error));
```

---

## API.on()

Listen for events within the Teachfloor platform.

### Usage

```javascript
API.on(eventIdentifier, callback)
```

### Parameters

- `eventIdentifier` (string): Event to listen for
- `callback` (function): Function called when event triggers

### Available Events

| Event | Description |
|-------|-------------|
| `environment.viewport.changed` | Viewport (page) changes within Teachfloor |
| `environment.path.changed` | URL path changes within Teachfloor |

### Example

```javascript
API.on('environment.viewport.changed', (viewport, objectContext) => {
  if (viewport === 'teachfloor.dashboard.course.detail') {
    console.log('Course page:', objectContext.course);
  }
});
```

---

## API.emit()

Trigger actions within the Teachfloor platform.

### Usage

```javascript
API.emit(action, parameters)
```

### Parameters

- `action` (string): Action identifier
- `parameters` (object): Action configuration

### Available Actions

#### `ui.toast.show`

Display a toast notification in the Teachfloor dashboard.

**Parameters:**
- `message` (string, required): Notification text
- `autoClose` (number, optional): Duration in milliseconds before auto-closing
- `color` (string, optional): Accent color (`gray`, `green`, `red`)

### Example

```javascript
API.emit('ui.toast.show', {
  message: 'Welcome to the course!',
  autoClose: 3000,
  color: 'green'
});
```

---

## Summary

- **API.get()**: Retrieve data from Teachfloor
- **API.on()**: Subscribe to platform events
- **API.emit()**: Trigger actions in Teachfloor

All methods use promises for asynchronous operations.
