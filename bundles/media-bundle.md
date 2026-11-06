---
title: "Media Bundle"
description: "Configure media types, upload and generate versions, render images and videos, and manage a media library with the Softspring Media Bundle."
---

# Media Bundle {#media-bundle}

The Media Bundle manages stored media files, generated versions, rendering helpers, and an optional admin media library.

It is designed around one idea: media behaviour is not hard-coded in PHP classes. It is defined by media types in configuration, and those types drive:

- what can be uploaded;
- which extra versions exist;
- which versions are generated or uploaded manually;
- how media is rendered in Twig;
- how admin create and update forms behave.

The bundle supports image and video media. Storage is delegated to a storage driver, and static image transformations use Imagine with Imagick by default or an explicitly selected GD driver. Animated images use FFmpeg.

## Documentation Map {#documentation-map}

Use this page as the full reference and start with the focused guides when you need to implement a specific part:

- [Install](./media-bundle/install.md)
  - package installation, base configuration, routes, and first required setup
- [Getting Started](./media-bundle/getting-started.md)
  - the shortest practical path to get uploads, versions, and rendering working
- [Concepts](./media-bundle/concepts.md)
  - media entities, versions, processing pipeline, migration, and duplicate detection
- [Configure Media Types](./media-bundle/media-types.md)
  - type config, upload rules, generated versions, pictures, and video sets
- [Using Medias](./media-bundle/using-medias.md)
  - Twig rendering, upload forms, selectors, and day-to-day usage patterns
- [Admin Medias](./media-bundle/admin-medias.md)
  - admin routes, permissions, list/create/update/read/delete, and migrate flows
- [Integrations](./media-bundle/integrations.md)
  - Doctrine migrations, CMS translation field caveats, and provider-based type configuration
- [Storage Options](./media-bundle/storage-options.md)
  - filesystem vs Google Cloud Storage and the implications of each
- [Name Generators](./media-bundle/name-generators.md)
  - how destination names are generated and how to replace that behavior
- [Extending Bundle](./media-bundle/extending-bundle.md)
  - type providers, processors, forms, listeners, templates, and extension points

If you are integrating the bundle for the first time, the usual reading order is: install, getting started, media types, using medias, then admin or extension guides depending on the project.

## Installation {#installation}

Install the bundle:

```bash
composer require softspring/media-bundle:^6.0
```

If your application does not use Symfony Flex, enable it manually:

```php
<?php

return [
    // ...
    Softspring\MediaBundle\SfsMediaBundle::class => ['all' => true],
];
```

## Requirements {#requirements}

The package requires:

- PHP `8.4` or higher
- Doctrine ORM
- PHP `gd`
- `imagine/imagine`
- `google/cloud-storage`

`google/cloud-storage` is a direct package requirement in the current codebase, even if your project only uses filesystem storage.

Install `ext-imagick` and the required ImageMagick codecs for the default static image driver, or select `imagine.driver: gd` explicitly. Animated processing requires FFmpeg and FFprobe. The configuration validator checks the selected driver and codecs before the container is built.

## Main Concepts {#main-concepts}

The bundle has three core runtime objects:

- `MediaInterface`
  - the logical media entry
  - stores type, metadata, privacy flag, SHA-1, and all versions
- `MediaVersionInterface`
  - one stored or generated version of a media
  - stores URL, mime type, dimensions, file size, timestamps, and processing options
- media type configuration
  - one configuration block under `sfs_media.types`
  - defines the business rules for that family of media

The bundle does not treat versions as simple strings. Every version is a Doctrine entity with its own metadata and storage URL.

## Data Model {#data-model}

By default, the bundle uses:

- `Softspring\MediaBundle\Entity\Media`
- `Softspring\MediaBundle\Entity\MediaVersion`

Those concrete entities extend these mapped model classes:

- `Softspring\MediaBundle\Model\Media`
- `Softspring\MediaBundle\Model\MediaVersion`

### Media Entity {#media-entity}

The media model stores:

- `id`
- `mediaType`
- `private`
- `type`
- `name`
- `description`
- `createdAt`
- `sha1`
- `altTexts`
- the collection of versions

The `type` field is the configured media type key such as `hero_image` or `homepage_video`.

