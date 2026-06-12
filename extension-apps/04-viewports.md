# Viewports System

Viewports define where your app displays within the Teachfloor platform.

## What is a Viewport?

A viewport is a specific location in the Teachfloor dashboard where your app renders its UI. Each viewport corresponds to a page or section of the platform.

### Viewport Structure

Viewports follow a hierarchical naming pattern:

```
teachfloor.{surface}.{resource}.{view-type}
```

**Example**: `teachfloor.dashboard.course.detail`
- `teachfloor`: Platform namespace
- `dashboard`: Surface where the view renders
- `course`: Resource being viewed
- `detail`: View type

## Available Viewports

### Quick Reference Table

| Viewport | Page | Path Pattern | Common Use Cases |
|----------|------|--------------|------------------|
| **Course Management** |
| `teachfloor.dashboard.course.list` | Course listing | `/:org/courses` | Course directory widgets, filters, quick actions |
| `teachfloor.dashboard.course.detail` | Course detail | `/:org/courses/:id` | Course analytics, notes, supplementary materials |
| `teachfloor.dashboard.course.module.list` | Module list | `/:org/courses/:id/modules` | Module overview, navigation tools |
| `teachfloor.dashboard.course.module.detail` | Module detail | `/:org/courses/:id/modules/:id` | Module tools, notes, progress tracking |
| `teachfloor.dashboard.course.element.detail` | Element detail | `/:org/courses/:id/modules/:id/elements/:id` | Element annotations, supplementary content |
| `teachfloor.dashboard.course.assessment.list` | Assessment list | `/:org/courses/:id/assessments` | Assessment overview, quiz management |
| `teachfloor.dashboard.course.progress.detail` | Course progress | `/:org/courses/:id/progress` | Progress tracking, completion status |
| `teachfloor.dashboard.course.calendar.detail` | Course calendar | `/:org/courses/:id/calendar` | Schedule view, deadline tracking |
| `teachfloor.dashboard.course.member.list` | Course members | `/:org/courses/:id/directory` | Student list, member management |
| **Community** |
| `teachfloor.dashboard.community.overview` | Community home (cross-channel feed) | `/:org/community` | Cross-channel feed widgets, suggested content, digest tools |
| `teachfloor.dashboard.community.channel.detail` | Single channel page | `/:org/community/:channelId` | Social features, engagement tools, moderation |
| `teachfloor.dashboard.community.post.detail` | Community post detail | `/:org/community/:channelId/posts/:postId` | Post sidebars, reaction tools, translation |
| `teachfloor.dashboard.community.member.list` | Community members | `/:org/community/:channelId/members` | Member profiles, networking tools, search |
| **Settings** |
| `teachfloor.dashboard.settings.general.detail` | General settings | `/:org/settings/general` | Organization-wide integrations, preferences |
| `teachfloor.dashboard.settings.customization.domain.detail` | Customization · Domain | `/:org/settings/customization/domain` | Domain configuration widgets, DNS helpers |
| `teachfloor.dashboard.settings.customization.appearance.detail` | Customization · Appearance | `/:org/settings/customization/appearance` | Theme tools, brand previews |
| `teachfloor.dashboard.settings.customization.smtp.detail` | Customization · SMTP | `/:org/settings/customization/smtp` | Email-delivery diagnostics |
| `teachfloor.dashboard.settings.customization.checkout.detail` | Customization · Checkout | `/:org/settings/customization/checkout` | Checkout experiments, payment tweaks |
| `teachfloor.dashboard.settings.customization.login.detail` | Customization · Login | `/:org/settings/customization/login` | Login-screen branding |
| `teachfloor.dashboard.settings.customization.labels.detail` | Customization · Labels | `/:org/settings/customization/labels` | Custom terminology tools |
| `teachfloor.dashboard.settings.customization.achievements.detail` | Customization · Achievements | `/:org/settings/customization/achievements` | Gamification configuration |
| `teachfloor.dashboard.settings.customization.profile-layout.detail` | Customization · Profile layout | `/:org/settings/customization/profile-layout` | Profile field arrangement tools |
| `teachfloor.dashboard.settings.team.list` | Team management | `/:org/settings/team` | Team collaboration tools, role management |
| `teachfloor.dashboard.settings.billing.detail` | Billing | `/:org/settings/billing` | Usage analytics, cost tracking |
| `teachfloor.dashboard.settings.integration.list` | Integrations | `/:org/settings/integrations` | Third-party integrations, API tools |
| `teachfloor.dashboard.settings.notification.list` | Notifications | `/:org/settings/notifications` | Custom notification rules, digest tools |
| `teachfloor.dashboard.settings.custom-field.list` | Custom fields | `/:org/settings/custom-fields` | Custom field management, data import tools |
| `teachfloor.dashboard.settings.branch.list` | Branches | `/:org/settings/branches` | Branch management, multi-tenant tools |
| `teachfloor.dashboard.settings.sanction.list` | Sanctions | `/:org/settings/sanctions` | Restriction policies, moderation tooling |
| `teachfloor.dashboard.settings.import.list` | Import data | `/:org/settings/import` | Bulk import tools, data migration |
| `teachfloor.dashboard.settings.app.list` | Apps | `/:org/settings/apps` | Installed apps, app marketplace |
| **User Management** |
| `teachfloor.dashboard.account.detail` | User account | `/:org/account` | Personal tools, user preferences, integrations |
| `teachfloor.dashboard.profile.detail` | User profile | `/:org/users/:userId` | Profile widgets, member insights |
| `teachfloor.dashboard.learner.list` | Learners | `/:org/learners` | Learner analytics, bulk actions, import tools |
| **Commerce** |
| `teachfloor.dashboard.payment.list` | Payments | `/:org/payments` | Payment analytics, invoicing tools |
| **Messaging** |
| `teachfloor.dashboard.messaging.thread.detail` | Thread | `/:org/messaging/threads/:thread` | Thread sidebars, translation tools, message-level integrations |
| **Workspace** |
| `teachfloor.dashboard.analytics.detail` | Analytics | `/:org/analytics` | Custom dashboards, exports, alerts |
| `teachfloor.dashboard.library.detail` | Library drive | `/:org/library/:drive` | Asset browsers, content pickers |
| `teachfloor.dashboard.automation.list` | Automations | `/:org/automations` | Workflow inspectors, audit tools |
| `teachfloor.dashboard.getstarted.detail` | Get started | `/:org/getstarted` | Onboarding hints, checklists |
| **App Settings** |
| `settings` | App settings | `/:org/settings/apps/:appId` | App configuration, user preferences |

