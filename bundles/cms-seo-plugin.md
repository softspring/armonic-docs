---
title: "CMS SEO Plugin (Experimental)"
description: "Analyze published HTML, page metadata, headings, images, robots, and sitemaps from CMS administration."
related: ["bundles/cms-bundle/sitemaps-and-robots", "bundles/cms-analytics-plugin"]
---

# CMS SEO Plugin {#cms-seo-plugin}

> **Experimental — 6.1-dev:** configuration, APIs and output may change before a stable release. Use this package in development and validate the workflows enabled in your application.

The SEO plugin analyzes the public HTML of CMS content and the discovery resources of CMS sites. It adds a dashboard and SEO tabs to content and site administration. The package is experimental.

## Install and register {#install-and-register}

```bash
composer require softspring/cms-seo-plugin:6.1.*@dev
```

Register the bundle when Flex does not do it automatically:

```php
// config/bundles.php
Softspring\CmsSeoPlugin\SfsCmsSeoPlugin::class => ['all' => true],
```

## Check published pages {#check-published-pages}

Publish the intended version and make its route reachable from the PHP application's runtime. Open its SEO tab to inspect status, HTML content type, title, description, canonical and alternate URLs, H1 and heading structure, image alternative text, robots directives, and approximate word count.

Content results are stored in `version.meta.sfs_cms_seo_analysis`. Reopening a tab can display that stored result; choose **Update data** to fetch and analyze again. The dashboard summarizes stored published-version results rather than recalculating every page on each visit.

## Configure site checks {#configure-site-checks}

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_cms_seo.yaml
sfs_cms_seo:
    site_analysis:
        enabled: true
        checks:
            robots_reachable: true
            robots_sitemap_index: true
            sitemap_index_reachable: true
            sitemap_index_contains_configured_sitemaps: true
            sitemap_reachable: true
            slash_route_redirect: true
```

```php
<?php

// config/packages/sfs_cms_seo.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_cms_seo', [
        'site_analysis' => [
            'enabled' => true,
            'checks' => [
                'robots_reachable' => true,
                'robots_sitemap_index' => true,
                'sitemap_index_reachable' => true,
                'sitemap_index_contains_configured_sitemaps' => true,
                'sitemap_reachable' => true,
                'slash_route_redirect' => true,
            ],
        ],
    ]);
};
```

<!-- config-tabs:end -->

Disable checks for resources intentionally absent from the project. Configure [sitemaps and robots](cms-bundle/sitemaps-and-robots.md) in the site definition before interpreting their analysis.

## Interpret the result {#interpret-the-result}

A score is a summary of deterministic checks, not a search ranking prediction. Read the failing check and its details, correct the template or content, recompile if needed, and fetch again.

The fetcher adds a random `sfs_cms_seo_bust` parameter to reduce reuse of stale proxy HTML. That does not regenerate stored compiled CMS output. The editor can still analyze old HTML if the affected published version was not recompiled.

For `.local` hosts, the current fetcher routes through `http://balancer:8008` with the original Host header. Projects with another local topology need an adapted fetcher; browser reachability from the developer's machine is insufficient.

## Extend analysis {#extend-analysis}

Implement `Content\Check\CheckInterface` for a content check or `Site\Check\SiteCheckInterface` for a site check in the plugin's namespace. Autoconfiguration adds the corresponding tags. Replace `SeoAnalysisDispatcherInterface` for a project-specific asynchronous dispatcher; the default implementation runs synchronously even when an integration exposes an async option.
