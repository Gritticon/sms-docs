---
title: Cache Control for Flutter Web
type: setup
project: sms-ui
last-updated: 2026-03-18
---

# Cache control for Flutter web

To avoid users having to clear cache to see new deployments, the **entry files** must not be cached:

- `index.html`
- `flutter_bootstrap.js`

This repo includes:

- **`_headers`** – used by Netlify and Cloudflare Pages. Ensure it is deployed with your app (e.g. in `build/web/` after `flutter build web`).
- **`.htaccess`** – for Apache; ensure `mod_headers` is enabled.

## Nginx

Add to your server/location block:

```nginx
location = /index.html {
    add_header Cache-Control "no-cache, no-store, must-revalidate";
}
location = /flutter_bootstrap.js {
    add_header Cache-Control "no-cache, no-store, must-revalidate";
}
```

If the app is served under a subpath (e.g. `/app/`), use that path in `location` (e.g. `location = /app/index.html`).

## AWS CloudFront

In the behavior for your app (or default):

1. **Cache policy**: Create a custom policy that sets `Cache-Control: no-cache, no-store, must-revalidate` for requests to `index.html` and `flutter_bootstrap.js`, or
2. **Origin response**: Use a Lambda@Edge or CloudFront Function to set the `Cache-Control` header on responses for `index.html` and `flutter_bootstrap.js` from origin.

Alternatively, create a separate behavior for `index.html` and `flutter_bootstrap.js` with a “CachingDisabled” or custom policy that sets `no-cache`.

## Firebase Hosting

Create `firebase.json` in the Flutter project root (e.g. `sms/`):

```json
{
  "hosting": {
    "public": "build/web",
    "headers": [
      {
        "source": "/index.html",
        "headers": [{ "key": "Cache-Control", "value": "no-cache, no-store, must-revalidate" }]
      },
      {
        "source": "/flutter_bootstrap.js",
        "headers": [{ "key": "Cache-Control", "value": "no-cache, no-store, must-revalidate" }]
      }
    ]
  }
}
```