### Course Management

#### `teachfloor.dashboard.course.list`
**Displays on**: Course listing page
**Path**: `/:organization/courses`
**Use cases**: Course directory widgets, filters, quick actions

```json
{
  "viewport": "teachfloor.dashboard.course.list",
  "component": "CourseListView"
}
```

#### `teachfloor.dashboard.course.detail`
**Displays on**: Individual course page
**Path**: `/:organization/courses/:courseId`
**Use cases**: Course analytics, notes, supplementary materials

```json
{
  "viewport": "teachfloor.dashboard.course.detail",
  "component": "CourseDetailView"
}
```

#### `teachfloor.dashboard.course.module.detail`
**Displays on**: Module detail page
**Path**: `/:organization/courses/:courseId/modules/:moduleId`
**Use cases**: Module-specific tools, notes, progress tracking

```json
{
  "viewport": "teachfloor.dashboard.course.module.detail",
  "component": "ModuleDetailView"
}
```

#### `teachfloor.dashboard.course.element.detail`
**Displays on**: Learning element page (assignments, videos, etc.)
**Path**: `/:organization/courses/:courseId/modules/:moduleId/elements/:elementId`
**Use cases**: Element annotations, supplementary content, tools

```json
{
  "viewport": "teachfloor.dashboard.course.element.detail",
  "component": "ElementDetailView"
}
```

#### `teachfloor.dashboard.course.module.list`
**Displays on**: Module list page
**Path**: `/:organization/courses/:courseId/modules`
**Use cases**: Module overview, navigation tools

```json
{
  "viewport": "teachfloor.dashboard.course.module.list",
  "component": "ModuleListView"
}
```

