---
title: "CMS Bundle Troubleshooting"
description: "Fix common CMS Bundle issues such as MySQL memory errors and silent failures related to ESI blocks and HTTP cache."
---

# Troubleshooting {#troubleshooting}

## MySQL memory allocation error {#mysql-memory-allocation-error}

In some situations, Doctrine can dispatch a "Doctrine\DBAL\Exception\DriverException" exception with an "out of sort memory" message.

> SQLSTATE[HY001]: Memory allocation error: 1038 Out of sort memory, consider increasing server sort buffer size

This MySQL-specific error and its `sort_buffer_size` setting do not apply to PostgreSQL. For PostgreSQL connection or encoding issues, see [Configure MySQL or PostgreSQL](../../getting-started/configure-database.md).

This is a common error if database instance has a low value for [sort_buffer_size](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_sort_buffer_size) variable.

Inspect the failing query and its indexes before changing server memory settings. Check the active session value with:

```sql
SHOW SESSION VARIABLES LIKE 'sort_buffer_size';
```

MySQL allocates a sort buffer per sorting session, so a larger global value increases memory use under concurrency. Choose a value from measurements on your own workload and apply it through the database administrator or managed service configuration. See the [MySQL sort buffer reference](https://dev.mysql.com/doc/refman/8.4/en/server-system-variables.html#sysvar_sort_buffer_size).

## Silent fail using a lot of ESI blocks or menus and http cache {#silent-fail-using-a-lot-of-esi-blocks-or-menus-and-http-cache}

With many ESI fragments, the HTTP cache trace can produce a large response header. If your proxy rejects the response, check the application and proxy logs for header-size errors.

Disable diagnostic cache tracing to reduce the header size, then repeat the request:

<!-- config-tabs:start -->

```yaml
# config/packages/framework.yaml
framework:
    http_cache:
        trace_level: none
    esi: true
```

```php
<?php

// config/packages/framework.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('framework', ['http_cache' => ['trace_level' => 'none'], 'esi' => true]);
};
```

<!-- config-tabs:end -->

## After deploying a new version assets are not loaded {#after-deploying-a-new-version-assets-are-not-loaded}

SfsCms compiles contents, and stores them in database by default to be used later and improve performance.

Sometimes, after an assets build for deploy, you can experience that assets are not loaded. This is because the 
 content is compiled and stored in database, and it includes assets references.

**Re-publish solution**

One solution is to re-publish the content version in admin panel.

**Disable content compilation storing**

Another solution is to disable content compilation storing in general:

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_cms.yaml
sfs_cms:
    content:
        save_compiled: false
```

```php
<?php

// config/packages/sfs_cms.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_cms', ['content' => ['save_compiled' => false]]);
};
```

<!-- config-tabs:end -->

or for a specific layout:

```yaml
# cms/layouts/<your-layout>/config.yaml
layout:
    save_compiled: false
```

or for a specific content type:

```yaml
# cms/contents/<your-content-type>/config.yaml
content:
    save_compiled: false
```

**Prefix compiled contents by version**

The last solution is to prefix compiled contents by version, so when you deploy a new version, the content is not found
 and is compiled and stored again.

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_cms.yaml
sfs_cms:
    content:
        prefix_compiled: '%env(APP_VERSION)%/'
```

```php
<?php

// config/packages/sfs_cms.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_cms', ['content' => ['prefix_compiled' => '%env(APP_VERSION)%/']]);
};
```

<!-- config-tabs:end -->

## Some site or locale content render has not styles or scripts {#some-site-or-locale-content-render-has-not-styles-or-scripts}

When a content is saved or republished, the content is compiled and stored in database. This content includes styles and scripts references.

If you use webpack encore, when an asset reference is rendered in a request, it won't be rendered again in the same request. 

SfsCms manage the _default webpack entrypoint, and automatically reset the entrypoint when a content is rendered.

If you use a custom entrypoint, you need to reset the entry for the site or locale to force the assets to be recompiled before using it.

```twig
{{ sfs_cms_encore_entry_reset('blog') }}
{{ encore_entry_link_tags('blog-styles', null, 'blog') }}
```
