---
title: "CMS API Plugin (Experimental)"
description: "Expose selected CMS configuration and content through authenticated read-only JSON endpoints."
related: ["components/cms-mcp-plugin", "bundles/cms-bundle/security-and-permissions"]
---

# CMS API Plugin {#cms-api-plugin}

> **Experimental — 6.1-dev:** configuration, APIs and output may change before a stable release. Use this package in development and validate the workflows enabled in your application.

The API plugin exposes CMS configuration and stored instances through read-only JSON endpoints. Use it for a controlled application client that needs content structure or published versions. The route and payload contracts are experimental.

## Install and register {#install-and-register}

```bash
composer require softspring/cms-api-plugin:6.1.*@dev
```

Register the bundle when Flex does not do it automatically:

```php
// config/bundles.php
Softspring\CmsApiPlugin\SfsCmsApiPlugin::class => ['all' => true],
```

Its dependencies include CMS core, Media Bundle, Sections Plugin, and NelmioApiDocBundle. Enable the dependent bundles before warming the application container.

<!-- config-tabs:start -->

```yaml
# config/routes/sfs_cms_api.yaml
_sfs_cms_api:
    resource: '@SfsCmsApiPlugin/config/routing/api.yaml'
    prefix: /api/cms
```

```php
<?php

// config/routes/sfs_cms_api.php
namespace Symfony\Component\Routing\Loader\Configurator;

return static function (RoutingConfigurator $routes): void {
    $routes->import('@SfsCmsApiPlugin/config/routing/api.yaml')->prefix('/api/cms');
};
```

<!-- config-tabs:end -->

Make the prefix part of an authenticated application firewall or gateway policy. The plugin does not implement an authentication scheme. A firewall restricted to `/admin` and `/app` will not automatically protect `/api/cms`.

## Choose an endpoint {#choose-an-endpoint}

With that prefix:

| Resource | GET path |
| --- | --- |
| Sites and content types | `/sites`, `/content-types` |
| Module, layout, block and media definitions | `/modules`, `/layouts`, `/block-types`, `/media-types` |
| Content list | `/content/{type}` |
| Content detail | `/content/{type}/{id}` |
| Published version | `/content/{type}/{id}/versions/published` |
| Other records | `/routes`, `/menus`, `/blocks`, `/sections`, `/media` |

Paths in the table are relative to `/api/cms`. Detail endpoints exist for those other records; sections also expose a published-version endpoint.

## Request published content explicitly {#request-published-content-explicitly}

For an authenticated client, a list request can use:

```http
GET /api/cms/content/page?publishedOnly=true&locale=en&limit=10
```

The content list defaults to `publishedOnly=false`, so a protected integration can see draft-bearing records unless it asks otherwise. List limits are capped at 100. Query options vary by resource; inspect its controller/OpenAPI contract instead of assuming every endpoint accepts every filter.

Use the published-version endpoint when the client needs the selected published payload. Content detail defaults to excluding version data unless `includeData` is enabled. Missing content types, records, or published versions return an error with HTTP 404.

## Limit configuration exposure {#limit-configuration-exposure}

Some endpoints accept `includeRawConfig`. Shared serializers redact known sensitive values, but redaction is not a substitute for deciding which clients may inspect configuration and metadata. Keep raw configuration off unless the consumer needs it.

This API does not import, edit, save, or publish content. Use the [data plugin](../bundles/cms-data-plugin.md) for import/export workflows and [CMS MCP](cms-mcp-plugin.md) for agent-oriented tools.
