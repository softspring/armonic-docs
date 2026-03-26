---
title: "Components"
description: "Use shared Twig layouts, paginators, flash messages, sidebars, and admin form themes from the Softspring Components package."
---

# Components {#components}

`softspring/components` is a shared Twig toolbox for Symfony applications.

Use it when you want a common base layout, a ready-to-use admin shell, reusable paginators, consistent flash messages, and a small set of UI templates that several bundles can share.

This package is not a UI framework on its own. It gives you practical Twig building blocks that you can adopt, override, and combine with your own templates.

## Installation {#installation}

Install the package:

```bash
composer require softspring/components:^6.0
```

If you use Symfony Flex, the recipe gives you a good starting point:

- it enables the bundle
- it creates `config/packages/sfs_components.yaml`
- it defines the Twig global `sfs_components_theme: 'bootstrap5'`
- it creates a local `templates/base.html.twig` that extends `@SfsComponents/base.html.twig`

## Recommended Starting Point {#recommended-starting-point}

For most new projects, start with this setup:

1. Keep `bootstrap5` as the active `sfs_components_theme`.
2. Extend `@SfsComponents/base.html.twig` from your application `base.html.twig`.
3. Extend `@SfsComponents/layout/admin.html.twig` for back office screens.
4. Use the paginator and flash templates in list and CRUD screens.
5. Override only the templates or blocks you really need to customize.

This is the pattern used in `armonic-standalone` and in several Softspring bundles.

## Themes {#themes}

The package resolves most entry templates through `sfs_components_theme`.

With the default setup:

```yaml
twig:
    globals:
        sfs_components_theme: 'bootstrap5'
```

These entry templates resolve automatically:

- `@SfsComponents/base.html.twig`
- `@SfsComponents/layout/admin.html.twig`
- `@SfsComponents/flash-messages/alerts.html.twig`
- `@SfsComponents/paginator/pager.html.twig`
- `@SfsComponents/paginator/table.html.twig`
- `@SfsComponents/sidebar/sidebar-list.html.twig`
- `@SfsComponents/sidebar/sidebar-pills.html.twig`
- `@SfsComponents/forms/admin-horizontal.html.twig`

`bootstrap5` is the main maintained theme.

There are also legacy Semantic UI paginator templates still used by older integrations such as `cms-sylius-bundle`, but the package does not provide a full Semantic UI layout system.

## Base Layout {#base-layout}

The package base layout is the simplest entry point:

```twig
{% extends '@SfsComponents/base.html.twig' %}
```

It gives you:

- HTML shell
- viewport and common meta blocks
- title block
- default favicon block
- Bootstrap 5 CDN CSS and JS blocks
- simple body structure

In practice, applications usually keep the template and override only a few blocks:

```twig
{% extends '@!SfsComponents/base.bootstrap5.html.twig' %}

{% block stylesheets %}
    {{ encore_entry_link_tags('app') }}
{% endblock %}

{% block favicon %}
    <link rel="icon" href="...">
{% endblock %}
```

That is the same pattern used in `armonic-standalone`.

## Admin Layout {#admin-layout}

For admin areas, extend:

```twig
{% extends '@SfsComponents/layout/admin.html.twig' %}
```

The admin layout gives you:

- fixed header area
- logo block
- user dropdown area
- sidebar area
- breadcrumb block
- content block
- flash messages include at the end of the page

This makes it a good default shell for bundle admin screens and back office pages.

### Typical Usage {#admin-layout-typical-usage}

```twig
{% extends '@SfsComponents/layout/admin.html.twig' %}

{% block title %}Products{% endblock %}

{% block breadcrums_content %}
    <li class="breadcrumb-item"><a href="{{ url('admin_dashboard') }}">Admin</a></li>
    <li class="breadcrumb-item active" aria-current="page">Products</li>
{% endblock %}

{% block content %}
    <h1>Products</h1>
    ...
{% endblock %}
```

### Integration Notes {#admin-layout-integration-notes}

The admin template is designed to work well inside the Softspring ecosystem.

In particular:

- the optional route checks use `route_defined()`
- sidebar entries use `active_for_routes()`
- the avatar area uses `sfs_user_is()`

Those helpers are typically provided by `softspring/twig-extra-bundle` and related user features. If your project does not use them, keep the layout but override the related blocks.

## Flash Messages {#flash-messages}

To render flash messages consistently:

```twig
{% include '@SfsComponents/flash-messages/alerts.html.twig' %}
```

This is used widely across bundle screens.

The template:

