---
title: "Current Account And Routes"
description: "Design account-aware routes and understand how Account Bundle resolves the current account at runtime."
---

# Current Account And Routes {#current-account-and-routes}

One of the most useful parts of `account-bundle` is not the ready-made screens. It is the current-account flow around the `/{_account}` route parameter.

Once the account is part of the route, the bundle can resolve it before your controller runs and reuse that context in Twig, access checks, and Doctrine filtering.

## Keep `_account` In The URL {#keep-account-in-the-url}

This is the safest route pattern:

```yaml
project_list:
    path: /account/{_account}/projects
    controller: App\Controller\ProjectController::list
```

Several internals still read `_account` directly:

- `AccountValueResolver`
- `AccountFilteredEventListener`
- `AccountFilter`

That is why `_account` should be your default route parameter name.

## Building Real Account Areas {#building-real-account-areas}

The pattern becomes more useful when you apply it to whole areas:

```yaml
_provider:
    resource: 'routes/provider/*'
    prefix: '/{_locale}/provider/{_account}'

_corporate:
    resource: 'routes/corporate/*'
    prefix: '/{_locale}/corporate/{_account}'
```

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

The request listener is still the more flexible runtime mechanism, especially when you rely on `find_field_name`.

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
