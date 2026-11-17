---
title: "CMS Module Collection"
description: "Install and customize the shared editing modules used by Armonic CMS pages."
related: ["bundles/cms-bundle/modules", "bundles/cms-bundle/collections"]
---

# CMS Module Collection {#cms-module-collection}

The module collection supplies reusable definitions, edit forms, templates, translations, and browser behavior for CMS modules. It gives a project a starting vocabulary while allowing its own collection to override the design.

## Load the collection {#load-the-collection}

```bash
composer require softspring/cms-module-collection:6.1.*@dev
```

Register the package root, which contains `modules/` directly:
<!-- config-tabs:start -->

```yaml
# config/packages/sfs_cms.yaml
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

Do not append `/cms` to this package's path. Merge the entry with existing collections and keep the application's overrides later in the collection order.

## Choose modules for a layout {#choose-modules-for-a-layout}

The collection includes headings, buttons, hero/content elements, and more specialized modules. Inspect the installed `modules/` directory and each module's `config.yaml` for its actual identifier, fields, revision, and group.

Expose the intended modules through the layout/content container's `allowed_modules`; installing a collection does not mean every module suits every container. Start with a heading and button, preview them in the application's theme, then add more complex elements.

## Load browser behavior {#load-browser-behavior}

For Encore:

```js
// assets/app.js
import '../vendor/softspring/cms-module-collection/assets/modules.js';
```

The published AssetMapper entry is `@softspring/cms-module-collection/modules.js`. The application must also load styles appropriate to the selected modules. A Bootstrap-oriented template does not install Bootstrap into the application automatically.

## Override a module {#override-a-module}

Create the same identifier under the application's `cms/modules/` directory and override the required files. For example, replace a button's render template while keeping its field structure.

If the data format changes, increment the revision and provide a migration. If only the public template changes, recompile affected published content. See [CMS collections](cms-bundle/collections.md) and [module migrations](cms-bundle/modules.md#migrate-stored-values).

## Give assistants module guidance {#ai-instructions}

Every module shipped by this collection declares `module.meta.ai_instructions` in its own `config.yaml`.
The guidance explains when to choose the module, how to compose its content and which alternatives fit better.
For example, ordinary body copy should use separate paragraph modules, rich HTML belongs in the appropriate
wysiwyg module, and hero composition should keep the message concise and text readable over its imagery.
Media and link guidance directs clients to use actual configured choices and existing identifiers.

These instructions are inherited when the collection is loaded. The CMS API/MCP expose them as `aiInstructions`
in configuration and module prototype metadata; no application copy is needed. Override only project-specific
preferences with a partial definition at the same identifier:

```yaml
# cms/modules/hero/config.yaml (override fragment)
module:
    meta:
        ai_instructions: |
            Use for the main introduction to a service page.
            Prefer the project's background media type and one clear primary action.
```

Changing guidance does not alter the data format or require a module revision migration. Clear the application
cache after changing definitions. Availability, allowed child modules, form validation and permissions still
come from the normal CMS workflow. See [instructions for all CMS elements](cms-bundle/configuration.md#ai-instructions).
