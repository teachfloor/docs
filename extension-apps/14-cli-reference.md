# CLI Reference

Complete reference for Teachfloor CLI commands and options.

## Installation

```bash
npm install -g @teachfloor/teachfloor-cli
```

## Command Reference

### Quick Reference Table

| Command | Description | Requires Auth | Requires App Folder |
|---------|-------------|---------------|---------------------|
| **Global** |
| `teachfloor version` | Display CLI version | No | No |
| `teachfloor login` | Authenticate with Teachfloor | No | No |
| `teachfloor logout` | Log out from account | No | No |
| `teachfloor whoami` | Show current user and org | Yes | No |
| **App Management** |
| `teachfloor apps create <name>` | Create new app | Yes | No |
| `teachfloor apps start` | Start dev server | Yes | Yes |
| `teachfloor apps upload` | Build and upload app | Yes | Yes |
| **View Management** |
| `teachfloor apps add view` | Add view to app | Yes | Yes |
| `teachfloor apps remove view` | Remove view from app | Yes | Yes |
| `teachfloor apps add settings` | Add settings view | Yes | Yes |
| `teachfloor apps add widget` | Add widget to app | Yes | Yes |
| `teachfloor apps remove widget` | Remove widget from app | Yes | Yes |
| **Permission Management** |
| `teachfloor apps grant permission` | Add permission | Yes | Yes |
| `teachfloor apps revoke permission` | Remove permission | Yes | Yes |
| **Webhook & OAuth** |
| `teachfloor apps set webhook` | Configure webhook URL and events | Yes | Yes |
| `teachfloor apps remove webhook` | Remove webhook block | Yes | Yes |
| `teachfloor apps set oauth` | Configure OAuth grant type | Yes | Yes |
| `teachfloor apps remove oauth` | Remove OAuth block | Yes | Yes |
| **Distribution** |
| `teachfloor apps set distribution` | Set public/private | Yes | Yes |
| **Inspection** |
| `teachfloor apps show` | Print a spec-sheet summary of the current app | Yes | Yes |
| **Diagnostics** |
| `teachfloor apps doctor` | Diagnose common setup issues | Yes | No |

## Non-Interactive Mode

Every prompt in every command has a matching flag. Pass all the flags a command needs and the CLI runs end-to-end without asking questions — required for scripts, CI pipelines, and AI-driven workflows.

**Three triggers** (any one flips the CLI into non-interactive mode):

- `--nonInteractive` on the command (alias: `--non-interactive`, `--no-interactive`).
- Environment variable: `TF_NON_INTERACTIVE=1` or `CI=1`.
- Automatic: whenever stdin isn't a TTY (piped input, subprocess spawn, headless agent).

**In non-interactive mode**:

- A prompt whose flag is set → uses the flag value (validated).
- A prompt whose flag is missing but has a default → uses the default (validated).
- A prompt whose flag is missing and has no default → hard error: `Missing required input in non-interactive mode: pass --<flag> to set "<field>".`
- Invalid values (bad choice, failed regex, empty required string) → error naming the flag, with the underlying validator's message.

**Example** — scripted `apps create`:

```bash
teachfloor apps create my-app \
  --name "My App" \
  --description "A test app" \
  --version 1.0.0
```

No `--nonInteractive` needed when stdin isn't a TTY — piping / subprocess spawn triggers it automatically. Add the flag explicitly in wrapper scripts if you want fail-fast behavior regardless of shell context.

**Flag → prompt mapping** (per command):

| Command | Flags |
|---|---|
| `apps create <name>` | `--appId`, `--name`, `--description`, `--version` |
| `apps add view` | `--viewport`, `--componentName`, `--withExample`, `--overwrite` |
| `apps add settings` | `--componentName`, `--withExample` |
| `apps add widget` | `--viewport`, `--id`, `--name`, `--description`, `--componentName`, `--withExample`, `--overwrite` |
| `apps remove view` | `--viewport`, `--removeComponent` |
| `apps remove widget` | `--id`, `--removeComponent` |
| `apps grant permission` | `--permissionName`, `--explanation` |
| `apps revoke permission` | `--permissionName` |
| `apps set webhook` | `--url`, `--events` (repeatable, or comma-separated) |
| `apps set oauth` | `--oauthType` |
| `apps set distribution` | `--distributionType` |

