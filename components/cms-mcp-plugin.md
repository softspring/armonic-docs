---
title: "CMS MCP Plugin (Experimental)"
description: "Expose read-only CMS tools to MCP consumers using published content and shared serializers."
related: ["components/cms-api-plugin", "bundles/cms-ai-plugin"]
---

# CMS MCP Plugin {#cms-mcp-plugin}

> **Experimental — 6.1@dev:** configuration, APIs and output may change before a stable release. Use this package in development and validate the workflows enabled in your application.

The MCP plugin registers read-only tools for site context, configuration, published content, internal links, menus, images, and optional traffic metrics. Use it when an agent should query CMS context through a structured tool interface.

## Install and register {#install-and-register}

```bash
composer require softspring/cms-mcp-plugin:6.1.*@dev
```

Register the bundle when Flex does not do it automatically:

```php
// config/bundles.php
Softspring\CmsMcpPlugin\SfsCmsMcpPlugin::class => ['all' => true],
```

The host application also needs Symfony MCP Bundle enabled and its server/transport configured. Registering these tool services does not create a universally named public endpoint. Choose transport, discovery, and access policies in that server configuration.

## Select tools by task {#select-tools-by-task}

| Task | Tool |
| --- | --- |
| Find sites | `sfs_cms_sites_list` |
| Inspect one site's context | `sfs_cms_sites_get_context` |
| Inspect content/layout/module definitions | `sfs_cms_configuration_get_context` |
| Find published pages | `sfs_cms_contents_search_published` |
| Read a published item | `sfs_cms_contents_get_published` |
| Find internal link destinations | `sfs_cms_routes_find_internal_links` |
| Read navigation | `sfs_cms_menus_get_context` |
| Find configured image types and media | `sfs_cms_media_images_list_types`, `sfs_cms_media_images_search`, `sfs_cms_media_images_get_context` |
| Read traffic metrics | `sfs_cms_analytics_get_site_metrics`, `sfs_cms_analytics_query_pages` |

## Search before reading a payload {#search-before-reading-a-payload}

For `sfs_cms_contents_search_published`, use an argument object such as:

```json
{"query":"accessibility","contentType":"page","site":"default","locale":"en","limit":10}
```

The tool returns summaries and caps the limit at 50. Use the returned content identifier with `sfs_cms_contents_get_published`; set `includeData: true` only when the payload is needed. Internal-link discovery can use the same query, site, locale, and limit context.

## Understand the boundary {#understand-the-boundary}

Published content tools select the published version rather than unsaved editor data. Configuration tools can expose definitions and, optionally, raw configuration with shared redaction. These are different kinds of context; read-only does not mean every payload is appropriate for every consumer.

Analytics tools consume the [Analytics Plugin](../bundles/cms-analytics-plugin.md) provider chain when available. The MCP package does not configure Plausible or GA4 credentials itself.

The tools do not save, import, export, or publish content. For an assistant that proposes changes in an open form, use the [CMS AI Plugin](../bundles/cms-ai-plugin.md) and retain its manual save/review workflow.
