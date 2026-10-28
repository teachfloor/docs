# Tools Reference

The 51 tools the MCP server offers, grouped by what they act on.

Each tool is purpose-built for a single operation, and every name is prefixed `teachfloor_` on the wire — `list_courses` is called as `teachfloor_list_courses`.

The **Access** column says how the assistant treats each one:

- **Read** — runs without asking you each time. Requires `mcp:read`.
- **Write** — adds something. Asks for confirmation. Requires `mcp:write`.
- **Modifies** — changes or removes existing data. Asks for confirmation. Requires `mcp:write`.

## Courses

| Tool | Access | |
|---|---|---|
| `list_courses` | Read | Every course in the organization, newest first |
| `get_course` | Read | One course by id |
| `search_courses` | Read | Courses matching a query |
| `get_course_modules` | Read | A course's modules, in order |
| `get_course_elements` | Read | Every element in a course, across all modules |
| `get_course_members` | Read | Members enrolled in a course |
| `get_course_member` | Read | One member's enrollment, with their course data |
| `get_course_custom_fields` | Read | Custom field definitions for course members |
| `create_course` | Write | |
| `join_course` | Write | Enrolls a member |
| `update_course` | Modifies | |
| `update_course_member` | Modifies | Custom field values on an enrollment |
| `delete_course` | Modifies | |
| `revoke_course_access` | Modifies | Removes a member's enrollment |

## Modules

| Tool | Access | |
|---|---|---|
| `list_modules` | Read | |
| `get_module` | Read | |
| `search_modules` | Read | |
| `get_module_elements` | Read | A module's elements, in order |
| `create_module` | Write | |
| `update_module` | Modifies | |
| `delete_module` | Modifies | |

## Elements

An element is a single piece of course content — a lesson, a video, a quiz, a discussion. Modules group them.

| Tool | Access | |
|---|---|---|
| `list_elements` | Read | |
| `get_element` | Read | |
| `search_elements` | Read | |
| `get_element_activities` | Read | Completion and submission records for one element |
| `create_element` | Write | |
| `update_element` | Modifies | |
| `delete_element` | Modifies | |

## Members

| Tool | Access | |
|---|---|---|
| `list_members` | Read | Every member in the organization, newest first |
| `get_member` | Read | |
| `search_members` | Read | |
| `get_member_courses` | Read | Courses a member is enrolled in |
| `get_member_teams` | Read | Teams a member belongs to |
| `list_custom_fields` | Read | Custom field definitions for members |
| `create_member` | Write | |
| `update_member` | Modifies | |

## Teams

Teams are named groups of members. Assigning a course to a team enrolls every member of that team in it.

| Tool | Access | |
|---|---|---|
| `list_teams` | Read | |
| `get_team` | Read | |
| `search_teams` | Read | |
| `get_team_members` | Read | |
| `get_team_courses` | Read | |
| `create_team` | Write | |
| `assign_team_member` | Write | Adds a member, enrolling them in the team's courses |
| `assign_team_course` | Write | Enrolls every member of the team |
| `update_team` | Modifies | |
| `delete_team` | Modifies | |
| `unassign_team_member` | Modifies | |
| `unassign_team_course` | Modifies | |

## Activities

Activities are completion and submission records.

| Tool | Access | |
|---|---|---|
| `list_activities` | Read | Filterable by course, module, element or member |
| `get_activity` | Read | |

## Uploads

| Tool | Access | |
|---|---|---|
| `upload_file` | Write | Takes base64 content, returns a stored URL |

The returned URL can then be set on an element — a video, a SCORM package, a file — or used as a member avatar.

## Permissions

Tools are scoped twice. The OAuth scope decides whether a token may call a tool at all, and the member's own Teachfloor permissions decide what that call can reach: a token approved by an assistant or a lecturer sees only the courses and members that person sees.
