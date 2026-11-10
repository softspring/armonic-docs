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

## Handle concurrent compilation {#concurrent-compilation}

When several visitors request the same uncached version, the CMS can compile its HTML at runtime. This cache fill does not save the editorial version or update the parent page's last-modified timestamp. A short database transaction checks the invalidation generation before storing the result. HTML compiled before a committed invalidation cannot repopulate the new generation's cache.

This works without configuring locks or Redis. Concurrent requests may repeat the same rendering work, but database serialization and uniqueness protect each version, context and generation. Apply the [compiled-cache migrations](../../UPGRADE-6.0.md#compiled-cache-migrations) before using the updated code.

### Share compilation locks between instances {#shared-compilation-locks}

Configure a named Symfony lock factory to reduce duplicate rendering across PHP processes and application instances. Use a Redis store shared by every instance of the application. Merge this named resource into the existing lock configuration, preserving other resources and their default store:

<!-- config-tabs:start -->

```yaml
# config/packages/lock.yaml
framework:
    lock:
        resources:
            sfs_cms_compile: '%env(CMS_COMPILE_LOCK_DSN)%'
```

```php
<?php

// config/packages/lock.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('framework', [
        'lock' => [
            'resources' => [
                'sfs_cms_compile' => '%env(CMS_COMPILE_LOCK_DSN)%',
            ],
        ],
    ]);
};
```

<!-- config-tabs:end -->

Set `CMS_COMPILE_LOCK_DSN` through the application's environment configuration, for example `redis://redis:6379` in a local container network. Choose the shared service address for a deployment; separate per-instance Redis servers or filesystem locks do not coordinate multiple instances. The bundle installs the `symfony/lock` dependency and consumes `lock.sfs_cms_compile.factory` when it exists.

A cache hit does not acquire a compilation lock. A competing request waits for at most two seconds, then renders without writing the compiled cache. Nested compilations do not wait for other compilation locks. If Redis fails, the visitor can still receive rendered HTML. A lost or expired lease prevents that request from writing the cache; a failed release expires with the lease's 300-second TTL.

Locks reduce duplicated work. They do not replace database transactions or the generation check used during invalidation. The [Sections Plugin](sections.md#concurrent-section-compilation) uses the same coordinator.

### Keep invalidation and extension code consistent {#compiled-cache-extensions}

Use the CMS recompile and clear-compiled actions to invalidate stored output. `cleanCompiled()` rotates the version's generation token; persist that change with the editorial operation. A custom invalidator using SQL must also rotate the affected version tokens and evict affected Doctrine second-level entity and collection caches. Clearing Symfony's container cache or deleting HTML rows alone is not equivalent to this invalidation. HTTP response caches are a separate layer and must also be invalidated when necessary.

Runtime compilation skips shared compiled-cache access while the caller owns a database transaction, and skips storage for unsaved versions and query-dependent requests. This keeps uncommitted or request-specific HTML out of the shared compiled cache. Template errors remain visible and are not saved as successful cache entries.

Applications needing custom cache persistence can replace `Softspring\CmsBundle\Cache\CompiledDataStoreInterface`. Its `find()`, `save()` and `isTransactionActive()` methods must preserve generation checks and caller-owned transaction boundaries. Cache persistence must not flush, clear or close the rendering EntityManager. The default store uses the configured Doctrine connection and the bundle's single-table compiled-data mapping with string version identifiers.

## Evolve saved data {#evolve-saved-data}

A module revision records its data format. Add a migration when renaming or restructuring saved fields; increasing `revision` alone does not transform existing versions. Test the migration with old saved data before publishing the updated module. See [Module data migrations](modules.md#migrate-stored-values).
