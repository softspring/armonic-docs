---
title: "Configure Medias"
description: "Configure media storage and media types used by Armonic CMS through the Media Bundle."
---

# Media Configuration (`config/packages/sfs_media.yaml`)

CMS media behavior is configured through **Media Bundle** (`sfs_media`), not through `cms/*`.

Main schema source:

- `media-bundle/src/DependencyInjection/Configuration.php`

## Minimal Practical Setup

```yaml
sfs_media:
    driver: filesystem
    filesystem:
        path: '%kernel.project_dir%/public/media'
        url: '/media'
    media:
        admin_controller: true
```

This is enough to store files and use media admin.

## Typical CMS Setup (with one media type)

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

## Key Sections

- `driver`: storage driver (`filesystem` or `google_cloud_storage`)
- `filesystem` / `google_cloud_storage`: driver-specific settings
- `media`: media entity/admin behavior (`class`, `find_field_name`, `admin_controller`)
- `version`: media version entity behavior (`class`, `find_field_name`)
- `types`: media type definitions used by CMS forms and rendering

### About `versions` (important)

`versions` are derived files generated from the original uploaded media (`_original`).

- You upload one original file.
- The bundle generates all configured versions (sizes/formats) for that media type.
- Those versions are then used in templates, pictures, thumbnails, and responsive rendering.

Typical use cases:

- `_thumbnail` for admin listings
- `sm`, `md`, `xl` for responsive front-end output

If a version includes its own `upload_requirements`, that version is expected as an uploaded version instead of being auto-generated.

### `types.<id>` common keys

- `type` (`image` or `video`, default `image`)
- `name`
- `private` (default `false`)
- `description`
- `generator` (filename generator class)
- `upload_requirements`
- `versions`
- `pictures` (for `<picture>` rendering)
- `video_sets` (for video rendering sets)

## Required vs Optional

- At root level, most keys are optional due to defaults.
- In real projects, you should at least define storage (`driver` + config block).
- `types` is optional technically, but practically required if editors must upload and use specific media definitions.
- Inside `types`, you should define at least a `name`; other fields depend on your rendering needs.

## Important Validation Notes

- If `driver: filesystem`, the `filesystem` block must exist.
- If `driver: google_cloud_storage`, the `google_cloud_storage` block must exist.
- Configured mime types and version output formats are validated against your PHP/image library support.
- Some formats (for example `avif`) require environment support; invalid combinations fail during config validation.

## Related Docs

- `bundles/media-bundle/media-types.md` for detailed `types`, `versions`, and `pictures` examples.
