---
title: "CMS sitemaps and robots.txt"
description: "Publish site sitemaps and robots directives and understand how they relate to indexability and SEO analysis."
related: ["bundles/cms-bundle/sites", "bundles/cms-seo-plugin"]
---

# Sitemaps and robots.txt {#sitemaps-and-robots-txt}

CMS sites can expose sitemaps, a sitemap index, and `robots.txt`. These help crawlers discover the public URLs already configured in the CMS. They do not publish a draft page or replace authentication for private content.

## Configure a site {#configure-a-site}

This complete local example uses English under `/en`:

```yaml
# cms/sites/default/config.yaml
site:
    locales: [en]
    default_locale: en
    https_redirect: false
    paths:
        - { path: /en, locale: en }
    robots:
        mode: static
        static_file: '@site/default/robots.txt.twig'
    sitemaps:
        pages:
            url: sitemap.xml
            cache_ttl: 300
            default_changefreq: weekly
            alternates_locales: true
            alternates_sites: true
            alternates_include_hreflang: true
    sitemaps_index:
        enabled: true
        url: sitemap-index.xml
        cache_ttl: 300
```

Merge the relevant blocks into an existing site instead of replacing its hosts, paths, or languages. Choose distinct URLs for robots, each sitemap, and the sitemap index: the configuration rejects reserved-path collisions.

Create the robots template shown below, then clear the Symfony cache after changing site definitions. In this example the matcher serves `/robots.txt`, `/sitemap.xml`, and `/sitemap-index.xml` at the site host's root; it does not prepend `/en`. Sitemap URLs can include an explicit path in configuration. When several sites share a host, use unambiguous hosts and resource URLs so the resolver selects the intended site.

## Use a static robots template {#use-a-static-robots-template}

The example uses `robots.mode: static` to render a project template. Create the corresponding file at `cms/sites/default/robots.txt.twig`:

```twig
User-agent: *
Disallow:
```

`robots.mode: dynamic` is accepted by the schema but its current request-matching branch throws a not-implemented exception. Use the static mode for a working robots response.

Add the actual deployed sitemap URL when appropriate. Do not copy a local hostname into a production robots file.

## Review indexability {#review-indexability}

For a page to appear, it must belong to the site and have a published version. Its indexing settings must enable `sitemap` and leave `noIndex` disabled. The generator includes route paths whose locale is enabled for the site. Configure the public hosts and schemes correctly before enabling alternates across sites.

Inspect the response status, XML or text content, public host, language alternates, and the URLs emitted for published content. A crawler directive is advisory; use security rules for access restrictions.

The current sitemap index generator does not include a non-standard port from the canonical host configuration. Check generated absolute URLs when testing on a local port or deploying behind a proxy.

The optional [SEO plugin](../cms-seo-plugin.md) checks public robots and sitemap resources from the application runtime. A successful schema load alone does not show that a reverse proxy serves those URLs correctly.