#### `teachfloor.dashboard.course.assessment.list`
**Displays on**: Assessment list page
**Path**: `/:organization/courses/:courseId/assessments`
**Use cases**: Assessment overview, quiz management

```json
{
  "viewport": "teachfloor.dashboard.course.assessment.list",
  "component": "AssessmentListView"
}
```

#### `teachfloor.dashboard.course.progress.detail`
**Displays on**: Course progress page
**Path**: `/:organization/courses/:courseId/progress`
**Use cases**: Progress tracking, completion status

```json
{
  "viewport": "teachfloor.dashboard.course.progress.detail",
  "component": "CourseProgressView"
}
```

#### `teachfloor.dashboard.course.calendar.detail`
**Displays on**: Course calendar page
**Path**: `/:organization/courses/:courseId/calendar`
**Use cases**: Schedule view, deadline tracking

```json
{
  "viewport": "teachfloor.dashboard.course.calendar.detail",
  "component": "CourseCalendarView"
}
```

#### `teachfloor.dashboard.course.member.list`
**Displays on**: Course members page
**Path**: `/:organization/courses/:courseId/directory`
**Use cases**: Learner list, member management

```json
{
  "viewport": "teachfloor.dashboard.course.member.list",
  "component": "CourseMemberListView"
}
```

### Community

#### `teachfloor.dashboard.community.overview`
**Displays on**: Community home (cross-channel feed) — the `/community` landing page
**Path**: `/:organization/community`
**Use cases**: Cross-channel feed widgets, suggested content, digest tools

The workspace-level home that aggregates posts across every channel the viewer can see. Stripe-style `overview` rather than `feed`/`list` because no single resource is being listed — distinct from `community.channel.detail`, which targets a single channel.

```json
{
  "viewport": "teachfloor.dashboard.community.overview",
  "component": "CommunityOverviewView"
}
```

#### `teachfloor.dashboard.community.channel.detail`
**Displays on**: Single channel page
**Path**: `/:organization/community/:channelId`
**Use cases**: Social features, engagement tools, moderation

The primary resource on this page is the Channel itself — posts are its content — so the viewport follows Stripe's `{resource}.detail` convention.

```json
{
  "viewport": "teachfloor.dashboard.community.channel.detail",
  "component": "CommunityChannelView"
}
```

#### `teachfloor.dashboard.community.post.detail`
**Displays on**: Individual community post page
**Path**: `/:organization/community/:channelId/posts/:postId`
**Use cases**: Post sidebars, reaction tools, translation, attached references

```json
{
  "viewport": "teachfloor.dashboard.community.post.detail",
  "component": "CommunityPostView"
}
```

#### `teachfloor.dashboard.community.member.list`
**Displays on**: Community members directory
**Path**: `/:organization/community/:channelId/members`
**Use cases**: Member profiles, networking tools, search

```json
{
  "viewport": "teachfloor.dashboard.community.member.list",
  "component": "CommunityMembersView"
}
```

### Settings

#### `teachfloor.dashboard.settings.general.detail`
**Displays on**: General settings page
**Path**: `/:organization/settings/general`
**Use cases**: Organization-wide integrations, preferences

```json
{
  "viewport": "teachfloor.dashboard.settings.general.detail",
  "component": "GeneralSettingsView"
}
```

#### `teachfloor.dashboard.settings.customization.domain.detail`
**Displays on**: Customization · Domain subtab
**Path**: `/:organization/settings/customization/domain`
**Use cases**: Domain configuration widgets, DNS helpers

```json
{
  "viewport": "teachfloor.dashboard.settings.customization.domain.detail",
  "component": "CustomizationDomainView"
}
```

#### `teachfloor.dashboard.settings.customization.appearance.detail`
**Displays on**: Customization · Appearance subtab
**Path**: `/:organization/settings/customization/appearance`
**Use cases**: Theme tools, brand previews

```json
{
  "viewport": "teachfloor.dashboard.settings.customization.appearance.detail",
  "component": "CustomizationAppearanceView"
}
```

#### `teachfloor.dashboard.settings.customization.smtp.detail`
**Displays on**: Customization · SMTP subtab
**Path**: `/:organization/settings/customization/smtp`
**Use cases**: Email-delivery diagnostics

