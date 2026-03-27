---
title: "Getting Started With Media Bundle"
description: "Follow a practical first integration path for Media Bundle with one media type, one upload flow, and one rendered output."
---

# Getting Started {#getting-started}

The fastest way to understand `media-bundle` is to wire one real media type end to end.

## Recommended First Flow {#recommended-first-flow}

Use this order:

1. configure filesystem storage
2. define one image type
3. upload a media entry with `MediaTypeUploadType`
4. render one configured version in Twig
5. open the admin media library only after the basic flow works

This lets you validate the real model before you add more formats, providers, or admin customization.

## Example First Type {#example-first-type}

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

This gives you:

- one original upload
- one smaller card version
- one larger article version

## Example Upload Form {#example-upload-form}

```php
use Softspring\MediaBundle\Form\MediaTypeUploadType;

$builder->add('heroImage', MediaTypeUploadType::class, [
    'media_type' => 'article_image',
]);
```

The form will create the `_original` upload field and any extra upload fields required by versions that define `upload_requirements`.

## Example Twig Rendering {#example-twig-rendering}

```twig
{{ media|sfs_media_render_image('article') }}
```

or, when you configure a picture set:

```twig
{{ media|sfs_media_render_picture('default') }}
```

## When To Enable Admin Early {#when-to-enable-admin-early}

Enable the admin media library early when:

- editors need manual upload and browse screens
- you want a shared media library across several forms
- you need modal media selectors in backoffice screens

If the project only needs one or two controlled upload flows, start with custom forms first and add admin later.

## Common First Mistakes {#common-first-mistakes}

- installing the bundle but not defining any media type
- importing admin routes but forgetting the role hierarchy import
- expecting private media to come with signed URLs or delivery access control
- treating video support as a transcoding system instead of a managed versioning and rendering layer

## Related Guides {#related-guides}

- [Media Bundle Overview](../media-bundle.md)
- [Install](./install.md)
- [Configure Media Types](./media-types.md)
- [Using Medias](./using-medias.md)
- [Storage Options](./storage-options.md)
