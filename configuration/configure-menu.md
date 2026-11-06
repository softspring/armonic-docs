---
title: "Configure CMS Menus"
description: "Define Armonic CMS menus with YAML configuration, rendering templates, cache settings, and custom form fields."
---

# Configure menus {#configure-menus}

A menu type defines the navigation structure editors can manage and the template that displays it. Define it under `cms/menus/<type>/config.yaml`, then create the corresponding menu record in the admin area.

For example, a shared `main` menu can link the homepage, an About page, and an existing Symfony route.

## Define the menu type {#define-menu-type}

```yaml
# cms/menus/main/config.yaml
menu:
    revision: 1
    singleton: true
    items: true
    esi: false
```

`items: true` enables editable menu items. `singleton: true` allows one record for this menu type. The example disables ESI so it can be used without an ESI-capable cache.

## Render the items {#render-items}

```twig
{# cms/menus/main/render.html.twig #}
<nav aria-label="Main navigation">
    <ul>
        {% for item in menu.items %}
            <li>
                <a href="{{ sfs_cms_url(item.symfonyRoute) }}">
                    {{ item.text|sfs_cms_trans }}
                </a>
            </li>
        {% endfor %}
    </ul>
</nav>
```

`menu` is the editable menu record. The URL helper resolves each item's route selection and parameters; the translation filter renders its label for the current locale.

The example handles top-level items. If your menu uses nested items, extend the template to render the hierarchy and provide keyboard-accessible controls for expandable groups.

Include the menu in a page layout:

```twig
{{ sfs_cms_menu('main') }}
```

The helper renders the singleton record of the `main` type. The default template is `@menu/main/render.html.twig`.

## Populate the navigation {#populate-navigation}

Clear the cache after creating the type:

```bash
php bin/console cache:clear
```

Open **Menus**, create the **main** menu, and add items with labels and route destinations. Save, open the site, and follow each link in every enabled locale. See [Create a menu](../user-manual/create-a-new-menu.md) for the admin workflow.

The route destination must exist. Creating a menu item does not create or publish the content it links to.

## Additional menu fields {#additional-menu-fields}

Use `form_fields` for settings beyond the item list, such as a heading:

```yaml
menu:
    revision: 1
    esi: false
    form_fields:
        heading:
            type: translation
```

These fields use the dynamic form system. Read saved values from the menu's data when customizing its template, and keep `items: true` when the record should also manage links.

## Cache and template options {#cache-template-options}

| Option | Purpose |
| --- | --- |
| `render_template` | Frontend template; defaults to `@menu/<type>/render.html.twig`. |
| `form_template` | Optional custom layout for the menu settings form. |
| `cache_ttl` | Fragment lifetime when using HTTP caching; omitted by default. |
| `esi` | ESI rendering, enabled by default; disable it for direct rendering. |
| `isolate_request` | Controls access to the user/session context for ESI rendering. |

Keep shared navigation independent of private account data when caching it publicly. Verify the output with the HTTP cache configuration used by the application.
