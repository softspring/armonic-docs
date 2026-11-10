---
title: "CMS plugins and integrations"
description: "Choose optional CMS features and find their setup, capabilities, and current integration limits."
related: ["bundles/cms-bundle", "bundles", "components"]
---

# Plugins and integrations {#plugins-and-integrations}

Start with CMS core and add the plugins needed by the application. A plugin may add a content type, admin action, provider, or tool integration; it does not necessarily add a public route automatically.

## Editorial features {#editorial-features}

| Need | Guide |
| --- | --- |
| Articles, authors, tags, and related content | [Blog](../cms-blog-plugin.md) |
| Reusable module-based content with its own versions | [Sections](sections.md) |
| Structured content import and export | [Data plugin](../cms-data-plugin.md) |
| Version translation and XLIFF exchange | [Translation plugin](../cms-translation-plugin.md) |
| Ready-made editing modules | [Module collection](../cms-module-collection.md) |

## Integrate into an existing admin {#integrate-into-an-existing-admin}

Use the normal CMS admin imports so plugin routing providers can contribute their actions. Import each resource once. A custom menu should link to the generated route and check the corresponding permission.

For Sylius, read the [installation and compatibility guide](../../getting-started/install-sylius-project.md). The current Sylius 2 admin-template incompatibility applies even when package installation and migrations succeed.

## Experimental integrations in 6.1-dev {#experimental-integrations}

These integrations are optional and their contracts may change before a stable release:

- [AI-assisted editing](../cms-ai-plugin.md)
- [Page analytics](../cms-analytics-plugin.md)
- [SEO analysis](../cms-seo-plugin.md)
- [Read-only HTTP API](../../components/cms-api-plugin.md)
- [MCP tools](../../components/cms-mcp-plugin.md)
- [Local Mate tools](../../components/cms-mate-extension.md)
- [Form schema extraction](../../components/form-schema.md)
