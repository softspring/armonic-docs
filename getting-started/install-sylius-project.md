---
title: "Install Armonic in a Sylius Project"
description: "Step-by-step guide to install Sylius and integrate Armonic CMS bundles."
next: "configuration"
related:
    - "bundles/cms-bundle/sections"
    - "bundles/cms-bundle/getting-started"
    - "build-a-small-website"
    - "configure-database"
---

# Install Armonic in Sylius {#install-sylius-project}

Armonic provides an integration package for editorial pages alongside a Sylius shop. This guide records the installation requirements and the current compatibility limit of the **6.0** integration.

> [!IMPORTANT]
> Installation and database migrations have been verified on Sylius 2.3, but the CMS administration is not currently usable there: its templates still depend on the older Sylius admin layout and `sylius_template_event` function. Do not treat Composer compatibility with `sylius/ui-bundle:^2.0` as proof of a working Sylius 2 admin. Complete that template migration before adopting this combination.

Start with a working Sylius installation and administrator account. If you need a new shop, install a compatible `sylius/sylius-standard` project and follow [Sylius' installation instructions](https://docs.sylius.com/getting-started-with-sylius). Complete its database installation before adding the CMS.

## 1. Check the database and frontend tools {#configure-database}

Keep the database engine used by the shop. For example, choose one connection in `.env.local`:

**MySQL:**

```dotenv
DATABASE_URL="mysql://app:app_password@127.0.0.1:33061/sylius?serverVersion=8.4&charset=utf8mb4"
```

**PostgreSQL:**

```dotenv
DATABASE_URL="postgresql://app:app_password@127.0.0.1:54321/sylius?serverVersion=17&charset=utf8"
```

Adjust the database name, credentials, and server version. The local services and PHP drivers are described in [Configure MySQL or PostgreSQL](configure-database.md). Keep the Doctrine mappings and identity generation strategy supplied by your Sylius version.

Use a Node.js version supported by the installed Sylius template. The Sylius 2.3 template used here requires `^22.13.0` or `>=24.0`. This guide uses npm; keep the project's existing package manager and lockfile if it uses Yarn instead.

## 2. Install the CMS packages {#install-armonic}

Configure the Softspring Flex endpoint, then install the matching development line:

```bash
composer config --json extra.symfony.endpoint '["https://api.github.com/repos/softspring/recipes/contents/index.json", "flex://defaults"]'
composer config minimum-stability dev
composer config prefer-stable true
composer require softspring/cms-bundle:6.1.*@dev softspring/cms-sylius-bundle:6.1.*@dev softspring/collection-form-type:6.1.*@dev -W
```

Accept and review the Flex recipes. Check that `config/bundles.php` registers the integration and collection assets bundle; add these entries if missing:

```php
Softspring\CmsSyliusBundle\SfsCmsSyliusBundle::class => ['all' => true],
Softspring\Component\CollectionFormType\SfsCollectionFormTypeBundle::class => ['all' => true],
```

The integration adds CMS menu entries automatically. Do not duplicate them in a separate hand-built sidebar configuration.

## 3. Keep Sylius migration ordering {#migration-ordering}

Sylius uses dependency-aware migration ordering. The generic Softspring comparator recipe must not replace that ordering.

If Flex creates `config/packages/sfs_doctrine_migrations.yaml` with a `Doctrine\Migrations\Version\Comparator` override, remove that override and its unused service definition from the application. Keep the existing Sylius migration configuration.

Add the CMS dependency on media to the existing Sylius migration graph:

<!-- config-tabs:start -->

```yaml
# config/packages/sylius_labs_doctrine_migrations_extra.yaml
sylius_labs_doctrine_migrations_extra:
    migrations:
        'Softspring\MediaBundle\Migrations': ~
        'Softspring\CmsBundle\Migrations': ['Softspring\MediaBundle\Migrations']
```

```php
<?php

// config/packages/sylius_labs_doctrine_migrations_extra.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sylius_labs_doctrine_migrations_extra', [
        'migrations' => [
            'Softspring\\MediaBundle\\Migrations' => null,
            'Softspring\\CmsBundle\\Migrations' => ['Softspring\\MediaBundle\\Migrations'],
        ],
    ]);
};
```

<!-- config-tabs:end -->

The media schema must exist before CMS migrations add media relationships. This is separate from migration version numbers inside each namespace. Keep the application's existing `App\Migrations` and plugin dependencies in this graph.

## 4. Configure routes and permissions {#admin-routes}

The CMS Flex recipe creates `config/routes/admin/sfs_cms.yaml` and imports it under `/admin/cms`. Keep one import for each route set. If you use the combined CMS admin routing file, do not also import the same media admin routes through another file.

For a manual route setup, the combined bundle resource is:

<!-- config-tabs:start -->

```yaml
# config/routes/sfs_cms.yaml
sfs_cms_admin:
    resource: '@SfsCmsBundle/config/routing/admin.yaml'
    prefix: /admin/cms
```

```php
<?php

// config/routes/sfs_cms.php
namespace Symfony\Component\Routing\Loader\Configurator;

return static function (RoutingConfigurator $routes): void {
    $routes->import('@SfsCmsBundle/config/routing/admin.yaml')->prefix('/admin/cms');
};
```

<!-- config-tabs:end -->

The standard Sylius administrator role is `ROLE_ADMINISTRATION_ACCESS`. Add the CMS permissions to the role your administrators actually have, merging with its existing hierarchy:

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_cms.yaml
imports:
    - { resource: '@SfsCmsBundle/config/security/admin_role_hierarchy.yaml' }
    - { resource: '@SfsMediaBundle/config/security/admin_role_hierarchy.yaml' }
security:
    role_hierarchy:
        ROLE_ADMINISTRATION_ACCESS:
            - ROLE_SFS_MEDIA_ADMIN_MEDIAS_RW
            - ROLE_SFS_CMS_ADMIN_BLOCKS_RW
            - ROLE_SFS_CMS_ADMIN_CONTENTS_RW
            - ROLE_SFS_CMS_ADMIN_ROUTES_RW
            - ROLE_SFS_CMS_ADMIN_MENUS_RW
            - ROLE_SFS_CMS_ADMIN_SITES_RO
sfs_cms:
    site:
        throw_not_found: false
```

```php
<?php

// config/packages/sfs_cms.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->import('@SfsCmsBundle/config/security/admin_role_hierarchy.yaml');

    $container->import('@SfsMediaBundle/config/security/admin_role_hierarchy.yaml');

    $container->extension('security', [
        'role_hierarchy' => [
            'ROLE_ADMINISTRATION_ACCESS' => [
                'ROLE_SFS_MEDIA_ADMIN_MEDIAS_RW',
                'ROLE_SFS_CMS_ADMIN_BLOCKS_RW',
                'ROLE_SFS_CMS_ADMIN_CONTENTS_RW',
                'ROLE_SFS_CMS_ADMIN_ROUTES_RW',
                'ROLE_SFS_CMS_ADMIN_MENUS_RW',
                'ROLE_SFS_CMS_ADMIN_SITES_RO',
            ],
        ],
    ]);

    $container->extension('sfs_cms', ['site' => ['throw_not_found' => false]]);
};
```

<!-- config-tabs:end -->

This example gives shop administrators CMS editing privileges. Use narrower roles when editorial staff should have fewer actions. Keep `throw_not_found: false` so requests outside CMS sites can continue through the shop.

## 5. Run migrations and prepare a site {#sylius-install}

After reviewing the configuration:

```bash
php bin/console cache:clear
php bin/console doctrine:migrations:sync-metadata-storage -n
php bin/console doctrine:migrations:migrate -n
```

Create or adapt `cms/sites/default/config.yaml` for a locale enabled in the shop. For a local HTTP example:

```yaml
site:
    allowed_content_types: ['page']
    locales: ['en_US']
    default_locale: en_US
    https_redirect: false
    paths:
        - { path: '/en_US/', locale: en_US, trailing_slash_on_root: true }
