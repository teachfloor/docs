# Deployment

This guide covers deploying your Teachfloor app from private testing to public marketplace publication.

## Overview

Teachfloor apps can be deployed in two ways:

1. **Private Deployment**: Available only to your organization
2. **Public Deployment**: Listed in the Teachfloor Marketplace for all organizations

## Distribution Types

### Private Apps

Default distribution type for new apps.

**Characteristics**:
- Only visible to your organization
- No review process required
- Instant deployment
- Cannot be installed by other organizations

**When to use**:
- Internal productivity tools
- Organization-specific integrations
- Testing and development
- Custom solutions for your team

### Public Apps

Marketplace-listed apps available to all organizations.

**Characteristics**:
- Listed in Teachfloor Marketplace
- Requires review and approval
- Available to all organizations
- Subject to quality standards

**When to use**:
- General-purpose tools
- Integrations with popular services
- Apps that benefit the community
- Commercial offerings

## Deployment Process

### Step 1: Prepare Your App

#### Update Version

Increment the version in your manifest before each deployment:

```json
{
  "version": "1.0.1"  // Changed from "1.0.0"
}
```

Follow [semantic versioning](https://semver.org/):
- **Major** (x.0.0): Breaking changes
- **Minor** (1.x.0): New features, backward compatible
- **Patch** (1.0.x): Bug fixes

#### Test Locally

```bash
teachfloor apps start
```

Ensure:
- All features work as expected
- No console errors
- All viewports display correctly
- Data storage works properly
- Permissions are correctly requested

#### Clean Build

```bash
# Clear cache
rm -rf node_modules/.cache
rm -rf dist/

# Fresh install
npm install

# Test build
npm run build
```

### Step 2: Set Distribution Type

#### For Private Deployment

Apps are private by default. To explicitly set:

```bash
teachfloor apps set distribution
```

Select `private` when prompted.

**Or edit manifest manually**:
```json
{
  "distribution_type": "private"
}
```

#### For Public Deployment

**Required** before submitting to marketplace:

```bash
teachfloor apps set distribution
```

Select `public` when prompted.

**Or edit manifest manually**:
```json
{
  "distribution_type": "public"
}
```

<scalar-callout type="warning">This must be set to `public` before the app can be submitted for marketplace review.</scalar-callout>

### Step 3: Upload Your App

Build and upload your app:

```bash
teachfloor apps upload
```

**What happens**:
1. Validates manifest
2. Checks version isn't already approved
3. Runs `npm run build`
4. Bundles all files from `dist/`
5. Replaces localhost URLs with production URLs
6. Uploads to Teachfloor platform
7. Creates new app version

**Output**:
```
✓ Uploading manifest...
✓ Starting upload...
✓ Building the production bundle...
✓ Uploading files...
✓ App uploaded successfully.
```

### Step 4: Submit for Review (Public Apps Only)

#### Via Dashboard

1. Log in to Teachfloor dashboard
2. Navigate to **Settings → Apps**
3. Find your app in the list
4. Click **"Submit for Review"**
5. Fill out submission form (if any)
6. Click **"Submit"**

#### What Gets Reviewed

- **Functionality**: App works as described
- **UI/UX**: Follows Teachfloor design guidelines
- **Security**: No vulnerabilities or malicious code
- **Privacy**: Complies with data protection standards
- **Permissions**: Requests only necessary permissions
- **Documentation**: Clear purpose and explanation

### Step 5: Review Process

#### Timeline

- **Initial Review**: 2-5 business days
- **Revisions**: 1-3 business days per iteration
- **Final Approval**: 1 business day

#### Status Tracking

Check status in dashboard:
- `UNPUBLISHED`: Draft, not submitted
- `SUBMITTED`: Awaiting review
- `REVIEWING`: Under review
- `PUBLISHED`: Approved and live
- `REJECTED`: Review failed

#### If Rejected

You'll receive feedback explaining:
- Issues found
- Required changes
- Guidelines violated

**Fix and resubmit**:
1. Make required changes
2. Update version number
3. Run `teachfloor apps upload`
4. Submit again via dashboard

## Version Management

### Creating New Versions

Once a version is approved/published, it becomes **locked** and cannot be modified.

**To release updates**:

1. Update version in manifest:
```json
{
  "version": "1.1.0"  // Increment from 1.0.0
}
```

2. Make your changes

3. Upload:
```bash
teachfloor apps upload
```

4. Submit for review (if public)

### Version States

| State | Editable | Installable | Visible in Marketplace |
|-------|----------|-------------|------------------------|
| UNPUBLISHED | ✅ Yes | ✅ Yes (dev mode) | ❌ No |
| SUBMITTED | ❌ No | ✅ Yes (dev mode) | ❌ No |
| REVIEWING | ❌ No | ✅ Yes (dev mode) | ❌ No |
| PUBLISHED | ❌ No | ✅ Yes | ✅ Yes (if public) |
| REJECTED | ✅ Yes | ✅ Yes (dev mode) | ❌ No |

### Rollback Strategy

You cannot rollback a published version. To address issues:
- Publish a fixed version with incremented version number
- Contact support to hide from marketplace if needed

## Private to Public Migration

If you want to make an existing private app public:

1. Set distribution to public:
```bash
teachfloor apps set distribution
# Select: public
```

2. Update manifest:
```json
{
  "distribution_type": "public",
  "description": "Clear, helpful description for marketplace"
}
```

3. Increment version (use major version for public release)
4. Upload: `teachfloor apps upload`
5. Submit via dashboard

<scalar-callout type="info">Existing private installations remain unaffected.</scalar-callout>

## Marketplace Guidelines

### App Requirements

**Required**:
- Clear, descriptive name
- App icon
- Accurate description (50-200 characters)
- Valid semantic version
- Working functionality

**Recommended**:
- Settings page for configuration
- Post-install action
- Clear permission purposes

### Quality Standards

Public apps must meet quality standards for performance, accessibility, design, error handling, and privacy.

## Next Steps

→ Continue to [CLI Reference](/apps/references/teachfloor-cli)

## Additional Resources

- [Best Practices](/apps/references/best-practices) - Deployment best practices and checklists
- [Troubleshooting Guide](/apps/references/troubleshooting) - Common deployment issues
- [App Manifest](/apps/core-concepts/app-manifest) - Manifest configuration
