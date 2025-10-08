# Quickstart Guide

Build your first Teachfloor app in 10 minutes! This guide walks you through creating a simple "Hello World" app that displays in the course list page.

## Prerequisites

- Node.js 18.0+ installed
- npm or yarn package manager
- A Teachfloor account

## Setup Steps

<scalar-steps>
  <scalar-step id="step-1" title="Install the CLI">

Open your terminal and install the Teachfloor CLI globally:

```bash
npm install -g @teachfloor/teachfloor-cli
```

Verify the installation:

```bash
teachfloor version
```

  </scalar-step>

  <scalar-step id="step-2" title="Authenticate">

Log in to your Teachfloor account:

```bash
teachfloor login
```

You'll be prompted for:
- **Email**: Your Teachfloor account email
- **Password**: Your account password
- **Organization**: Select your organization (if you have multiple)

Once authenticated, verify your session:

```bash
teachfloor whoami
```

  </scalar-step>

  <scalar-step id="step-3" title="Create Your First App">

Create a new app called "hello-world":

```bash
teachfloor apps create hello-world
```

You'll be prompted for:
- **App ID**: Press Enter to accept the default (auto-generated)
- **Display Name**: Enter "Hello World"
- **Description**: Enter "My first Teachfloor app"
- **Version**: Press Enter to accept "1.0.0"

The CLI will:
1. Create the app on the platform
2. Generate the project structure
3. Install dependencies (React, Extension Kit, etc.)

```
✓ Creating app...
✓ Setting up app structure...
✓ Installing npm dependencies...
✓ App "Hello World" created successfully in "hello-world".
```

  </scalar-step>

  <scalar-step id="step-4" title="Explore the Project Structure">

Navigate into your app directory:

```bash
cd hello-world
```

Your project structure looks like this:

```
hello-world/
├── src/
│   ├── index.js          # Entry point
│   └── views/
│       └── App.jsx       # Main view component
├── public/
│   └── index.html        # HTML template with SDK
├── teachfloor-app.json   # App manifest
├── package.json          # Dependencies
├── webpack.config.js     # Webpack config
├── tsconfig.json         # TypeScript config
└── .gitignore
```

  </scalar-step>

  <scalar-step id="step-5" title="Add a View">

Add a view to display your app in the course list page:

```bash
teachfloor apps add view
```

Select:
- **Viewport**: `teachfloor.dashboard.course.list`
- **Component Name**: Press Enter to accept "CourseListView"
- **Generate example**: Yes

This creates `src/views/CourseListView.jsx` and updates your manifest.

  </scalar-step>

  <scalar-step id="step-6" title="Customize Your Component">

Open `src/views/CourseListView.jsx` and modify it:

```jsx
import React from 'react'
import {
  Container,
  Text,
  Button,
  SimpleGrid,
  showToast,
  useExtensionContext,
} from '@teachfloor/extension-kit'

const CourseListView = () => {
  const { userContext } = useExtensionContext()

  const handleClick = () => {
    showToast('Hello from your app!', { type: 'success' })
  }

  return (
    <Container>
      <SimpleGrid verticalSpacing="md">
        <Text size="xl" fw={700}>
          Hello, {userContext.full_name}!
        </Text>
        <Text size="sm" c="dimmed">
          Welcome to your first Teachfloor app.
        </Text>
        <Button onClick={handleClick}>
          Click Me
        </Button>
      </SimpleGrid>
    </Container>
  )
}

export default CourseListView
```

  </scalar-step>

  <scalar-step id="step-7" title="Start the Development Server">

Run the development server:

```bash
teachfloor apps start
```

This will:
1. Validate your manifest
2. Upload the manifest to the platform
3. Open your browser to install the app
4. Start the webpack dev server on `http://localhost:3000`

```
✓ Manifest file updated
Install URL: https://app.teachfloor.com/your-org/courses?app=abc123@1.0.0
Starting development server...
```

  </scalar-step>

  <scalar-step id="step-8" title="Test Your App">

1. Your browser will open to the install URL
2. Click "Install" to add the app to your organization
3. Navigate to the course list page
4. You should see your app displayed!

The app will hot-reload as you make changes to the code.

  </scalar-step>

  <scalar-step id="step-9" title="Build for Production">

When you're ready to deploy:

1. Stop the dev server (Ctrl+C)
2. Build the production bundle:

```bash
teachfloor apps upload
```

This will:
1. Run `npm run build`
2. Bundle your app for production
3. Upload files to the platform
4. Create a new app version

```
✓ Building the production bundle...
✓ Uploading files...
✓ App uploaded successfully.
```

  </scalar-step>

  <scalar-step id="step-10" title="Publish to Marketplace (Optional)">

If you want to make your app available in the public marketplace:

**Set Distribution to Public**

First, set your app's distribution type to public:

```bash
teachfloor apps set distribution
```

Select `public` when prompted. This updates your manifest:

```json
{
  "distribution_type": "public"
}
```

**Upload and Submit**

1. Upload your app with the updated manifest:

```bash
teachfloor apps upload
```

2. Go to your Teachfloor dashboard
3. Navigate to Settings → Apps
4. Find your app and click "Submit for Review"
5. Wait for approval from the Teachfloor team

    <scalar-callout type="info">
Only apps with `distribution_type: "public"` can be submitted for marketplace review. Private apps (`distribution_type: "private"`) are only visible to your organization and don't require review.
    </scalar-callout>

  </scalar-step>
</scalar-steps>

## What's Next?

Congratulations! You've built your first Teachfloor app. Here's what to explore next:

### Learn Core Concepts
- [App Manifest](./core-concepts/app-manifest) - Configure your app
- [Viewports](./core-concepts/viewports-system) - Understand placement options
- [Extension Kit](./core-concepts/extension-kit/components) - Explore available components

### Add More Functionality
- [Data Storage](./advanced-topics/data-storage) - Persist user data
- [Permissions](./advanced-topics/permissions) - Access platform resources
- [Extension Kit Integration](./core-concepts/extension-kit/integration) - Use platform features

### See Examples
- [Example Apps](./references/examples) - Sample code and patterns

---

**Ready to dive deeper?** Continue to [App Manifest](./core-concepts/app-manifest)
