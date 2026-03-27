---
title: "Storage Options"
description: "Choose between filesystem and Google Cloud Storage in Media Bundle and understand the tradeoffs of each."
---

# Storage Options {#storage-options}

The bundle ships two storage drivers:

- `filesystem`
- `google_cloud_storage`

Both implement the same storage contract, so the rest of the bundle can keep the same behavior.

## Filesystem Storage {#filesystem-storage}

```yaml
sfs_media:
    driver: filesystem
    filesystem:
        path: '%kernel.project_dir%/public/media'
        url: '/media'
```

This is the simplest setup for:

- local development
- small and medium applications
- environments where media files live directly under the application public path

The driver stores internal URLs with the `sfs-media-filesystem://` prefix and can translate them to public URLs later.

## Google Cloud Storage {#google-cloud-storage}

```yaml
sfs_media:
    driver: google_cloud_storage
    google_cloud_storage:
        bucket: '%env(MEDIA_BUCKET_NAME)%'
```

This is the better fit when:

- media should live outside the application container or host
- several app instances share the same storage
- deployment and media lifetime must be decoupled

Stored URLs use the `gs://bucket/path` format internally and the driver exposes them as standard Google Storage HTTPS URLs.

## Driver Responsibilities {#driver-responsibilities}

Both drivers implement:

- `store()`
- `remove()`
- `download()`
- `url()`

That is important because regeneration and migration may need to download the original file again even long after the first upload.

## Private Media Caveat {#private-media-caveat}

`private: true` marks media as private in the entity and is used in some admin and search flows.

It does not provide a signed-URL delivery layer or a full secure asset access system by itself.

## Current Filesystem URL Caveat {#current-filesystem-url-caveat}

In the current implementation, some rendering paths still assume `/media/...` for filesystem URLs.

So a custom `filesystem.url` is not reflected uniformly in every code path yet. Keep that in mind if your project needs a different public prefix.

## Related Guides {#related-guides}

- [Media Bundle Overview](../media-bundle.md)
- [Install](./install.md)
- [Getting Started](./getting-started.md)
- [Concepts](./concepts.md)
- [Extending Bundle](./extending-bundle.md)
