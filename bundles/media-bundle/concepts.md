---
title: "Media Bundle Concepts"
description: "Understand the core model, version lifecycle, processing pipeline, and migration behavior of Media Bundle."
---

# Concepts {#concepts}

The bundle revolves around three runtime concepts:

- the media entity
- the media version entity
- the media type configuration

## Media Entity {#media-entity}

`MediaInterface` represents the logical media entry. It stores:

- type key such as `article_image`
- normalized media category such as image or video
- name and description
- privacy flag
- SHA-1
- translated alt texts
- all versions

The media entry is the stable business object. Versions are derived or uploaded files attached to it.

## Media Version Entity {#media-version-entity}

`MediaVersionInterface` represents one stored or generated file version.

It stores metadata such as:

- URL
- mime type
- dimensions
- file size
- upload or generation timestamps
- normalized options
- SHA-1

`_original` is the special original uploaded file. Every media has one.

## Generated And Uploaded Versions {#generated-and-uploaded-versions}

Configured versions are either:

- generated versions
  - derived automatically from another version, usually `_original`
- uploaded versions
  - require their own upload because they define `upload_requirements`

This distinction matters for migrations and admin flows.

## Processing Pipeline {#processing-pipeline}

The version lifecycle is driven by Doctrine listeners and tagged processors:

1. `UploadedImageSizeProcessor`
2. `VersionFileCopyProcessor`
3. `ImagineProcessor`
4. `StoreFileProcessor`

That means version generation is part of the entity lifecycle, not a separate batch command you must call manually after every upload.

## Migration When Type Config Changes {#migration-when-type-config-changes}

When type definitions change, old database rows and stored files do not update automatically.

Use:

```bash
php bin/console sfs:media:types-migration
```

The migration logic classifies versions as:

- `ok`
- `new`
- `changed`
- `delete`
- `manual`

That is what lets the bundle regenerate generated files while preserving manual-upload expectations.

## Duplicate Detection {#duplicate-detection}

`MediaManager` also includes duplicate helpers based on:

- media type
- SHA-1 of the original version

This is useful when cleaning or consolidating large media libraries.

## Important Current Limitations {#important-current-limitations}

- video support is rendering- and storage-oriented, not a transcoding pipeline
- private media is a persisted flag, not a complete secure delivery system
- some rendering paths still assume the default filesystem public URL behavior

These are not reasons to avoid the bundle. They are integration details you should understand before depending on those behaviors.

## Related Guides {#related-guides}

- [Media Bundle Overview](../media-bundle.md)
- [Configure Media Types](./media-types.md)
- [Using Medias](./using-medias.md)
- [Storage Options](./storage-options.md)
- [Extending Bundle](./extending-bundle.md)
