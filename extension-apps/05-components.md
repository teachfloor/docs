# Extension Kit Components

This guide covers the **visual components** provided by the Teachfloor Extension Kit (`@teachfloor/extension-kit`) - everything you need to build your app's user interface.

## What This Document Covers

This document focuses exclusively on **UI components** for building your app's interface:

- **Form Components**: TextInput, Select, Checkbox, Switch, etc.
- **Layout Components**: Container, Grid, Stack, Group
- **Display Components**: Button, Text, Badge, Avatar, Tabs
- **Chart Components**: BarChart, LineChart
- **Special Components**: SettingsView
- **Setup Components**: ExtensionContextProvider, ExtensionViewLoader

All components are designed to match Teachfloor's design system, ensuring your app feels native to the platform.

### Quick Reference

| What you need | Where to find it |
|---------------|------------------|
| Buttons, Forms, Layouts, Charts | **This document** (05-components.md) |
| Events, Storage, Navigation, Toasts | [Extension Kit Integration](./06-integration.md) |

> **Looking for platform integration?** To communicate with the platform (events, storage, navigation, toasts), see the [Extension Kit Integration](./06-integration.md) guide.

## Installation

The Extension Kit is automatically included when you create an app with the CLI:

```bash
teachfloor apps create my-app
```

To install manually:

```bash
npm install @teachfloor/extension-kit react react-dom
```

## Setup Components

### Extension Context Provider

Wrap your app with the provider to enable platform context throughout your app:

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { ExtensionContextProvider } from '@teachfloor/extension-kit'
import App from './App'

const root = ReactDOM.createRoot(document.getElementById('root'))

root.render(
  <ExtensionContextProvider>
    <App />
  </ExtensionContextProvider>
)
```

**Props**:
- `autoInit`: (Optional) Automatically signal app ready to platform (default: true)

> **Accessing context data?** To use the `useExtensionContext()` hook and access platform data, see the [Extension Kit Integration](./06-integration.md#context-api) guide.

### View Loader

Automatically load the correct component based on viewport:

```jsx
import React from 'react'
import { ExtensionViewLoader } from '@teachfloor/extension-kit'
import manifest from '../teachfloor-app.json'

const App = () => {
  return (
    <ExtensionViewLoader
      manifest={manifest}
      componentResolver={(componentName) => import(`./views/${componentName}`)}
    />
  )
}

export default App
```

**Props**:
- `manifest`: Your app's manifest object (required)
- `componentResolver`: Function that returns a dynamic import (optional, defaults to null)
- `fallback`: Loading component (optional, defaults to null)
- `basePath`: Base path for components (optional, defaults to `'./'`)

## Layout Components

### Container

Main content wrapper with consistent padding:

```jsx
import { Container } from '@teachfloor/extension-kit'

<Container>
  <h1>My Content</h1>
</Container>

// With props
<Container size="sm" p="xl">
  <h1>Compact Container</h1>
</Container>
```

**Props**:
- `size`: `"xs"` | `"sm"` | `"md"` | `"lg"` | `"xl"` (default: `"md"`)
- `p`: Padding
- `m`: Margin

### Box

Flexible container for layout:

```jsx
import { Box } from '@teachfloor/extension-kit'

<Box p="md" style={{ backgroundColor: '#f0f0f0' }}>
  <p>Content in a box</p>
</Box>
```

**Props**:
- All standard HTML div props
- Spacing props: `p`, `m`, `px`, `py`, `mx`, `my`

### Grid

Responsive grid layout:

```jsx
import { Grid } from '@teachfloor/extension-kit'

<Grid>
  <Grid.Col span={6}>
    <p>Half width</p>
  </Grid.Col>
  <Grid.Col span={6}>
    <p>Half width</p>
  </Grid.Col>
</Grid>

// Responsive columns
<Grid>
  <Grid.Col span={12} md={6} lg={4}>
    <p>Responsive column</p>
  </Grid.Col>
</Grid>
```

**Props**:
- `gutter`: Gap between columns
- `align`: Vertical alignment
- `justify`: Horizontal alignment

### SimpleGrid

Automatic grid with equal columns:

```jsx
import { SimpleGrid } from '@teachfloor/extension-kit'

<SimpleGrid cols={3} spacing="lg">
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</SimpleGrid>

// Responsive
<SimpleGrid cols={1} sm={2} md={3}>
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</SimpleGrid>
```

**Props**:
- `cols`: Number of columns
- `spacing`: Gap between items
- `verticalSpacing`: Vertical gap

### Group

Horizontal layout with flex:

```jsx
import { Group } from '@teachfloor/extension-kit'

<Group spacing="md" position="apart">
  <Button>Left</Button>
  <Button>Right</Button>
</Group>
```

**Props**:
- `spacing`: Gap between items
- `position`: `"left"` | `"center"` | `"right"` | `"apart"`
- `align`: Vertical alignment

## Form Components

### TextInput

Single-line text input:

```jsx
import { TextInput } from '@teachfloor/extension-kit'

<TextInput
  label="Name"
  placeholder="Enter your name"
  required
  onChange={(e) => console.log(e.target.value)}
