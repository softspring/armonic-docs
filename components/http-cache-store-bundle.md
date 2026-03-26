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

```yaml
# config/packages/framework.yaml
framework:
    http_cache:
        enabled: true
```

If your project does not use Symfony HttpCache, this bundle has nothing to replace.

## Default Configuration {#default-configuration}

By default, the bundle uses:

- `cache.app` as adapter
- no logger

That means the minimum configuration is often no extra configuration at all.

## Main Options {#main-options}

The bundle exposes two practical options:

```yaml
sfs_http_cache_store:
    adapter: 'cache.app'
    logger: null
```

- `adapter`: the cache pool service id used to store HttpCache data
- `logger`: an optional logger service id used for hit, miss, and write logs

## Use A Dedicated Cache Pool {#use-a-dedicated-cache-pool}

For real projects, a dedicated cache pool is usually the better setup.

Example:

```yaml
# config/services.yaml
services:
    app.http_cache_pool:
        parent: 'cache.adapter.redis'
        public: true
        tags:
            - { name: 'cache.pool', namespace: 'http_cache' }

# config/packages/sfs_http_cache_store.yaml
sfs_http_cache_store:
    adapter: 'app.http_cache_pool'
```

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

```yaml
sfs_http_cache_store:
    logger: 'monolog.logger.http_cache'
```

A dedicated Monolog channel is usually the cleanest option:

```yaml
monolog:
    channels: ['http_cache']

sfs_http_cache_store:
    logger: 'monolog.logger.http_cache'
```

This helps when you want to understand:

- cache hits
- cache misses
- cache writes
- fragment cache activity
- `Vary`-related differences between requests

## Typical Usage Pattern {#typical-usage-pattern}

For production projects, the usual recommendation is:

1. enable Symfony HttpCache
2. use a dedicated Redis or Memcached-backed pool
3. configure a dedicated logger channel
4. clear only that pool when you want to reset HttpCache data

That keeps reverse proxy cache storage separate from the rest of the application.

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

### Best default for serious projects {#best-default-for-serious-projects}

- dedicated cache pool
- dedicated logger channel
- backend suited for shared cache traffic

### Best default for small projects {#best-default-for-small-projects}

- keep `cache.app`
- skip custom logger until needed

### Best default for debugging cache behavior {#best-default-for-debugging-cache-behavior}

- dedicated logger channel
- inspect hits, misses, and writes while testing vary and fragments

## Limits And Notes {#limits-and-notes}

The most important practical limits are:

- the bundle only applies when Symfony HttpCache is enabled
- it changes storage, not the caching strategy itself
- a dedicated pool is recommended if you want clean operations
- purging one URL removes the main cached entry for that URL, but storage cleanup is still driven by the cache backend and TTLs

## Good Fit Summary {#good-fit-summary}

Choose this bundle when your goal is:

- better storage control for Symfony HttpCache
- easier operations through Symfony cache pools
- cleaner isolation of reverse proxy cache data
- better observability through logging

That is the core value of the bundle.
