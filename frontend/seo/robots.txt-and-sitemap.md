# Robots.txt and Sitemap

## Contents 
- [Testing tools](#testing-tools)
- [Why do you need robots.txt and sitemap.xml files?](#why-do-you-need-robotstxt-and-sitemapxml-files)
- [Robots.txt](#robotstxt)
  - [Robots.txt syntax](#robotstxt-syntax)
    - [User-agent](#user-agent)
    - [Disallow](#disallow)
    - [Allow](#allow)
  - [How to check that robots.txt works](#how-to-check-that-robotstxt-works)
  - [Sitemap](#sitemap)
  - [Host](#host)
- [Sitemap.xml](#sitemapxml)
  - [Structure of sitemap.xml](#structure-of-sitemapxml)
  - [Tag descriptions](#tag-descriptions)
  - [Recommendations for setting priorities](#recommendations-for-setting-priorities)
  - [Sitemap limits](#sitemap-limits)
- [Setting up automatic generation of robots.txt and sitemap.xml in Next.js](#setting-up-automatic-generation-of-robotstxt-and-sitemapxml-in-nextjs)

## Testing tools
- [Google Search Console](https://search.google.com/search-console/about) 
- [Yandex.Webmaster](https://webmaster.yandex.ru/welcome/)

## Why do you need robots.txt and sitemap.xml files?

The robots.txt and sitemap.xml files play an important role in optimizing a website for search engines. These files help search engine robots index your site correctly, which affects how the site's pages appear in search results.

## Robots.txt 

Robots.txt is just a text file that helps search engine robots understand which pages should be indexed and which ones should not.

### Robots.txt syntax

The file is built on several key directives.

#### User-agent

This shows which robot the rules are meant for.
```
- User-agent: * – rules for all robots.

- User-agent: Googlebot – only for Google.

- User-agent: YandexBot – only for Yandex.
```

#### Disallow

This blocks scanning of the specified path.
```
- Disallow: (empty value) – blocks nothing.

- Disallow: /components – blocks any URL that starts with /components.

- Disallow: /admin/ – blocks the whole admin directory and everything inside it.
```

Please note: a slash / at the end of the path means the whole directory. Without a slash, any URL that starts with the given string is blocked.

#### Allow

This allows scanning inside a blocked directory. It is useful for exceptions.

```
User-agent: *
Disallow: /catalog/
Allow: /catalog/main-page/
```

### How to check that robots.txt works

You need to add `/robots.txt` to the URL after the domain, for example `https://example.com/robots.txt`. After that, you should see the contents of robots.txt.

### Sitemap

This shows the robot the path to the site map. It is placed on a separate line outside the User-agent blocks.

```
User-agent: *
Disallow: /components

Sitemap: https://example.com/sitemap.xml
```

### Host

An outdated field that you don't have to include.

## Sitemap.xml

The sitemap.xml file is a site map that helps search engines better understand the structure of your site and find new or updated pages faster. Unlike robots.txt, which limits access, sitemap.xml is aimed at improving indexing. This file contains a list of all the site's URLs, as well as additional information about each URL, such as the date of the last update, the update frequency, and the priority.

A sitemap is especially useful when:

- You have more than a few hundred pages.

- The content is updated often (an online store, a news site, a blog).

- The site is new and has few external links.

### Structure of sitemap.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com/</loc>
    <lastmod>2026-10-12</lastmod>
    <changefreq>daily</changefreq>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://example.com/news/</loc>
    <lastmod>2026-10-13</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.8</priority>
  </url>
</urlset>
```

### Tag descriptions:

- loc – the full URL of the page. Required tag *.

- lastmod – the date of the last change. Google uses this tag and recommends giving the real date, not the current one.

- changefreq – how often the page changes (daily, weekly, monthly). In practice, Google says it ignores this tag, but Yandex takes it into account.

- priority – the priority from 0.0 to 1.0 relative to other pages on the site. It doesn't affect rankings, but it hints to the robot what to scan first.

### Recommendations for setting priorities:

Home page (/) – 1.0

Key sections and categories – 0.8 – 0.9

Articles, products, subcategories – 0.6 – 0.7

Secondary pages (blog, FAQ, contacts) – 0.4 – 0.5

Technical and service pages – 0.1 – 0.3

### Sitemap limits

A single sitemap.xml file can contain no more than 50,000 URLs and weigh no more than 50 MB. If the site is large, you need to use an index file. This is an XML file that links to several regular sitemaps.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <sitemap>
    <loc>https://example.com/sitemap1.xml.gz</loc>
    <lastmod>2026-08-15</lastmod>
  </sitemap>
  <sitemap>
    <loc>https://example.com/sitemap2.xml.gz</loc>
    <lastmod>2026-06-05</lastmod>
  </sitemap>
</sitemapindex>
```

## Setting up automatic generation of robots.txt and sitemap.xml in Next.js

> An example configuration will be provided for Next.js projects using the Pages Router.

1. Install the npm package [next-sitemap](https://www.npmjs.com/package/next-sitemap) it allows you to generate `robots.txt` and `sitemap.xml` automatically after the project build.

2. Create a file named `next-sitemap.config.js` in the project root it will contain all the configuration.

Example configuration for a home site:

```js
module.exports = {
  // Base url of your website
  siteUrl: `https://tourmalinecore.com`,
  // Generate a robots.txt file and list the generated sitemaps. Default false
  generateRobotsTxt: true,
  // Array of relative paths (wildcard pattern supported) to exclude from listing on sitemap.xml or sitemap-*.xml.
  // e.g.: ['/page-0', '/page-*', '/private/*'].
  exclude: [
    '/**/components',
    '/**/components/*',
    '/**/404',
    '/**/500',
  ],
  // Settings for generating robots.txt
  robotsTxtOptions: {
    policies: [
      process.env.ENABLE_SEO_INDEXING === `true` ? {
        userAgent: `*`,
        disallow: [
          '/components',
          '/*/components',
        ],
      }: {
        userAgent: `*`,
        disallow: `/`,
      }
    ],
    ...(process.env.ENABLE_SEO_INDEXING === `true` && {
      // Additional sitemaps that need to be specified in robots.txt.
      // This is useful if you have a dynamic sitemap that is generated by a separate API endpoint.
      additionalSitemaps: [`https://tourmalinecore.com/api/get-sitemap`],
    }),
  },
};
```

You can find the rest of the settings [here](https://github.com/iamvishnusankar/next-sitemap#configuration-options)

3. Add the script `"postbuild": "next-sitemap"` to package.json.

```json
"scripts": {
    "build": "npm run copy:pdf-worker && next build",
    "postbuild": "npx next-sitemap",
}
```

Run the `build`, after it finishes, `robots.txt` and `sitemap.xml` should be generated in the public folder.
