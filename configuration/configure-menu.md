---
title: "Configure CMS Menus"
description: "Define Armonic CMS menus with YAML configuration, rendering templates, cache settings, and custom form fields."
---

# Menu Configuration (`cms/menus`)

Each menu is configured in:

- `cms/menus/<menu_name>/config.yaml`

The schema is defined in `cms-bundle/src/Config/Model/Menu.php`.

## Minimal Example

```yaml
menu:
    revision: 1
```

Only `revision` is required. The rest is optional and has defaults.
If you clear the cache, you’ll be able to see the menu in the drop-down menu of the ‘New menu’ button.

## Practical Example

```yaml
menu:
    revision: 1
    render_template: '@menu/main/render.html.twig'
    esi: true
    isolate_request: true
    cache_ttl: 600
    singleton: true
    items: true
    form_template: '@menu/main/form.html.twig'
    form_fields:
        cssClass:
            type: text
        maxDepth:
            type: integer
            type_options:
                required: false
                empty_data: 2
```

## Option Reference

Top-level `menu` keys:

- `revision` (required, integer)
- `render_template` (optional, default: `@menu/<menu_name>/render.html.twig`)
- `esi` (optional, default: `true`)
- `isolate_request` (optional, `true|false|null`, default: `null`)
- `cache_ttl` (optional, integer, default: `false`)
- `singleton` (optional, boolean, default: `true`)
- `items` (optional, boolean, default: `true`)
- `form_template` (optional, string)
- `form_fields` (optional map)

### `form_fields`

Each field entry supports:

- `type` (required)
- `type_options` (optional map)

Example:

```yaml
form_fields:
    title:
        type: translatable
    showIcons:
        type: checkbox
        type_options:
            required: false
```

## What Is Usually Required In Real Projects

- Always set `revision`.
- Override `render_template` when the default path does not match your Twig structure.
- Set `cache_ttl` for menus that are expensive to build or mostly static.
- Use `form_fields` only if editors need extra configurable values beyond menu items.

## Notes and Behavior

- `isolate_request` is intended for ESI sub-requests. Keep `esi: true` when using it.
- `items: true` means this menu handles menu items in the CMS.
- `singleton: true` is the common setup (one configuration per menu id).
- Increase `revision` when your menu structure/config expectations change.
