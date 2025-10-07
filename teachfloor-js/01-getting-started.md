# Getting Started

## Prerequisites

- A Teachfloor account with access to the Dashboard
- A web page where you'll integrate the SDK

## Step 1: Create a Teachfloor App

1. Log in to the [Teachfloor Dashboard](https://app.teachfloor.com)
2. Navigate to **Developers** → **Apps**
3. Click **Create App**
4. Provide:
   - App name
   - Icon and color theme
   - URL where the SDK will be used
5. Save to generate your unique app ID

## Step 2: Install the SDK

Add the SDK script to your page's `<head>` section:

```html
<head>
  <script>
    (function(t,c,h,f,l,r){
      t.tf = t.tf || function() {(t.tf.q = t.tf.q || []).push(arguments)}
      t._tfOpts = { id: 'YOUR_UNIQUE_APP_ID' }
      l = c.getElementsByTagName('head')[0];
      r = c.createElement('script');
      r.async = 1;
      r.src = h + t._tfOpts.id + f + Date.now();
      l.appendChild(r)
    })(window, document, 'https://app.teachfloor.com/apps/', '.js?ver=')

    tf('onInit', function(API) {
      // SDK is ready - start using API methods
      console.log('Teachfloor SDK initialized');
    });
  </script>
</head>
```

Replace `YOUR_UNIQUE_APP_ID` with the ID from your Teachfloor App.

## Step 3: Use the SDK

Once initialized, use the `API` object to interact with Teachfloor:

```javascript
tf('onInit', function(API) {
  // Get authenticated user
  API.get('auth.user')
    .then(user => console.log('User:', user));

  // Listen for viewport changes
  API.on('environment.viewport.changed', (viewport) => {
    console.log('Current viewport:', viewport);
  });
});
```

## Next Steps

- Explore [SDK Methods](./02-sdk-methods.md)
- Review [Viewports Reference](./03-viewports-reference.md)
