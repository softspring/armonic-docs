---
title: "CMS forms and editorial metadata"
description: "Add editorial properties with dynamic form fields and choose when custom Symfony forms or mapped entities are needed."
related: ["configure-content-types", "components/dynamic-form-type"]
---

# Forms and editorial metadata {#forms-and-editorial-metadata}

Use metadata for properties of a content record, such as a summary, reading time, or card image. Use modules for the content composed inside a layout. This keeps listing data separate from page body structure.

## Add properties to an existing type {#add-properties-to-an-existing-type}

The following override extends the existing `page` type. It inherits that type's entity and templates:

```yaml
# cms/contents/page/config.yaml
content:
    revision: 3
    extra_fields:
        summary:
            type: translation
            type_options:
                type: textarea
        reading_minutes:
            type: integer
            type_options:
                required: false
```

Values are stored in `Content.extraData`; adding them does not create separate SQL columns. Use a mapped entity field or association when the domain needs relational integrity, indexing, or efficient database queries.

In a template with a content version:

```twig
{% set page = version.content %}
<p>{{ page.extraData.summary|default('')|sfs_cms_trans }}</p>
{% if page.extraData.reading_minutes|default(0) %}
    <p>{{ page.extraData.reading_minutes }} min</p>
{% endif %}
```

Use the same pattern for a media selection or an application-specific field type. Define media types and output restrictions before adding the selector.

## Reuse Symfony form behavior {#reuse-symfony-form-behavior}

CMS dynamic fields resolve application form types, CMS types, and Symfony types. A field's `type_options` are Symfony options; place `attr`, validation constraints, and choice settings inside that block.

See [Dynamic Form Type](../../components/dynamic-form-type.md) and the [custom rating form](modules.md#build-your-own-form-types-for-modules). Keep validation in the form and application model, including for generated or imported values.

## Customize an admin form {#customize-an-admin-form}

Content definitions expose admin actions such as `create`, `update`, `version_create`, and `version_update`. Their `type` selects the Symfony form class and `view` selects the Twig template. Override the specific action that needs different fields instead of replacing the entire controller.

A new content type needs its own entity mapping and a complete definition with `entity_class`. An override of an existing type can inherit it. Clear configuration cache after changing definitions and test both creation and update forms with existing content.

## Store the value at the right level {#store-the-value-at-the-right-level}

Site metadata, content extra data, version metadata, and module data have different lifetimes. Site-wide instructions belong to a site; record-wide properties belong to content; values that must vary with each draft belong to versioned module data or an application version extension. Republishing an old version does not rewind every record-level property.
