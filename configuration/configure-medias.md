---
title: "Configure Medias"
description: "Configure media storage and media types used by Armonic CMS through the Media Bundle."
---

# Media Configuration (`config/packages/sfs_media.yaml`) {#media-configuration-config-packages-sfs-media-yaml}

CMS media behavior is configured through **Media Bundle** (`sfs_media`), not through `cms/*`.

Media types control accepted uploads, generated sizes, and frontend rendering. Define storage first, then the types your editors need. Admin routes and permissions must also be enabled; see [Install Media Bundle](../bundles/media-bundle/install.md).

## Minimal Practical Setup {#minimal-practical-setup}

<!-- config-tabs:start -->

```yaml
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
    $container->extension('sfs_media', [
        'driver' => 'filesystem',
        'filesystem' => ['path' => '%kernel.project_dir%/public/media', 'url' => '/media'],
        'media' => ['admin_controller' => true],
    ]);
};
```

<!-- config-tabs:end -->

This selects local filesystem storage and enables the admin controller services. The upload directory must be writable by PHP. Configure the admin routes and permissions before opening the screens.

## Typical CMS Setup (with one media type) {#typical-cms-setup-with-one-media-type}

<!-- config-tabs:start -->

```yaml
sfs_media:
    driver: filesystem
    filesystem:
        path: '%kernel.project_dir%/public/media'
        url: '/media'
    types:
        background:
            type: image
            name: 'Background image'
            imagine_driver: gd
            upload_requirements:
                minWidth: 1280
                minHeight: 450
                mimeTypes: ['image/png', 'image/jpeg']
            versions:
                _thumbnail: { type: 'jpeg', scale_width: 300, jpeg_quality: 90 }
                xl: { type: 'jpeg', scale_width: 1280, jpeg_quality: 90 }
            pictures:
                _default:
                    img:
                        src_version: xl
```

```php
<?php

// config/packages/sfs_media.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_media', [
        'driver' => 'filesystem',
        'filesystem' => ['path' => '%kernel.project_dir%/public/media', 'url' => '/media'],
        'types' => [
            'background' => [
                'type' => 'image',
                'name' => 'Background image',
                'imagine_driver' => 'gd',
                'upload_requirements' => ['minWidth' => 1280, 'minHeight' => 450, 'mimeTypes' => ['image/png', 'image/jpeg']],
                'versions' => [
                    '_thumbnail' => ['type' => 'jpeg', 'scale_width' => 300, 'jpeg_quality' => 90],
                    'xl' => ['type' => 'jpeg', 'scale_width' => 1280, 'jpeg_quality' => 90],
                ],
                'pictures' => ['_default' => ['img' => ['src_version' => 'xl']]],
            ],
        ],
    ]);
};
```

<!-- config-tabs:end -->

The example uses GD for JPEG generation, matching the required `ext-gd` extension. The bundle's default image driver is Imagick; install `ext-imagick` and the required codecs before using that driver instead.

To render the configured picture in Twig:

```twig
{{ media|sfs_media_render_picture('_default', {'class': 'img-fluid'}) }}
```

Here `media` is a saved record of the `background` type. After adding or changing generated versions, regenerate existing files through the media admin; changing configuration alone does not recreate stored images.

## Key Sections {#key-sections}

- `driver`: storage driver (`filesystem` or `google_cloud_storage`)
- `filesystem` / `google_cloud_storage`: driver-specific settings
- `media`: media entity/admin behavior (`class`, `find_field_name`, `admin_controller`)
- `version`: media version entity behavior (`class`, `find_field_name`)
- `types`: media type definitions used by CMS forms and rendering

### About `versions` (important) {#about-versions-important}

`versions` are derived files generated from the original uploaded media (`_original`).

- You upload one original file.
- The bundle generates all configured versions (sizes/formats) for that media type.
- Those versions are then used in templates, pictures, thumbnails, and responsive rendering.

Typical use cases:

- `_thumbnail` for admin listings
- `sm`, `md`, `xl` for responsive front-end output

If a version includes its own `upload_requirements`, that version is expected as an uploaded version instead of being auto-generated.

### `types.<id>` common keys {#types-id-common-keys}

- `type` (`image` or `video`, default `image`)
- `name`
- `private` (default `false`)
- `description`
- `generator` (filename generator class)
- `upload_requirements`
- `versions`
- `pictures` (for `<picture>` rendering)
- `video_sets` (for video rendering sets)

## Required vs Optional {#required-vs-optional}

- At root level, most keys are optional due to defaults.
- In real projects, you should at least define storage (`driver` + config block).
- `types` is optional technically, but practically required if editors must upload and use specific media definitions.
- Inside `types`, you should define at least a `name`; other fields depend on your rendering needs.

## Important Validation Notes {#important-validation-notes}

- If `driver: filesystem`, the `filesystem` block must exist.
- If `driver: google_cloud_storage`, the `google_cloud_storage` block must exist.
- Configured mime types and version output formats are validated against your PHP/image library support.
- Some formats (for example `avif`) require environment support; invalid combinations fail during config validation.

## Related Docs {#related-docs}

- [Media Bundle guide](../bundles/media-bundle.md) for `types`, `versions`, `pictures`, rendering, and admin examples.
