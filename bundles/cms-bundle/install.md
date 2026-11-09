---
title: "Install the CMS Bundle"
description: "Install the Softspring CMS Bundle in a Symfony application using Composer and Symfony Flex conventions."
---

# Install CMS Bundle {#installation}

CMS Bundle provides Armonic's content model, editor, routing, and rendering services. Install the `softspring/armonic` package when you want the usual CMS modules and plugins together; install this bundle directly when integrating only the core runtime into an existing application.

Check the [Armonic requirements](../../getting-started/requirements.md) and configure [MySQL or PostgreSQL](../../getting-started/configure-database.md) first.

## Install with Composer {#applications-that-use-symfony-flex}

For the development packages used by this `6.0` documentation:

```bash
composer config --json extra.symfony.endpoint '["https://api.github.com/repos/softspring/recipes/contents/index.json", "flex://defaults"]'
composer config minimum-stability dev
composer config prefer-stable true
composer require softspring/cms-bundle:6.1.*@dev -W
```

Review the Flex recipe changes to routes, services, permissions, and CMS configuration. Merge them with the application's existing setup rather than replacing working security configuration.

For a complete installation with a login screen, follow [Install Armonic in a new Symfony project](../../getting-started/install-new-symfony-project.md). For a shop, use the [Sylius integration guide](../../getting-started/install-sylius-project.md).

## Register bundles without Flex {#applications-that-dont-use-symfony-flex}

Register CMS Bundle and the Symfony bundles required by its installed dependencies in `config/bundles.php`. The CMS entry is:

```php
Softspring\CmsBundle\SfsCmsBundle::class => ['all' => true],
```

This entry alone does not configure Doctrine mappings, authentication, routes, or frontend assets. Use the package's `6.0` recipe as a checklist for manual integration, and register optional plugins before executing their migrations.

## Frontend assets {#frontend-assets}

Choose the asset system used by the application:

- **AssetMapper:** the bundle registers its published `assets/dist/` files when AssetMapper is available. Your admin layout must load the appropriate script and stylesheet entrypoints.
- **Webpack Encore:** install the reusable frontend packages from their package roots under `vendor/softspring/`, then import their source entrypoints. Do not use the old `file:.../assets` package paths.

The [standalone installation](../../standalone/install-armonic-standalone.md) provides an Encore-based application and an admin layout using the shared shell. Keep the package assets in sync with the installed PHP package line.

## Continue with application setup {#continue-setup}

Enable the admin routes, configure permissions for the application's existing administrator role, and run migrations against the chosen database. See [Get started with CMS Bundle](getting-started.md).
