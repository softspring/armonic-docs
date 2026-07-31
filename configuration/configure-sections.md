---
title: "Configure CMS Sections"
description: "Reference for the sfs_cms_sections configuration block: section entities, compiled output, automatic compilation, and recompile behavior."
---

# Section Configuration (`sfs_cms_sections`) {#section-configuration-sfs-cms-sections}

Sections are configured at application level with the `sfs_cms_sections` Symfony configuration.

This page is only about configuration. For creating, editing, rendering, publishing, and using sections in CMS content, see [CMS Bundle Sections](../bundles/cms-bundle/sections.md).

This is different from modules. Modules are reusable content types defined in `cms/modules/<module_name>/config.yaml`. Sections are editable content records created from the CMS admin. The configuration below controls how those records are stored and compiled.

## Minimal Example {#minimal-example}

The plugin works with defaults:

```yaml
sfs_cms_sections: ~
```

With this configuration, Armonic uses the default section and section version entities provided by the plugin.

## Practical Example {#practical-example}

```yaml
sfs_cms_sections:
    section:
        class: App\Entity\Cms\Section
        version_class: App\Entity\Cms\SectionVersion
        find_field_name: id
        save_compiled: true
        autocompile_on_save: false
        autocompile_on_publish: true
        prefix_compiled: ''
        recompile: true
```

Use this when the project needs custom Doctrine entities for sections or when you need to tune how compiled section output is generated and stored.

## Option Reference {#option-reference}

Top-level `sfs_cms_sections.section` keys:

- `class` (optional, default: `Softspring\CmsSectionsPlugin\Entity\Section`)
- `version_class` (optional, default: `Softspring\CmsSectionsPlugin\Entity\SectionVersion`)
- `find_field_name` (optional, default: `id`)
- `save_compiled` (optional, default: `true`)
- `autocompile_on_save` (optional, default: `false`)
- `autocompile_on_publish` (optional, default: `true`)
- `prefix_compiled` (optional, default: empty string)
- `recompile` (optional, default: `true`)

## What Each Option Does {#what-each-option-does}

### `class` {#class}

Doctrine entity used for sections.

Keep the default unless the project needs extra section metadata or custom relations. A custom class must implement `Softspring\CmsSectionsPlugin\Model\SectionInterface`.

### `version_class` {#version-class}

Doctrine entity used for section versions.

Keep the default unless the project needs extra version metadata or custom relations. A custom class must implement `Softspring\CmsSectionsPlugin\Model\SectionVersionInterface`.

### `find_field_name` {#find-field-name}

Lookup field configured for section managers and related admin services. The built-in frontend section render route receives the section id.

### `save_compiled` {#save-compiled}

Controls whether compiled section output is stored in the CMS compiled data storage.

Keep it enabled when sections are rendered through the normal compiled-content flow, especially when using ESI, AJAX, or cached fragments.

### `autocompile_on_save` {#autocompile-on-save}

Controls whether a section version is compiled when it is saved.

The default is `false`. This avoids compiling every draft save. The plugin only autocompiles on save when there is an active request and `save_compiled` is enabled.

### `autocompile_on_publish` {#autocompile-on-publish}

Controls whether a section version is compiled before it is published.

The default is `true`. Publishing compiles the section for every configured site and every locale enabled on the section. If compilation produces errors, the version is not published.

### `prefix_compiled` {#prefix-compiled}

Prefix added to compiled data keys.

Most projects can leave it empty. Use it only when the project needs to separate compiled keys for a specific integration.

### `recompile` {#recompile}

Enables the admin recompile action for section versions.

Keep it enabled when editors or administrators may need to regenerate compiled section output after a template, module, or configuration change.

## Difference From Module Configuration {#difference-from-module-configuration}

Do not create section definitions in `cms/sections`. Sections are not configured like module types.

The plugin adds its own `section` CMS module through a CMS collection:

```yaml
sfs_cms:
    collections:
        - vendor/softspring/cms-sections-plugin/cms
```

That module is what editors use to insert an existing section into a page or content type. The module itself comes from the plugin; the section records are created in the admin area.

## What To Configure First {#what-to-configure-first}

- Start with `sfs_cms_sections: ~`.
- Configure routes and permissions as described in the sections guide.
- Add custom `class` and `version_class` only when the project needs custom Doctrine entities.
- Keep `autocompile_on_publish: true` unless publishing should be allowed without precompilation.
- Keep `recompile: true` in projects where templates or module configuration can change after content has been published.