/>

// With validation
<TextInput
  label="Email"
  type="email"
  error="Invalid email address"
/>
```

**Props**:
- `label`: Input label
- `placeholder`: Placeholder text
- `required`: Required field
- `error`: Error message
- `description`: Help text
- All standard input props

### Textarea

Multi-line text input:

```jsx
import { Textarea } from '@teachfloor/extension-kit'

<Textarea
  label="Description"
  placeholder="Enter description"
  minRows={4}
  autosize
/>
```

**Props**:
- `minRows`: Minimum rows
- `maxRows`: Maximum rows
- `autosize`: Auto-grow with content
- All TextInput props

### Select

Dropdown selection:

```jsx
import { Select } from '@teachfloor/extension-kit'

<Select
  label="Choose option"
  placeholder="Select one"
  data={[
    { value: 'react', label: 'React' },
    { value: 'vue', label: 'Vue' },
    { value: 'angular', label: 'Angular' }
  ]}
/>
```

**Props**:
- `data`: Array of options
- `searchable`: Enable search
- `clearable`: Allow clearing
- All TextInput props

### MultiSelect

Multiple selection dropdown:

```jsx
import { MultiSelect } from '@teachfloor/extension-kit'

<MultiSelect
  label="Select multiple"
  placeholder="Choose options"
  data={[
    { value: '1', label: 'Option 1' },
    { value: '2', label: 'Option 2' },
    { value: '3', label: 'Option 3' }
  ]}
/>
```

### NumberInput

Numeric input with controls:

```jsx
import { NumberInput } from '@teachfloor/extension-kit'

<NumberInput
  label="Age"
  placeholder="Enter age"
  min={0}
  max={120}
  step={1}
/>
```

### PasswordInput

Password input with toggle:

```jsx
import { PasswordInput } from '@teachfloor/extension-kit'

<PasswordInput
  label="Password"
  placeholder="Enter password"
  required
/>
```

### Checkbox

Checkbox input:

```jsx
import { Checkbox } from '@teachfloor/extension-kit'

<Checkbox
  label="I agree to terms"
  checked={checked}
  onChange={(e) => setChecked(e.target.checked)}
/>
```

### Radio

Radio button input:

```jsx
import { Radio } from '@teachfloor/extension-kit'

<Radio.Group
  label="Select option"
  value={value}
  onChange={setValue}
>
  <Radio value="1" label="Option 1" />
  <Radio value="2" label="Option 2" />
</Radio.Group>
```

### Switch

Toggle switch:

```jsx
import { Switch } from '@teachfloor/extension-kit'

<Switch
  label="Enable notifications"
  checked={enabled}
  onChange={(e) => setEnabled(e.target.checked)}
/>
```

### ColorInput

Color picker:

```jsx
import { ColorInput } from '@teachfloor/extension-kit'

<ColorInput
  label="Choose color"
  placeholder="Pick a color"
/>
```

## Display Components

### Text

Styled text component:

```jsx
import { Text } from '@teachfloor/extension-kit'

<Text size="lg" fw={700}>Large bold text</Text>
<Text size="sm" c="dimmed">Small dimmed text</Text>
<Text c="red">Error message</Text>
```

**Props**:
- `size`: `"xs"` | `"sm"` | `"md"` | `"lg"` | `"xl"`
- `fw`: Font weight (100-900)
- `c`: Color
- `align`: Text alignment

### Button

Action button:

```jsx
import { Button } from '@teachfloor/extension-kit'

<Button onClick={() => console.log('Clicked')}>
  Click Me
</Button>

// Variants
<Button variant="filled">Filled</Button>
<Button variant="outline">Outline</Button>
<Button variant="subtle">Subtle</Button>

// Sizes
<Button size="xs">Extra Small</Button>
<Button size="sm">Small</Button>
<Button size="md">Medium</Button>
<Button size="lg">Large</Button>

// Colors
<Button color="blue">Blue</Button>
<Button color="red">Red</Button>
<Button color="green">Green</Button>

// Loading state
<Button loading>Loading...</Button>

// Disabled
<Button disabled>Disabled</Button>
```

**Props**:
- `variant`: `"filled"` | `"outline"` | `"subtle"`
- `size`: `"xs"` | `"sm"` | `"md"` | `"lg"`
- `color`: Color name
- `loading`: Show loading state
- `disabled`: Disable button

### ButtonGroup

Group related buttons:

```jsx
import { ButtonGroup } from '@teachfloor/extension-kit'

<ButtonGroup>
  <Button>Left</Button>
  <Button>Middle</Button>
  <Button>Right</Button>
</ButtonGroup>
```

### Badge

Label or status indicator:

```jsx
import { Badge } from '@teachfloor/extension-kit'

<Badge>Default</Badge>
<Badge color="blue">Info</Badge>
<Badge color="green">Success</Badge>
<Badge color="red">Error</Badge>
<Badge variant="dot">With dot</Badge>
```

### Chip

Interactive chip/tag:

```jsx
import { Chip } from '@teachfloor/extension-kit'

<Chip checked={checked} onChange={setChecked}>
  Selectable Chip
