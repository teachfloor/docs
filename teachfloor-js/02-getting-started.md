Getting Started
Overview
To begin using the Teachfloor.js SDK, you first need to create a custom Teachfloor App through the Teachfloor Dashboard. This process will generate a unique SDK script tailored for your application. This script is the key to integrating your custom page with the Teachfloor platform, enabling communication and interaction between your custom implementation and Teachfloor’s rich environment.

Step 1: Create a Teachfloor App
To create a Teachfloor App and obtain your unique SDK script, follow these steps:

Log in to the Teachfloor Dashboard:

Access the Teachfloor Dashboard using your account credentials.

Navigate to the Developers Page:

Once logged in, locate the Developers page. This is where you'll manage your custom apps.

Create a New App:

Go to the Apps Tab:

Inside the Developers section, find and click on the Apps tab. This is where you can view existing apps or create new ones.

Click on "Create App":

Click the Create App button to start the process of defining your new application.

Provide App Details:

Fill in the required information, such as the app name, icon, and color theme, which will represent your app within Teachfloor.

Specify the URL of the page where the SDK will be included. This URL is where your custom JS script will be hosted and should correspond to the page that will interact with Teachfloor.

Receive Your Unique SDK Script:

Once the app is created, Teachfloor will generate a unique SDK script containing your application’s ID. This script is crucial for initializing the SDK on your custom page.

Step 2: Initialize the SDK Script
With your Teachfloor App created, the next step is to integrate the SDK into your custom web page. The script provided after app creation should be included in the <head> section of your HTML file to ensure it is loaded properly.

Here’s an example of how to initialize the SDK:

```php-template
<head>
    <script>
        // Initialize the Teachfloor SDK
        (function(t,c,h,f,l,r){
            t.tf = t.tf || function() {(t.tf.q = t.tf.q || []).push(arguments)}
            t._tfOpts = { id: 'YOUR_UNIQUE_APP_ID' }
            l = c.getElementsByTagName('head')[0];
            r = c.createElement('script');
            r.async = 1;
            r.src = h + t._tfOpts.id + f + Date.now();
            l.appendChild(r)
        })(window, document, 'https://app.teachfloor.com/apps/', '.js?ver=')

        // Wait for the SDK to be ready
        tf('onInit', function(API) {
            // Communicate with Teachfloor using API object
        });
    </script>
</head>
```

**Key Points:**
- SDK Initialization:
  - The script initializes the SDK by loading the necessary JavaScript file from Teachfloor’s servers. The unique app ID ensures that the SDK is correctly configured for your specific app.
- SDK Ready Event:
  - The tf('onInit', function(API) { ... }) block is where you can start interacting with Teachfloor once the SDK is fully initialized. The API object provided in this callback is your gateway to accessing all the SDK functionalities.

Final Steps
After setting up the SDK script, you’re ready to start using the powerful features provided by the Teachfloor.js SDK. You can now build interactive, responsive applications that integrate seamlessly with the Teachfloor platform, offering enhanced functionality and a tailored user experience.