---
title: "CMS API Plugin (Experimental)"
description: "Reuse CMS queries and controlled editorial operations through shared services and optional REST endpoints."
related: ["components/cms-mcp-plugin", "bundles/cms-bundle/security-and-permissions"]
---

# CMS API Plugin {#cms-api-plugin}

> **Experimental — 6.1-dev:** configuration, APIs and output may change before a stable release. Use this package in development and validate the workflows enabled in your application.

The API plugin provides shared CMS queries and controlled operations for REST clients, MCP tools, AI assistants, and application code.
Import its REST routes only when the project needs HTTP endpoints. Registering the bundle alone exposes no REST routes.
The service, route, and payload contracts are experimental.

## Install and register {#install-and-register}

```bash
composer require softspring/cms-api-plugin:6.1.*@dev
```

Register the bundle when Flex does not do it automatically:

```php
// config/bundles.php
Softspring\CmsApiPlugin\SfsCmsApiPlugin::class => ['all' => true],
```

Its runtime dependencies include CMS core, Media Bundle, and Sections Plugin. Enable those bundles before warming
the application container. NelmioApiDocBundle is optional: install and enable it only when generating OpenAPI
documentation for REST endpoints. MCP clients do not need it.

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

## Preview a content version {#preview-content-version}

The authenticated resource API can render a draft with the CMS compiler without publishing it:

```http
GET /api/cms/resources/content_preview/CONTENT_ID?type=article&version=VERSION_ID&locale=es&site=blog
```

Replace the identifiers, content type, locale and site with values from your application. `version` defaults to the
last version, `locale` to the content default, and `site` to the first enabled site assigned to that content.
The requested version must belong to the content. The locale must be enabled for both the content and site.
This resource requires the host permission `CMS_API_READ`.

The response contains `ok`, `html`, `errors`, `containerErrors`, and the selected content, version, locale and site.
Check `ok` and the error lists before reviewing the HTML and its assets in a browser. The compiler does not attach
or persist compiled data for this preview. It does not publish the version or make its public route available.

## Limit configuration exposure {#limit-configuration-exposure}

Some endpoints accept `includeRawConfig`. Shared serializers redact known sensitive values, but redaction is not a substitute for deciding which clients may inspect configuration and metadata. Keep raw configuration off unless the consumer needs it.

This API does not import, edit, save, or publish content. Use the [data plugin](../bundles/cms-data-plugin.md) for import/export workflows and [CMS MCP](cms-mcp-plugin.md) for agent-oriented tools.

## Reuse CMS operations in your application {#reuse-cms-operations}

The API Plugin provides autowired read services under `Softspring\CmsApiPlugin\Application\Query`.
Use these services from an application service, command, controller, or integration when you need CMS data.
They require this API plugin to be enabled, but do not require imported REST routes, Swagger, MCP, an AI platform, or a user bundle.

| Service | Purpose |
| --- | --- |
| `ContentQuery` | Read content records and selected version data |
| `PublishedContentQuery` | Search and resolve published content |
| `SiteQuery` | Read site definitions and site context |
| `MenuQuery` | Read menus and their item trees |
| `ConfigurationQuery`, `ConfigurationContextQuery` | Read CMS definitions and project context |
| `RouteQuery`, `BlockQuery`, `SectionQuery` | Read routes, reusable blocks, and sections |
| `MediaQuery`, `MediaTypeQuery`, `MediaImageQuery` | Read media records, definitions, and image context |

The API Plugin also provides `SectionQuery` using the Sections Plugin managers. `AnalyticsQuery` is registered
only when the optional Analytics Plugin is enabled, and uses its configured statistics provider chain. All these
queries belong to `Softspring\CmsApiPlugin\Application\Query`.

For example, a service can search published pages without making an HTTP request to its own API:

```php
<?php

namespace App\Content;

use Softspring\CmsApiPlugin\Application\Query\PublishedContentQuery;

class PublishedPageSearch
{
    public function __construct(private readonly PublishedContentQuery $contents)
    {
    }

    public function search(string $site, string $locale, string $text): ?array
    {
        return $this->contents->searchPublished('page', $text, $site, $locale, 10);
    }
}
```

API controllers and MCP tools call these same services. MCP depends on this plugin, without calling its REST endpoints. Content and media queries share their repository lookup
and filtering through `ContentProvider` and `MediaProvider`; serializers produce the requested projection.
The HTTP adapter chooses status codes, while the MCP adapter adds tool schemas and response metadata.

Query services are read-only. Use the controlled operation service described below for writes; it reuses the CMS
forms, events and editorial permissions rather than writing entities directly.

### Apply your application's access policy {#apply-your-applications-access-policy}

These services do not authenticate callers or select an OAuth implementation. Protect HTTP routes through Symfony
Security and enforce the application's resource permissions before calling a query, including calls from local agents
and commands. A site filter selects data; it is not an authorization check.

For external clients, map authenticated identities and delegated scopes to the application's CMS permissions.
Do not put OAuth token validation in the query services, and do not assume a particular user entity or user bundle.

## Migrate custom API services {#migrate-custom-api-services}

The former `Softspring\CmsApiPlugin\Api\*Api` classes remain as deprecated subclasses for the 6.1 development
transition. HTTP paths and response contracts remain available. Update custom wiring and direct constructor calls
to use `Softspring\CmsApiPlugin\Application\Query` services and their dependencies. `RequestQuery` remains in
the HTTP layer because it reads HTTP query parameters.

## Execute controlled CMS operations {#execute-controlled-cms-operations}

The experimental `Application\Operation\OperationService` provides a catalog, form inspection and execution
for configured CMS actions. `Application\Query\ResourceQuery` provides permission-checked resource and version
reads. Both are shared by REST and MCP. Read the [operation guide](cms-mcp-plugin.md#manage-cms-elements) for
inputs, endpoints, authorization attributes, supported workflows, and current limits.

## Read version SEO statistics {#version-seo-statistics}

When the optional SEO Plugin is enabled, inject
`Softspring\CmsApiPlugin\Application\Query\SeoStatisticsQuery` and call
`analyze($contentType, $contentId, $versionId, $locale, $site, $limit)`.
It requires the host permission `CMS_API_READ` and checks the version/content relationship, including for drafts.

With the protected REST resource routes imported, the equivalent request is:

```http
GET /api/cms/resources/seo_statistics/CONTENT_ID?type=article&version=VERSION_ID&locale=en&site=main&limit=20
```

Use identifiers from your application. The version is required; locale and site are optional. The limit is 1–100.
Missing records or an unrelated version return HTTP 404; missing read permission returns 403, and invalid context
returns 400. The resource returns 503 when the SEO Plugin is not enabled. A module rendering failure returns an
analysis with `ok: false`, so callers must inspect the payload as well as HTTP status.

The result contains word counts, density, frequency and emphasis rankings, tag contexts and algorithm parameters.
It delegates to the [SEO Plugin's version analysis](../bundles/cms-seo-plugin.md#version-word-statistics), without
publishing, modifying metadata, or requiring an AI provider. The SEO Plugin remains optional for other API resources.
