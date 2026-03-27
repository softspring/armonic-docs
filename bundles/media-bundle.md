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

The bundle supports image and video media. Storage is delegated to a storage driver, and image transformations are handled through GD via `imagine/imagine`.

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

Image processing support also depends on what your GD build can actually read and write. The configuration validator checks that before the container is built.

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
- that version output formats are supported by the current GD build.

This is especially relevant for:

- `webp`
- `avif`
- `apng`

If your environment cannot encode or decode one of those formats, container compilation fails with a configuration error instead of failing later at runtime.

### APNG Support {#apng-support}

The bundle has explicit APNG support:

- the config validator accepts `image/apng` when PNG support exists;
- the upload validator removes false-positive mime-type violations for APNG files detected as `image/png`;
- `StoreFileProcessor` stores APNG uploads with `image/apng`;
- `ImagineProcessor` does not attempt to scale APNG files when the target remains APNG.

That is one of the more specialized parts of the bundle and is worth relying on only if your environment actually supports the needed image functions.

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
- `ImagineProcessor` `0`
- `StoreFileProcessor` `-100`

That order explains the lifecycle:

1. read image dimensions from uploaded files;
2. copy or download the original into a temp file for generated versions;
3. transform the temp file when the version is generated from an image;
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

This processor handles generated image versions.

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

## Type Migration {#type-migration}

When type definitions change, old database rows and stored files do not update automatically.

That is what `sfs:media:types-migration` is for:

```bash
php bin/console sfs:media:types-migration
```

The command loads all media entities, switches the manager into migration mode, and checks every media against the current type config.

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

### Filesystem Public URL Caveat {#filesystem-public-url-caveat}

`MediaVersion::getPublicUrl()` hardcodes filesystem media URLs as `/media/...` when the stored URL uses the internal filesystem prefix.

Because `MediaRenderer` prefers `getPublicUrl()` over `StorageDriverInterface::url()`, a custom configured `filesystem.url` is not always reflected in rendered URLs.

That means the documentation-friendly configuration:

```yaml
filesystem:
    url: '/custom-media'
```

does not fully match the current rendering behaviour in every code path.

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