```json
{
  "viewport": "teachfloor.dashboard.settings.customization.smtp.detail",
  "component": "CustomizationSmtpView"
}
```

#### `teachfloor.dashboard.settings.customization.checkout.detail`
**Displays on**: Customization · Checkout subtab
**Path**: `/:organization/settings/customization/checkout`
**Use cases**: Checkout experiments, payment tweaks

```json
{
  "viewport": "teachfloor.dashboard.settings.customization.checkout.detail",
  "component": "CustomizationCheckoutView"
}
```

#### `teachfloor.dashboard.settings.customization.login.detail`
**Displays on**: Customization · Login subtab
**Path**: `/:organization/settings/customization/login`
**Use cases**: Login-screen branding

```json
{
  "viewport": "teachfloor.dashboard.settings.customization.login.detail",
  "component": "CustomizationLoginView"
}
```

#### `teachfloor.dashboard.settings.customization.labels.detail`
**Displays on**: Customization · Labels subtab
**Path**: `/:organization/settings/customization/labels`
**Use cases**: Custom terminology tools

```json
{
  "viewport": "teachfloor.dashboard.settings.customization.labels.detail",
  "component": "CustomizationLabelsView"
}
```

#### `teachfloor.dashboard.settings.customization.achievements.detail`
**Displays on**: Customization · Achievements subtab (visible only when the workspace has the achievements feature enabled)
**Path**: `/:organization/settings/customization/achievements`
**Use cases**: Gamification configuration

```json
{
  "viewport": "teachfloor.dashboard.settings.customization.achievements.detail",
  "component": "CustomizationAchievementsView"
}
```

#### `teachfloor.dashboard.settings.customization.profile-layout.detail`
**Displays on**: Customization · Profile layout subtab (visible only to owners and admins)
**Path**: `/:organization/settings/customization/profile-layout`
**Use cases**: Profile field arrangement tools

```json
{
  "viewport": "teachfloor.dashboard.settings.customization.profile-layout.detail",
  "component": "CustomizationProfileLayoutView"
}
```

#### `teachfloor.dashboard.settings.team.list`
**Displays on**: Team management page
**Path**: `/:organization/settings/team`
**Use cases**: Team collaboration tools, role management

```json
{
  "viewport": "teachfloor.dashboard.settings.team.list",
  "component": "TeamSettingsView"
}
```

#### `teachfloor.dashboard.settings.billing.detail`
**Displays on**: Billing settings page
**Path**: `/:organization/settings/billing`
**Use cases**: Usage analytics, cost tracking

```json
{
  "viewport": "teachfloor.dashboard.settings.billing.detail",
  "component": "BillingView"
}
```

#### `teachfloor.dashboard.settings.integration.list`
**Displays on**: Integrations page
**Path**: `/:organization/settings/integrations`
**Use cases**: Third-party integrations, API tools

```json
{
  "viewport": "teachfloor.dashboard.settings.integration.list",
  "component": "IntegrationView"
}
```

#### `teachfloor.dashboard.settings.notification.list`
**Displays on**: Notification settings page
**Path**: `/:organization/settings/notifications`
**Use cases**: Custom notification rules, digest tools

```json
{
  "viewport": "teachfloor.dashboard.settings.notification.list",
  "component": "NotificationView"
}
```

#### `teachfloor.dashboard.settings.custom-field.list`
**Displays on**: Custom fields settings page
**Path**: `/:organization/settings/custom-fields`
**Use cases**: Custom field management, data import tools

```json
{
  "viewport": "teachfloor.dashboard.settings.custom-field.list",
  "component": "CustomFieldView"
}
```

#### `teachfloor.dashboard.settings.branch.list`
**Displays on**: Branches settings page
**Path**: `/:organization/settings/branches`
**Use cases**: Branch management, multi-tenant tools

```json
{
  "viewport": "teachfloor.dashboard.settings.branch.list",
  "component": "BranchListView"
}
```

#### `teachfloor.dashboard.settings.sanction.list`
**Displays on**: Sanctions settings page (visible only when the workspace has the sanctions feature enabled)
**Path**: `/:organization/settings/sanctions`
**Use cases**: Restriction policies, moderation tooling

```json
{
  "viewport": "teachfloor.dashboard.settings.sanction.list",
  "component": "SanctionListView"
}
```