`login` remains interactive-only (browser OAuth).

## Global Commands

### `teachfloor version`

Display CLI version and check for updates.

```bash
teachfloor version
```

**Output**:
```
teachfloor version 1.2.0
A newer version of the Teachfloor CLI is available: 1.3.0
```

---

### `teachfloor login`

Authenticate with your Teachfloor account.

```bash
teachfloor login
```

**Prompts**:
- Email address
- Password
- Organization (if you have multiple)

**What it does**:
1. Authenticates with your Teachfloor account
2. Stores credentials securely
3. Saves selected organization

**Example**:
```bash
$ teachfloor login
✔ Enter your email: john@example.com
✔ Enter your password: ••••••••
✔ Select an organization: My Organization
✓ Login successful!
```

---

### `teachfloor logout`

Log out from your Teachfloor account.

```bash
teachfloor logout
```

**What it does**:
1. Removes stored credentials
2. Clears organization selection

---

### `teachfloor whoami`

Display current authenticated user and organization.

```bash
teachfloor whoami
```

**Output**:
```
User: john@example.com
Organization: my-organization
```

---

## App Management

### `teachfloor apps create <app-name>`

Create a new Teachfloor app.

```bash
teachfloor apps create my-awesome-app
```

**Arguments**:
- `app-name`: Name of the folder to create

**Prompts** (interactive mode) / **Flags** (non-interactive):
- **App ID** — `--appId <value>` (alias `--id`; default: `<slug>-<timestamp>`)
- **Display Name** — `--name <value>` (required)
- **Description** — `--description <value>` (required)
- **Version** — `--version <value>` (semver, default: `1.0.0`)

**What it does**:
1. Creates app on the platform
2. Generates project structure with all necessary files
3. Installs dependencies

**Generated Structure**:
```
my-awesome-app/
├── src/
│   ├── index.js
│   └── views/
│       └── App.jsx
├── public/
│   └── index.html
├── teachfloor-app.json
├── package.json
├── webpack.config.js
└── tsconfig.json
```

**Example**:
```bash
$ teachfloor apps create notes-app
✔ App ID: notes-1234567890
✔ Display Name: Notes App
✔ Description: Take notes while learning
✔ Version: 1.0.0
✓ Creating app...
✓ Setting up app structure...
✓ Installing npm dependencies...
✓ App "Notes App" created successfully in "notes-app".

OAuth credentials for this app:
  Client ID:     9f8e7d6c-1234-4abc-9def-0123456789ab
  Client Secret: rN7pQ8E4fD0sUjLvX2mK5H1a3bT9wY6c
```

:::caution
**Save the Client Secret now.** It's shown once at create time and never returned by the API again — if you lose it, you can retrieve it from Developers → Apps → your app → **OAuth Client Secret** in the Teachfloor dashboard. The Client ID is also delivered inside every `app.installed` webhook payload; the Client Secret is not. See [OAuth](./oauth) for how to use these credentials for the refresh flow.
:::

---

### `teachfloor apps start`

Start development server for your app.

```bash
teachfloor apps start
```

**Options**:
- `-m, --manifest <path>`: Use custom manifest file

**What it does**:
1. Validates your app manifest
2. Uploads manifest to platform
3. Opens browser to install the app
4. Starts development server with auto-reload

**Requirements**:
- Must be run inside an app folder
- Must be logged in
- Version must not be approved/published

**Example**:
```bash
$ cd my-app
$ teachfloor apps start
✓ Manifest file updated
Install URL: https://app.teachfloor.com/myorg/courses?app=abc123@1.0.0
Starting development server...
webpack 5.x.x compiled successfully
```

**With custom manifest**:
```bash
teachfloor apps start --manifest teachfloor-app.dev.json
```

---

### `teachfloor apps upload`

Build and upload your app to the platform.

```bash
teachfloor apps upload
```

