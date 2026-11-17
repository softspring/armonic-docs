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

## Inspect words in a content version {#version-word-statistics}

The version information page and the **SEO Analysis** tab show two rankings for each content language:

- **Most frequent words**: occurrences and percentage of all content words.
- **Most emphasized words**: terms appearing in semantic headings or emphasis tags, ordered by their emphasis weight.

This analysis also works with drafts. It renders the selected version's module containers through the CMS renderer,
without publishing, saving metadata, fetching a public page, or persisting compiled HTML. Results depend on the selected
site and locale, and on reusable content included by those modules. They are computed from the current server render.
If a module cannot render, the result is unavailable rather than an apparently successful empty analysis.

The scope is the version's containers. Global layout navigation, headers and footers, including a title produced only
by the layout, are excluded. SEO metadata is not counted as body text. The SEO Analysis tab shows the published version
by default, or the last version when the content has no published version. The panel displays its version number.
Use the separate SEO form to adjust metadata after reviewing the statistics; changing metadata does not change body counts.

Each term includes `word`, `count`, `density`, `weightedCount`, `highlightScore`, `highlightedCount`, and per-tag
`contexts`. The result also includes `summary`, `frequentTerms`, `prominentTerms`, the selected version, locale and site,
and `parameters` describing the algorithm. The requested limit only truncates the two rankings, not the totals.

Density is `100 * count / summary.wordCount`. That denominator includes common words before filtering, but excludes
pure numbers and ignored markup. `analyzedWordCount` counts occurrences after filtering; `uniqueWordCount` counts
their distinct terms. A highlighted occurrence contributes once even when several formatting tags are nested.

The default weights are H1=6, H2=5, H3=4, H4=3, H5/H6=2, strong/bold=3, emphasis/italics=2, mark=3 and underline=2.
Each occurrence takes its highest enclosing weight. `weightedCount` sums those weights, using 1 for ordinary text;
`highlightScore` sums only the extra weight above 1. It is an editorial emphasis measure, not a search ranking score.
Per-tag counts may overlap: one occurrence inside H2 and strong contributes to both contexts, but only once to the total.

The tokenizer is case-insensitive, preserves accents, joins words split by inline HTML and separates block elements.
It filters common English and Spanish words, short words, scripts, styles, navigation, footers, forms and explicitly
hidden elements. Other languages can supply stop-word lists. It does not stem word variants, infer related concepts,
segment languages without word separators, or compute CSS styles. These are descriptive counts, not keyword research
or recommended density targets. No AI model or external SEO provider is required.

## Reuse the statistics in Twig or PHP {#reuse-word-statistics}

The Twig function accepts a content version object, then optional locale, site identifier and term limit:

```twig
{% set statistics = sfs_cms_seo_word_statistics(version, 'en', 'main', 20) %}
{% if statistics.ok %}
    {% for term in statistics.frequentTerms %}
        <p>{{ term.word }}: {{ term.count }} ({{ term.density }}%)</p>
    {% endfor %}
{% endif %}
```

Without a locale, it selects the content default. Without a site, it chooses the first enabled content site supporting
that locale. Use your application's identifiers. The selected language must be configured for the content and site.
Twig returns `ok: false` when that context is unavailable; templates should handle it.

An application service can inject `Softspring\CmsSeoPlugin\Content\WordStatistics\VersionWordStatistics` and call:

```php
$statistics = $versionWordStatistics->analyze($version, 'en', 'main', 20);
```

This low-level service accepts a CMS version object and expects the caller to authorize access to it. It rejects an
invalid locale, site or limit with `InvalidArgumentException`. The API adapter applies the host's `CMS_API_READ`
permission and verifies that the requested version belongs to the requested content before invoking it.

## Configure word filtering and weights {#configure-word-statistics}

This is an override fragment. Unspecified settings retain their defaults; custom stop words extend the built-in lists.
Language lists such as `en` apply to regional locales too. Exact locale lists can add further exclusions.

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_cms_seo.yaml
sfs_cms_seo:
    word_statistics:
        limit: 20
        min_word_length: 2
        stop_words:
            en: ['example']
            es: ['ejemplo']
        weights:
            h2: 5
            strong: 3
```

```php
<?php

// config/packages/sfs_cms_seo.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_cms_seo', [
        'word_statistics' => [
            'limit' => 20,
            'min_word_length' => 2,
            'stop_words' => ['en' => ['example'], 'es' => ['ejemplo']],
            'weights' => ['h2' => 5, 'strong' => 3],
        ],
    ]);
};
```

<!-- config-tabs:end -->

Limits must be between 1 and 100, minimum word length between 1 and 20, and weights between 1 and 20. A weight of 1
adds no emphasis bonus. The complete available weight keys are `h1` through `h6`, `strong`, `em`, `mark` and `u`.
Applications may decorate the HTML analyzer, stop-word provider or version statistics service for a different policy.

The [API query](../components/cms-api-plugin.md#version-seo-statistics) and
[MCP tool](../components/cms-mcp-plugin.md#version-seo-statistics) expose this same analysis when their plugins are enabled.

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
