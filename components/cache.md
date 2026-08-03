---
title: "Cache Configuration"
description: "Configure application, system, Doctrine, HTTP, and CMS caches for an Armonic project."
---

# Cache Configuration {#cache-configuration}

An Armonic project uses several cache layers. Each layer has a different lifetime and must be configured for a specific purpose.

A practical setup separates:

- application and system caches managed by Symfony
- Doctrine metadata and query caches
- HTTP response storage and ESI processing
- CMS response caching and compiled content

Do not use a local cache for mutable data that must be immediately consistent across application instances.

## Application And System Caches {#application-and-system-caches}

Start with explicit adapters for `cache.app` and `cache.system`:

```yaml
# config/packages/cache.yaml
framework:
    cache:
        prefix_seed: 'app_%env(APP_VERSION)%'
        app: cache.adapter.filesystem
        system: cache.adapter.apcu
```

This setup gives each cache a clear role:

- `cache.app` stores application data on the local filesystem
- `cache.system` stores framework and container-related data in local APCu memory
- `prefix_seed` creates stable namespaces and separates cache keys between application versions

`APP_VERSION` is a project environment variable. Set it to a value that changes with each deployment when cached data must not be reused by the new version.

Changing the prefix prevents the application from reading entries created by an older version. It does not necessarily delete those entries from the backend, so long-lived cache storage may still need pruning.

### Local Cache Boundaries {#local-cache-boundaries}

Filesystem and APCu adapters are local to the application instance. They are a good fit for derived or disposable values that can be rebuilt.

Do not use them as the source of truth for:

- locks shared by several instances
- counters that must be globally accurate
- mutable state that must be visible immediately on every instance
- workflow or business data

Use the database or a shared cache backend for those cases.

## Doctrine Caches {#doctrine-caches}

Doctrine metadata and parsed queries can reuse Symfony's system cache:

```yaml
# config/packages/doctrine_cache.yaml
framework:
    cache:
        pools:
            doctrine.system_cache_pool:
                adapter: cache.system

doctrine:
    orm:
        query_cache_driver:
            type: pool
            pool: doctrine.system_cache_pool
        metadata_cache_driver:
            type: pool
            pool: doctrine.system_cache_pool
```

This avoids creating a separate backend for data that is derived from code and mapping configuration.

The pool above stores query parsing and metadata information. It does not cache query results or Doctrine entities.

## HTTP Cache And ESI {#http-cache-and-esi}

When a CDN or another reverse proxy is the shared HTTP cache, Symfony HttpCache can remain enabled only for ESI processing. In that deployment model, an in-memory store is enough:

```yaml
# config/packages/http_cache.yaml
services:
    http_cache.store.adapter:
        parent: 'cache.adapter.array'
        public: true

framework:
    http_cache:
        enabled: true
        private_headers: ['Authorization']
        trace_level: none

monolog:
    channels: ['http_cache']

sfs_http_cache_store:
    adapter: 'http_cache.store.adapter'
    logger: 'monolog.logger.http_cache'
```

This configuration has deliberate limits:

- `cache.adapter.array` is process-local and non-persistent
- restarting the process removes its stored responses
- the upstream CDN or reverse proxy remains responsible for shared response caching
- `trace_level: none` avoids HttpCache trace information when it is not needed
- the dedicated logger channel keeps cache activity identifiable in normal application logs

Use this model only when persistent HTTP response storage already exists outside the Symfony process.

### Cookie And Authorization Safety {#cookie-and-authorization-safety}

Setting `private_headers` to only `Authorization` means that a request containing cookies is not automatically treated as private by that rule.

Use this configuration only when cookie-bearing responses are safe to cache and the application sets `Cache-Control`, `Vary`, and private response rules correctly. Personalized responses must never enter a shared public cache.

### When Symfony Is The Shared HTTP Cache {#when-symfony-is-the-shared-http-cache}

If Symfony HttpCache is responsible for storing shared responses, do not use `cache.adapter.array`. Configure a persistent pool that matches the deployment topology instead.

See [HTTP cache store bundle](http-cache-store-bundle.md) for filesystem, Redis, Memcached, and dedicated pool options.

## CMS Cache Strategy {#cms-cache-strategy}

The CMS can use TTL-based HTTP caching and versioned compiled content:

```yaml
# config/packages/sfs_cms.yaml
sfs_cms:
    cache:
        type: 'ttl'

    content:
        save_compiled: true
        recompile: false
        prefix_compiled: '%env(ENVIRONMENT)%/%env(APP_VERSION)%/'
```

The options solve two different problems:

- `cache.type: ttl` selects the CMS response cache strategy
- `save_compiled: true` stores compiled content for reuse
- `prefix_compiled` separates compiled content by environment and application version
- `recompile: false` disables the manual content recompilation feature

`ENVIRONMENT` and `APP_VERSION` are project environment variables. A versioned prefix prevents a deployment from reusing compiled content that contains references generated by an older asset build.

CMS response caching and compiled content are not the same storage layer. Clearing Symfony cache does not remove compiled CMS data stored in the database. Changing `prefix_compiled` makes the CMS use a new compile key instead.

## Recommended Baseline {#recommended-baseline}

For a container-based Armonic deployment:

1. use filesystem cache for disposable application values
2. use APCu for system and Doctrine metadata caches
3. include the application version in cache namespaces
4. use an in-memory HttpCache store only when a CDN or reverse proxy owns shared response caching
5. use versioned CMS compile keys when deployments can change generated asset references
6. keep shared mutable state in a shared backend or the database

## Clearing The Right Cache {#clearing-the-right-cache}

Clear Symfony application and system caches with the normal environment-aware command:

```bash
bin/console cache:clear --env=prod
```

Clear only the Doctrine system pool when that is the affected layer:

```bash
bin/console cache:pool:clear doctrine.system_cache_pool
```

An array-backed HttpCache store disappears with the process and does not need persistent cleanup. CMS compiled content requires its own lifecycle; use a new version prefix or the CMS recompilation flow rather than relying on `cache:clear`.

## Common Mistakes {#common-mistakes}

- treating local filesystem or APCu cache as shared application state
- using an array HttpCache adapter without an upstream shared cache
- changing `private_headers` without reviewing personalized responses
- assuming Doctrine's query cache contains query results
- expecting `cache:clear` to remove compiled CMS content from the database
- reusing one cache namespace across deployments with incompatible data or assets

## Related Pages {#related-pages}

- [HTTP cache store bundle](http-cache-store-bundle.md)
- [Logging](logging.md)
- [Configure blocks](../configuration/configure-blocks.md)
- [Configure menus](../configuration/configure-menu.md)
- [CMS troubleshooting](../bundles/cms-bundle/troubleshooting.md)
