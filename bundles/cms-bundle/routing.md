---
title: "CMS routes and Symfony URLs"
description: "Connect CMS pages to public URLs, resolve links in Twig, and keep application routes distinct from CMS content."
related: ["create-routes", "bundles/cms-bundle/sitemaps-and-robots"]
---

# CMS routes and Symfony URLs {#cms-routes-and-symfony-urls}

A content record can have different public paths by site and language. The CMS route identifier is the stable reference used by menus and modules; a route path is the URL a visitor opens.

For example, one page can use `/en/about` and `/es/nosotros`. Moving one path should not require replacing every stored reference to the route.

## Register the frontend {#register-the-frontend}

Import the frontend route resource as described in [Getting started](getting-started.md#frontend-routes). Match its locale prefix to the site paths and keep more specific Symfony routes ahead of the CMS catch-all routes.

If the application also serves APIs, a shop, or other pages outside CMS sites, configure:
<!-- config-tabs:start -->

```yaml
# config/packages/sfs_cms.yaml
sfs_cms:
    site:
        throw_not_found: false
```

```php
<?php

// config/packages/sfs_cms.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_cms', ['site' => ['throw_not_found' => false]]);
};
```

<!-- config-tabs:end -->

This lets requests outside a matching CMS site continue through the application. It does not create CMS paths or make missing published content available.

## Create the route record {#create-the-route-record}

Create the page, save and publish a version, and add its CMS route with the intended site, language, and path. See [Create routes](../../user-manual/create-routes.md). Check for collisions with existing Symfony routes, robots and sitemap paths, and other CMS paths.

## Generate URLs in templates {#generate-urls-in-templates}

For a saved route-selection value named `button_link`:

```twig
{% if button_link.route_name|default(false) %}
    <a href="{{ sfs_cms_url(button_link) }}">
        Read more
    </a>
{% endif %}
```

Use the CMS helper for values that may reference a CMS route. For a known application route, Symfony's `path()` and `url()` remain available. When editors need anchors or external URLs as well as routes, use the [link field](module/form-types/link.md).

## Protect application paths {#protect-application-paths}

The application-level `sfs_cms.route.restricted_paths` option reserves paths that CMS routes must not occupy. Add your own login, API, and application prefixes to the existing list rather than assuming every route is editorial content. Keep the restriction aligned with the site's actual path prefixes.

Test a changed URL in each enabled locale, including links from menus and modules. A working route does not imply that the selected content has a published version, and an admin preview URL is not the public URL.
