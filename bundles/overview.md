---
title: "Armonic and Softspring Bundles"
description: "Choose application features for content management, media, users, accounts, and email."
related:
    - "getting-started"
    - "configuration"
    - "components"
    - "user-manual"
---

# Bundles {#bundles}

These packages provide features you can integrate into a Symfony application. Follow each guide for installation, configuration, and ways to extend it.

## Content and media {#content-and-media}

- **[CMS bundle](../bundles/cms-bundle.md):** configure and manage pages, sites, layouts, modules, and reusable content. Start with [installation](../bundles/cms-bundle/install.md) if you are adding the CMS to a project.
- **[Media bundle](../bundles/media-bundle.md):** manage uploaded files, generated versions, rendering, and the media library.

## Users and accounts {#users-and-accounts}

- **[User bundle](../bundles/user-bundle.md):** add login, registration, account settings, and user administration.
- **[Account bundle](../bundles/account-bundle.md):** make application data belong to accounts, with users accessing it through memberships.

## Email {#email}

- **[Mailer bundle](../bundles/mailer-bundle.md):** manage mail templates, preview messages, and integrate email history.

## Looking for technical building blocks? {#technical-building-blocks}

The [Components](../components/overview.md) section covers forms, Doctrine helpers, HTTP integrations, translations, permissions, and other infrastructure packages.

## Extend the CMS {#extend-the-cms}

The [CMS plugin catalog](cms-bundle/plugins.md) covers blog articles, sections, translation workflows, data import/export, and ready-made modules. Choose the plugin by the workflow it adds and check its current compatibility requirements.

## Experimental development packages {#experimental-development-packages}

These optional packages belong to `6.1@dev`. Their public contracts may change before a stable release.

- [cms-ai-plugin (Experimental)](cms-ai-plugin.md)
- [cms-analytics-plugin (Experimental)](cms-analytics-plugin.md)
- [cms-seo-plugin (Experimental)](cms-seo-plugin.md)
