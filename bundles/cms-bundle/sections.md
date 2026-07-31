---
title: "CMS Bundle Sections"
description: "Create, publish, render, and configure reusable CMS sections with the CMS Sections Plugin."
---

# Sections {#sections}

Sections are reusable editable content fragments. They are useful for content that appears in several pages or templates, such as a shared banner, a newsletter call to action, a support strip, or a reusable product block.

A section is managed outside a page, has its own versions, and can be rendered from another CMS page, from a module, or directly from Twig.

## When to use sections {#when-to-use-sections}

Use a section when the same editable content must be maintained once and displayed in more than one place.

Use a normal page module when the content belongs only to that page. Use a block when you need the block system behavior and its block-specific configuration. A section is closer to a small reusable CMS page: editors can build it with modules, preview it, publish a version, and then reference it elsewhere.

Common examples:

- site-wide calls to action
- reusable header or footer fragments
- campaign strips reused across landing pages
- editorial components shared by several content types
- fragments rendered asynchronously or through ESI

## Install the plugin {#install-the-plugin}

Sections are provided by `softspring/cms-sections-plugin`.

Install the package with Composer:

```bash
composer require softspring/cms-sections-plugin
```

If your project does not use Symfony Flex, enable the plugin in `config/bundles.php`:

```php
return [
    // ...
    Softspring\CmsSectionsPlugin\SfsCmsSectionsPlugin::class => ['all' => true],
];
```

The plugin prepends its CMS collection automatically:

```yaml
sfs_cms:
    collections:
        - vendor/softspring/cms-sections-plugin/cms
```

That collection provides the `section` module and its form, edit, and render templates.

## Register routes {#register-routes}

Add the frontend route used to render sections:

```yaml
# config/routes/sfs_cms.yaml
_sfs_cms_sections_frontend_:
    resource: "@SfsCmsSectionsPlugin/config/routing/frontend_sections.yaml"
    prefix: /{_locale}
```

The plugin exposes the section render route at:

```text
/{_locale}/__/s/{section}
```

Add the admin routes where you want the CMS backoffice to manage sections:

```yaml
# config/routes/admin/sfs_cms.yaml
_sfs_cms_sections_:
    resource: "@SfsCmsSectionsPlugin/config/routing/admin_sections.yaml"
    prefix: "/cms/sections"
```

These routes add the section list, create, update, delete, preview, content edition, version list, publish, unpublish, recompile, clear compiled content, and cleanup actions.

### Register the admin routing provider {#register-the-admin-routing-provider}

The admin route file includes an internal plugin route type named `sfs_cms_plugin_admin_section`. If your installed plugin version does not register a routing provider for this type, Symfony fails while loading routes with this error:

```text
Cannot load resource ".". Make sure there is a loader supporting the "sfs_cms_plugin_admin_section" type.
```

In that case, register a small routing provider in the application:

```php
<?php

declare(strict_types=1);

namespace App\Cms\Infrastructure\Routing;

use Softspring\CmsBundle\Routing\Provider\RoutingProviderInterface;
use Symfony\Component\Routing\RouteCollection;

final class SectionsAdminRoutingProvider implements RoutingProviderInterface
{
    public function supportedTypes(): array
    {
        return ['sfs_cms_plugin_admin_section'];
    }

    public function supports(string $type): bool
    {
        return in_array($type, $this->supportedTypes(), true);
    }

    public function getAdminRoutes(string $type): RouteCollection
    {
        return new RouteCollection();
    }
}
```

Register it with the CMS routing provider tag:

```yaml
# config/services.yaml
services:
    App\Cms\Infrastructure\Routing\SectionsAdminRoutingProvider:
        tags: [ 'sfs_cms.routing_provider' ]
```

This provider only makes Symfony accept the plugin route type. It does not add routes, because the section routes are already declared explicitly by the plugin route file.

## Add security roles {#add-security-roles}

Import the role hierarchy shipped by the plugin:

```yaml
# config/packages/security.yaml
imports:
    - { resource: '@SfsCmsSectionsPlugin/config/security/admin_role_hierarchy.yaml' }
```

Then give your admin role one of the section roles:

```yaml
security:
    role_hierarchy:
        ROLE_ADMIN:
            - ROLE_SFS_CMS_ADMIN_SECTIONS_RW
```

Available roles:

- `ROLE_SFS_CMS_ADMIN_SECTIONS_RO` can list, read, preview, and inspect versions.
- `ROLE_SFS_CMS_ADMIN_SECTIONS_CREATOR` can also create sections and versions.
- `ROLE_SFS_CMS_ADMIN_SECTIONS_PUBLISHER` can update, publish, unpublish, delete, keep versions, recompile, clear compiled data, and clean old versions.
- `ROLE_SFS_CMS_ADMIN_SECTIONS_RW` includes the publisher role.

## Add the admin menu entry {#add-the-admin-menu-entry}

If your project builds the admin menu manually, add an entry that points to `sfs_cms_admin_sections_list`:

```yaml
twig:
    globals:
        admin_menu:
            cms:
                sections:
                    translation_key: 'sidebar.menu.cms.sections'
                    route: 'sfs_cms_admin_sections_list'
                    role: PERMISSION_SFS_CMS_ADMIN_SECTION_LIST
                    active_expression: 'sfs_cms_admin_sections_'
                    icon: 'bi bi-grid-fill me-2'
```

## Add admin assets {#add-admin-assets}

The plugin includes JavaScript for section previews and for the `section` form type messages. Add the asset package and import its admin entry when your admin build does not already include it:

```bash
yarn add "file:vendor/softspring/cms-sections-plugin/assets" --dev
```

