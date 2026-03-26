---
title: "Twig Extra Bundle"
description: "Add practical Twig helpers and an extensible app variable to Symfony applications."
---

# Twig Extra Bundle {#twig-extra-bundle}

`softspring/twig-extra-bundle` adds a small set of Twig helpers that solve common template problems in Symfony applications.

Its most important feature is not a filter or a function. It is the ability to extend the global Twig `app` variable so other bundles and your own application can expose extra runtime context such as `app.account`.

## Installation {#installation}

```bash
composer require softspring/twig-extra-bundle:^6.0
```

The bundle works with Symfony Twig applications and does not need extra setup for the default helpers.

## What This Bundle Adds {#what-this-bundle-adds}

The bundle provides these building blocks:

- an extensible Twig `app` variable
- `active_for_routes`
- `route_defined`
- `date_span`
- the `instanceof` Twig test
- optional `encore_entry_css_source` and `encore_entry_js_source`

Most helpers are enabled by default. The Encore source helpers are opt-in.

## Configuration Overview {#configuration-overview}

You can enable or disable each helper under `sfs_twig_extra`:

```yaml
# config/packages/sfs_twig_extra.yaml
sfs_twig_extra:
    active_for_routes_extension:
        enabled: true
    routing_extension:
        enabled: true
    date_span_extension:
        enabled: true
    instanceof_extension:
        enabled: true
    encore_entry_sources_extension:
        enabled: false
        public_path: '%kernel.project_dir%/public'
```

This is useful when you want to keep the template API small and only expose the helpers your project really uses.

## Extensible App Variable {#extensible-app-variable}

This bundle replaces Symfony's default `twig.app_variable` class with `Softspring\TwigExtraBundle\Twig\ExtensibleAppVariable`.

That means other runtime services can set extra values and make them available in Twig as part of `app`.

### Why This Matters {#why-this-matters}

This is the feature that allows patterns such as:

```twig
{{ app.account.name }}
```

That pattern is used by other Softspring bundles. For example, `account-bundle` resolves the current account from the request and injects it into the extensible app variable, which is why `app.account` becomes available in Twig.

### How To Use It In Your App {#how-to-use-it-in-your-app}

The usual approach is to set values from a request listener or another runtime service:

```php
<?php

namespace App\EventListener;

use Softspring\TwigExtraBundle\Twig\ExtensibleAppVariable;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpKernel\Event\RequestEvent;
use Symfony\Component\HttpKernel\KernelEvents;

class StoreRequestListener implements EventSubscriberInterface
{
    public function __construct(private readonly ExtensibleAppVariable $twigAppVariable)
    {
    }

    public static function getSubscribedEvents(): array
    {
        return [
            KernelEvents::REQUEST => ['onRequest', 30],
        ];
    }

    public function onRequest(RequestEvent $event): void
    {
        $this->twigAppVariable->setStore('uk');
    }
}
```

Then in Twig:

```twig
{{ app.store }}
```

This is the cleanest way to expose request-scoped or application-scoped context to templates.

## active_for_routes {#active-for-routes}

`active_for_routes` helps you build menus, tabs, and navigation items with an active class based on the current route.

Basic usage:

```twig
<a href="{{ url('admin_products_list') }}"
   class="{{ active_for_routes('admin_products_') }}">
    Products
</a>
```

If the current route starts with `admin_products_`, the function returns `active`.

You can also:

- add an extra boolean condition
- change the returned CSS class

```twig
{{ active_for_routes('admin_products_', canSeeMenu) }}
{{ active_for_routes('admin_products_', null, 'is-current') }}
```

This is much easier to read than repeating route prefix checks inline in every menu template.

## route_defined {#route-defined}

`route_defined` helps templates check whether a route exists before trying to generate a link.

```twig
{% if route_defined('sfs_user_register') %}
    <a href="{{ url('sfs_user_register') }}">Sign up</a>
{% endif %}
```

This is especially useful in shared templates or reusable bundles where some routes may or may not be available depending on the installed features.

The helper considers a route as defined even when URL generation would still need route parameters. That makes it suitable for capability checks, not for validating complete URL input.

## date_span {#date-span}

`date_span` formats a date for the visitor timezone while preserving UTC information in a tooltip.

```twig
{{ order.createdAt|date_span('H:i:s d-m-Y') }}
```

The filter reads the `utz` cookie to know the user timezone.

When the timezone is different from UTC, it renders a small HTML span with:

- the user-local time as visible text
- the user-local time and UTC time in the `title`

This is useful in admin interfaces or operational screens where local time is nice for the user but UTC is still important for support and debugging.

### Fallback Behavior {#date-span-fallback-behavior}

If there is no request, no timezone cookie, or the cookie contains an invalid timezone, the filter falls back to UTC instead of breaking the template.

## instanceof Twig Test {#instanceof-twig-test}

The bundle adds a Twig test named `instanceof`:

```twig
{% if block is instanceof('App\\\\Cms\\\\Block\\\\HeroBlock') %}
    ...
{% endif %}
```

This is a small helper, but it is practical when templates work with polymorphic objects and you need a direct class check.

## Encore Entry Source Helpers {#encore-entry-source-helpers}

The Encore helpers are disabled by default.

When enabled, they expose:

- `encore_entry_css_source`
- `encore_entry_js_source`

They read the built file contents for a given Encore entry:

```twig
<style>{{ encore_entry_css_source('email') }}</style>
<script>{{ encore_entry_js_source('email') }}</script>
```

This is useful for cases such as:

- inline email rendering
- HTML exports
- pages where you need asset source instead of asset URLs

### Requirements {#encore-entry-source-requirements}

This feature requires `symfony/webpack-encore-bundle`.

You must also configure the correct `public_path` when your built assets are not stored in Symfony's default public directory.

## Disabling Helpers {#disabling-helpers}

Each helper can be disabled independently.

That matters when:

- you want to keep the template API minimal
- you do not want project-specific teams to depend on a helper
- the project does not use one of the optional integrations

## Extending The Bundle {#extending-the-bundle}

### Expose New app.* Values {#expose-new-app-values}

The main extension point is the extensible app variable.

Use it to expose data such as:

- current account
- current store
- current tenant
- current site

This is one of the cleanest ways to share runtime context with Twig without filling templates with service calls.

### Build On Top Of The Existing Helpers {#build-on-top-of-the-existing-helpers}

You can keep the helpers enabled and still add your own Twig extensions beside them.

This bundle does not try to own the whole Twig layer. It gives you a small common base.

## Current Limits {#current-limits}

- `date_span` depends on the `utz` cookie convention for user timezone support
- `route_defined` checks route existence through generation, not through router metadata inspection
- `encore_entry_*_source` reads files from disk, so it depends on built assets being available locally
- `encore_entry_*_source` is opt-in and requires `symfony/webpack-encore-bundle`

## Recommended Use {#recommended-use}

Use this bundle when you want a small set of template helpers that remove repeated Twig boilerplate and let your application expose extra `app.*` context cleanly.

Its biggest value is usually the extensible app variable. The other helpers are small, but together they remove a lot of repetitive template code.