The `mediaType` field is the normalized numeric category:

- image
- video
- unknown

### Media Version Entity {#media-version-entity}

Each version stores:

- `id`
- `media`
- `version`
- `url`
- `width`
- `height`
- `fileSize`
- `fileMimeType`
- `uploadedAt`
- `generatedAt`
- `options`
- `sha1`

`_original` is a special version name. It always represents the main uploaded file.

Generated versions also keep a non-persisted `originalVersion` reference while they are being processed.

### Replacing The Entity Classes {#replacing-the-entity-classes}

You can replace the default entity classes:

```yaml
# config/packages/sfs_media.yaml
sfs_media:
    media:
        class: App\Entity\Media
    version:
        class: App\Entity\MediaVersion
```

The bundle resolves `MediaInterface` and `MediaVersionInterface` to those configured classes through a compiler pass.

## Storage Drivers {#storage-drivers}

The bundle ships two storage drivers:

- `filesystem`
- `google_cloud_storage`

### Filesystem Driver {#filesystem-driver}

Example:

```yaml
sfs_media:
    driver: filesystem
    filesystem:
        path: '%kernel.project_dir%/public/media'
        url: '/media'
```

If you do not configure it explicitly, the bundle falls back to those same defaults.

The filesystem driver stores logical URLs with the internal prefix:

```text
sfs-media-filesystem://...
```

and can translate them into public URLs later.

### Google Cloud Storage Driver {#google-cloud-storage-driver}

Example:

```yaml
sfs_media:
    driver: google_cloud_storage
    google_cloud_storage:
        bucket: '%env(MEDIA_BUCKET_NAME)%'
```

The storage driver stores URLs internally as:

```text
gs://bucket/path/to/file
```

and exposes them publicly through the standard Google Storage HTTPS URL.

### Delayed Google Cloud Storage deletion {#delayed-google-cloud-storage-deletion}

Set `sfs_media.google_cloud_storage.delayed_deletion_days` to a positive integer to keep a removed object available at its existing URL until the configured delay has elapsed:

```yaml
sfs_media:
    google_cloud_storage:
        bucket: media-bucket
        delayed_deletion_days: 90
```

The driver sets the object's `Custom-Time` to the delayed deletion date. Configure the bucket with this Google Cloud Storage lifecycle rule, without a `matchesPrefix` condition:

```json
{
  "rule": [
    {
      "action": {"type": "Delete"},
      "condition": {"daysSinceCustomTime": 0}
    }
  ]
}
```

For example, save it as `lifecycle.json` and apply it with:

```bash
gcloud storage buckets update gs://media-bucket --lifecycle-file=lifecycle.json
```

Without that lifecycle rule, marked objects are not deleted automatically.

This rule only affects objects with `Custom-Time` set. Ensure that no other process using the same bucket sets `Custom-Time`, or its objects will also be eligible for deletion when that time is reached.

### Driver Responsibilities {#driver-responsibilities}

Both drivers implement the same four operations:

- `store()`
- `remove()`
- `download()`
- `url()`

That matters because generated versions may need to download the original file again during regeneration or migration.

## Media Types {#media-types}

Everything important in this bundle starts with `sfs_media.types`.

A media type defines:

- whether the media is an image or a video;
- whether it is private;
- which upload constraints apply;
- which derived versions exist;
- how pictures or video sets are rendered;
- which file name generator to use.

Example:

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

## Type Configuration Fields {#type-configuration-fields}

The most important fields are:

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

### Private Media {#private-media}

`private: true` marks the media entity as private at form submit time.

The bundle does not add a signed-URL system or an access-control layer for private assets by itself. It only persists the flag and uses it in some admin search flows.

### Name Generators {#name-generators}

The `generator` field points to a name generator service class.

The default generator is `Softspring\MediaBundle\Media\DefaultNameGenerator`.

It generates a path based on:

- the media id when available;
- the version name;
- the uploaded file extension.

You can plug another generator by implementing `NameGeneratorInterface`. Services implementing that interface are auto-tagged and collected by `NameGeneratorProvider`.

## Upload Requirements {#upload-requirements}

The bundle validates uploads through a custom `Image` constraint and `ImageValidator`.

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

