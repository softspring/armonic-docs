---
title: "CMS Bundle"
description: "Choose the CMS building blocks, editorial workflows, and integrations needed by a Symfony application."
related: ["bundles/cms-bundle/getting-started", "configuration", "user-manual"]
---

# CMS Bundle {#cms-bundle}

CMS Bundle provides versioned editorial content inside a Symfony application. Developers define the content structure and rendering; editors create records, compose modules, preview changes, and publish a selected version.

## Choose the right building block {#choose-the-right-building-block}

| Need | Building block |
| --- | --- |
| A public page with its own URL | Content type, layout, content record, and route |
| A repeated element inside a page | Module |
| Shared editable contact details or call to action | Block |
| Shared content composed from modules with its own publication history | Section |
| Navigation maintained by editors | Menu |
| Another host, language, or path prefix | Site |

For example, a website can use a `page` content type with a `landing` layout, a hero module inside its `main` container, a shared contact block, and a main menu. The site determines its languages; route records connect published pages to URLs.

## Start a project {#start-a-project}

Follow [installation](cms-bundle/install.md) and [getting started](cms-bundle/getting-started.md), then configure [sites](../configuration/configure-sites.md), [content types](../configuration/configure-content-types.md), [layouts](../configuration/configure-layouts.md), and [modules](../configuration/configure-modules.md).

[Build a small website](../user-manual/build-a-small-website.md) follows the complete editorial journey. [Content versions](cms-bundle/content-versions.md) explains drafts, preview, publication, and compiled output.

## Integrate application features {#integrate-application-features}

Use [forms and metadata](cms-bundle/forms-and-metadata.md) for editorial properties and custom fields, [routes](cms-bundle/routing.md) for Symfony/CMS URL integration, and [admin permissions](cms-bundle/security-and-permissions.md) to assign editor access. Configure [assets](cms-bundle/frontend-assets.md) before validating the editor in a browser.

The [plugin catalog](cms-bundle/plugins.md) covers blog articles, import/export, translations, SEO, analytics, APIs, and AI tooling. Plugins add their own requirements; installing CMS core does not automatically configure every integration.
