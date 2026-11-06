---
title: "HTTP Cache Store Bundle"
description: "Replace Symfony HttpCache's default store with one backed by a configurable Symfony cache pool."
---

# HTTP Cache Store Bundle {#http-cache-store-bundle}

`softspring/http-cache-store-bundle` lets Symfony HttpCache store its data in a Symfony cache pool.

Use it when you want your HttpCache data to live in:

- `cache.app`
- a dedicated Redis pool
- a Memcached pool
- another Symfony cache backend already used by your project

This is mainly a practical infrastructure bundle. Its value is not adding new HTTP cache rules, but making storage easier to control, isolate, and operate.

For the complete project-level setup, including Symfony, Doctrine, CMS, and deployment-specific cache choices, see [Cache configuration](cache.md).

## When To Use It {#when-to-use-it}

This bundle is a good fit when:

- you already use Symfony HttpCache
- you want cache data outside the default store
- you want to isolate reverse proxy cache data in its own backend
- you want easier operational control through Symfony cache pools

Typical reasons to adopt it:

- move HttpCache storage to Redis
- separate HttpCache data from application cache
- clear HttpCache data through one known cache pool
- log cache hits and misses more easily

## Installation {#installation}

```bash
composer require softspring/http-cache-store-bundle:^6.0
```

## Quick Start {#quick-start}

The normal setup is simple:

1. enable Symfony HttpCache
2. install the bundle
3. keep the default `cache.app` adapter or point the bundle to a dedicated pool
4. optionally configure a logger

For many projects, that is enough.

## Enable Symfony HttpCache {#enable-symfony-httpcache}

This bundle only matters if Symfony HttpCache is enabled:

<!-- config-tabs:start -->

```yaml
# config/packages/framework.yaml
framework:
    http_cache:
        enabled: true
```

```php
<?php

// config/packages/framework.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('framework', ['http_cache' => ['enabled' => true]]);
};
```

<!-- config-tabs:end -->

If your project does not use Symfony HttpCache, this bundle has nothing to replace.

## Default Configuration {#default-configuration}

By default, the bundle uses:

- `cache.app` as adapter
- no logger

That means the minimum configuration is often no extra configuration at all.

## Main Options {#main-options}

The bundle exposes two practical options:

<!-- config-tabs:start -->

```yaml
sfs_http_cache_store:
    adapter: 'cache.app'
    logger: null
```

```php
<?php

// config/packages/framework.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_http_cache_store', ['adapter' => 'cache.app', 'logger' => null]);
};
```

<!-- config-tabs:end -->

- `adapter`: the cache pool service id used to store HttpCache data
- `logger`: an optional logger service id used for hit, miss, and write logs

## Use A Dedicated Cache Pool {#use-a-dedicated-cache-pool}

Use a dedicated persistent pool when Symfony HttpCache is responsible for shared response storage.

Example:

<!-- config-tabs:start -->

```yaml
# config/packages/cache.yaml
framework:
    cache:
        pools:
            app.http_cache_pool:
                adapter: cache.adapter.redis
                provider: '%env(REDIS_URL)%'

sfs_http_cache_store:
    adapter: app.http_cache_pool
```

```php
<?php

// config/packages/cache.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('framework', [
        'cache' => [
            'pools' => [
                'app.http_cache_pool' => ['adapter' => 'cache.adapter.redis', 'provider' => '%env(REDIS_URL)%'],
            ],
        ],
    ]);

    $container->extension('sfs_http_cache_store', ['adapter' => 'app.http_cache_pool']);
};
```

<!-- config-tabs:end -->

Set `REDIS_URL`, for example `redis://127.0.0.1:6379`, and install a Redis client supported by Symfony Cache. The pool can remain private because the compiler pass references it directly. Confirm that a Redis restart or a pool clear has the expected effect before relying on this store across application instances.

This is useful when you want:

- isolated storage
- easier cleanup
- a backend tuned for cache traffic
- fewer surprises when clearing cache pools

