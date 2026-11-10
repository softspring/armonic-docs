---
title: "CMS Analytics Plugin (Experimental)"
description: "Read page traffic in CMS administration with site-specific Plausible or Google Analytics 4 providers."
related: ["bundles/cms-seo-plugin", "components/cms-mcp-plugin"]
---

# CMS Analytics Plugin {#cms-analytics-plugin}

> **Experimental — 6.1-dev:** configuration, APIs and output may change before a stable release. Use this package in development and validate the workflows enabled in your application.

The analytics plugin adds a **Statistics** tab to CMS content and a provider-independent statistics API. Use it to compare traffic for the public URLs associated with a page. The package is experimental.

## Install and register {#install-and-register}

```bash
composer require softspring/cms-analytics-plugin:6.1.*@dev
```

Register the bundle when Flex does not do it automatically:

```php
// config/bundles.php
Softspring\CmsAnalyticsPlugin\SfsCmsAnalyticsPlugin::class => ['all' => true],
```

## Configure Plausible for a site {#configure-plausible-for-a-site}

Merge this fragment into the site's existing definition:

```yaml
# cms/sites/default/config.yaml
site:
    extra:
        analytics:
            driver: plausible
            plausible:
                enabled: true
                api_base_url: 'https://plausible.io'
                api_key: '%env(PLAUSIBLE_STATS_API_KEY)%'
                site_id: '%env(PLAUSIBLE_SITE_ID)%'
```

Use a Stats API key and the provider's site identifier. Set credentials outside committed configuration. For self-hosted Plausible, adapt `api_base_url` to that installation.

## Use Google Analytics 4 instead {#use-google-analytics-4-instead}

Install the optional `google/auth` dependency and select the other driver:

```yaml
site:
    extra:
        analytics:
            driver: google_analytics_4
            google_analytics_4:
                enabled: true
                property_id: '%env(GA4_PROPERTY_ID)%'
                credentials_path: '%env(GA4_CREDENTIALS_PATH)%'
```

The identity must have read access to the numeric GA4 property. The resolver supports `credentials_json`, `credentials_path`, or Application Default Credentials when neither is supplied. Use one credential source appropriate to the runtime.

## Read the statistics {#read-the-statistics}

The plugin resolves content URLs by site, route, and language, then fetches metrics for each URL. Supported ranges include 7, 30, and 91 days, 6 and 12 months, and the current year. Per-URL responses are cached for 15 minutes, so a reload need not make an external request.

Check unconfigured-provider and API-error rows separately from zero traffic. Visitor counts summed across URLs are not unique site-wide visitors: one person can contribute to several URL rows.

The plugin reads statistics; it does not install a browser tracking script or start collecting visits. Configure collection and any required consent behavior in the application/provider integration.

## Add another provider {#add-another-provider}

Implement `Softspring\CmsAnalyticsPlugin\Analytics\StatisticsProviderInterface` and tag the service `sfs_cms_analytics.statistics_provider`. Its API resolves site configuration, returns aggregate metrics, and queries page statistics. Keep provider-specific authentication there rather than coupling consumers such as [CMS MCP](../components/cms-mcp-plugin.md) to one vendor.
