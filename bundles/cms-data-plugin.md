---
title: "CMS Data Plugin"
description: "Import and export CMS content with its references and organize reusable local fixture data."
related: ["bundles/cms-bundle/content-versions", "bundles/cms-bundle/forms-and-metadata"]
---

# CMS Data Plugin {#cms-data-plugin}

Use the data plugin to move structured CMS content between compatible projects or environments. It adds content import, version import, and version export actions, plus serializers and fixture support for routes, menus, blocks, media, and their references.

## Install and register {#install-and-register}

```bash
composer require softspring/cms-data-plugin:6.0.x-dev
```

Register the bundle when Flex does not do it automatically:

```php
// config/bundles.php
Softspring\CmsDataPlugin\SfsCmsDataPlugin::class => ['all' => true],
```

The plugin contributes actions through the CMS content routing providers. Keep the standard content admin imports so those actions can be registered. There is no separate mandatory global configuration block.

## Grant import and export access {#grant-import-and-export-access}

<!-- config-tabs:start -->

```yaml
# config/packages/security.yaml
imports:
    - { resource: '@SfsCmsDataPlugin/config/security/admin_role_hierarchy.yaml' }
```

```php
<?php

// config/packages/security.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->import('@SfsCmsDataPlugin/config/security/admin_role_hierarchy.yaml');
};
```

<!-- config-tabs:end -->

Assign the imported capabilities through the application's role hierarchy. The action defaults check `PERMISSION_SFS_CMS_ADMIN_CONTENT_IMPORT`, `PERMISSION_SFS_CMS_ADMIN_CONTENT_VERSION_IMPORT`, and `PERMISSION_SFS_CMS_ADMIN_CONTENT_VERSION_EXPORT`.

## Move a version {#move-a-version}

1. Export the intended content version from the source admin.
2. Confirm that the destination has compatible content types, layouts, module revisions, media types, and templates.
3. Import into a disposable destination first. Inspect the selected content, imported version, routes, and referenced media.
4. Preview the result in every relevant language before making it public.

A ZIP can carry structured data and media files. It does not install PHP classes, render templates, Symfony routes, or external storage permissions. Keep exports scoped to data intended for the destination.

## Organize fixtures {#organize-fixtures}

The loader reads from the application's `cms/fixtures` directory:

```text
cms/fixtures/
├── contents/
├── routes/
├── blocks/
├── menus/
└── media/
```

Media descriptors use JSON with companion files; other supported elements are read through the structured YAML storage. References are resolved across imported elements instead of treating every identifier as an unrelated string.

The CMS fixture group is `sfs_cms`. Its importer uses fixture-origin versions and requests automatic publication. Loading fixture data therefore has editorial effects; use synthetic fixtures in isolated databases. The dump command currently writes to `/srv/cms/fixtures` and clears exported media files before dumping, so it is not a generic configurable backup destination. Keep operational fixture procedures in the package's contribution guidance.

## Adapt application data {#adapt-application-data}

Implement `EntityTransformerInterface` for additional entity types or `FieldTransformerInterface` for specialized saved fields. With autoconfiguration enabled, the plugin tags these services as `sfs_cms.data.entity_transformer` and `sfs_cms.data.field_transformer`. Check priorities and preserve reference resolution when decorating a transformer.

Content definitions can override the `admin.import`, `admin.version_import`, and `admin.export_version` actions. Use this for project-specific permissions or import forms while retaining the standard workflow.
