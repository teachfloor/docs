# Troubleshooting

Common issues and solutions for Teachfloor app development.

## CLI Issues

### "Not logged in" Error

**Symptoms**:
```
Error: Not logged in. Please run 'teachfloor login' first.
```

**Solution**:
```bash
teachfloor login
```

Enter your credentials and select your organization.

---

### "Login failed: Request failed with status code 400"

**Cause**: OAuth client not properly configured

**Solution**:
1. Check if API URL is correct:
```bash
echo $API_URL
```

2. For self-hosted instances, ensure OAuth client is set up (see web-app/README.md)

3. Contact platform administrator to configure OAuth client

---

### "Invalid token" Error

**Cause**: Token expired or corrupted

**Solution**:
```bash
teachfloor logout
teachfloor login
```

---

### "Not in app folder" Error

**Symptoms**:
```
Error: This command must be run inside a Teachfloor extension app folder.
```

**Solution**:
```bash
# Check for manifest file
ls teachfloor-app.json

# If not found, navigate to app folder
cd path/to/my-app

# Or create new app
teachfloor apps create my-app
cd my-app
```

---

### CLI Command Not Found

**Symptoms**:
```
bash: teachfloor: command not found
```

**Solution**:
```bash
# Install globally
npm install -g @teachfloor/teachfloor-cli

# Verify installation
teachfloor version

# If still not found, check PATH
echo $PATH

# Use npx as alternative
npx @teachfloor/teachfloor-cli login
```

---

## Build Issues

### "npm run build" Fails

**Symptoms**:
```
Module not found: Error: Can't resolve...
```

**Solution**:
```bash
# Clear cache and reinstall
rm -rf node_modules
rm package-lock.json
npm install

# Clear webpack cache
rm -rf node_modules/.cache

# Try build again
npm run build
```

---

### "Version already exists" Error

**Symptoms**:
```
Error: Version 1.0.0 is already approved.
```

**Solution**:
Increment version in `teachfloor-app.json`:
```json
{
  "version": "1.0.1"
}
```

Then upload again:
```bash
teachfloor apps upload
```

---

### Build Size Too Large

**Symptoms**:
Warning about bundle size exceeding recommended limit

**Solution**:
```bash
# Analyze bundle
npm install --save-dev webpack-bundle-analyzer
ANALYZE=true npm run build

# Reduce size:
# 1. Remove unused dependencies
npm uninstall unused-package

# 2. Use dynamic imports
const Component = lazy(() => import('./Component'))

# 3. Minify images
# 4. Tree-shake imports
import { Button } from '@teachfloor/extension-kit' // ✅
import * as Kit from '@teachfloor/extension-kit'    // ❌
```

---

## Runtime Issues

### App Not Displaying

**Checklist**:
1. Check viewport matches current page
2. Verify app is installed
3. Check browser console for errors
4. Verify SDK is loading

**Solution**:
```javascript
// Add debug logging
const { environment } = useExtensionContext()
console.log('Current viewport:', environment.viewport)
console.log('App initialized:', environment.initialized)
```

---

### "Cannot read property of undefined"

**Cause**: Accessing context before initialization

**Solution**:
```javascript
// ❌ Bad
const { userContext } = useExtensionContext()
return <div>{userContext.name}</div> // Error!

// ✅ Good
const { userContext, environment } = useExtensionContext()

if (!environment.initialized) {
  return <Loader />
}

return <div>{userContext.name}</div>
```

---

### Data Not Persisting

**Cause**: Missing permissions or incorrect source

**Solution**:
1. Check permissions in manifest:
```json
{
  "permissions": [
    {
      "permission": "user_read",
      "purpose": "Display your profile"
    }
  ]
}
```

2. Verify data source:
```javascript
// Correct source
await store('key', data, 'userdata')  // ✅
await store('key', data, 'appdata')   // ✅

// Wrong source
await store('key', data, 'wrong')     // ❌
```

3. Check for errors:
```javascript
try {
  await store('key', data, 'userdata')
  console.log('Saved successfully')
} catch (error) {
  console.error('Save failed:', error)
}
```

