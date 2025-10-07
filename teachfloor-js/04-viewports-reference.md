Viewports Reference
Viewports in Teachfloor
A viewport in Teachfloor specifies a particular page or view within the Teachfloor dashboard. Each viewport is uniquely identified by a viewport ID , which is dynamically generated based on the current path within the application.

Viewports allow your custom page, integrated through the Teachfloor SDK, to understand and react to the specific context of the Teachfloor environment. By subscribing to viewport changes, your application can execute different logic depending on which part of the Teachfloor dashboard the user is currently interacting with.

A viewport in the Teachfloor platform can provide an objectContext object, which gives you additional context about the Teachfloor object associated with the current page. This is particularly useful when you need to interact with specific objects, such as courses, modules, or elements, directly within your custom implementation.

Available Viewports
The following table lists all the viewports available within the Teachfloor platform, along with their corresponding paths.

VIEWPORT ID

PATHS

OBJECT TYPE

teachfloor.dashboard.course.list

/:organization/courses

null

teachfloor.dashboard.course.detail

/:organization/courses/:course

course

teachfloor.dashboard.course.module.detail

/:organization/courses/:course/modules/:module

course module

teachfloor.dashboard.course.element.detail

/:organization/courses/:course/modules/:module/elements/:element

course module element

teachfloor.dashboard.community.post.list

/:organization/community/:community

null

teachfloor.dashboard.community.member.list

/:organization/community/:community/members

null

teachfloor.dashboard.settings.general.detail

/:organization/settings/general

null

teachfloor.dashboard.settings.customization.detail

/:organization/settings/customization

null

teachfloor.dashboard.settings.team.list

/:organization/settings/team

null

teachfloor.dashboard.settings.billing.detail

/:organization/settings/billing

null

teachfloor.dashboard.settings.integration.list

/:organization/settings/integrations

null

teachfloor.dashboard.settings.notification.list

/:organization/settings/notifications

null

teachfloor.dashboard.settings.custom-field.list

/:organization/settings/custom-fields

null

teachfloor.dashboard.account.detail

/:organization/account

null

teachfloor.dashboard.learner.list

/:organization/learners

null

teachfloor.dashboard.payment.list

/:organization/payments

null