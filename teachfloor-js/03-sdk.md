Teachfloor.js
Overview of Teachfloor.js API Methods
The Teachfloor.js API provides a simple yet powerful interface to interact with the Teachfloor platform. Currently, the API supports two essential methods: API.get and API.on . These methods are designed to facilitate seamless communication between your custom web page and the Teachfloor environment, allowing you to retrieve data and respond to events in real-time.

API.get()
The API.get() method enables you to retrieve specific data from Teachfloor. This method is particularly useful when you need to access detailed information about various entities within the Teachfloor platform, such as the authenticated user.

Usage
API.get(objectIdentifier)

Parameters
objectIdentifier (string): The identifier for the object you want to retrieve. This could be, for example, auth.user to get the authenticated user’s details.

Object Identifiers
Object identifiers in the Teachfloor.js SDK allow you to access specific pieces of information from the Teachfloor platform. Below is a list of the currently available object identifiers, which you can use with the API.get() method to retrieve details from Teachfloor.

Object Identifier

Description

auth.user

Retrieves the details of the authenticated user, including information like the user's ID, name, email, and any other relevant profile data.

Return
A Promise that resolves with the requested data once it’s successfully retrieved, or rejects if an error occurs.

Example
javascript
API.get('auth.user')
    .then(user => {
        console.log('Authenticated user:', user);
    })
    .catch(error => {
        console.error('Error retrieving user data:', error);
    });
This example demonstrates how to use API.get() to fetch the details of the currently authenticated user. The method returns a promise, allowing you to handle the response asynchronously.

API.on()
The API.on() method allows your custom web page to listen for specific events that occur within the Teachfloor platform. By subscribing to these events, your page can react to changes in the Teachfloor environment, enabling dynamic content updates or triggering specific actions based on user interactions or platform changes.

Usage
API.on(eventIdentifier, callback)

Parameters
eventIdentifier (string): The identifier representing the Teachfloor event you want to listen to. For example, environment.viewport.changed to track when the viewport changes.

callback (function): A function that will be executed when the specified event is triggered. This function receives event-specific data as an argument.

Event Identifiers
Event identifiers in the Teachfloor.js SDK allows you to listen for specific events within the Teachfloor platform, enabling your custom application to react to changes and updates in real-time. Below is a list of the currently available event identifiers that you can subscribe to using the API.on() method.

Event Name

Description

environment.viewport.changed

Triggered whenever the viewport (a specific page within the Teachfloor dashboard) changes.

environment.path.changed

Triggered when the URL path within the Teachfloor application changes.

Example
javascript
API.on('environment.viewport.changed', function(viewport, objectContext) {
    console.log('Viewport changed:', viewport, objectContext);

    if (viewport === 'teachfloor.dashboard.course.detail') {
        // Handle the course detail page specific logic
    }
});
In this example, API.on() is used to listen for changes to the viewport within Teachfloor. When the viewport changes (for instance, navigating to a course detail page), the callback function is triggered, allowing your application to respond appropriately.

API.emit()
The API.emit method in the Teachfloor SDK allows your custom application to trigger specific actions within the Teachfloor platform. One of the currently supported actions is displaying a toast notification in the Teachfloor dashboard using the ui.toast.show event.

Usage
API.emit(action, parameters)

Parameters
action (string): The identifier representing the action you want to trigger. For example, ui.toast.show displays a toast notification in the Teachfloor dashboard.

eventParameters (object): This object contains the configuration of the action.

Actions
Actions in the Teachfloor.js SDK allows you to trigger specific commands within the Teachfloor platform. Below is a list of the currently available actions that you can trigger using the API.emit() method.

Action Identifier

Description

Parameters

ui.toast.show

Displays a toast notification in the Teachfloor dashboard.

message (string): The text to display in the toast notification. This is a required parameter.

autoClose (number): The duration (in milliseconds) for which the toast will be visible before automatically closing. If not provided, the toast may stay open until the user dismisses it.

color (string): The accent color of the toast notification. This helps to differentiate the toast based on the type of message (e.g., success, error, information). Common values might include gray , green , red , etc.

Example
javascript
API.emit('ui.toast.show', {
    message: 'Your message here',
    autoClose: 3000,
    color: 'gray',
});
The ui.toast.show event is used to display a toast notification within the Teachfloor dashboard. This notification appears as a small, temporary message that provides feedback to the user. For example, you might use it to welcome users to a particular section of your course, or display an error message.

When API.emit('ui.toast.show') is called, the toast will display according to the parameters provided. The message parameter is required, and it defines the content of the toast. The autoClose parameter allows you to control how long the toast remains visible, making it disappear automatically after the specified time. The color parameter lets you style the toast for emphasis, helping to convey the type or urgency of the message.

Summary
The Teachfloor.js API methods, API.get() and API.on() , provide a robust foundation for building interactive and responsive integrations with the Teachfloor platform.

Use API.get() to retrieve data and integrate Teachfloor’s resources directly into your custom page.

Use API.on() to subscribe to platform events and react to changes in real-time, ensuring your application remains in sync with the Teachfloor environment.

These methods, coupled with the promise-based architecture of the SDK, allow for streamlined development, enabling you to create highly responsive and feature-rich integrations.

As Teachfloor continues to evolve, additional methods and capabilities will be introduced, expanding the possibilities for your custom implementations.