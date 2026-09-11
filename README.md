# portal-forw

Config-only Vercel project that reverse-proxies **[portal.tepl.com.pk](https://portal.tepl.com.pk)** to the origin app at `http://154.57.198.206:8080`. It serves no application of its own.

## How the rewrite works

`vercel.json` defines a single catch-all rewrite:

- `source`: `/:path*` matches every path, including `/`
- `destination`: `http://154.57.198.206:8080/:path*` forwards the same path to the origin

The browser URL stays on `portal.tepl.com.pk`. Vercel’s CDN proxies the request and automatically preserves the query string.

Example: `https://portal.tepl.com.pk/login?next=/home` is fetched from `http://154.57.198.206:8080/login?next=/home`.

## Known limitations

This is a CDN rewrite to a **plain-HTTP** origin, not a full reverse proxy.

- **No WebSockets or streaming.** Upgrade requests, Socket.IO, SSE, and long-lived streams are not supported through this rewrite.
- **Unencrypted last hop.** Visitors reach Vercel over HTTPS, but Vercel talks to the origin over HTTP. Traffic between Vercel and `154.57.198.206:8080` is not TLS-encrypted.
- **Request duration limits.** Proxied requests are subject to Vercel’s platform timeouts. Slow or long-running origin responses can be cut off.
- **Absolute URLs leak the raw IP.** If the origin app emits absolute links, redirects, or asset URLs using `http://154.57.198.206:8080`, those will appear in the browser instead of `portal.tepl.com.pk`. The origin should use relative URLs (or generate URLs from the public host).
