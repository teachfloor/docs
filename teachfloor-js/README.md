# Teachfloor.js SDK

JavaScript SDK for integrating custom web pages with the Teachfloor platform.

## Overview

The Teachfloor.js SDK enables seamless communication between your custom web page and Teachfloor. Built on a promise-based architecture, it allows you to retrieve data, listen to events, and trigger actions within the Teachfloor platform.

## Quick Links

- [Getting Started](./getting-started)
- [SDK Methods](./sdk-methods)
- [Viewports Reference](./viewports-reference)

## Quick Start

```html
<script>
  (function(t,c,h,f,l,r){
    t.tf = t.tf || function() {(t.tf.q = t.tf.q || []).push(arguments)}
    t._tfOpts = { id: 'YOUR_APP_ID' }
    l = c.getElementsByTagName('head')[0];
    r = c.createElement('script');
    r.async = 1;
    r.src = h + t._tfOpts.id + f + Date.now();
    l.appendChild(r)
  })(window, document, 'https://app.teachfloor.com/apps/', '.js?ver=')

  tf('onInit', function(API) {
    // Start using the SDK
  });
</script>
```