#### `teachfloor.dashboard.settings.import.list`
**Displays on**: Import data page
**Path**: `/:organization/settings/import`
**Use cases**: Bulk import tools, data migration

```json
{
  "viewport": "teachfloor.dashboard.settings.import.list",
  "component": "ImportView"
}
```

#### `teachfloor.dashboard.settings.app.list`
**Displays on**: Apps settings page
**Path**: `/:organization/settings/apps`
**Use cases**: Installed apps, app marketplace

```json
{
  "viewport": "teachfloor.dashboard.settings.app.list",
  "component": "AppListView"
}
```

### User Management

#### `teachfloor.dashboard.account.detail`
**Displays on**: User account settings page
**Path**: `/:organization/account`
**Use cases**: Personal tools, user preferences, integrations

```json
{
  "viewport": "teachfloor.dashboard.account.detail",
  "component": "AccountView"
}
```

#### `teachfloor.dashboard.profile.detail`
**Displays on**: User profile page
**Path**: `/:organization/users/:userId`
**Use cases**: Profile widgets, member insights, supplementary user info

```json
{
  "viewport": "teachfloor.dashboard.profile.detail",
  "component": "ProfileDetailView"
}
```

#### `teachfloor.dashboard.learner.list`
**Displays on**: Learners management page
**Path**: `/:organization/learners`
**Use cases**: Learner analytics, bulk actions, import tools

```json
{
  "viewport": "teachfloor.dashboard.learner.list",
  "component": "LearnerListView"
}
```

### Commerce

#### `teachfloor.dashboard.payment.list`
**Displays on**: Payments page
**Path**: `/:organization/payments`
**Use cases**: Payment analytics, invoicing tools

```json
{
  "viewport": "teachfloor.dashboard.payment.list",
  "component": "PaymentView"
}
```

### Messaging

#### `teachfloor.dashboard.messaging.thread.detail`
**Displays on**: Messaging thread page
**Path**: `/:organization/messaging/threads/:threadId`
**Use cases**: Thread sidebars, translation tools, message-level integrations

```json
{
  "viewport": "teachfloor.dashboard.messaging.thread.detail",
  "component": "MessagingThreadView"
}
```

### Workspace

#### `teachfloor.dashboard.analytics.detail`
**Displays on**: Analytics page
**Path**: `/:organization/analytics`
**Use cases**: Custom dashboards, exports, alerts

```json
{
  "viewport": "teachfloor.dashboard.analytics.detail",
  "component": "AnalyticsView"
}
```

#### `teachfloor.dashboard.library.detail`
**Displays on**: Library drive page
**Path**: `/:organization/library/:drive`
**Use cases**: Asset browsers, content pickers

```json
{
  "viewport": "teachfloor.dashboard.library.detail",
  "component": "LibraryDriveView"
}
```

#### `teachfloor.dashboard.automation.list`
**Displays on**: Automations page
**Path**: `/:organization/automations`
**Use cases**: Workflow inspectors, audit tools

```json
{
  "viewport": "teachfloor.dashboard.automation.list",
  "component": "AutomationListView"
}
```

#### `teachfloor.dashboard.getstarted.detail`
**Displays on**: Get started page
**Path**: `/:organization/getstarted`
**Use cases**: Onboarding hints, checklists

```json
{
  "viewport": "teachfloor.dashboard.getstarted.detail",
  "component": "GetStartedView"
}
```

### App Settings

#### `settings`
**Displays on**: Your app's settings page
**Path**: `/:organization/settings/apps/:appId`
**Use cases**: App configuration, user preferences

```json
{
  "viewport": "settings",
  "component": "AppSettingsView"
}
```

:::info
The `settings` viewport provides a dedicated settings page for your app.
:::

## Viewport Context

When your app renders in a viewport, it receives context about the current page:

```javascript
import { useExtensionContext } from '@teachfloor/extension-kit'

const MyView = () => {
  const { environment } = useExtensionContext()

  console.log(environment.viewport)  // "teachfloor.dashboard.course.detail"
  console.log(environment.path)      // "/myorg/courses/123"

  return <div>Current viewport: {environment.viewport}</div>
}
```