**What it does**:
1. Builds your app for production
2. Uploads bundled files to the platform
3. Creates a new version

**Requirements**:
- Must be run inside an app folder
- Must be logged in
- Version must not be already published

**Example**:
```bash
$ teachfloor apps upload
✓ Building the production bundle...
✓ Uploading files...
✓ App uploaded successfully.
```

---

## View Management

### `teachfloor apps add view`

Add a new view to your app.

```bash
teachfloor apps add view
```

**Prompts** (interactive mode) / **Flags** (non-interactive):
- **Select viewport** — `--viewport <id>` (must be one of the app's available viewports)
- **Component name** — `--componentName <PascalCase>` (alias `--component`; defaults to a name derived from the viewport)
- **Generate example** — `--withExample` (alias `--with-example`; default: `false`)
- **Overwrite existing file** — `--overwrite` (only prompted when the target file exists; default: `false`)

**What it does**:
1. Shows available viewports for your app
2. Creates component file in `src/views/`
3. Updates your app manifest

**Example**:
```bash
$ teachfloor apps add view
✔ Select the viewport for your view: teachfloor.dashboard.course.list
✔ Enter the name of your component: CourseListView
✔ Generate a "Getting Started" example view? Yes
✓ Component view created at src/views/CourseListView.jsx
✓ Manifest file updated
✓ View "CourseListView" added successfully under "teachfloor.dashboard.course.list".
```

**Generated Component**:
```jsx
import React from 'react'
import { Container, Text } from '@teachfloor/extension-kit'

const CourseListView = () => {
  return (
    <Container>
      <Text>CourseListView</Text>
    </Container>
  )
}

export default CourseListView
```

---

### `teachfloor apps remove view`

Remove a view from your app.

```bash
teachfloor apps remove view
```

**Prompts** (interactive mode) / **Flags** (non-interactive):
- **Select viewport** — `--viewport <id>` (must match an existing view in the manifest)
- **Delete component file too** — `--removeComponent` (alias `--remove-component`; default: `false`)

**What it does**:
1. Removes view from your app manifest
2. Note: Component file remains in `src/views/` (delete manually if needed)

**Example**:
```bash
$ teachfloor apps remove view
✔ Select the viewport to remove: teachfloor.dashboard.course.list
✓ View removed successfully.
```

---

### `teachfloor apps add settings`

Add a settings view to your app.

```bash
teachfloor apps add settings
```

**Prompts** (interactive mode) / **Flags** (non-interactive):
- **Component name** — `--componentName <PascalCase>` (alias `--component`; default: `AppSettings`)
- **Generate example** — `--withExample` (alias `--with-example`; default: `false`)

**What it does**:
1. Creates settings component in `src/views/`
2. Adds `settings` viewport to your manifest

**Example**:
```bash
$ teachfloor apps add settings
✔ Enter the name of your component: AppSettings
✔ Generate a "Getting Started" example view? Yes
✓ Settings view created at src/views/AppSettings.jsx
✓ Manifest file updated
```

---

### `teachfloor apps add widget`

Add a new widget to your app. See [Surfaces](./surfaces) for the concepts (widget id, `<WidgetView>`, admin picker).

```bash
teachfloor apps add widget
```

**Prompts** (interactive mode) / **Flags** (non-interactive):
- **Select viewport** — `--viewport <pattern>` (`"*"` for universal, or a concrete widget-hosting viewport)
- **Widget id** — `--id <slug>` (lowercase slug, `^[a-z][a-z0-9_]*$`, unique per app across all widget declarations)
- **Widget name** — `--name <string>` (≤60 chars; shown in the admin's widget picker and the app install-consent surfaces list)
- **Widget description** — `--description <string>` (≤200 chars; shown alongside the name in the picker)
- **Component name** — `--componentName <PascalCase>` (alias `--component`; defaults to `<PascalId>Widget` derived from the widget id — e.g. `streak_daily` → `StreakDailyWidget`)
- **Generate example** — `--withExample` (alias `--with-example`; default: `false`)
- **Overwrite existing file** — `--overwrite` (only prompted when the target file exists; default: `false`)

**What it does**:
1. Prompts for the widget's scoping viewport
2. Validates the widget id locally against ids already in your manifest (fails fast before the server round-trip)
3. Creates the component file in `src/views/`
4. Appends a `surface: "widget"` view entry to your manifest with the nested `widget: { id, name, description }` block

**Example**:
```bash
$ teachfloor apps add widget
✔ Select the viewport for your widget: *
✔ Enter the widget id (lowercase slug): learning_streak
✔ Enter the widget name: Learning Streak
✔ Enter the widget description: Current daily study streak with a 7-day heatmap.
✔ Enter the name of your component: LearningStreakWidget
✔ Generate a "Getting Started" example widget? Yes
✓ Component view created at src/views/LearningStreakWidget.jsx
✓ Manifest file updated
✓ Widget "Learning Streak" added successfully under "*".
```

**Generated Manifest Entry**:
```json
{
  "surface": "widget",
  "viewport": "*",
  "component": "LearningStreakWidget",
  "widget": {
    "id": "learning_streak",
    "name": "Learning Streak",
    "description": "Current daily study streak with a 7-day heatmap."
  }
}
```

---

### `teachfloor apps remove widget`

Remove a widget from your app.

```bash
teachfloor apps remove widget
```

**Prompts** (interactive mode) / **Flags** (non-interactive):
- **Select widget** — `--id <slug>` (must match a widget id declared in your manifest; picker lists each widget as `<id> — <name> (<viewport>)`)
- **Delete component file too** — `--removeComponent` (alias `--remove-component`; default: `false`)

**What it does**:
1. Removes the widget's view entry from your manifest (matched by `widget.id`, not viewport — multiple widgets can share the same viewport)
2. Optionally deletes the component file in `src/views/` when `--removeComponent` is set

**Example**:
```bash
$ teachfloor apps remove widget
✔ Select the widget you want to remove: learning_streak — Learning Streak (*)
✔ Do you want to delete the component file as well? Yes
✓ Component file "src/views/LearningStreakWidget.jsx" deleted
✓ Manifest file updated
✓ Widget "Learning Streak" removed successfully.
```

---

## Permission Management

### `teachfloor apps grant permission`

Add a permission to your app.

```bash
teachfloor apps grant permission
```

**Prompts** (interactive mode) / **Flags** (non-interactive):
- **Select permission** — `--permissionName <key>` (alias `--permission`; must be one of the available permissions and not already granted)
- **Purpose** — `--explanation <text>` (required — user-facing reason shown on the install screen)

**Available Permissions**:

Contextual data (SDK):
- `user:read`: Read user profile
- `user_events:read`: Read user activity
- `courses:read`: Read course data (also unlocks `GET /v0/courses/*` when OAuth is opted in)
- `modules:read`: Read module content (also unlocks `GET /v0/modules/*` when OAuth is opted in)
- `elements:read`: Read learning elements (also unlocks `GET /v0/elements/*` when OAuth is opted in)

Data storage (SDK):
- `appdata:read` / `appdata:write`: Organization-wide app storage
- `userdata:read` / `userdata:write`: User-specific storage
- `usercollection:read` / `usercollection:write`: User-specific collection storage

AI (SDK):
- `ai:text_generate`: Consume AI credits to generate text
- `ai:context_external_send`: Include platform data in external AI requests

Realtime (SDK):
- `realtime`: Publish and subscribe to the app's realtime channels

Public API only (backend/OAuth):
- `members:read`: Read organization members and enrollments via `GET /v0/members/*`
- `activities:read`: Read activity records via `GET /v0/activities/*`

See [Permissions](/docs/apps/advanced-topics/permissions) for the full description of each permission and [OAuth](/docs/apps/advanced-topics/oauth) for the scope mapping.

**Example**:
```bash
$ teachfloor apps grant permission
✔ Select permission: courses:read
✔ Enter purpose: Display course information in notes
✓ Permission added to manifest.
```

**Updates Manifest**:
```json
{
  "permissions": [
    {
      "permission": "courses:read",
      "purpose": "Display course information in notes"
    }
  ]
}
```

---

### `teachfloor apps revoke permission`

Remove a permission from your app.

```bash
teachfloor apps revoke permission
```

**Prompts** (interactive mode) / **Flags** (non-interactive):
- **Select permission** — `--permissionName <key>` (alias `--permission`; must match an existing granted permission)

**Example**:
```bash
$ teachfloor apps revoke permission
✔ Select permission to revoke: courses:read
✓ Permission removed from manifest.
```

---

## Webhook & OAuth

### `teachfloor apps set webhook`

Configure the app's webhook URL and the events it subscribes to. Re-run to reconfigure — the whole block is rewritten each time.

```bash
teachfloor apps set webhook
```

**Prompts** (interactive mode) / **Flags** (non-interactive):
- **URL** — `--url <value>` (required — must start with `https://`, max 2048 chars)
- **Events** — `--events <name>` (multi-select checkbox picker; repeatable flag or comma-separated: `--events a --events b` OR `--events "a,b"`)

**Available Events**:

Populated from the server catalog at run time so the CLI always offers exactly the events the platform will accept on upload. Current set includes:

- `organization.join`, `course.created`, `course.updated`, `course.completed`, `course.join`
- `module.created`, `module.updated`
- `element.created`, `element.updated`, `element.deleted`, `element.completed`
- `member.login`

The lifecycle events `app.installed` and `app.uninstalled` are always delivered — no need to include them in the manifest.

**Example**:
```bash
$ teachfloor apps set webhook
✔ Webhook URL (must be https://): https://myapp.example.com/teachfloor/hook
✔ Select events to subscribe to (space to toggle): course.completed, element.completed
✓ Webhook set to "https://myapp.example.com/teachfloor/hook" (2 events).
```

**Updates Manifest**:
```json
{
  "webhook": {
    "url": "https://myapp.example.com/teachfloor/hook",
    "events": ["course.completed", "element.completed"]
  }
}
```

---

### `teachfloor apps remove webhook`

Strip the `webhook` block from the manifest. The app becomes SDK-only for platform events — no signed deliveries, no installer identity disclosure to the app, and no OAuth credentials on install (OAuth requires a webhook to deliver them).

```bash
teachfloor apps remove webhook
```

---

### `teachfloor apps set oauth`

Configure the OAuth block. Presence of this block is the developer's explicit opt-in for install-integrated OAuth — see [OAuth](./oauth).

```bash
teachfloor apps set oauth
```

**Prompts** (interactive mode) / **Flags** (non-interactive):
- **Type** — `--oauthType <value>` (alias `--type`; currently only `install`)

**Example**:
```bash
$ teachfloor apps set oauth
✔ Select the OAuth grant type: install
✓ OAuth type set to "install".
```

**Updates Manifest**:
```json
{
  "oauth": { "type": "install" }
}
```

:::info
OAuth credentials are delivered inside the `app.installed` webhook payload. If your app has no webhook block, no tokens will be minted even with `oauth` set. Run `apps set webhook` first (or after) to complete the setup.
:::

---

### `teachfloor apps remove oauth`

Strip the `oauth` block from the manifest. New installs will no longer receive an access token / refresh token pair on install. In-app SDK permissions still work.

```bash
teachfloor apps remove oauth
```

---

## Distribution

### `teachfloor apps set distribution`

Set app distribution type (public or private).

```bash
teachfloor apps set distribution
```

**Prompts** (interactive mode) / **Flags** (non-interactive):
- **Distribution type** — `--distributionType <value>` (alias `--type`; one of `private` or `public`)

**Distribution Types**:
- **private**: Only your organization (default)
- **public**: Listed in marketplace (requires review)

**Example**:
```bash
$ teachfloor apps set distribution
✔ Select distribution type: public
✓ Distribution type updated to public.
```

**Updates Manifest**:
```json
{
  "distribution_type": "public"
}
```

**Important for Marketplace Submission**:
You **must** set distribution to `public` before submitting your app for marketplace review. Apps with `distribution_type: "private"` cannot be submitted to the public marketplace.

**Workflow for Public Apps**:
```bash
# 1. Set distribution to public
teachfloor apps set distribution
# Select: public

# 2. Upload your app
teachfloor apps upload

# 3. Submit via dashboard
# Navigate to Settings → Apps → Your App → Submit for Review
```

---

## Inspection

### `teachfloor apps show`

Print a spec-sheet summary of the current app — manifest metadata, webhook + OAuth config, permissions (with legacy-alias flagging), views, and install state on your own org. The quickest way to see "what does this app look like right now."

```bash
teachfloor apps show
```

**Options**:
- `-v, --verbose`: Expand widget declarations to show `id`/`name`/`description` per view (default is compact `Views (2 widgets, 1 drawer)`)
- `--json`: Emit machine-readable JSON instead of the pretty output (pipe into `jq`)
- `--no-remote`: Skip the permissions-catalog fetch; render local manifest only. Loses legacy-alias flagging and OAuth scope derivation but works fully offline

**What it prints**:

- **Header** — app name, version, id, distribution type
- **Metadata** — description + post-install action (only when set)
- **Webhook** — URL, subscribed events, note that lifecycle events (`app.installed` / `app.uninstalled`) are always delivered
- **OAuth** — type + "Scopes on install" (the OAuth scopes derived from the manifest's permissions, showing exactly what token an install would mint)
- **Permissions** — every entry with its purpose; legacy snake_case names show the canonical form with `(legacy alias: course_read)` inline
- **Views** — count by surface by default; expanded to per-view detail with `--verbose`

Sections only render when the corresponding manifest field is present.

**Example**:
```bash
$ teachfloor apps show

Webhook Test App                                            v1.0.0
6a640834d241e                                              private

  Description      Webhook test app
  Post-install     external  →  https://example.com

Webhook
  URL              https://webhook.site/5c1ab7fb-168a-4102-83a0-0425ffc073db
  Events           element.completed
                   (+ lifecycle: app.installed, app.uninstalled — always delivered)

OAuth
  Type             install
  Scopes on install  courses:read, elements:read, activities:read, members:read

Permissions (5)
  courses:read     Course permission
  courses:read     Read permission                (legacy alias: course_read)
  elements:read    Element read permission        (legacy alias: element_read)
  activities:read  Activity permission
  members:read     User read permission
```

**Scripting with `--json`**:
```bash
# List the OAuth scopes this app's install would mint
teachfloor apps show --json | jq -r '.computed.oauth_scopes_on_install[]'

# List every permission entry that's still using a legacy alias
teachfloor apps show --json | jq '.permissions[] | select(.is_legacy)'
```

---

## Diagnostics

### `teachfloor apps doctor`

Run a sequence of checks against your local setup + the current app, printing a pass/warn/fail line for each. Useful when something isn't working and you're not sure whether the problem is auth, the manifest, the app state on the platform, or your webhook/OAuth config.

```bash
teachfloor apps doctor
```

**Options**:
- `-v, --verbose`: Print the detail line for every check, not just non-passing ones

**What it checks**:

Environment
- **Authenticated** — token present, org selected, and `/whoami` still accepts it (the most common failure mode is a silently-expired token)

Manifest (only when run inside an app folder)
- **Inside app folder** — `teachfloor-app.json` exists
- **Manifest is valid JSON** — file parses
- **Manifest required fields** — `id`, `name`, `version` are all set
- **App exists on the platform** — the manifest's `id` resolves via `GET /apps/{id}`

Metadata (checked when the field is present)
- **Description** — warns when empty; marketplace listings render blank otherwise
- **Distribution type** — must be `private` or `public`
- **post_install_action** — shape check: `type` is required; when `type: "external"`, `url` is required and must be `https://`

Views (only when `ui_extension.views` is declared)
- **Views** — every view has `surface`, `viewport`, `component`; surface exists in the server catalog; widget-surface views additionally follow server rules (widget id matches `^[a-z][a-z0-9_]*$`, name ≤ 60 chars, description ≤ 200 chars, ids unique per app)

Permissions (only when declared)
- **Permissions** — every entry is `{ permission, purpose }`; each permission exists in the server catalog; legacy snake_case names (`course_read`) surface as warnings so you know to migrate to the canonical form (`courses:read`)

Webhook block (only when declared)
- **Webhook URL declared / uses HTTPS / length** — server-side rules mirrored locally so `apps upload` doesn't 422 on preventable issues
- **Webhook events subscribed** — every event in `webhook.events` is in the server's catalog

OAuth block (only when declared)
- **OAuth type is valid** — currently only `install` is accepted
- **OAuth prerequisites** — the three-prereq gate (`oauth` block + `webhook` block + at least one permission) — warns when credentials wouldn't actually be minted on install

**Exit code**: `0` when there are no errors (warnings still exit 0); `1` when any check failed.

**Example**:
```bash
$ teachfloor apps doctor

Running diagnostics...

  ✓  Authenticated
  ✓  Inside app folder
  ✓  Manifest is valid JSON
  ✓  Manifest required fields
  ✓  App exists on the platform
  ✓  Webhook URL — myapp.example.com
  ✓  Webhook events subscribed
  ⚠  OAuth prerequisites — no webhook block — credentials would have nowhere to land

7 passed, 1 warning
```

---

## Command Requirements

### Authentication Required

These commands require authentication:
- `teachfloor apps create`
- `teachfloor apps start`
- `teachfloor apps upload`

**Check authentication**:
```bash
teachfloor whoami
```

**Re-authenticate**:
```bash
teachfloor logout
teachfloor login
```

### App Folder Required

These commands must be run inside an app folder:
- `teachfloor apps start`
- `teachfloor apps upload`
- `teachfloor apps add view`
- `teachfloor apps remove view`
- `teachfloor apps add settings`
- `teachfloor apps add widget`
- `teachfloor apps remove widget`
- `teachfloor apps grant permission`
- `teachfloor apps revoke permission`
- `teachfloor apps set distribution`

**Check if in app folder**:
```bash
ls teachfloor-app.json
```

---

## Common Workflows

### Create and Test App

```bash
# 1. Install CLI
npm install -g @teachfloor/teachfloor-cli

# 2. Login
teachfloor login

# 3. Create app
teachfloor apps create my-app
cd my-app

# 4. Add a view
teachfloor apps add view

# 5. Start dev server
teachfloor apps start

# 6. Make changes and test
# (dev server auto-reloads)

# 7. Upload when ready
teachfloor apps upload
```

### Update Existing App

```bash
# 1. Navigate to app folder
cd my-app

# 2. Make code changes
# edit src/views/MyView.jsx

# 3. Update version in manifest
# Edit teachfloor-app.json: "version": "1.1.0"

# 4. Test locally
teachfloor apps start

# 5. Upload new version
teachfloor apps upload
```

### Add View to Existing App

```bash
cd my-app

# Add view
teachfloor apps add view
# Select viewport and enter component name

# Implement component
# edit src/views/NewView.jsx

# Test
teachfloor apps start
```

---

## Troubleshooting

### "Not logged in" error

**Solution**:
```bash
teachfloor login
```

### "Not in app folder" error

**Check for manifest**:
```bash
ls teachfloor-app.json
```

**Create new app if needed**:
```bash
teachfloor apps create my-app
cd my-app
```

### "Version already approved" error

**Solution**: Increment version in `teachfloor-app.json`:
```json
{
  "version": "1.0.1"
}
```

### Build errors

**Clear cache**:
```bash
rm -rf node_modules
npm install
npm run build
```

---

## Getting Help

### Built-in Help

```bash
teachfloor --help
teachfloor apps --help
teachfloor apps create --help
```

### Version Info

```bash
teachfloor version
```

### Support Channels

- **Email**: support@teachfloor.com
- **Documentation**: [docs.teachfloor.com](https://docs.teachfloor.com)

---

## Next Steps

Learn about best practices:

→ Continue to [Best Practices](./best-practices)

## Additional Resources

- [Quickstart Guide](/docs/apps/quickstart)
- [App Manifest](/docs/apps/core-concepts/app-manifest)
- [Deployment](/docs/apps/advanced-topics/deployment)
