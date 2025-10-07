# Viewports Reference

## Overview

A viewport identifies a specific page or view within the Teachfloor dashboard. Use viewports to execute context-specific logic based on where users are in the platform.

## Object Context

Some viewports provide an `objectContext` object containing details about the current page's associated objects (courses, modules, elements).

## Available Viewports

| Viewport ID | Path | Object Types |
|-------------|------|--------------|
| `teachfloor.dashboard.course.list` | `/:organization/courses` | - |
| `teachfloor.dashboard.course.detail` | `/:organization/courses/:course` | course |
| `teachfloor.dashboard.course.module.detail` | `/:organization/courses/:course/modules/:module` | course, module |
| `teachfloor.dashboard.course.element.detail` | `/:organization/courses/:course/modules/:module/elements/:element` | course, module, element |
| `teachfloor.dashboard.community.post.list` | `/:organization/community/:community` | - |
| `teachfloor.dashboard.community.member.list` | `/:organization/community/:community/members` | - |
| `teachfloor.dashboard.settings.general.detail` | `/:organization/settings/general` | - |
| `teachfloor.dashboard.settings.customization.detail` | `/:organization/settings/customization` | - |
| `teachfloor.dashboard.settings.team.list` | `/:organization/settings/team` | - |
| `teachfloor.dashboard.settings.billing.detail` | `/:organization/settings/billing` | - |
| `teachfloor.dashboard.settings.integration.list` | `/:organization/settings/integrations` | - |
| `teachfloor.dashboard.settings.notification.list` | `/:organization/settings/notifications` | - |
| `teachfloor.dashboard.settings.custom-field.list` | `/:organization/settings/custom-fields` | - |
| `teachfloor.dashboard.account.detail` | `/:organization/account` | - |
| `teachfloor.dashboard.learner.list` | `/:organization/learners` | - |
| `teachfloor.dashboard.payment.list` | `/:organization/payments` | - |

## Usage Example

```javascript
API.on('environment.viewport.changed', (viewport, objectContext) => {
  if (viewport === 'teachfloor.dashboard.course.detail') {
    // Access course object
    const course = objectContext.course;
    console.log('Current course:', course);
  }
});
```