### Mime Type Validation {#mime-type-validation}

The configuration layer already validates:

- that configured mime types are supported by the current PHP environment;
- that image types only allow image mime types;
- that video types only allow video mime types;
- that version output formats are supported by the selected image driver and its installed codecs.

This is especially relevant for:

- `webp`
- `avif`
- `apng`

If your environment cannot encode or decode one of those formats, container compilation fails with a configuration error instead of failing later at runtime.

### APNG Support {#apng-support}

The detector recognizes animation from file contents, including PNG/APNG uploads reported as `image/png`. Animated inputs use FFmpeg and select its APNG demuxer explicitly, including files with a separate default image. Static inputs use the configured Imagine driver. See [Animated image versions](#animated-image-versions) for policies, limits, and runtime requirements.

## Generation defaults {#generation-defaults}

Generated image versions inherit processing options from `sfs_media.defaults`
and then from their media type's `defaults` block. Explicit version options
have the highest priority. Nested animation settings are merged by key.
Only options relevant to the output format are passed to its processor.
Manual uploads and video types do not inherit image processing defaults.

```yaml
sfs_media:
    defaults:
        avif_quality: 85
        animation: { max_frames: 500, speed: 6 }
    types:
        content:
            defaults:
                avif_quality: 88
                animation: { speed: 5 }
            versions:
                normal: { type: avif } # Quality 88; max_frames 500; speed 5.
                detailed: { type: avif, avif_quality: 90 }
```

## Versions {#versions}

Every media always has an `_original` version.

Configured versions fall into two categories:

- generated versions
- uploaded versions

### Generated Versions {#generated-versions}

A generated version is a version without its own `upload_requirements`.

It is derived from another version, usually `_original`, using processing options such as:

- `type`
- `from`
- `scale_width`
- `scale_height`
- `png_compression_level`
- `webp_quality`
- `jpeg_quality`
- `avif_quality`
- `flatten`
- `resolution-x`
- `resolution-y`
- `resolution-units`
- `resampling-filter`

If `type` is omitted for image versions, the bundle normalizes it to `keep`.

If `resampling-filter` or `resolution-units` are omitted for generated image versions, the bundle fills them with defaults in `Configuration::fixConfigTypes()`.

### Uploaded Versions {#uploaded-versions}

A version with its own `upload_requirements` is not generated automatically from another file. It is expected to be uploaded explicitly.

This is how you model things such as:

- alternative manually exported formats;
- video poster images;
- video encodings that come from an external pipeline.

### Version Options In Database {#version-options-in-database}

When a version file is finally stored, `StoreFileProcessor` removes some configuration-only keys before persisting the version options:

- `upload_requirements`
- `from`

That is why migration compares actual stored version options against normalized type config instead of just comparing the raw config blocks directly.

## Static image processing driver {#static-image-processing-driver}

Static generated images use Imagick by default. Override the default globally,
per media type or per version (version > type > global):

```yaml
sfs_media:
    imagine:
        driver: imagick # imagick | gd
        allow_upscale: false
        resize_tolerance: 0.02
        preserve_original_encoding: true
    types:
        content:
            imagine_driver: gd
            versions:
                small:
                    type: webp # inherits gd
                    scale_width: 400
                large:
                    type: avif
                    imagine_driver: imagick
                    scale_width: 1200
```

Install `ext-imagick` and the ImageMagick codecs used by your sources and outputs.
On Alpine, AVIF support requires `imagemagick-heic` (libheif), in addition to the
PHP Imagick extension. Configuration checks the selected driver and its codecs;
there is no silent fallback to GD. Set `imagine.driver: gd` to keep using GD.

The resolved `imagine_driver` is stored in each static generated version's options,
so adding or changing the effective driver is detected by the media migration
checker, including existing versions created before this option existed. Run the
normal media migration explicitly to regenerate files; changing configuration
does not itself run a migration. Type/global driver changes do not affect versions
with an explicit override. Animated FFmpeg versions and manual uploads do not
inherit an Imagine driver. `resampling-filter` is passed to the resize operation;
Imagick supports configurable filters, while GD keeps its native resampling.

The three resize/encoding options follow the same global → type → version
inheritance (use their names directly on a type or version). Their effective values
are also persisted and checked for regeneration:

- `allow_upscale: false` caps dimensions at the source size.
- `resize_tolerance: 0.02` skips proportional reductions of 2% or less. For example,
  a 1127px source requested at 1120px stays at 1127px. Set `0` for strict resizing.
  Tolerance does not apply when both width and height are explicitly requested.
- `preserve_original_encoding: true` keeps an independent byte-for-byte copy when
  dimensions and format remain unchanged, unless flattening or an explicit output
  resolution is requested. Quality settings do not force recompression in this
  case; set this option to `false` to force encoding. Format conversions and larger
  reductions still encode normally.

Picture width descriptors use the actual generated width and duplicate widths
within a source are removed. These options apply only to static generated images,
not animated FFmpeg versions or manual uploads. Existing stored files are unchanged
until a media migration/regeneration is explicitly run.

## Animated image versions {#animated-image-versions}

Image types accept still and animated uploads without separate animation types.
The detector inspects PNG/APNG, GIF, WebP and AVIF files independently of their
extension. Animated inputs use FFmpeg; still images use the configured Imagine
driver. Image animations render in pictures without a manual poster. Video
versions and their video sets keep their existing processing path.

```yaml
sfs_media:
    defaults:
        avif_quality: 85
        animation: { max_frames: 500 }
    ffmpeg:
        binary: ffmpeg
        probe_binary: ffprobe
        timeout: 300
    types:
        content:
            defaults:
                animation: { speed: 6 }
            upload_requirements:
                animation: accepted
                mimeTypes: [image/png, image/apng, image/avif, image/gif]
            versions:
                small_avif: { type: avif, scale_width: 400 }
                small_webp: { type: webp, scale_width: 400, webp_quality: 82 }
                still: { type: avif, scale_width: 400, animated: no }
```

`upload_requirements.animation` accepts `accepted` (the default), `no` or
`required`. Compatible-type choices use known animation metadata and creation
validates the downloaded original again. Generated versions use `animated: keep`
by default for animation-capable formats. Set `animated: no` to extract a still
frame; JPEG always produces a still image. Legacy boolean values on versions
are normalized to `keep`/`no`; migrate configurations to the explicit values.

Supported animated targets are AVIF, WebP, PNG/APNG and GIF; `type: keep` retains
the input format. Timing and frame rate are preserved unless `animation.fps` is
set. PNG input explicitly selects the APNG demuxer, including files with a
separate default image outside the animation. Animated WebP output uses
independent full frames to avoid composition trails in browsers.

FFmpeg and FFprobe must be installed when processing animated uploads. A profile
requiring animation checks both binaries during configuration compilation.
Accepted animation does not make static-only processing depend on FFmpeg.
Animated WebP input uses an Imagick-to-APNG bridge when passed to FFmpeg, preserving frame timing and transparency even when FFmpeg cannot decode animated WebP directly.

Animation processing options (in global/type defaults or per version):

- `fps`: output frame rate, 1–120; omitted preserves source timing.
- `loop`: loop count; `0` means infinite.
- `crf`: AVIF quality, 0–63; derived from `avif_quality` when omitted.
- `speed`: AVIF encoding speed, 0–8.
- `keyframe_interval`: encoder GOP size.
- `max_duration`: maximum source duration in seconds.
- `max_frames`: maximum source frame count.

Limits reject excessive animations instead of silently truncating them.
Processing is synchronous; regenerate large collections outside HTTP requests.
Manual uploads do not inherit generation defaults. The former `Tools\Apng`
utility is replaced by the unified `Tools\AnimationDetector` service.

## Processing Pipeline {#processing-pipeline}

The real upload and generation flow is driven by Doctrine entity listeners plus tagged processors.

### When Version Entities Are Created {#when-version-entities-are-created}

`MediaListener::preFlush()` ensures that:

- `_original` exists;
- every configured version entity exists.

This happens automatically on regular media saves.

The listener skips this behaviour while the media manager is in migration mode, because migration handles versions explicitly.

### Processor Order {#processor-order}

Processors are auto-tagged and ordered by priority:

- `UploadedImageSizeProcessor` `255`
- `VersionFileCopyProcessor` `200`
- `FfmpegProcessor` `10` for animated inputs
- `ImagineProcessor` `0` for static inputs
- `StoreFileProcessor` `-100`

That order explains the lifecycle:

1. read image dimensions from uploaded files;
2. copy or download the original into a temp file for generated versions;
3. transform the temp file with FFmpeg for animated images or Imagine for static images;
4. store the final file and persist the storage URL.

### UploadedImageSizeProcessor {#uploadedimagesizeprocessor}

This processor runs only for uploaded files.

It extracts width and height for image uploads before storage.

### VersionFileCopyProcessor {#versionfilecopyprocessor}

This processor prepares generated versions.

If a non-original version has an `originalVersion`:

- and that original still has a local uploaded file, it copies it;
- otherwise, it downloads the original from the storage driver into a temp file.

This is what makes post-persist generation and later migrations possible.

### ImagineProcessor {#imagineprocessor}

This processor handles static generated image versions. Animated inputs are handled by `FfmpegProcessor` before this stage.

It only supports:

- non-`_original` versions;
- versions without `upload_requirements`;
- versions linked to an original image file;
- target output types in `jpeg`, `png`, `gif`, `webp`, `avif`, or `keep`.

It can:

- resize by width;
- resize by height;
- resize to fixed width and height;
- change format;
- set image save options such as quality or compression.

It does not support arbitrary video transcoding. Video versions that need conversion must come from an external pipeline and be uploaded as explicit version files.

### StoreFileProcessor {#storefileprocessor}

This processor runs for every version that currently has an upload file.

It:

- normalizes stored options;
- detects APNG uploads;
- fills mime type and file size;
- computes the SHA-1;
- asks the selected name generator for the destination name;
- stores the file through the configured storage driver;
- removes the temp file unless `keepTmpFile` is true.

This processor is the final step of the pipeline.

## Regeneration And Original Replacement {#regeneration-and-original-replacement}

`MediaVersionListener::onFlush()` contains an important regeneration rule.

When the `_original` version is updated with a new upload:

- its URL is reset;
- generated versions that already exist and have `generatedAt` are also reset;
- those generated versions get the new original version as source.

This forces the bundle to regenerate derived versions from the new original file.

This behaviour is one of the main reasons the version model is more than just a static metadata table.

## Rendering In Twig {#rendering-in-twig}

The bundle provides both Twig filters and Twig functions:

- `sfs_media_render_image`
- `sfs_media_render_picture`
- `sfs_media_render_video`
- `sfs_media_render_video_set`
- `sfs_media_render`
- `sfs_media_image_url`
- `sfs_media_type_config`

### Basic Rendering Examples {#basic-rendering-examples}

```twig
{{ media|sfs_media_render_image('card') }}

{{ media|sfs_media_render_picture('default') }}

{{ media|sfs_media_render_video('mp4', { controls: true }) }}

{{ media|sfs_media_render_video_set('default') }}

{{ media|sfs_media_image_url('article') }}
```

The generic renderer accepts strings like:

```twig
{{ media|sfs_media_render('image#card') }}
{{ media|sfs_media_render('picture#default') }}
{{ media|sfs_media_render('video#mp4') }}
{{ media|sfs_media_render('videoSet#default') }}
```

### Picture Rendering {#picture-rendering}

`renderPicture()` reads the current media type config and builds:

- one `<picture>` tag;
- one `<source>` tag per configured source block;
- one final `<img>` tag for the fallback image.

If the picture config name does not exist for the media type, the renderer throws an exception.

### Picture display dimensions {#picture-display-dimensions}

By default, a rendered `<picture>` takes its `<img>` width and height from `img.src_version`. When that fallback has different dimensions from the responsive sources, use `img.dimensions_version` to provide the intrinsic display size without changing the fallback URL:

```yaml
pictures:
    card:
        sources:
            - { srcset: [ { version: card, suffix: '1x' }, { version: card_retina, suffix: '2x' } ] }
        img:
            src_version: fallback
            dimensions_version: card
```

Explicit image attributes passed to the renderer still take precedence over the configured dimensions version.

### Animated images in pictures {#animated-images-in-pictures}

AVIF and WebP versions preserve animation automatically when the original is
animated. The same picture configuration works for static images and animations;
sources are rendered in their configured order without animation filtering:

```yaml
versions:
    avif: { type: avif }
    webp: { type: webp }
pictures:
    _default:
        sources:
            - srcset: [{ version: avif }]
              attrs: { type: image/avif }
            - srcset: [{ version: webp }]
              attrs: { type: image/webp }
        img:
            src_version: avif
```

The default `animated: keep` retains animation in supported image formats;
`animated: no` explicitly generates a still image. JPEG is always static.
Animated images do not need a poster. Upload and regeneration store detection
in `media.metadata.animated`; picture rendering never downloads originals or
runs FFprobe. Animated WebP inputs use Imagick and a lossless RGBA APNG bridge
when passed to FFmpeg, preserving frame timing and transparency.

`webx` is not a supported output format. WebM belongs in video sets, while
picture sources and `<img>` use image formats.

Configured storage public base URLs apply consistently to image URLs, picture
sources, image fallbacks, video sources, and posters. Stored `gs://` references
remain unchanged when using a CDN or local proxy hostname.

### Video Set Rendering {#video-set-rendering}

`renderVideoWithSources()` reads the `video_sets` block and builds:

- one `<video>` tag;
- one `<source>` tag per configured version;
- an optional `poster` attribute when `poster_version` is configured.

This is useful for video formats such as MP4 and WebM served together.

### Alt Text Fallback {#alt-text-fallback}

For image rendering, the `<img>` `alt` attribute is resolved in this order:

1. translated `altTexts`
2. media `description`
3. media `name`

That fallback chain is worth following in your data model if you want accessible defaults.

### Rendering By Id Or Entity {#rendering-by-id-or-entity}

The renderer accepts either:

- a `MediaInterface` object;
- or a media id string.

If you pass a string id, the renderer loads the media through Doctrine.

## Forms {#forms}

The bundle exposes several forms, but the two most important families are:

- upload forms
- media selection forms

### MediaTypeUploadType {#mediatypeuploadtype}

`MediaTypeUploadType` creates a media upload form for one configured media type.

Important options:

- `media_type`
- `required_uploads`
- `allow_name_field`
- `allow_description_field`
- `allow_alt_text_field`

The form automatically adds:

- `name`
- `description`
- `_original`
- one field for every version with `upload_requirements`
- `altTexts`

At submit time, it also sets:

- the media type key;
- the `private` flag from the type config.

### MediaVersionUploadType {#mediaversionuploadtype}

`MediaVersionUploadType` is the inner form used for each version upload field.

It applies the custom `Image` constraint built from the configured upload requirements.

### Alt Text Field Caveat {#alt-text-field-caveat}

In the current code, `MediaTypeUploadType` uses `Softspring\CmsBundle\Form\Type\TranslationType` for `altTexts`.

If your project does not expose that form type, disable the field:

```php
$builder->add('media', MediaTypeUploadType::class, [
    'media_type' => 'article_image',
    'allow_alt_text_field' => false,
]);
```

or replace the form type in your own application.

### MediaChoiceType {#mediachoicetype}

`MediaChoiceType` is an `EntityType` specialized for selecting existing media entries.

It filters by `media_types` and can preload preview HTML in the choice attributes.

The `media_types` option is not just a list of allowed type keys. It also defines which rendering mode and version should be used for previews, for example:

```php
[
    'article_image' => ['picture' => 'default'],
    'teaser_video' => ['video' => 'mp4'],
]
```

Preview attributes are only generated when the form field has `data-media-preview-input`.

### Modal Selectors {#modal-selectors}

`MediaModalType` and `MediaVersionModalType` support the admin-style media picker.

They can:

- restrict the valid media types;
- expose preview configuration to the frontend through data attributes;
- open the admin search route for media selection.

The search route is built around public media only.

## Configured media selections {#configured-media-selections}

Media fields restrict both their modal choices and submitted values to
`media_types`. For example, `content: { picture: two_columns }` exposes only
`picture#two_columns`, not raw image versions or other picture sets. Image,
video, picture and video-set choices follow the configured order and must exist
on the selected media. The version selector is hidden for zero or one choice.
Existing invalid selections remain visible as invalid; migrate saved module data
before expecting it to validate under a narrower allow-list. Rebuild/install the
bundle's assets when updating.

## Admin UI {#admin-ui}

The bundle can expose a full media library UI through `softspring/crudl-controller`.

Enable the built-in admin controller:

```yaml
sfs_media:
    media:
        admin_controller: true
```

Import routes:

```yaml
_sfs_media_admin:
    resource: '@SfsMediaBundle/config/routing/admin_media.yaml'
    prefix: /admin/media
```

Import the role hierarchy when you use the default permission names:

```yaml
imports:
    - { resource: '@SfsMediaBundle/config/security/admin_role_hierarchy.yaml' }
```

### Admin Actions {#admin-actions}

The admin controller defines these actions:

- list
- create
- update
- delete
- read
- read_ajax
- search_type
- create_ajax
- migrate

This is a richer surface than the current docs page suggested.

### Search Type Action {#search-type-action}

The `search_type` action is the route used by modal selectors.

`MediaSearchTypeListener` forces:

- `private = false`
- `type__in = <valid types from the route>` when not already filtered

So the modal search flow is intentionally limited to public media.

### Migrate Action {#migrate-action}

The `migrate` admin action runs the same migration logic as the command, but only for one media item.

It is useful when:

- one type definition changed;
- one media entry is missing generated files;
- one media needs repair without migrating the whole library.

## Regenerate versions from the admin {#regenerate-versions-from-the-admin}

The media detail page includes **Regenerate versions** when the user has the
media migration permission and the media has an original file and generated
versions. This opens a confirmation page listing the generated version keys.
Only a confirmed, CSRF-protected POST starts the synchronous regeneration.
Originals and configured manual uploads are preserved; generated versions are
rebuilt even when their options already match. Normal cleanup of obsolete
versions still applies. On completion the user returns to the media detail page
with a success/error notification. Large animations may take time: wait for the
result before retrying. The standard and Sylius admin templates are supported.

### Sylius admin integration {#sylius-admin-regeneration}

With `softspring/cms-sylius-bundle` installed, media detail pages provide the same **Regenerate versions** action with a Semantic UI confirmation page. The templates reuse Media Bundle's controller, permissions, CSRF protection, and generated-version plan. Originals and configured manual uploads are preserved.

Update both packages together: the Sylius templates require the Media Bundle 6.0 regeneration endpoint and shared confirmation form. No additional assets are needed for this integration.

## Create from an existing original {#create-from-an-existing-original}

The media detail screen offers a **Create from this** type dropdown. It creates a
new media from the source's `_original`, copying its name, description and all alt
translations. Original and generated files belong to the new media; deleting or
editing either media does not modify the other's files or translations.

Destinations must be public image types other than the source type, satisfy the
original's stored dimensions, orientation, ratio, pixels, MIME type and size
requirements, and have no independently uploaded versions. The downloaded file is
validated again against the destination's upload requirements before creation.
Generation uses the existing synchronous processing pipeline. The POST action
requires create/read permissions and a source-specific CSRF token.

## Type Migration {#type-migration}

When type definitions change, old database rows and stored files do not update automatically.

That is what `sfs:media:types-migration` is for:

```bash
php bin/console sfs:media:types-migration
```

The command paginates media IDs, switches the manager into migration mode, and checks every selected media against the current type config. It clears the Doctrine identity map after each media.

### Parallel media migrations {#parallel-media-migrations}

The migration command defaults to a single process. Start with two workers and
increase only after measuring CPU and memory usage, especially with animated
media and AVIF (the encoders may also use multiple threads).

```bash
php bin/console sfs:media:types-migration --workers=2 --batch-size=100
php bin/console sfs:media:types-migration --workers=2 --type=background --dry-run
php bin/console sfs:media:types-migration --workers=4 --type=background --force --dry-run
php bin/console sfs:media:types-migration --workers=4 --type=background --force --no-debug
```

Workers partition media IDs deterministically. A media and all its versions are
processed by one worker, in version order. IDs are paginated and Doctrine's
identity map is cleared after each media. Each worker reports timings and counts;
the parent returns a nonzero exit code if any worker fails. Dry runs report
version changes without modifying media or storage. Existing matching versions
are not regenerated; rerunning checks the remaining differences.

Use `--force` to rebuild generated versions even when their configuration is
unchanged, for example after updating image codecs. It preserves `_original` and
configured manual/uploaded versions. Normal migration cleanup of obsolete
versions still applies. Combine it with `--type` to limit its scope and with
`--dry-run` to inspect the plan without writing anything. Without `--dry-run`,
forced regeneration replaces generated versions and uploads their new files;
it can be expensive. Force is opt-in and is forwarded to every worker.

Distributed runners can use `--worker-index=0 --worker-count=4` (indexes 0–3),
instead of `--workers`. Every task must use the same configuration, filters and
worker count, and a unique index. Do not run overlapping migration jobs against
the same media concurrently: partitioning is not a cross-job lock. Do not change
media type configuration during a run. No Messenger worker or PHP threading
extension is required; local workers use Symfony Process and the project's
`bin/console`. The worker count must fit the resources allocated to the container.

### What Migration Compares {#what-migration-compares}

`TypeChecker::checkMedia()` classifies versions into:

- `ok`
- `new`
- `changed`
- `delete`
- `manual`

Those categories mean:

- `ok`
  - the stored version matches the current config
- `new`
  - the version exists in config but not in database and can be generated automatically
- `changed`
  - the version exists but its stored options differ from the current config
- `delete`
  - the version exists in database but no longer exists in config
- `manual`
  - the version is new in config but needs a manual upload because it has `upload_requirements`

### What Migration Actually Does {#what-migration-actually-does}

During migration, the media manager:

- creates missing auto-generated versions;
- recreates changed versions;
- deletes removed versions;
- keeps manual-upload versions marked as manual;
- updates missing SHA-1 values;
- updates missing file sizes.

This distinction between generated and manual versions is one of the most important operational details of the bundle.

## Duplicate Detection {#duplicate-detection}

`MediaManager` also contains duplicate helpers based on:

- media type
- SHA-1 of the `_original` version

Available helpers:

- `findMediaDuplicates()`
- `findDuplicates()`
- `getDuplicatesStats()`

They are implemented directly in the manager today rather than in a repository, but they are still useful when cleaning a large media library.

## Doctrine Migrations Path {#doctrine-migrations-path}

The bundle prepends its own migrations path into Doctrine Migrations:

- `Softspring\MediaBundle\Migrations` => `@SfsMediaBundle/src/Migrations`

This is part of the runtime setup and matters if your project relies on bundle-provided schema migrations.

## Important Implementation Details {#important-implementation-details}

These are the parts of the current codebase that are easy to miss and worth knowing before you depend on a behaviour.

### Configured Public URLs {#filesystem-public-url-caveat}

Rendering resolves stored URLs through the configured storage driver before falling back to the entity's public URL. The configured public base URL therefore applies to images, picture sources and fallbacks, video sources, and posters. Stored `gs://` references remain unchanged when a CDN or local proxy hostname is used.

### Video Processing Is Storage-Oriented, Not Transcoding-Oriented {#video-processing-is-storage-oriented-not-transcoding-oriented}

The bundle understands video media types and video set rendering, but it does not include a real transcoder.

Video versions with their own uploads are modeled as manually uploaded files, not as FFmpeg-style generated outputs.

### Processor Pipeline Depends On Doctrine Lifecycle {#processor-pipeline-depends-on-doctrine-lifecycle}

The processing pipeline is attached to Doctrine entity listeners.

That means version generation and storage happen when version entities are persisted or updated, not when a standalone helper service is called directly.

### Modal Search Excludes Private Media {#modal-search-excludes-private-media}

The modal search flow forces `private = false`.

If your application needs private media selection in those modals, you need to override that listener or provide a different search flow.

### Type Config Is Merged From Providers {#type-config-is-merged-from-providers}

The default provider reads `sfs_media.types`, but the bundle architecture allows extra `MediaTypeProviderInterface` services.

So type definitions can come from more than one place.

For advanced projects, that is the better extension point than trying to keep all type config in one very large YAML file.
