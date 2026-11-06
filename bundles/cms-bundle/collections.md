---
title: "CMS Bundle Collections"
description: "Extend the CMS with reusable module collections distributed as composer packages and loaded through configuration."
---

# Extending with collections {#extending-with-collections}

A collection groups CMS definitions and templates that can be reused across applications. It can provide sites, content types, layouts, modules, blocks, menus, and their translations.

Create a Composer package containing the same directory structure as the application's `cms/` directory, then add the directory containing those definitions to `sfs_cms.collections`:

<!-- config-tabs:start -->

```yaml
sfs_cms:
    collections:
         - vendor/softspring/cms-module-collection
```

```php
<?php

// config/packages/sfs_cms.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_cms', ['collections' => ['vendor/softspring/cms-module-collection']]);
};
```

<!-- config-tabs:end -->

## Organize a collection {#organize-collection}

```text
vendor/example/editorial-components/cms/
├── modules/
│   └── rating/
│       ├── config.yaml
│       └── render.html.twig
└── layouts/
    └── landing/
        ├── config.yaml
        └── render.html.twig
```

For this structure, register `vendor/example/editorial-components/cms`, not the package root. Collection paths are relative to the project directory. Later collections override earlier definitions; the application's own collection can customize a shared module without editing `vendor/`.

Use the module and layout identifiers consistently in configuration and Twig namespaces. Clear the Symfony cache after adding a collection. Test existing content when changing a published collection: stored module data may require a revision migration, and compiled pages may require recompilation.
