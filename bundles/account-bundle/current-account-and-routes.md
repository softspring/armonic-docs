---
title: "Current Account And Routes"
description: "Design account-aware routes and understand how Account Bundle resolves the current account at runtime."
---

# Current Account And Routes {#current-account-and-routes}

One of the most useful parts of `account-bundle` is not the ready-made screens. It is the current-account flow around the `/{_account}` route parameter.

Once the account is part of the route, the bundle can resolve it before your controller runs and reuse that context in Twig, access checks, and Doctrine filtering.

## Keep `_account` In The URL {#keep-account-in-the-url}

This is the safest route pattern:

<!-- config-tabs:start -->

```yaml
project_list:
    path: /account/{_account}/projects
    controller: App\Controller\ProjectController::list
```

```php
<?php

// config/routes.php
namespace Symfony\Component\Routing\Loader\Configurator;

return static function (RoutingConfigurator $routes): void {
    $routes->add('project_list', '/account/{_account}/projects')
        ->controller('App\\Controller\\ProjectController::list');
};
```

<!-- config-tabs:end -->

`route_param_name` controls which request attribute the shared context resolver reads. The default `_account` keeps the examples and bundled routes consistent. The resolver uses `find_field_name` and reuses an entity already resolved into the attribute.

## Building Real Account Areas {#building-real-account-areas}

The pattern becomes more useful when you apply it to whole areas:

<!-- config-tabs:start -->

```yaml
_provider:
    resource: 'routes/provider/*'
    prefix: '/{_locale}/provider/{_account}'

_corporate:
    resource: 'routes/corporate/*'
    prefix: '/{_locale}/corporate/{_account}'
```

```php
<?php

// config/routes.php
namespace Symfony\Component\Routing\Loader\Configurator;

return static function (RoutingConfigurator $routes): void {
    $routes->import('routes/provider/*')->prefix('/{_locale}/provider/{_account}');

    $routes->import('routes/corporate/*')->prefix('/{_locale}/corporate/{_account}');
};
```

<!-- config-tabs:end -->

This lets your product areas stay account-aware without repeating account-loading code in every controller.

## Request Listener Resolution {#request-listener-resolution}

`AccountRequestListener` runs after routing. When the request contains the configured route attribute:

1. it reads the raw route value
2. it loads the account repository for `AccountInterface`
3. it searches with `find_field_name`
4. it writes the resolved entity back into the request attribute
5. it stores the entity in router context
6. it injects the entity into Twig as `app.account` by default

That means a route parameter such as `/{_account}` turns into a real account object before your controller logic starts.

## Using The Current Account In Controllers {#using-the-current-account-in-controllers}

The simplest pattern is to read the resolved account from the request:

```php
/** @var AccountInterface $account */
$account = $request->attributes->get('_account');
```

On Symfony versions with `ValueResolverInterface`, the bundle also registers `AccountValueResolver`, so controller arguments typed as `AccountInterface` can be resolved directly:

```php
use Softspring\AccountBundle\Model\AccountInterface;
use Symfony\Component\HttpFoundation\Response;

public function settings(AccountInterface $account): Response
{
    // ...
}
```

The request listener and value resolver share the same context resolver, including `find_field_name`. Protect the route with a stateful firewall so account access is checked against the authenticated user.

## Using The Current Account In Twig {#using-the-current-account-in-twig}

With `softspring/twig-extra-bundle` installed, templates can use:

```twig
{{ app.account.name }}
```

This is useful for account navigation, headers, breadcrumbs, and account-scoped UI.

## Practical Advice {#practical-advice}

- keep `_account` as the current-account key
- use account-aware prefixes early, not only on one page
- validate the design by building one real product area such as projects, provider dashboard, or account settings
- if that first area feels awkward, review the entity model before adding more screens

## Related Guides {#related-guides}

- [Account Bundle Overview](../account-bundle.md)
- [Install](./install.md)
- [Model And Entities](./model-and-entities.md)
- [Filter And Scoped Data](./filter-and-scoped-data.md)
- [Admin And Security](./admin-and-security.md)
