# Performance Improvements in Next.js

***This document is focused on `Pages router`, and some sections will not be relevant for the `App router`.***

## Overview
- [Diagnostic Tools](#diagnostic-tools)
- [GetStaticProps vs GetServerSideProps](#getstaticprops-vs-getserversideprops)
- [Images](#images)
- [Fonts](#fonts)
- [Scripts](#scripts)
- [Navigation (Link)](#navigation-link)
- [Dynamic imports](#dynamic-imports)
- [Installing Dependencies](#installing-dependencies)

---

## Diagnostic Tools
- Lighthouse.
- The Performance tab in developer tools. You can find detailed instructions on how to use it [here](https://www.debugbear.com/blog/lcp-request-discovery).
- The website [WebPageTest](https://www.webpagetest.org), where you can run performance tests on your site and get different metrics. You can also check if there are any blocking scripts.
- Package [@next/bundle-analyzer](https://www.npmjs.com/package/@next/bundle-analyzer) allows you to analyze the project bundle and see heavy libraries or scripts that may affect the site's performance.

## GetStaticProps vs GetServerSideProps

[`getServerSideProps`](https://nextjs.org/docs/pages/building-your-application/data-fetching/get-server-side-props) and [`getStaticProps`](https://nextjs.org/docs/pages/building-your-application/data-fetching/get-static-props) — these are two key functions for preloading data in Next.js (they are used only in the Pages Router). They are needed to load data on the server side and pass it to components as props before the page is sent to the browser. This is very important for SEO (search engines see ready HTML with content right away) and for security (you can hide secret keys or the server domain).

The difference between them is that `getStaticProps` fetches the data at build time, while `getServerSideProps` fetches the data on every user request.

Use `getStaticProps + ISR (revalidate)` for pages that don't change every second (articles, catalogs, static pages). This helps you get the lowest TTFB (Time to First Byte) and reduce the load on the server.

If the data comes from a CMS, don't forget to set the `revalidate` option and specify the time in seconds after which Next.js will rebuild the page in the background (make a request to the CMS for new data). 

Without `revalidate`, the page is generated once during the build and never updates again — changes in the CMS won't appear until the next deploy.

Use `getServerSideProps` only when the data needs to be up to date on every page load (shopping cart, dashboard). SSR increases TTFB (Time to First Byte)
by the time it takes to run all requests on the server — the slower the CMS/database, the worse the TTFB.

## Images
For most images on the website, it is recommended to use the Next.js [`Image`](https://nextjs.org/docs/pages/api-reference/components/image) tag. It automatically optimizes images: it converts them to a modern format, adjusts them to the device screen size, and has lazy loading (this is the default setting of the tag, so you don't need to add it manually). With lazy loading, the image starts loading only after it enters the viewport.

**When you should NOT use the Image tag:**
- Vector images — Next.js does not optimize the SVG format. However, if you still want to use an SVG with the Image tag, remember to add the [`unoptimized`](https://nextjs.org/docs/pages/api-reference/components/image#unoptimized) property. This way, you clearly tell Next.js not to waste resources trying to optimize it.
- Background images — when using background-image in CSS. The Image component does not directly support CSS background properties.

**Optimization methods:**
- Compress images manually using tools like [`tinypng`](https://tinypng.com) and [`SVGOMG`](https://jakearchibald.github.io/svgomg), or other similar tools.
- Use the [`priority`](https://nextjs.org/docs/pages/api-reference/components/image#priority) property and fetchPriority="high" for images that are in the viewport when the page loads (for example, the first blocks like Hero sections).
- Use the [`loading="eager"`](https://nextjs.org/docs/pages/api-reference/components/image#loading) property for images inside modals or popups. This allows such images to start loading as soon as they appear in the DOM tree.
- Control the caching time for optimized images (this only affects images added through the Image component) using the [`minimumCacheTTL`](https://nextjs.org/docs/pages/api-reference/components/image#minimumcachettl) property.
- It is recommended to use conversion to webp and not to use conversion to avif [see here](https://nextjs.org/docs/pages/api-reference/components/image#formats). Although the avif format compresses about 20% more, it uses a lot of system resources and takes about 50% more time compared to webp. We have already had a bad experience with using avif on a corporate website. When opening a page with many images, the site crashed because the virtual machine where it was hosted did not have enough resources.
- Use the [`sizes`](https://nextjs.org/docs/pages/api-reference/components/image#sizes) property for images that use the [`fill`](https://nextjs.org/docs/pages/api-reference/components/image#fill) property. This helps the browser choose the right image size based on the `sizes` settings.

```js
import Image from "next/image";

<Image 
  fill 
  src="/example.png"
  sizes="(max-width: 767px) 100vw, (max-width: 1365px) 50vw, 33vw"
/>
```

This code tells the browser the following:
- If the viewport is 767px or smaller, the image should take up 100% of the viewport width.
- If the viewport width is between 768px and 1365px, the image should take up 50% of the viewport width.
- Otherwise (for viewports 1366px and larger), the image should take up 33% of the viewport width.

## Fonts
To add fonts to your project, use the [next/font](https://nextjs.org/docs/app/api-reference/components/font) component. It helps make your fonts load faster and more efficiently.

Use the `display: swap` option. This tells the browser to show the text right away using a backup font. Then, when your main font is ready, the browser will switch to it automatically. This way, users can read the text without waiting.

## Scripts
To add third-party scripts, use the Next.js [`Script`](https://nextjs.org/docs/pages/api-reference/components/script) tag.

- Use [`strategy='beforeInteractive'`](https://nextjs.org/docs/pages/api-reference/components/script#beforeinteractive) only for critically important scripts. In this case, the script loads before any other Next.js modules.
- Use the [`strategy='lazyOnLoad'`](https://nextjs.org/docs/pages/api-reference/components/script#strategy) parameter for scripts that do not critically affect page loading and functionality. This delays the script loading until after all high-priority content has loaded.

In all other cases, it is recommended to keep the default loading behavior with the [`afterInteractive`](https://nextjs.org/docs/pages/api-reference/components/script#afterinteractive) strategy.

## Navigation (Link)
For internal navigation, always use the Next.js [`Link`](https://nextjs.org/docs/pages/api-reference/components/link) component instead of standard HTML `<a>` tags. This enables client-side routing, preventing full page reloads and significantly improving the perceived performance of your site.

**When to disable prefetching**
Use [`prefetch={false}`](https://nextjs.org/docs/pages/api-reference/components/link#prefetch) for links that are unlikely to be clicked or exist in large quantities (e.g., footer links, large catalogs, pagination, or sidebar menus with dozens of items). This prevents unnecessary background requests, saving bandwidth and reducing server load. Note that with `prefetch={false}`, Next.js will still prefetch on hover.

**External links**
Do not use `next/link` for external URLs. Use a standard `<a>` tag with `rel="noopener noreferrer"` for security and to avoid interfering with Next.js routing.

**Example:**
```js
import Link from 'next/link'

<Link href="/about">About Us</Link>
```

## Dynamic imports
Dynamic imports help reduce the size of the initial page load and speed up your website without losing functionality. You should use dynamic imports selectively — do not use them everywhere, as this can negatively affect the user experience.

**You should use them for:**
- Components that only appear after user interaction: modals, tooltips, popups, dropdowns, carousels.
- Components with a lot of complex logic: 3D animations, graphics.
- Components that use large third-party libraries: video players, interactive maps.

**You should NOT use them for:**
- Components with `ssr: false` that are located above the fold (on the first screen) — this worsens LCP and SEO. However, with `ssr: true`, dynamic imports are acceptable even for critical components.
- Small and lightweight components — this may worsen the user experience (the component may appear with a jerky or laggy effect).

**Example of a dynamic import:**
```js
import dynamic from "next/dynamic";

const Modal = dynamic(
  () => import('@/components/Modal').then((component) => component.Modal),
  {
    ssr: false, // do not render on the server
  }
);
```

If you use a dynamic import for a component that contains important information for SEO, make sure to set `ssr: true`.

## Installing Dependencies
Install packages in the right environment. This helps make the final bundle smaller, speeds up the build process, and reduces server load.

### What should go in dependencies
Only put packages in dependencies if they are actually used when your app is running in production:
- Frameworks and UI libraries (react, next, react-dom)
- HTTP API Client (axios)
- Date handling packages (dayjs, moment)

To install a package in dependencies, use this command:

```bash
npm install <packageName>
```

### What should go in devDependencies
Put everything in devDependencies that is only needed during development or building:
- Type definitions (@types/*, typescript)
- Linters and formatters (eslint, stylelint)
- Build and post-processing tools (postcss, autoprefixer)
- Testing frameworks (jest, playwright, cypress)

To install a package in devDependencies, use this command:

```bash
npm install -D <packageName>
```