## Keep It On cache.app When {#keep-it-on-cache-app-when}

Keeping the default `cache.app` setup is fine when:

- the project is small
- HttpCache volume is low
- you do not need separate operations for reverse proxy data
- you just want the bundle without extra infrastructure work

That is often enough for development, staging, or smaller deployments.

## Configure Logging {#configure-logging}

You can point the bundle to a logger service:

<!-- config-tabs:start -->

```yaml
sfs_http_cache_store:
    logger: 'monolog.logger.http_cache'
```

```php
<?php

// config/packages/sfs_http_cache_store.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_http_cache_store', ['logger' => 'monolog.logger.http_cache']);
};
```

<!-- config-tabs:end -->

A dedicated Monolog channel is usually the cleanest option:

<!-- config-tabs:start -->

```yaml
monolog:
    channels: ['http_cache']

sfs_http_cache_store:
    logger: 'monolog.logger.http_cache'
```

```php
<?php

// config/packages/monolog.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('monolog', ['channels' => ['http_cache']]);

    $container->extension('sfs_http_cache_store', ['logger' => 'monolog.logger.http_cache']);
};
```

<!-- config-tabs:end -->

This helps when you want to understand:

- cache hits
- cache misses
- cache writes
- fragment cache activity
- `Vary`-related differences between requests

## Typical Usage Pattern {#typical-usage-pattern}

Choose the store from the deployment topology:

1. enable Symfony HttpCache when the application needs HTTP caching or ESI processing
2. use a dedicated Redis or Memcached-backed pool when Symfony owns shared response storage
3. use a non-persistent array adapter when an upstream CDN owns shared caching and Symfony only processes ESI
4. configure a dedicated logger channel when cache activity needs to be inspected

This avoids adding shared cache infrastructure where it is not needed while keeping persistent storage available when Symfony is responsible for it.

## Clear The Cache Store {#clear-the-cache-store}

When you use a dedicated pool, clearing the store is straightforward:

```bash
bin/console cache:pool:clear app.http_cache_pool
```

This is one of the main practical benefits of the bundle.

If you use `cache.app`, clearing that pool may also affect unrelated application cache entries, so a dedicated pool is usually safer.

## Working With Vary {#working-with-vary}

The bundle supports normal Symfony HttpCache behavior around `Vary`.

That means you can use it safely with responses that vary by headers such as:

- `Accept-Language`
- `Accept-Encoding`
- other request headers used by your application

From an application point of view, the important part is simple:

- keep using `Vary` correctly in your responses
- the bundle will keep distinct variants in the configured store

## Working With Fragments {#working-with-fragments}

The bundle can log extra information for fragment cache requests.

This is mostly useful for debugging pages with ESI or fragment-heavy rendering, where cache behavior is harder to follow from route-level logs alone.

You do not need special configuration beyond enabling a logger.

## Recommended Project Decisions {#recommended-project-decisions}

If you are deciding how to use the bundle, these are the choices that matter most:

### Symfony Owns Shared Response Storage {#symfony-owns-shared-response-storage}

- dedicated cache pool
- dedicated logger channel
- backend suited for shared cache traffic

### An Upstream CDN Owns Shared Response Storage {#an-upstream-cdn-owns-shared-response-storage}

- non-persistent array adapter for ESI processing
- dedicated logger channel when cache diagnostics are useful
- no duplicated persistent response cache inside the application

### Small Or Single-Instance Projects {#small-or-single-instance-projects}

- keep `cache.app`
- skip custom logger until needed

### Best default for debugging cache behavior {#best-default-for-debugging-cache-behavior}

- dedicated logger channel
- inspect hits, misses, and writes while testing vary and fragments

## Limits And Notes {#limits-and-notes}

The most important practical limits are:

- the bundle only applies when Symfony HttpCache is enabled
- it changes storage, not the caching strategy itself
- the adapter must match whether Symfony or an upstream proxy owns shared response storage
- purging one URL removes the main cached entry for that URL, but storage cleanup is still driven by the cache backend and TTLs
