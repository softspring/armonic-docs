---
title: "CMS Blog Plugin"
description: "Add articles, authors, tags, related article lists, and editorial metadata to Armonic CMS."
related: ["bundles/cms-bundle/forms-and-metadata", "bundles/cms-bundle/content-versions"]
---

# CMS Blog Plugin {#cms-blog-plugin}

The blog plugin adds an `article` content type to the CMS, with authors, publication dates, tags, and article-list modules. Articles still use the CMS version, layout, route, and publication workflow.

## Install and register {#install-and-register}

```bash
composer require softspring/cms-blog-plugin:6.1.*@dev
```

Register the bundle when Flex does not do it automatically:

```php
// config/bundles.php
Softspring\CmsBlogPlugin\SfsCmsBlogPlugin::class => ['all' => true],
```

## Configure an author {#configure-an-author}

The author class is required:
<!-- config-tabs:start -->

```yaml
# config/packages/sfs_cms_blog.yaml
sfs_cms_blog:
    author:
        class: App\Entity\User
```

```php
<?php

// config/packages/sfs_cms_blog.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_cms_blog', ['author' => ['class' => \App\Entity\User::class]]);
};
```

<!-- config-tabs:end -->

The configured Doctrine entity must implement `Softspring\CmsBlogPlugin\Model\AuthorInterface`, whose methods are `getDisplayName(): string` and `getAvatarUrl(): string`. Implement both on the chosen user or author entity; merely having a user identifier is insufficient.

The plugin contributes its CMS collection and migration namespace automatically. Run the installed migrations and review any application entity changes before creating articles. The default article definition selects an `article` layout; provide or override that layout in the application rather than assuming the plugin supplies the site's final article design.

## Import article administration {#import-article-administration}

<!-- config-tabs:start -->

```yaml
# config/routes/sfs_cms_blog.yaml
_sfs_cms_blog:
    resource: '@SfsCmsBlogPlugin/config/routing/admin_articles.yaml'
    prefix: /admin/cms
```

```php
<?php

// config/routes/sfs_cms_blog.php
namespace Symfony\Component\Routing\Loader\Configurator;

return static function (RoutingConfigurator $routes): void {
    $routes->import('@SfsCmsBlogPlugin/config/routing/admin_articles.yaml')->prefix('/admin/cms');
};
```

<!-- config-tabs:end -->

With this import, articles are managed under `/admin/cms/articles`. Reuse the CMS content role hierarchy and keep the area behind the application's admin firewall. Add the `article` identifier to the site's `allowed_content_types`.

## Add editorial metadata {#add-editorial-metadata}

Extend `cms/contents/article/config.yaml` with application fields such as a summary, reading time, or a restricted card image. Use `extra_fields` for JSON metadata and mapped entity associations for relational data. See [Forms and metadata](cms-bundle/forms-and-metadata.md).

Define the frontend layout and module templates, create an article, set its author and tags, save a draft, add the public route, preview, and publish. The publication date and a published version are separate inputs used by article listings.

## Show related articles {#show-related-articles}

```twig
{% for article in sfs_cms_related_blog_articles('guides', currentArticle, 3) %}
    <p>{{ article.name }}</p>
{% endfor %}
```

`currentArticle` is the current article object passed by the application. The helper excludes it, selects published articles whose publication time has arrived, filters by the current request locale and tag, and limits the result. The default order is `published_at_desc`; other supported values are `published_at_asc`, `name_asc`, and `name_desc`.

The collection also includes `article_list` and `article_link_list` modules. Check their fields and preview templates before adapting them to a different design.

## Database integration {#database-integration}

CMS core supports MySQL and PostgreSQL, but this plugin also registers MySQL/MariaDB JSON DQL functions. Validate article filters and custom queries on PostgreSQL before claiming the whole blog integration is portable; a working core database connection does not prove every plugin query works on that engine.