</Chip>
```

### Avatar

User avatar:

```jsx
import { Avatar } from '@teachfloor/extension-kit'

<Avatar src="https://..." alt="User" />
<Avatar radius="xl">AB</Avatar>
<Avatar color="blue">JD</Avatar>
```

**Props**:
- `src`: Image URL
- `alt`: Alt text
- `radius`: Border radius
- `size`: Size
- `color`: Background color for initials

### Image

Optimized image component:

```jsx
import { Image } from '@teachfloor/extension-kit'

<Image
  src="https://..."
  alt="Description"
  width={400}
  height={300}
/>
```

### Divider

Visual separator:

```jsx
import { Divider } from '@teachfloor/extension-kit'

<Divider />
<Divider label="Section Title" />
<Divider variant="dashed" />
```

### Tooltip

Hover tooltip:

```jsx
import { Tooltip } from '@teachfloor/extension-kit'

<Tooltip label="Helpful information">
  <Button>Hover me</Button>
</Tooltip>
```

### Loader

Loading indicator:

```jsx
import { Loader } from '@teachfloor/extension-kit'

<Loader />
<Loader size="lg" />
<Loader variant="dots" />
```

### Tabs

Tab navigation:

```jsx
import { Tabs } from '@teachfloor/extension-kit'

<Tabs defaultValue="first">
  <Tabs.List>
    <Tabs.Tab value="first">First Tab</Tabs.Tab>
    <Tabs.Tab value="second">Second Tab</Tabs.Tab>
  </Tabs.List>

  <Tabs.Panel value="first">
    <p>First panel</p>
  </Tabs.Panel>

  <Tabs.Panel value="second">
    <p>Second panel</p>
  </Tabs.Panel>
</Tabs>
```

## Chart Components

### BarChart

Bar chart visualization:

```jsx
import { BarChart } from '@teachfloor/extension-kit'

const data = [
  { name: 'Jan', value: 400 },
  { name: 'Feb', value: 300 },
  { name: 'Mar', value: 500 }
]

<BarChart
  data={data}
  dataKey="value"
  height={300}
/>
```

### LineChart

Line chart visualization:

```jsx
import { LineChart } from '@teachfloor/extension-kit'

const data = [
  { name: 'Week 1', value: 100 },
  { name: 'Week 2', value: 150 },
  { name: 'Week 3', value: 120 }
]

<LineChart
  data={data}
  dataKey="value"
  height={300}
/>
```

## Special Components

### SettingsView

Pre-built settings page template:

```jsx
import { SettingsView, TextInput, Select, SimpleGrid } from '@teachfloor/extension-kit'
import { useState } from 'react'

const AppSettings = () => {
  const [status, setStatus] = useState('')

  const saveSettings = async (values) => {
    setStatus('Saving...')
    try {
      await fetch('https://api.example.com/save', {
        method: 'POST',
        body: JSON.stringify(values)
      })
      setStatus('Saved!')
    } catch (error) {
      setStatus('Error saving settings')
    }
  }

  return (
    <SettingsView onSave={saveSettings} statusMessage={status}>
      <SimpleGrid>
        <TextInput
          name="apiKey"
          label="API Key"
          placeholder="Enter your API key"
        />
        <Select
          name="region"
          label="Region"
          data={[
            { value: 'us', label: 'US' },
            { value: 'eu', label: 'EU' }
          ]}
        />
      </SimpleGrid>
    </SettingsView>
  )
}
```

**Props**:
- `onSave`: Save handler function
- `statusMessage`: Status message to display
- `children`: Form fields

## Spacing System

All components support spacing props:

| Prop | Description |
|------|-------------|
| `p` | Padding (all sides) |
| `px` | Padding horizontal |
| `py` | Padding vertical |
| `pt` | Padding top |
| `pr` | Padding right |
| `pb` | Padding bottom |
| `pl` | Padding left |
| `m` | Margin (all sides) |
| `mx` | Margin horizontal |
| `my` | Margin vertical |
| `mt` | Margin top |
| `mr` | Margin right |
| `mb` | Margin bottom |
| `ml` | Margin left |

**Values**: `xs`, `sm`, `md`, `lg`, `xl` or pixel number

**Example**:
```jsx
<Box p="md" m="lg">Content</Box>
<Container px="xl" py="sm">Content</Container>
```

## Color System

Available colors:
- `blue`, `red`, `green`, `yellow`, `orange`
- `purple`, `pink`, `teal`, `cyan`
- `gray`, `dark`, `dimmed`

**Example**:
```jsx
<Text c="blue">Blue text</Text>
<Badge color="green">Success</Badge>
<Button color="red">Delete</Button>
```

## Next Steps

Learn how to integrate with the platform:

→ Continue to [Extension Kit Integration](./06-integration.md)

## Additional Resources

- [Extension Kit Integration](./06-integration.md) - Events, storage, navigation, and platform integration
- [Data Storage](./07-data-storage.md) - Deep dive into data persistence
- [Examples](./12-examples.md) - Sample apps and code snippets
- [Best Practices](./11-best-practices.md) - Development patterns and tips
