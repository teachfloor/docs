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
| **Permission Management** |
| `teachfloor apps grant permission` | Add permission | Yes | Yes |
| `teachfloor apps revoke permission` | Remove permission | Yes | Yes |
| **Distribution** |
| `teachfloor apps set distribution` | Set public/private | Yes | Yes |

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

**Prompts**:
- **App ID**: Unique identifier (auto-generated)
- **Display Name**: User-facing name
- **Description**: Short description
- **Version**: Semantic version (default: 1.0.0)

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

**Prompts**:
1. **Select viewport**: Choose from available viewports
2. **Component name**: React component name (PascalCase)
3. **Generate example**: Include example code

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

**Prompts**:
- **Select viewport**: Choose viewport to remove

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

**Prompts**:
- **Component name**: Settings component name
- **Generate example**: Include example code

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

## Permission Management

### `teachfloor apps grant permission`

Add a permission to your app.

```bash
teachfloor apps grant permission
```

**Prompts**:
1. **Select permission**: Choose from available permissions
2. **Purpose**: User-facing explanation

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

**Prompts**:
- **Select permission**: Choose permission to remove

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

**Prompts**:
- **Distribution type**: public or private

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

→ Continue to [Best Practices](./11-best-practices.md)

## Additional Resources

- [Quickstart Guide](./02-quickstart.md)
- [App Manifest](./03-app-manifest.md)
- [Deployment](./09-deployment.md)
