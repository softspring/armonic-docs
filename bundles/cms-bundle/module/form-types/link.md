---
title: "CMS link field"
description: "Let editors choose an internal route, an anchor, or an external URL using the CMS link form type."
related: ["bundles/cms-bundle/modules/symfony-route", "configure-modules"]
---

# Link field {#link-field}

Use `link` when one module can point to an internal route, an anchor on the page, or an external URL. The field extends the CMS Symfony-route type and adds destination mode and browsing target controls.

## Define the field {#define-the-field}

This is a fragment of an existing module definition:

```yaml
module:
    module_options:
        form_fields:
            button_link:
                type: link
```

The form contains `type`, `route_name`, `route_params`, `anchor`, `url`, `target`, and `custom_target`. The CMS admin assets show the controls relevant to the selected destination type.

## Render it {#render-it}

```twig
{% if button_link|default(false) %}
    <a {{ sfs_cms_link_attr(button_link) }}>
        {{ button_text|default('Read more')|sfs_cms_trans }}
    </a>
{% endif %}
```

The helper builds the destination and target attributes. Anchor values are normalized to one leading `#`; route destinations go through CMS URL resolution. Keep the link label escaped.

## Understand the saved structure {#understand-the-saved-structure}

For an external URL, the relevant values look like this:

```json
{
    "type": "url",
    "route_name": null,
    "route_params": "{}",
    "anchor": null,
    "url": "https://example.test/contact",
    "target": "_self",
    "custom_target": null
}
```

For an external link the unused route parameters may remain a JSON string in the submitted value, as shown above. A route link uses `type: route` plus its route name and a decoded parameter map. An anchor link uses `type: anchor` plus the anchor value. The target can be `_self`, `_blank`, `_parent`, `_top`, or a custom browsing-context name.

## Restrict destinations {#restrict-destinations}

Inherited route filtering options apply to internal route choices; see [Symfony route fields](symfony-route.md). They do not validate arbitrary external URL schemes. Add application validation for the URL policies your editors need rather than treating a display helper as a security validator.

When replacing an older route-only field, migrate existing module data to the new structure before requiring `type` and `target` in the template. The [module migration guide](../../modules.md#migrate-stored-values) explains revision-based conversions.
