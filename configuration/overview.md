---
title: "Armonic Configuration"
description: "Prepare your CMS by configuring sites, content types, layouts, modules, media, blocks, menus, and sections."
next: "configure-sites"
related:
    - "getting-started"
    - "user-manual"
    - "build-a-small-website"
---

# Armonic configuration {#armonic-configuration}

These guides are for developers preparing the CMS for editors. Configuration defines which sites exist, how pages are structured, and what editors can add to them.

## Start with the site and its pages {#start-with-the-site-and-its-pages}

1. [Configure sites](../configuration/configure-sites.md): define hosts, languages, and site behaviour.
2. [Configure content types](../configuration/configure-content-types.md): define the kinds of content your application manages.
3. [Configure layouts](../configuration/configure-layouts.md): provide the page structure and its editable areas.
4. [Configure modules](../configuration/configure-modules.md): choose the content elements available inside those areas.

## Add shared content and media {#add-shared-content-and-media}

- [Media](../configuration/configure-medias.md): configure the files and generated versions your content uses.
- [Blocks](../configuration/configure-blocks.md): define reusable pieces of content.
- [Menus](../configuration/configure-menu.md): define the navigation editors can manage.
- [Sections](../configuration/configure-sections.md): configure reusable content sections when using the sections plugin.

## See how the pieces fit {#see-how-the-pieces-fit}

[Build and publish a small website](../user-manual/build-a-small-website.md) walks through a layout, a menu, a reusable block, and two pages. For the CMS package reference and extension points, see the [CMS bundle guides](../bundles/cms-bundle/getting-started.md).

## Integrate the configured CMS {#integrate-the-configured-cms}

Read the [forms and metadata guide](../bundles/cms-bundle/forms-and-metadata.md) for editorial properties, [route integration](../bundles/cms-bundle/routing.md) for public URLs, [sitemaps and robots](../bundles/cms-bundle/sitemaps-and-robots.md) for discovery resources, and [admin permissions](../bundles/cms-bundle/security-and-permissions.md) for editorial access. Check [frontend assets](../bundles/cms-bundle/frontend-assets.md) before validating the editor in a browser.
