# UI Security Approaches (CORS, CSP)

The frontend app is the first thing an attacker sees, so we should not forget about security here. On the UI side, security mostly comes from HTTP response headers. They tell the browser who can read our responses, whether the page can be put inside a frame, which browser APIs are available, and where scripts, styles and other resources can be loaded from.

Here we describe the approach we follow in Next.js projects. An example implementation is [pelican-ui](https://github.com/TourmalineCore/pelican-ui). How we added CSP is described in the [ADR about Content Security Policy](https://github.com/TourmalineCore/pelican-documentation/blob/master/architecture%20decision%20records/Content%20Security%20Policy.md), and CORS is described in the [ADR about Cross-Origin Resource Sharing](https://github.com/TourmalineCore/pelican-documentation/blob/master/architecture%20decision%20records/Cross-Origin%20Resource%20Sharing.md).

## Contents
1. [Where headers are set](#where-headers-are-set)
2. [CORS](#cors)
3. [Other security headers](#other-security-headers)
4. [Content Security Policy (CSP)](#content-security-policy-csp)
5. [Subresource Integrity (SRI)](#subresource-integrity-sri)
6. [Development and rollout](#development-and-rollout)
7. [Testing](#testing)
8. [Pros and cons](#pros-and-cons)

## Where headers are set

| Headers | Where | Why |
|---|---|---|
| CORS, `Cross-Origin-Opener-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy` | `headers()` in `next.config.mjs` | The values are the same for all requests |
| `Content-Security-Policy` | `src/middleware.ts` | The policy has a nonce that changes with every request |

Values that depend on the environment are passed through environment variables.

```bash
# CORS
CORS_ORIGIN=https://example.com

# CSP
CSP_ENABLED=true
CSP_SCRIPT_SRC_URLS="https://analytics.example.com"
CSP_IMG_SRC_URLS="https://cdn.example.com"
CSP_FONT_SRC_URLS="https://cdn.example.com"
CSP_STYLE_SRC_URLS="https://cdn.example.com"
CSP_MEDIA_SRC_URLS="https://storage.example.com"
CSP_FRAME_SRC_URLS="https://video.example.com"
CSP_CONNECT_SRC_URLS="https://cdn.example.com"
```

## CORS

Browsers follow the **Same-Origin Policy**: JavaScript that runs on a page of one origin (scheme + host + port) cannot read responses from another origin. **CORS** (Cross-Origin Resource Sharing) is a set of headers that a server uses to clearly allow other origins to read its responses.

It is important to understand what CORS does:
- CORS is checked by the **browser**. It does not block requests from curl, Postman or from another server.
- CORS does not let JavaScript **read the response**, but it does not stop the request from **being sent**. "Simple" requests (`GET`, form `POST`) will reach the server anyway. For other requests (`PUT`, `DELETE`, custom headers) the browser first sends a **preflight** `OPTIONS` request and checks the CORS headers it gets back.
- CORS headers are set by the **server**. The UI server headers only decide who can read the resources of the UI itself through fetch/XHR. If the UI calls an API on another domain, CORS must be set up **on the API side**.
- CORS does not control normal embedding of our images, scripts and styles on other sites. If you need to limit this, use the `Cross-Origin-Resource-Policy` header.

### Our configuration

```javascript
// next.config.mjs
async headers() {
  return [
    // ...
    {
      source: '/(.*)',
      headers: [
        // Responses to requests with cookies are not given to other origins
        {
          key: "Access-Control-Allow-Credentials",
          value: "false",
        },

        // Only pages from the given origin are allowed to read responses with JavaScript
        {
          key: "Access-Control-Allow-Origin",
          value: process.env.NODE_ENV === "production"
            ? process.env.CORS_ORIGIN
            : process.env.CORS_ORIGIN || "http://localhost:3000",
        },
        // The browser checks this list only before requests with preflight
        {
          key: "Access-Control-Allow-Methods",
          value: "GET, OPTIONS",
        },

        // Not related to CORS: another origin does not get a link to our window
        // Protects from tabnabbing (our tab is replaced with a phishing one) and XS-Leaks (getting user data in indirect ways)
        {
          key: "Cross-Origin-Opener-Policy",
          value: "same-origin",
        },

        // ...
```

## Other security headers

These headers are set in the same `headers()` as CORS:

```javascript
// next.config.mjs
// Allows only pages of our origin to put the site inside a frame
// When CSP is on, the browser uses the stricter frame-ancestors 'none' instead
{
  key: 'X-Frame-Options',
  value: "SAMEORIGIN",
},

// Does not let the browser guess the file type: a script and a style will run only if they have the correct MIME type (protection from scripts disguised as images, etc.)
{
  key: 'X-Content-Type-Options',
  value: "nosniff",
},

// Does not send the page URL (it can contain important data) in the Referer header
{
  key: 'Referrer-Policy',
  value: "no-referrer",
},

// Turns off browser APIs that the site does not need
{
  key: 'Permissions-Policy',
  value: "interest-cohort=(), camera=(), microphone=(), geolocation=(), fullscreen=(self), payment=(), usb=(), accelerometer=(), display-capture=(), gyroscope=(), magnetometer=(), midi=(), picture-in-picture=(self), xr-spatial-tracking=()",
}
```

## Content Security Policy (CSP)

**Content Security Policy** is set with the `Content-Security-Policy` HTTP header and decides which content sources (scripts, styles, images, etc.) are allowed on the page. It is an extra level of protection from **XSS** and from loading harmful resources: even if an attacker adds code to the page, the browser will not run it.

### Directives

- **default-src** - the default value for all `*-src` directives that are not set directly
- **script-src** - sources of scripts
- **style-src** - sources of styles
- **img-src** - sources of images
- **connect-src** - addresses for `fetch`, XHR, WebSocket
- **font-src** - sources of fonts
- **media-src** - sources of audio and video
- **frame-src** - sources of frames that are embedded in our page
- **manifest-src** - sources of the app manifest
- **base-uri** - allowed addresses for the `<base>` element
- **form-action** - where forms are allowed to be sent
- **frame-ancestors** - which sites can put our page inside a frame

### Source values

| Value                 | Meaning                                                                                                          |
|-----------------------|------------------------------------------------------------------------------------------------------------------|
| `'self'`              | The same origin as the page                                                                                      |
| `'none'`              | Nothing is allowed                                                                                               |
| `https://example.com` | A specific trusted origin                                                                                        |
| `'nonce-<value>'`     | Allows `<script>`/`<style>` elements with a matching `nonce` attribute                                           |
| `'strict-dynamic'`    | A script with a nonce can load other scripts. Addresses and `'self'` are ignored                                 |
| `'unsafe-inline'`     | Allows any inline scripts/styles and `style` attributes. It is ignored if the same directive has a nonce         |
| `'unsafe-eval'`       | Allows `eval()`, `new Function()` and similar                                                                    |

### Our policy

```typescript
// src/common/middleware/setCspHeaders.ts
export function setCspHeaders({
  headers,
}: {
  headers: Headers;
}): void {
  // Create a unique random nonce for every request
  const nonce = Buffer.from(crypto.randomUUID())
    .toString(`base64`);

  const csp = `
    default-src 'none';
    script-src 'self' 'strict-dynamic' 'nonce-${nonce}' ${process.env.CSP_SCRIPT_SRC_URLS};
    style-src 'self' 'unsafe-inline' ${process.env.CSP_STYLE_SRC_URLS};
    img-src 'self' ${process.env.CSP_IMG_SRC_URLS};
    font-src 'self' ${process.env.CSP_FONT_SRC_URLS};
    media-src 'self' ${process.env.CSP_MEDIA_SRC_URLS};
    frame-src ${process.env.CSP_FRAME_SRC_URLS};
    connect-src 'self' ${process.env.CSP_CONNECT_SRC_URLS};
    manifest-src 'self';
    base-uri 'none';
    frame-ancestors 'none';
    form-action 'none';
    upgrade-insecure-requests;
  `
    .replace(/\s{2,}/g, ` `) // Replace two or more spaces with one space
    .trim(); // Remove spaces at the start and at the end

  // Add the nonce and CSP to the headers
  headers.set(`x-nonce`, nonce);
  headers.set(`Content-Security-Policy`, csp);
}
```

Why the policy is built this way:
- **`default-src 'none'`**: by default everything is blocked, and each resource type is allowed directly. A forgotten directive leads to a blocked resource, not to a security hole.
- **`script-src` with nonce and `'strict-dynamic'`**, without `'unsafe-inline'` and `'unsafe-eval'`: only the scripts that we wrote and the scripts they load will run. An attacker does not know the nonce, so an added `<script>` is blocked.
- **`style-src 'self' 'unsafe-inline'`** without nonce: Next.js renders inline `style` attributes (for example, in `next/image`), and nonce does not work for attributes. With a nonce, `'unsafe-inline'` would be ignored, and these styles would be blocked. Added styles are less dangerous than scripts, so we accept this trade-off.
- **`base-uri 'none'`**: does not allow adding a `<base href>` that would send relative URLs to the attacker's host.
- **`form-action 'none'`**: HTML forms cannot be sent. The directive does not affect sending data with `fetch`.

### Creating the nonce in middleware

The nonce must be **unique for every request**: if you set it once in `next.config`, it will be the same for all users, and it can be copied from any page. That is why CSP is set in middleware. Its `matcher` skips API, static files and prefetch requests: only HTML pages need CSP.

```typescript
// src/middleware.ts
export function middleware(request: NextRequest) {
  // Clone original request headers
  const requestHeaders = new Headers(request.headers);

  // Create the next response with request headers
  const response = NextResponse.next({
    request: {
      headers: requestHeaders,
    },
  });

  if (process.env.CSP_ENABLED === `true`) {
    // Set the CSP header in the actual HTTP response to the browser
    setCspHeaders({
      headers: response.headers,
    });
  }

  return response;
}

// Middleware configuration: match all paths EXCEPT specific static and API files
export const config = {
  matcher: [
    {
      // Apply the middleware to everything except Next.js API routes and static assets
      source: `/((?!api|_next/static|_next/image|favicon.ico).*)`,
      // These headers are added by Next.js or browsers when preloading routes or resources
      // We skip applying middleware to avoid unnecessary work or CSP processing for them
      missing: [
        {
          type: `header`,
          key: `next-router-prefetch`,
        },
        {
          type: `header`,
          key: `purpose`,
          value: `prefetch`,
        },
      ],
    },
  ],
};
```
Framework scripts are rendered by the `<Head>` and `<NextScript>` components in `_document.tsx`. So the nonce is read from the request in `getInitialProps` of the custom Document and passed to these components and to every `<Script>`:

```tsx
// src/pages/_document.tsx
class AppDocument extends Document {
  static async getInitialProps(ctx: DocumentContext) {
    const initialProps = await Document.getInitialProps(ctx);
    const nonce = ctx.req?.headers[`x-nonce`] as string | undefined;
    return {
      ...initialProps,
      nonce,
    };
  }

  render() {
    const {
      nonce,
    } = (this.props as any);

    // ...

    return (
      <Html lang="ru">
        <Head nonce={nonce}>
          <script
            nonce={nonce}
            dangerouslySetInnerHTML={{
              __html: `window.__NONCE__ = ${JSON.stringify(nonce)};`,
            }}
          />
          {/* ... */}
        </Head>

        <body>
          {/* ... */}

          <Main />
          <NextScript nonce={nonce} />

          {/* ... */}

          <Script
            strategy="lazyOnload"
            nonce={nonce}
            src="https://culturaltracking.ru/static/js/spxl.js?pixelId=19304"
            data-pixel-id="19304"
            async
          />

          {/* ... */}
        </body>
      </Html>
    );
  }
}
```

## Subresource Integrity (SRI)

The nonce allows our scripts, but if a third-party CDN is hacked, the script it serves will still run. Subresource Integrity protects from this: the browser calculates the hash of the loaded file and does not run it if the hash does not match the `integrity` attribute.

```html
<script
  src="https://cdn.example.com/lib@1.2.3/lib.min.js"
  integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
  crossorigin="anonymous"
></script>
```

- `integrity` - the hash of the expected file content
- `crossorigin="anonymous"` is required for files from another origin, otherwise the browser cannot check the hash

SRI works only for **external files with a fixed version**. The provider updates analytics loaders without changing the URL, and a hash would break them. For such scripts the risk of the provider being hacked stays, so there should be as few third-party scripts as possible.

### Why we want to stop using hashes for inline scripts

Right now in pelican-ui, the inline scripts of Yandex Metrica and the Gosuslugi widget get an `integrity` attribute with the hash of their content:

```tsx
// src/pages/_document.tsx
const yMetricHash = getHash({
  content: yMetricScript,
});

// ...

<Script
  id="yandex-metrika"
  strategy="lazyOnload"
  nonce={nonce}
  integrity={yMetricHash}
  crossOrigin="anonymous"
  dangerouslySetInnerHTML={{
    __html: yMetricScript,
  }}
/>
```

But this check does not protect anything:
1. The browser ignores `integrity` and `crossOrigin` for these scripts without `src`.
2. `getHash` calculates a hash that always matches.
3. The inline script itself is in our code, and the provider cannot replace it because of CSP. The provider can only replace the external file that this script loads. We can only trust the security of the platform we load the script from.

So `getHash` and the `integrity`/`crossOrigin` attributes on inline scripts are not needed, and we will remove them in the future.

## Development and rollout

### Local development
In dev mode, Next.js and React use `eval` and inline scripts, so the site does not work locally with our policy. Locally, CSP is turned off with `CSP_ENABLED=false`. On all deployed environments (local-env, production) it must be turned on, so that CSP problems are caught by e2e tests.

### Adding a new policy
Adding CSP to an existing site blocks everything that was not taken into account. Follow these steps:
1. Instead of `Content-Security-Policy`, send the `Content-Security-Policy-Report-Only` header. It does not block anything, it only shows violations in the console. Remove the `upgrade-insecure-requests` directive from it: it does not work in Report-Only.
2. Go through all pages and fix the violations: add missing sources, add a nonce to inline scripts, remove inline code you do not need.
3. Switch to `Content-Security-Policy` when there are no violations left.

## Testing

Security headers are easy to lose during refactoring or a framework update, so we cover them with [Playwright tests](https://github.com/TourmalineCore/pelican-ui/blob/master/playwright-tests/security-headers-tests/security-headers.spec.ts). The tests check headers not only on the HTML page, but also on JS, images and fonts. CSP is checked only if it is turned on. The test takes the expected nonce from `window.__NONCE__`.

Besides automated tests, check by hand:
- There are no CSP errors in the browser console on all page types. Later we will add an automated check that the console has no errors.
- The nonce in the `Content-Security-Policy` header matches the `nonce` of the scripts on the page and changes after a reload. Compare with the page source code (View Page Source), because the browser hides the nonce value in the Elements panel.
- Online scanners: [Mozilla HTTP Observatory](https://developer.mozilla.org/en-US/observatory), [Google CSP Evaluator](https://csp-evaluator.withgoogle.com/), where you can check the security coverage percentage of the site.