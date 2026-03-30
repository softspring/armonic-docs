---
title: "Name Generators"
description: "Control how Media Bundle generates destination names and paths for stored media files."
---

# Name Generators {#name-generators}

Each media type can choose the service class that generates stored file names.

## Default Generator {#default-generator}

The default generator is:

- `Softspring\MediaBundle\Media\DefaultNameGenerator`

It builds names from:

- the media id when available
- the version name
- the uploaded file extension

That is enough for many projects, especially when the storage path only needs stable uniqueness and a light relationship to the media entity.

## Configure A Generator Per Type {#configure-a-generator-per-type}

```yaml
sfs_media:
    types:
        article_image:
            type: image
            generator: App\Media\ArticleImageNameGenerator
```

## Custom Generator Contract {#custom-generator-contract}

Create a service that implements `NameGeneratorInterface`.

The bundle auto-tags services implementing that interface and collects them through `NameGeneratorProvider`.

This is useful when:

- the path must include business grouping such as year, locale, or tenant
- migration from a previous storage layout needs predictable names
- you want a naming convention that editors or other systems can inspect easily

## Practical Advice {#practical-advice}

- keep names deterministic enough for maintenance
- avoid encoding too much business meaning in the physical path
- treat the generator as a storage detail, not as the main source of metadata

The media entity and version entity should still carry the business metadata. The generator should only decide where the file lives.

## Related Guides {#related-guides}

- [Media Bundle Overview](../media-bundle.md)
- [Configure Media Types](./media-types.md)
- [Storage Options](./storage-options.md)
- [Extending Bundle](./extending-bundle.md)
- [Integrations](./integrations.md)
