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

## Visibility and analysis {#visibility-and-analysis}

[SEO](../cms-seo-plugin.md) checks the HTML and discovery resources served by published URLs. [Analytics](../cms-analytics-plugin.md) reads traffic metrics from Plausible or Google Analytics 4. These solve different problems: valid metadata does not imply traffic, and traffic metrics do not validate HTML.

## API and assistant integrations {#api-and-assistant-integrations}

- [CMS API](../../components/cms-api-plugin.md) exposes read-only JSON endpoints.
- [CMS MCP](../../components/cms-mcp-plugin.md) exposes read-only tools to an MCP server.
- [CMS AI](../cms-ai-plugin.md) adds an admin assistant and draft editing workflows.
- [CMS Mate](../../components/cms-mate-extension.md) lets a local AI Mate process inspect the host application's CMS.

The API, analytics, SEO, MCP, AI, and Mate integrations are evolving. Read their current limits before exposing them to users or clients. The host application remains responsible for authentication, provider credentials, and deployment configuration.

## Integrate into an existing admin {#integrate-into-an-existing-admin}

Use the normal CMS admin imports so plugin routing providers can contribute their actions. Import each resource once. A custom menu should link to the generated route and check the corresponding permission.

For Sylius, read the [installation and compatibility guide](../../getting-started/install-sylius-project.md). The current Sylius 2 admin-template incompatibility applies even when package installation and migrations succeed.
