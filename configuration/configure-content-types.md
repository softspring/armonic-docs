---
title: "Configure CMS Content Types"
description: "Define Armonic CMS content types with YAML configuration, admin form fields, rendering rules, and entity mapping."
---

# Configure content types {#configure-content-types}

A content type defines the entity used by a page, the layouts available to its editors, and additional metadata fields. Definitions live in `cms/contents/<type>/config.yaml`.

Armonic ships a `page` type. Start by extending that type when you need extra metadata without introducing a new entity. For example, pages can have a translated summary and a category used in listing cards.

## Add metadata to pages {#add-page-metadata}

```yaml
# cms/contents/page/config.yaml
content:
    revision: 3
    entity_class: 'Softspring\CmsBundle\Entity\Page'
    default_layout: default
    allowed_layouts: ['default', 'landing']
    extra_fields:
        summary:
            type: translation
            type_options:
                type: textarea
        category:
            type: text
            type_options:
                required: false
```

The project definition is merged with the `page` definition supplied by the bundle. Keep only layout identifiers that exist in your application; `landing` refers to the layout created in [Configure layouts](configure-layouts.md).

`extra_fields` configures the create and update forms. Values are stored in the content's `extraData` JSON field, rather than becoming new database columns. Use the [dynamic form types](../components/dynamic-form-type.md) for available field types and options.

The `entity_class` is required in a complete content definition. It can be inherited when overriding an existing type; it is not optional when introducing a new type.

## Display the metadata {#display-metadata}

In a layout or another template with access to the content version:

```twig
{% set page = version.content %}
<p class="page-summary">{{ page.extraData.summary|default('')|sfs_cms_trans }}</p>
{% if page.extraData.category|default('') %}
    <span class="page-category">{{ page.extraData.category }}</span>
{% endif %}
```

The translation filter selects the summary for the current locale. The category remains ordinary escaped text. These fields can also be read from content objects in application code to build listings or custom views.

## Choose layouts {#choose-layouts}

`default_layout` selects the initial layout for new versions. `allowed_layouts` limits the editor's choices. A content type can also restrict modules in named `containers`; keep these names aligned with its layouts.

## Create a distinct content type {#create-content-type}

Use a new type when content has its own domain behavior, such as articles with an author relation. Create its Doctrine entity and mapping before referring to it in `entity_class`, then add `cms/contents/article/config.yaml`.

A new type needs a concrete entity implementing `Softspring\CmsBundle\Model\ContentInterface`. Do not assume that adding a YAML file creates an entity, a database table, or an application listing. Register any additional Doctrine mapping and generate the application migration for your chosen database engine.

Add the type identifier to the relevant site's `allowed_content_types`. A type can exist in the configuration but remain unavailable for a site that does not allow it.

## Customize the admin screens {#customize-admin-screens}

The `admin` section overrides the default screen templates. For example, after creating these templates:

```yaml
content:
    revision: 3
    entity_class: 'Softspring\CmsBundle\Entity\Page'
    admin:
        create:
            view: '@content/page/admin/create.html.twig'
        update:
            view: '@content/page/admin/update.html.twig'
        list:
            page_view: '@content/page/admin/list-page.html.twig'
```

Keep the existing form fields, CSRF protection, actions, and layout blocks when extending the default templates. A template override changes presentation; it does not replace the controller's authorization rules.

## Apply the configuration {#apply-configuration}

Clear the cache, create or update a page, and check that the metadata fields and layout choices appear:

```bash
php bin/console cache:clear
```

`revision` identifies the configuration revision; it does not generate a Doctrine migration or automatically convert stored data. When changing existing data structures, provide the corresponding migration strategy and recompile affected published content.
