---
title: "Using CMS menus"
description: "Render translated navigation and nested menu items while preserving Symfony and CMS route behavior."
related: ["configure-menu", "bundles/cms-bundle/routing"]
---

# Using menus {#using-menus}

A menu is an editable navigation record. Its type defines the settings and rendering template; its items hold translated labels, route selections, and nested children.

Follow [Configure menus](../../configuration/configure-menu.md) to create a `main` type and [Create a menu](../../user-manual/create-a-new-menu.md) to populate its record.

## Render shared navigation {#render-shared-navigation}

```twig
{{ sfs_cms_menu('main') }}
```

The renderer loads the menu record by type and passes it to `@menu/main/render.html.twig` as `menu`. Creating the type alone does not create that record. A menu link does not create or publish its destination page.

## Render nested items {#render-nested-items}

This template handles route items and submenu headings:

```twig
{# cms/menus/main/render.html.twig #}
{% macro items(entries) %}
    <ul>
        {% for item in entries %}
            <li>
                {% if item.symfonyRoute.route_name|default(false) %}
                    <a href="{{ sfs_cms_url(item.symfonyRoute) }}">
                        {{ item.text|sfs_cms_trans }}
                    </a>
                {% else %}
                    <span>{{ item.text|sfs_cms_trans }}</span>
                {% endif %}
                {% if item.items|length %}
                    {{ _self.items(item.items) }}
                {% endif %}
            </li>
        {% endfor %}
    </ul>
{% endmacro %}

<nav aria-label="Main navigation">
    {{ _self.items(menu.items|filter(item => item.parent is null)) }}
</nav>
```

The translation filter resolves labels for the active locale. The CMS URL helper also handles CMS route references. Keep normal Twig escaping for labels.

## Add menu settings {#add-menu-settings}

Use `form_fields` in the type for settings such as a translated navigation heading. The editable item list remains available when `items: true`. The template can read additional saved settings from the menu's data.

## Cache the correct context {#cache-the-correct-context}

Menus can use ESI and `cache_ttl`; use direct rendering while validating a new template. Check the result for every site and language. Avoid caching personalized navigation as public shared HTML. Permission-dependent application navigation may be better rendered by the application outside the shared CMS menu.
