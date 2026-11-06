---
title: "Install Media Bundle"
description: "Install and configure Media Bundle with the minimum setup needed to manage and render media files in Symfony."
---

# Install Media Bundle {#install-media-bundle}

`softspring/media-bundle` manages uploaded media as Doctrine entities, generates derived versions, renders images and videos in Twig, and can expose an admin media library.

## Install The Package {#install-the-package}

```bash
composer require softspring/media-bundle:6.0.x-dev
```

If your application does not use Symfony Flex, enable the bundle manually:

```php
<?php

return [
    // ...
    Softspring\MediaBundle\SfsMediaBundle::class => ['all' => true],
];
```

## Runtime Requirements {#runtime-requirements}

The current package requires:

- PHP `8.4` or higher
- Doctrine ORM
- PHP `gd`
- `imagine/imagine`
- `google/cloud-storage`

The default static image driver also needs `ext-imagick` and the codecs used by your sources and outputs. To use GD instead, set `sfs_media.imagine.driver: gd`. Animated processing needs FFmpeg and FFprobe; animated WebP decoding also needs Imagick. See [Static image processing driver](../media-bundle.md#static-image-processing-driver) and [Animated image versions](../media-bundle.md#animated-image-versions).

Even if you use filesystem storage only, `google/cloud-storage` is still a direct package dependency in the current line.

## Add Base Configuration {#add-base-configuration}

Start with a minimal filesystem setup:

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_media.yaml
imports:
    - { resource: '@SfsMediaBundle/config/security/admin_role_hierarchy.yaml' }

sfs_media:
    driver: filesystem
    filesystem:
        path: '%kernel.project_dir%/public/media'
        url: '/media'
    media:
        admin_controller: true
```

```php
<?php

// config/packages/sfs_media.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->import('@SfsMediaBundle/config/security/admin_role_hierarchy.yaml');

    $container->extension('sfs_media', [
        'driver' => 'filesystem',
        'filesystem' => ['path' => '%kernel.project_dir%/public/media', 'url' => '/media'],
        'media' => ['admin_controller' => true],
    ]);
};
```

<!-- config-tabs:end -->

This is enough to boot the bundle, but it is not enough to upload useful media yet. Real uploads start when you define at least one media type under `sfs_media.types`.

## Import Routes {#import-routes}

If you want the built-in admin media library, import the admin routes:

<!-- config-tabs:start -->

```yaml
# config/routes/sfs_media.yaml
_sfs_media_admin:
    resource: '@SfsMediaBundle/config/routing/admin_media.yaml'
    prefix: /admin/media
```

```php
<?php

// config/routes/sfs_media.php
namespace Symfony\Component\Routing\Loader\Configurator;

return static function (RoutingConfigurator $routes): void {
    $routes->import('@SfsMediaBundle/config/routing/admin_media.yaml')->prefix('/admin/media');
};
```

<!-- config-tabs:end -->

If you disable `media.admin_controller`, do not import those routes.

## Define The First Media Type {#define-the-first-media-type}

The bundle is configuration-driven. Before you can upload or render anything useful, define a media type:

<!-- config-tabs:start -->

```yaml
sfs_media:
    types:
        article_image:
            type: image
            name: 'Article image'
            upload_requirements:
                mimeTypes: ['image/jpeg', 'image/png', 'image/webp']
                minWidth: 1200
                minHeight: 675
            versions:
                card:
                    type: webp
                    scale_width: 480
                    webp_quality: 80
                article:
                    type: webp
                    scale_width: 1280
                    webp_quality: 85
```

```php
<?php

// config/routing/admin_media.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_media', [
        'types' => [
            'article_image' => [
                'type' => 'image',
                'name' => 'Article image',
                'upload_requirements' => [
                    'mimeTypes' => ['image/jpeg', 'image/png', 'image/webp'],
                    'minWidth' => 1200,
                    'minHeight' => 675,
                ],
                'versions' => [
                    'card' => ['type' => 'webp', 'scale_width' => 480, 'webp_quality' => 80],
                    'article' => ['type' => 'webp', 'scale_width' => 1280, 'webp_quality' => 85],
                ],
            ],
        ],
    ]);
};
```

<!-- config-tabs:end -->

Without type configuration, the bundle knows how to manage media entities, but it does not know what files, versions, or rendering variants your application expects.

## Practical First Milestone {#practical-first-milestone}

A good first milestone is:

1. install the bundle
2. configure filesystem storage
3. define one image type
4. upload one media with `MediaTypeUploadType`
5. render it in Twig

That is enough to validate the full path from upload to stored file to rendered output.

## Related Guides {#related-guides}

- [Media Bundle Overview](../media-bundle.md)
- [Getting Started](./getting-started.md)
- [Configure Media Types](./media-types.md)
- [Admin Medias](./admin-medias.md)
- [Storage Options](./storage-options.md)
