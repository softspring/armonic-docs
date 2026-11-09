---
title: "CMS content versions and publication"
description: "Understand drafts, preview, publication, compilation, and shared content when editing a CMS page."
related: ["preview-and-publish-a-page", "components/cache"]
---

# Content versions and publication {#content-versions-and-publication}

A content record identifies a page across its editing history. Its versions store the layout and module data at a particular point in time. Saving a new version does not automatically make that version public.

## Follow a change from draft to publication {#follow-a-change-from-draft-to-publication}

1. Open a page and edit its content.
2. Save a version. This is a draft until it is published.
3. Preview the saved version in each enabled language and relevant site.
4. Publish that version. Public rendering uses the published version.
5. Open the public route in a separate browser session to verify the visitor result.

A route and a published version solve different problems: the route makes the page reachable, while publication chooses which saved content visitors receive. A working admin preview does not prove either one.

## Return to earlier content {#return-to-earlier-content}

Use version history to inspect and preview an older version. Publish the intended version or create a new draft from it, depending on whether more changes are needed. Keep versions that must survive cleanup.

Shared blocks, menus, and media are separate records. Selecting an old page version does not restore the historical state of every shared record it references. A reusable section also has its own publication history.

## Understand compiled output {#understand-compiled-output}

The CMS can store rendered HTML for a content version. This avoids repeating rendering work, but it means a Twig or asset change may not appear in HTML already compiled and stored.

The following application configuration compiles on publication while avoiding compilation on every draft save:
<!-- config-tabs:start -->

```yaml
# config/packages/sfs_cms.yaml
sfs_cms:
    content:
        save_compiled: true
        autocompile_on_save: false
        autocompile_on_publish: true
        recompile: true
```

```php
<?php

// config/packages/sfs_cms.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_cms', [
        'content' => [
            'save_compiled' => true,
            'autocompile_on_save' => false,
            'autocompile_on_publish' => true,
            'recompile' => true,
        ],
    ]);
};
```

<!-- config-tabs:end -->

Content and layout definitions can override compilation behavior. After changing a render template, recompile the affected published versions through the admin. Symfony's cache clear does not by itself replace stored compiled HTML.

`prefix_compiled` can distinguish application releases. It does not migrate module data or repair an invalid template. See [Cache configuration](../../components/cache.md) when combining compiled output with HTTP caching.

## Evolve saved data {#evolve-saved-data}

A module revision records its data format. Add a migration when renaming or restructuring saved fields; increasing `revision` alone does not transform existing versions. Test the migration with old saved data before publishing the updated module. See [Module data migrations](modules.md#migrate-stored-values).
