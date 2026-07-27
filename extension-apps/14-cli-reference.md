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
| **Distribution** |
| `teachfloor apps set distribution` | Set public/private | Yes | Yes |

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
```

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
- `user_read`: Read user profile
- `user_events_read`: Read user activity
- `course_read`: Read course data
- `module_read`: Read module content
- `element_read`: Read learning elements

**Example**:
```bash
$ teachfloor apps grant permission
✔ Select permission: course_read
✔ Enter purpose: Display course information in notes
✓ Permission added to manifest.
```

**Updates Manifest**:
```json
{
  "permissions": [
    {
      "permission": "course_read",
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
✔ Select permission to revoke: course_read
✓ Permission removed from manifest.
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
