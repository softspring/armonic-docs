---
title: "Configure Media Types"
description: "Define media types, upload requirements, generated versions, pictures, and video sets in Media Bundle."
---

# Configure Media Types {#configure-media-types}

The most important part of `media-bundle` is `sfs_media.types`. Media behavior is not hard-coded in PHP classes. It is driven by type configuration.

## What A Media Type Defines {#what-a-media-type-defines}

A media type decides:

- whether the media is image or video
- whether it is private
- which uploads are allowed
- which versions exist
- how pictures are rendered
- how video sets are rendered
- which name generator to use

## Example Image Type {#example-image-type}

```yaml
sfs_media:
    types:
        article_image:
            type: image
            name: 'Article image'
            description: 'Responsive images used inside article pages'
            private: false
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
            pictures:
                default:
                    sources:
                        - srcset:
                              - { version: card, suffix: '480w' }
                              - { version: article, suffix: '1280w' }
                          attrs:
                              sizes: '100vw'
                    img:
                        src_version: article
```

## Most Important Fields {#most-important-fields}

- `type`
  - `image` or `video`
- `name`
- `description`
- `private`
- `generator`
- `upload_requirements`
- `versions`
- `pictures`
- `video_sets`

## Upload Requirements {#upload-requirements}

Supported upload requirements include:

- `minWidth`
- `minHeight`
- `maxWidth`
- `maxHeight`
- `minRatio`
- `maxRatio`
- `minPixels`
- `maxPixels`
- `allowSquare`
- `allowLandscape`
- `allowPortrait`
- `detectCorrupted`
- `mimeTypes`
- `maxSize`
- `binaryFormat`

The configuration layer also validates whether the configured mime types and output formats are supported by the current PHP and GD environment.

## Generated Versions {#generated-versions}

Generated versions do not define their own `upload_requirements`.

They are created from another version using options such as:

- `type`
- `from`
- `scale_width`
- `scale_height`
- `png_compression_level`
- `webp_quality`
- `jpeg_quality`
- `avif_quality`
- `flatten`

If `type` is omitted for image versions, the bundle normalizes it to `keep`.

## Uploaded Versions {#uploaded-versions}

If a version defines `upload_requirements`, it becomes a manual upload field instead of an auto-generated version.

Use that for cases such as:

- manually exported video encodings
- poster images provided separately
- externally generated files

## Pictures And Video Sets {#pictures-and-video-sets}

Use `pictures` for responsive image rendering and `video_sets` for multi-source video rendering.

These blocks are what the Twig renderer uses to build `<picture>` and `<video>` structures.

## APNG, WebP, And AVIF {#apng-webp-and-avif}

The bundle has explicit support for more specialized formats such as APNG, WebP, and AVIF, but only when the current PHP environment supports them.

That is why unsupported configurations fail at container compilation time instead of failing later during uploads.

## Related Guides {#related-guides}

- [Media Bundle Overview](../media-bundle.md)
- [Install](./install.md)
- [Getting Started](./getting-started.md)
- [Using Medias](./using-medias.md)
- [Name Generators](./name-generators.md)