### Available Context

```typescript
environment: {
  initialized: boolean      // SDK ready state
  viewport: string         // Current viewport ID
  path: string            // Current URL path
}
```

## Viewport Matching

### How Matching Works

1. User navigates to a page (e.g., `/myorg/courses`)
2. Platform determines viewport (`teachfloor.dashboard.course.list`)
3. Platform finds apps with views matching the exact viewport
4. Matching app views are rendered in their designated positions

:::caution
Viewports require exact string matches. Wildcard patterns are not supported.
:::

### Example Scenario

Your app manifest:
```json
{
  "ui_extension": {
    "views": [
      {
        "viewport": "teachfloor.dashboard.course.detail",
        "component": "CourseDetailView"
      },
      {
        "viewport": "teachfloor.dashboard.course.list",
        "component": "CourseListView"
      },
      {
        "viewport": "settings",
        "component": "SettingsView"
      }
    ]
  }
}
```

**User visits course detail page (`/org/courses/123`)**:
- ✅ `CourseDetailView` renders (viewport: `teachfloor.dashboard.course.detail`)
- ❌ `CourseListView` does not render (different viewport)

**User visits course list page (`/org/courses`)**:
- ✅ `CourseListView` renders (viewport: `teachfloor.dashboard.course.list`)
- ❌ `CourseDetailView` does not render (different viewport)

**User visits app settings page (`/org/settings/apps/yourapp`)**:
- ✅ `SettingsView` renders (viewport: `settings`)

## View Component Structure

### Basic View Component

```jsx
import React from 'react'
import { Container, Text, useExtensionContext } from '@teachfloor/extension-kit'

const CourseListView = () => {
  const { environment, userContext } = useExtensionContext()

  return (
    <Container>
      <Text>Hello {userContext.full_name}</Text>
      <Text size="sm" c="dimmed">
        You're viewing: {environment.viewport}
      </Text>
    </Container>
  )
}

export default CourseListView
```

### Viewport-Aware Component

```jsx
import React from 'react'
import { useExtensionContext } from '@teachfloor/extension-kit'

const SmartView = () => {
  const { environment } = useExtensionContext()

  // Render different UI based on viewport
  if (environment.viewport.includes('course.detail')) {
    return <CourseSpecificUI />
  }

  if (environment.viewport.includes('settings')) {
    return <SettingsUI />
  }

  return <DefaultUI />
}
```

### Dynamic View Loader

```jsx
import React from 'react'
import { ExtensionViewLoader } from '@teachfloor/extension-kit'
import manifest from '../../teachfloor-app.json'

const App = () => {
  return (
    <ExtensionViewLoader
      manifest={manifest}
      componentResolver={(componentName) => import(`./${componentName}`)}
    />
  )
}

export default App
```

## Adding Views to Your App

### Using CLI

```bash
teachfloor apps add view
```

1. Select viewport from list
2. Enter component name (PascalCase)
3. Choose whether to generate example code

### Manual Addition

1. Create component file:
```bash
touch src/views/CourseListView.jsx
```

2. Update manifest:
```json
{
  "ui_extension": {
    "views": [
      {
        "viewport": "teachfloor.dashboard.course.list",
        "component": "CourseListView"
      }
    ]
  }
}
```

3. Implement component:
```jsx
import React from 'react'
import { Container, Text } from '@teachfloor/extension-kit'

const CourseListView = () => {
  return (
    <Container>
      <Text>My Course List Widget</Text>
    </Container>
  )
}

export default CourseListView
```

## Removing Views

### Using CLI

```bash
teachfloor apps remove view
```

### Manual Removal

1. Delete component file
2. Remove from manifest:
```json
{
  "ui_extension": {
    "views": [
      // Remove the view object
    ]
  }
}
```

## Next Steps

→ Continue to [Extension Kit Components](./extension-kit/components)

## Additional Resources

- [Best Practices](/docs/apps/references/best-practices) - Viewport selection and performance optimization
- [Troubleshooting Guide](/docs/apps/references/troubleshooting) - Viewport issues and debugging
- [Examples](/docs/apps/references/examples) - Multi-viewport app examples
- [Extension Kit Integration](./extension-kit/integration)
