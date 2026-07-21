# Surfaces

Surfaces describe **how** your app renders inside Teachfloor — as a drawer that slides in from the app dock, or as a widget embedded inline in a layout. Combined with a [viewport](./viewports), the pair `(viewport, surface)` fully specifies where and how your app is mounted.

## Two Axes: Viewport and Surface

Every entry in your manifest's `ui_extension.views` is defined by two orthogonal properties:

| Axis | Answers | Example |
|---|---|---|
| **Viewport** | *Where* is the user? | `teachfloor.dashboard.course.detail` |
| **Surface** | *How* is your app rendered? | `drawer` or `widget` |

An app can register **multiple entries at the same viewport** as long as they use different surfaces (or, for widgets, different widget ids — see [Multiple Widgets](#multiple-widgets-at-the-same-viewport)). Kit picks the correct component using both fields.

## Available Surfaces

### Drawer

A right-side panel opened from the app dock. This is the default surface — if a manifest entry omits `surface`, it renders as a drawer for back-compat with pre-surfaces apps.

**Typical use:** on-demand focus interactions — open the app, look at something, close it. Cohort presence, quick notes, calculators, help articles.

```json
{
  "surface": "drawer",
  "viewport": "teachfloor.dashboard.course.detail",
  "component": "CourseHelper"
}
```

The `surface` field is optional here — omitting it produces the same result:

```json
{
  "viewport": "teachfloor.dashboard.course.detail",
  "component": "CourseHelper"
}
```

### Widget

A cell in a layout that hosts widget slots. Widgets are always visible on the page (no click-to-open) and sit alongside native platform widgets.

**Typical use:** always-on displays — dashboards, progress panels, live counters, feature highlights.

Widget entries carry an extra `widget` block with three required fields:

```json
{
  "surface": "widget",
  "viewport": "teachfloor.dashboard.dashboard.detail",
  "component": "CourseProgressWidget",
  "widget": {
    "id": "course_progress",
    "name": "Course Progress",
    "description": "Aggregate progress across all enrolled courses."
  }
}
```

- **`id`** — stable slug (`^[a-z][a-z0-9_]*$`), **unique per app** across all widget declarations. Used as the runtime identifier when your bundle contains multiple widget components. Part of the composite key stored in the dashboard row, so renaming your component doesn't invalidate placed layouts.
- **`name`** — shown in the admin's widget picker and in the app install-consent surface list. ≤60 chars.
- **`description`** — one-line explanation shown alongside the name in the picker. ≤200 chars.

## A Full Manifest Example

An app registering a drawer plus two widgets:

```json
{
  "id": "6a42a271ae2ee",
  "version": "1.0.0",
  "name": "Course Progress Panel",
  "description": "See your progress across all enrolled courses.",
  "distribution_type": "public",
  "ui_extension": {
    "views": [
      {
        "surface": "drawer",
        "viewport": "teachfloor.dashboard.dashboard.detail",
        "component": "ProgressDetailView"
      },
      {
        "surface": "widget",
        "viewport": "teachfloor.dashboard.dashboard.detail",
        "component": "CourseProgressWidget",
        "widget": {
          "id": "course_progress",
          "name": "Course Progress",
          "description": "Aggregate progress across all enrolled courses."
        }
      },
      {
        "surface": "widget",
        "viewport": "teachfloor.dashboard.dashboard.detail",
        "component": "LearningStreakWidget",
        "widget": {
          "id": "learning_streak",
          "name": "Learning Streak",
          "description": "Current daily study streak with a 7-day heatmap."
        }
      }
    ]
  },
  "permissions": []
}
```

The two widgets share the same viewport — Kit disambiguates by `widget.id` at render time.

## Reading the Surface at Runtime

The current surface is exposed via `useExtensionContext()`, grouped inside `environment`:

```jsx
import { useExtensionContext, SURFACES } from '@teachfloor/extension-kit'

const MyView = () => {
  const { environment } = useExtensionContext()

  if (environment.surface === SURFACES.WIDGET) {
    // compact layout — widget slot is usually small
  }

  if (environment.surface === SURFACES.DRAWER) {
    // full drawer layout with headers and sections
  }
}
```

Prefer the `SURFACES` constant over the bare `'drawer'` / `'widget'` strings — it's exported specifically to survive future renames.

Apps that don't need surface-aware behavior can ignore `environment.surface` entirely — most components render the same regardless of surface.

## Widget Auto-Height with `<WidgetView>`

Wrap your widget's root element in `<WidgetView>` and the slot in the dashboard grid will automatically fit your content's height. No aspect ratio needed:

```jsx
import { WidgetView, Text, Stack } from '@teachfloor/extension-kit'

const LearningStreakWidget = () => (
  <WidgetView sx={{ padding: 16 }}>
    <Stack spacing="xs">
      <Text size="lg" weight={700}>7-day streak</Text>
      <Text size="sm" color="dimmed">Keep going — you're on fire.</Text>
    </Stack>
  </WidgetView>
)
```

Under the hood, `WidgetView` observes its own DOM height and emits it to the host over RPC. The host applies the reported height to the slot's container, so a widget with sparse content stays compact and a widget with a scrolling list grows to match. The same mechanism powers `<SettingsView>`.

**Admin override:** if the dashboard admin explicitly picks an aspect ratio when configuring the widget slot, that ratio wins over the emitted height. Auto-height is the default; forced ratios are the escape hatch.

## Multiple Widgets at the Same Viewport

A single app bundle can register multiple widgets at the same viewport as long as each has a distinct `widget.id`. This lets one deploy provide, say, a *Course Progress* widget and a *Learning Streak* widget from the same codebase.

Kit picks each widget's component using `environment.view.id`:

```jsx
import { useExtensionContext, SURFACES } from '@teachfloor/extension-kit'

import CourseProgressWidget from './widgets/CourseProgressWidget'
import LearningStreakWidget from './widgets/LearningStreakWidget'

const WIDGETS = {
  course_progress: CourseProgressWidget,
  learning_streak: LearningStreakWidget,
}

const App = () => {
  const { environment } = useExtensionContext()

  if (environment.surface === SURFACES.WIDGET) {
    const Component = WIDGETS[environment.view.id]
    return Component ? <Component /> : null
  }

  return <DrawerView />
}
```

In most cases you can skip this dispatch entirely and let `ExtensionViewLoader` (from `@teachfloor/extension-kit`) resolve the right component by mapping `widget.id` to your components — see the [Extension Kit Components](./extension-kit/components) chapter.

### Runtime `environment.viewport` is always the actual route

A widget declared as `viewport: '*'` that lands on the dashboard receives `environment.viewport === 'teachfloor.dashboard.dashboard.detail'` at runtime — never `'*'`. Wildcards are declaration scope; the runtime value is always concrete. This lets your widget code react to *where it landed* (fetch course-specific data on a course page, org-wide data on the main dashboard) without conflating declaration with location.

## CLI Helpers

The Teachfloor CLI scaffolds drawer views and widgets with the correct manifest shape:

```bash
# Add a new drawer view (prompts for viewport + component name)
teachfloor apps add view

# Add a new widget (prompts for viewport, widget id, name, description, component)
teachfloor apps add widget

# Remove a widget by id
teachfloor apps remove widget
```

See the [CLI reference](./references/cli) for the full flag list.

## Continue to

- [Extension Kit Components](./extension-kit/components) — the UI primitives you'll use inside your surfaces, including `<WidgetView>` and `<SettingsView>`
- [Extension Kit Integration](./extension-kit/integration) — surface-agnostic APIs (events, storage, navigation) that work identically across drawer and widget
