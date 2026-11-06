---
title: "CMS Bundle Getting Started"
description: "Connect the CMS admin and frontend routes, prepare content definitions, and publish a first page."
---

# Getting started {#getting-started}

After [installing CMS Bundle](install.md), connect its routes to your application and define the content editors will manage. CMS Bundle provides the editing and publishing infrastructure; the application supplies its design, authentication, and content definitions.

## Register admin routes {#admin-routes}

The combined admin file includes pages, routes, menus, blocks, sites, and the Media Bundle library:

<!-- config-tabs:start -->

```yaml
# config/routes/sfs_cms_admin.yaml
_sfs_cms_admin:
    resource: '@SfsCmsBundle/config/routing/admin.yaml'
    prefix: /admin/cms
```

```php
<?php

// config/routes/sfs_cms_admin.php
namespace Symfony\Component\Routing\Loader\Configurator;

return static function (RoutingConfigurator $routes): void {
    $routes->import('@SfsCmsBundle/config/routing/admin.yaml')->prefix('/admin/cms');
};
```

<!-- config-tabs:end -->

Import it once. If Armonic's recipe already imports this file, keep that import rather than adding another. A custom content type can import `admin_content_type.yaml` separately with its own `_content_type` default and route name prefix.

Protect the admin prefix through your application's firewall and access-control rules. Import the CMS role hierarchy and grant your administrator the corresponding roles as described in the installation guide. Route registration does not create a login form or administrator account.

## Prepare the content model {#prepare-content-model}

Start with one site, one page content type, one layout, and one module:

1. [Configure a local site](../../configuration/configure-sites.md) with the hosts or locale paths you will browse.
2. [Configure the page content type](../../configuration/configure-content-types.md) and its allowed layouts.
3. [Create a layout](../../configuration/configure-layouts.md) with an editable container.
4. [Create a module](../../configuration/configure-modules.md) and allow it in that container.

Keep these files under the application's `cms/` collection. Clear the Symfony cache after changing definitions. The core bundle does not provide a universal welcome-fixtures command; any demo fixtures belong to the specific collection or application that supplies them.

## Register frontend routes {#frontend-routes}

<!-- config-tabs:start -->

```yaml
# config/routes/sfs_cms_frontend.yaml
_sfs_cms_frontend:
    resource: '@SfsCmsBundle/config/routing/frontend.yaml'
    prefix: /{_locale}
```

```php
<?php

// config/routes/sfs_cms_frontend.php
namespace Symfony\Component\Routing\Loader\Configurator;

return static function (RoutingConfigurator $routes): void {
    $routes->import('@SfsCmsBundle/config/routing/frontend.yaml')->prefix('/{_locale}');
};
```

<!-- config-tabs:end -->

Place the CMS frontend import after more specific application routes when they share a prefix. Its content routes should not take over your login, API, or application controllers. Configure the site's locale paths to match the import.

## Publish a first page {#publish-first-page}

Open the pages list, create a page, choose the configured layout, and add a module to its container. Save a draft and preview it. Create a CMS route for the page in the selected site and language, then publish the version you want visitors to see.

Check the public URL in a separate browser session. A successful admin preview does not prove that the route exists or that a version has been published. Follow [Build a small website](../../user-manual/build-a-small-website.md) for the full editorial workflow.