```js
// assets/admin.js
import '@softspring/cms-sections-plugin/scripts/admin-cms';
```

## Configuration reference {#configuration-reference}

The plugin works with defaults, but projects can tune the section entity classes, compiled output, automatic compilation, and recompile behavior.

For the option reference, see [Configure CMS Sections](../../configuration/configure-sections.md).

## Create a section {#create-a-section}

In the admin area, open **CMS / Sections** and create a new section.

The section form asks for:

- `name`, used by editors to identify the section.
- `ttl`, stored as section extra data and used for HTTP cache max age when the section is rendered through the section endpoint.
- `defaultLocale`, used as the base locale for editable fields.
- `locales`, the languages enabled for this section.
- `notes`, shown to editors when the section is selected from a page module or form field.

After creating the section, edit its content. A section version uses the same module collection editor as CMS pages, so editors can compose the section with the configured CMS modules.

Saving creates a version. Publishing marks that version as the public version of the section.

## Insert a section in CMS content {#insert-a-section-in-cms-content}

The plugin provides a `section` module. It can be added to a page or content type like any other CMS module.

The module fields are:

- `id`, optional HTML id wrapper.
- `class`, optional CSS classes wrapper.
- `section`, the section to render.
- `mode`, the render mode.

Render modes:

- `embedded` renders the section through Symfony sub-request rendering.
- `esi` renders an `<esi:include>` tag that points to the section endpoint.
- `ajax` renders an empty placeholder and loads the section endpoint in the browser.

Choose `embedded` when the section is part of the page response and should be available immediately. Choose `esi` when your HTTP cache or reverse proxy is configured to process ESI. Choose `ajax` when the fragment can load after the page and you want to keep it independent from the page response.

The admin form warns editors when a draft section is selected or when a published section uses ESI/AJAX with or without a TTL.

## Render sections from Twig {#render-sections-from-twig}

You can render sections directly from Twig.

Render by id:

```twig
{{ sfs_cms_section('018f2f6a-8d89-7a0c-9a4c-8e4b9c2f5d31') }}
```

Find and render:

```twig
{% set section = sfs_cms_section_find({ name: 'Footer CTA' }) %}
{{ sfs_cms_section(section) }}
```

Render with ESI:

```twig
{{ sfs_cms_section_esi(section) }}
```

Render with AJAX:

```twig
{{ sfs_cms_section_ajax(section) }}
```

You can also use dynamic field helpers:

```twig
{{ sfs_cms_section_embed_by_name('Footer CTA') }}
{{ sfs_cms_section_esi_by_name('Footer CTA') }}
{{ sfs_cms_section_ajax_by_name('Footer CTA') }}
```

The `find_by_*` and render `*_by_*` helpers add the dynamic field to the criteria. For example, `sfs_cms_section_embed_by_name('Footer CTA')` searches with `name = Footer CTA`.

## Render behavior and cache {#render-behavior-and-cache}

The frontend render controller loads the section by id and renders its published version. If the section has no published version, the route returns a not found response unless `do_not_throw_not_found` is passed.

Admin preview is different: when a section has no published version, preview can use the last version so editors can inspect drafts.

When the response is successful and the section has a `ttl`, the controller marks the response as public and sets `max-age` to that value. This matters most for ESI and AJAX rendering, because both call the section endpoint separately.

When compiled content contains render errors, the section endpoint returns HTTP 500. This makes broken reusable fragments visible during publishing or direct rendering instead of silently serving invalid content.

## Versions and publishing {#versions-and-publishing}

Each section keeps its own versions. A version contains the module data, linked medias, linked routes, linked sections, compiled data, and version metadata.

The admin area supports:

- creating a version from the last version or from a selected previous version
- previewing a version
- publishing a version
- keeping or unkeeping a version so cleanup does not remove it
- editing the version note
- recompiling a version
- clearing compiled data
- deleting a version
- cleaning versions that are eligible for cleanup
- unpublishing the section

When `autocompile_on_publish` is enabled, publishing compiles the section for every configured site and every locale enabled on the section. The published version is also marked as kept.

## Linked usages {#linked-usages}

The CMS tracks section relationships in content versions and compiled data. This lets the admin area show where a section is used.

The relationship is created when a section is selected by the `section` module or by a field using the `section` form type. This is useful before editing or unpublishing a shared section, because editors can inspect which pages or other sections depend on it.

## Use the section form type in custom modules {#use-the-section-form-type-in-custom-modules}

The plugin registers a CMS form field type named `section`. Use it in module configuration when a custom module needs to reference a reusable section:

```yaml
module:
    revision: 1
    group: 'cms-blocks'
    edit_template: '@module/my_module/edit.html.twig'
    form_template: '@module/my_module/form.html.twig'
    render_template: '@module/my_module/render.html.twig'
    module_options:
        form_fields:
            section:
                type: 'section'
```

The field lists sections by name. Draft sections are marked in the choice label, and the current section is filtered out when editing a section so editors cannot create a direct self-reference.

The form type adds data attributes used by the admin JavaScript:

- section detail URL
- draft state
- TTL value
- section notes
- preview URLs for each configured site and enabled locale

## Extend the section model {#extend-the-section-model}

Projects can use custom Doctrine entities for sections and section versions. Configure them with `sfs_cms_sections.section.class` and `sfs_cms_sections.section.version_class`.

The section entity should extend the base model or implement `SectionInterface`. The version entity should extend the base version model or implement `SectionVersionInterface`.

Use this when the project needs extra metadata, project-specific relations, or custom admin behavior around reusable fragments. Keep the standard versioning and compilation behavior unless there is a clear reason to change it, because the render controller, Twig helpers, admin actions, and relationship listeners expect the section and version contracts.