---

### Permission Denied Errors

**Symptoms**:
```
Error: Permission denied: user_read
```

**Solution**:
1. Add permission to manifest:
```bash
teachfloor apps grant permission
```

2. Upload new version:
```bash
teachfloor apps upload
```

3. Users may need to reinstall app to approve new permissions

---

## Development Issues

### Hot Reload Not Working

**Solution**:
```bash
# Stop dev server (Ctrl+C)
# Clear cache
rm -rf node_modules/.cache

# Restart
teachfloor apps start
```

---

### Port Already in Use

**Symptoms**:
```
Error: Port 3000 is already in use
```

**Solution**:
Change port in `webpack.config.js`:
```javascript
module.exports = {
  devServer: {
    port: 3001, // Change port
  }
}
```

Or kill process using port:
```bash
# Find process
lsof -i :3000

# Kill process
kill -9 <PID>
```

---

### Changes Not Reflecting

**Solution**:
```bash
# Hard refresh browser
Ctrl+Shift+R (Windows/Linux)
Cmd+Shift+R (Mac)

# Clear browser cache
# In DevTools: Right-click refresh → Empty Cache and Hard Reload

# Restart dev server
# Stop server (Ctrl+C)
teachfloor apps start
```

---

## Manifest Issues

### "Invalid manifest" Error

**Cause**: JSON syntax error or missing required fields

**Solution**:
```bash
# Validate JSON syntax
cat teachfloor-app.json | jq .

# Check required fields
jq '.id, .version, .name' teachfloor-app.json

# Common issues:
# - Missing comma
# - Extra comma
# - Unquoted strings
# - Wrong bracket type
```

---

### Component Not Found

**Symptoms**:
```
Error: Cannot find module './MyView'
```

**Solution**:
1. Check component name matches manifest:
```json
{
  "ui_extension": {
    "views": [
      {
        "component": "MyView"  // Must match filename
      }
    ]
  }
}
```

2. Verify file exists:
```bash
ls src/views/MyView.jsx
```

3. Check export:
```javascript
// Must use default export
export default MyView  // ✅
export { MyView }      // ❌
```

---

## Browser Issues

### Browser Console Errors

**Common Errors**:

1. **"Uncaught ReferenceError: tf is not defined"**
   - SDK not loaded
   - Check `public/index.html` has SDK script

2. **"Cannot read property 'emit' of undefined"**
   - Calling SDK before initialization
   - Wrap in `useEffect` with initialization check

3. **"Failed to fetch"**
   - CORS issue or network error
   - Check API URL
   - Verify network connection

---

### App Shows Blank Page

**Debug Steps**:
1. Open browser console (F12)
2. Check for JavaScript errors
3. Verify platform is initialized:
```javascript
import { useExtensionContext } from '@teachfloor/extension-kit'

const { environment } = useExtensionContext()
console.log('Initialized:', environment.initialized)
```

4. Signal app is ready:
```javascript
import { initialize } from '@teachfloor/extension-kit'

initialize()
```

---

## Installation Issues

### Cannot Install npm Packages

**Solution**:
```bash
# Clear npm cache
npm cache clean --force

# Delete lock file
rm package-lock.json

# Reinstall
npm install

# Try with legacy peer deps
npm install --legacy-peer-deps
```

---

### Node Version Issues

**Solution**:
```bash
# Check version
node --version

# Should be 18.0.0 or higher
# Install/switch version
nvm install 20
nvm use 20
```

---

## Deployment Issues

### Upload Fails

**Symptoms**:
```
Error: Failed to upload app
```

**Solution**:
1. Check authentication:
```bash
teachfloor whoami
```

2. Verify network:
```bash
ping app.teachfloor.com
```

3. Check manifest is valid
4. Try again

---

### Review Rejected

**Common Reasons**:
- Unclear permission purposes
- Security vulnerabilities
- Poor user experience
- Broken functionality
- Missing documentation