```

Keep editorial page paths distinct from existing shop routes. Configure the deployed host and HTTPS behavior before using the site outside local development.

## 6. Build and open the admin {#install-assets}

```bash
npm install
npm run build
php bin/console cache:clear
symfony server:start -d
```

Open the URL printed by Symfony CLI and sign in through **`/admin/login`** using your existing Sylius administrator. The example imports register CMS pages under **`/admin/cms/pages/`**.

On the tested Sylius 2.3 installation, opening that URL fails with `Unknown "sylius_template_event" function` in the CMS admin layout. Database migration success does not remove this template incompatibility. Keep the integration out of a working shop until its admin templates and assets have been adapted and the editorial workflow has been exercised.


Create a page, open its content editor, add a module, and preview the result. Then [configure layouts and modules](../configuration/overview.md) for the editorial pages your shop needs.

## Integration troubleshooting {#troubleshooting}

### Twig reports a duplicate StringLoaderExtension {#duplicate-string-loader}

If both integrations register the string loader, merge this definition into the existing `services` configuration:

<!-- config-tabs:start -->

```yaml
# config/services.yaml
services:
    sfs_cms.twig.extension.loader:
        class: Twig\Extension\StringLoaderExtension
        autoconfigure: false
```

```php
<?php

// config/services.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $services = $container->services();

    $services->set('sfs_cms.twig.extension.loader', \Twig\Extension\StringLoaderExtension::class)
        ->autoconfigure(false);
};
```

<!-- config-tabs:end -->

### Frontend build cannot find LoaderDependency {#webpack-loader-dependency}

With Encore 4 and the affected Stimulus loader, the compatible Webpack version used in the test project is `5.105.4`:

```bash
npm install --save-dev --save-exact webpack@5.105.4
npm run build
```

Recheck this pin when updating Encore or the Sylius frontend template. Do not patch templates or JavaScript directly inside `vendor/`; install an integration version compatible with the Sylius admin UI you use.
