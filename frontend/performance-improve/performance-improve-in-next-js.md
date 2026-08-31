# Performance improve in Next.js

## Diagnostic Tools
- Lighthouse.
- The Performance tab in developer tools. You can find detailed instructions on how to use it [here](https://www.debugbear.com/blog/lcp-request-discovery).
- The website [WebPageTest](https://www.webpagetest.org), where you can run performance tests on your site and get different metrics. You can also check if there are any blocking scripts.

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
<Image 
  fill 
  src="/example.png"
  sizes={(max-width: 767px) 100vw, (max-width: 1365px) 50vw, 33vw}
/>
```

This code tells the browser the following:
- If the viewport is 767px or smaller, the image should take up 100% of the viewport width.
- If the viewport width is between 768px and 1365px, the image should take up 50% of the viewport width.
- Otherwise (for viewports 1366px and larger), the image should take up 33% of the viewport width.

## Scripts
To add third-party scripts, use the Next.js [`Script`](https://nextjs.org/docs/pages/api-reference/components/script) tag.

- Use [`strategy='beforeInteractive'`](https://nextjs.org/docs/pages/api-reference/components/script#beforeinteractive) only for critically important scripts. In this case, the script loads before any other Next.js modules.
- Use the [`strategy='lazyOnLoad'`](https://nextjs.org/docs/pages/api-reference/components/script#strategy) parameter for scripts that do not critically affect page loading and functionality. This delays the script loading until after all high-priority content has loaded.

In all other cases, it is recommended to keep the default loading behavior with the [`afterInteractive`](https://nextjs.org/docs/pages/api-reference/components/script#afterinteractive) strategy.

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
const Modal = dynamic(
  () => import('@/components/Modal').then((component) => component.Modal),
  {
    ssr: false, // do not render on the server
  }
);
```

If you use a dynamic import for a component that contains important information for SEO, make sure to set `ssr: true`.