**Solution**:
1. Read rejection feedback carefully
2. Fix identified issues
3. Increment version
4. Upload and resubmit

---

## Performance Issues

### App Loading Slowly

**Solution**:
```javascript
// 1. Lazy load components
const HeavyComponent = lazy(() => import('./Heavy'))

// 2. Memoize expensive calculations
const result = useMemo(() => expensive(data), [data])

// 3. Debounce frequent operations
const debouncedFn = useMemo(
  () => debounce(fn, 500),
  []
)

// 4. Cache data
const cachedData = useMemo(() => data, [data])
```

---

### High Memory Usage

**Solution**:
```javascript
// Clean up timers and intervals
useEffect(() => {
  const timer = setTimeout(() => {
    // Do something
  }, 1000)

  return () => {
    clearTimeout(timer)  // Cleanup!
  }
}, [])

// Clear timers
useEffect(() => {
  const timer = setTimeout(() => {}, 1000)

  return () => {
    clearTimeout(timer)  // Cleanup!
  }
}, [])
```

---

## Getting Help

### Before Asking for Help

1. Check browser console for errors
2. Check CLI output for error messages
3. Verify manifest is valid
4. Check this troubleshooting guide
5. Search GitHub issues

### Where to Get Help

- **GitHub Issues**: [teachfloor/teachfloor-cli/issues](https://github.com/teachfloor/teachfloor-cli/issues)
- **Email**: support@teachfloor.com
- **Community**: [community.teachfloor.com](https://community.teachfloor.com)

### What to Include

When reporting issues:
- Teachfloor CLI version (`teachfloor version`)
- Node version (`node --version`)
- Operating system
- Full error message
- Steps to reproduce
- Manifest file (if relevant)
- Console errors (screenshot)

---

## Debug Mode

### Enable Debug Logging

Add to `public/index.html`:
```html
<script>
  window._tfOpts = {
    id: 'your-app-id',
    debug: true  // Enable debug mode
  }
</script>
```

This will log SDK events to console.

---

## Common Mistakes

### 1. Not Handling Loading States
```javascript
// ❌ Bad
const { userContext } = useExtensionContext()
return <div>{userContext.name}</div>

// ✅ Good
const { userContext, environment } = useExtensionContext()
if (!environment.initialized) return <Loader />
return <div>{userContext.name}</div>
```

### 2. Not Handling Async Operations
```javascript
// ❌ Bad
useEffect(async () => {
  const data = await retrieve('key')
  setData(data)
}, [])

// ✅ Good
useEffect(() => {
  retrieve('key').then(setData)
}, [])
```

### 3. Over-Requesting Permissions
```javascript
// ❌ Bad - requesting everything
"permissions": [
  { "permission": "user_read", ... },
  { "permission": "user_events_read", ... },
  { "permission": "course_read", ... },
  // ... not all needed
]

// ✅ Good - only what's needed
"permissions": [
  { "permission": "course_read", "purpose": "Display course info" }
]
```

### 4. Not Validating Data
```javascript
// ❌ Bad
await store('data', userInput, 'userdata')

// ✅ Good
if (!isValid(userInput)) {
  throw new Error('Invalid input')
}
const sanitized = sanitize(userInput)
await store('data', sanitized, 'userdata')
```

---

## Quick Reference

### Reset Everything

```bash
# Logout
teachfloor logout

# Clear cache
rm -rf node_modules/.cache
rm -rf dist/

# Reinstall
rm -rf node_modules
npm install

# Login again
teachfloor login

# Rebuild
npm run build
```

### Debug Checklist

```
□ Check browser console for errors
□ Verify environment.initialized is true
□ Check manifest is valid JSON
□ Confirm permissions are listed
□ Verify component files exist
□ Check SDK script is loaded
□ Confirm app is installed
□ Check viewport matches page
□ Verify data storage permissions
□ Test with hard refresh
```

---

## Additional Resources

- [Quickstart Guide](./quickstart-guide)
- [CLI Reference](./references/teachfloor-cli)
- [Best Practices](./references/best-practices)
- [Examples](./references/examples)