- reads the Symfony flash bag
- maps `error` to Bootstrap `danger`
- renders dismissible alerts

If you already have your own flash container, keep using your own include and reuse this template only when it helps.

## Paginators {#paginators}

The paginator templates are one of the most reused parts of the package.

### Pager {#paginator-pager}

Use:

```twig
{% include '@SfsComponents/paginator/pager.html.twig' with {
    collection: collection
} %}
```

The pager expects a paginated collection object with page information such as:

- current page
- previous and next page
- total items
- collapsed page list

You can pass route data explicitly:

```twig
{% include '@SfsComponents/paginator/pager.html.twig' with {
    collection: collection,
    pagination_route: 'admin_products_list',
    pagination_route_attributes: { section: 'active' }
} %}
```

If you do not pass the route explicitly, the template falls back to the current request route.

### Table Wrapper {#paginator-table}

The table helper is useful for admin list pages:

```twig
{% embed '@SfsComponents/paginator/table.html.twig' with {
    collection: products,
    pagination_route: 'admin_products_list'
} %}
    {% block ths %}
        <th>Name</th>
        <th>Status</th>
    {% endblock %}

    {% block tds %}
        <td>{{ element.name }}</td>
        <td>{{ element.status }}</td>
    {% endblock %}
{% endembed %}
```

It gives you:

- optional filter form area
- table wrapper
- row loop with overridable blocks
- empty state
- pager footer when the collection exposes pages

This is the same integration pattern used in `account-bundle`, `user-bundle`, `cms-bundle`, `mailer-bundle`, and others.

### Semantic UI Legacy Usage {#paginator-semantic-ui}

Older projects still use the Semantic UI table and pager variants:

```twig
{% set sfs_components_theme = 'semantic-ui' %}
{% embed '@SfsComponents/paginator/table.html.twig' with {
    collection: entities,
    classes: 'table ui celled'
} %}
...
{% endembed %}
```

That support is kept mainly for legacy integrations. For new projects, prefer Bootstrap 5.

## Sidebar Templates {#sidebar-templates}

The package provides two admin sidebar views:

- `@SfsComponents/sidebar/sidebar-list.html.twig`
- `@SfsComponents/sidebar/sidebar-pills.html.twig`

In practice, projects usually select one of them through admin menu configuration:

```yaml
twig:
    globals:
        admin_menu:
            _view: '@SfsComponents/sidebar/sidebar-pills.html.twig'
            _cacheable: false
            _translation_domain: 'sfs_components'
            _active_for_routes_class: 'active'
```

Each block can define menu entries with:

- `translation_key`
- `route`
- `active_expression`
- `icon`
- optional `role`

The sidebar templates:

- hide routes that do not exist
- optionally hide entries that the user cannot access
- translate labels before rendering

This is the pattern used in `armonic-standalone` and older `base-project` setups.

## Admin Form Theme {#admin-form-theme}

The package also ships a horizontal Bootstrap 5 form theme:

```twig
{% form_theme form '@SfsComponents/forms/admin-horizontal.html.twig' %}
```

This theme is especially useful for:

- admin filter forms
- back office edit forms
- CRUD screens that need a compact horizontal layout

It customizes:

- labels
- row layout
- expanded choice rows
- submit, reset, button, and checkbox rows

You can use it for a single form or combine it with your application theme strategy.

## Overriding Templates {#overriding-templates}

The package is meant to be overridden.

A common pattern is:

```twig
{# templates/bundles/SfsComponentsBundle/layout/admin.bootstrap5.html.twig #}
{% extends '@!SfsComponents/layout/admin.bootstrap5.html.twig' %}

{% block header %}
    ...
{% endblock %}
```

Use this when you want to keep the shared structure but replace:

- branding
- assets
- header actions
- breadcrumb defaults
- sidebar presentation

This is usually better than copying the whole bundle template into many packages.

## When To Use This Package {#when-to-use-this-package}

Use `softspring/components` when you want a shared Twig UI base across several bundles or Symfony apps.

It is especially useful when:

- several packages should look coherent
- you need quick admin pages with reusable structure
- paginated tables appear in many places
- you want bundle templates to start from the same base layout

If your project already has a complete design system and custom layout stack, you may only want to reuse the paginator or flash templates.

## Current Limits {#current-limits}

- The package is template-only. It does not provide controllers or domain logic.
- The admin layout becomes more useful when combined with `twig-extra-bundle` and related user routes.
- Bootstrap 5 is the main supported theme. Semantic UI support is limited and mainly kept for older integrations.
