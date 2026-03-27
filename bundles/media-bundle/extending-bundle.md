---
title: "Extending Media Bundle"
description: "Extend Media Bundle through providers, processors, forms, listeners, templates, and other safe integration points."
---

# Extending Bundle {#extending-bundle}

`media-bundle` is designed to be configured heavily and extended selectively. In most projects, you do not replace everything. You keep the storage, versioning, and rendering infrastructure, and override only the parts that carry product-specific rules.

## Good Extension Points {#good-extension-points}

The current codebase exposes several practical extension points:

- custom media entities and media version entities
- custom media type providers
- custom name generators
- custom processors
- replaced forms
- replaced listeners
- overridden templates

## Media Type Providers {#media-type-providers}

Use `MediaTypeProviderInterface` when media type definitions should come from more than one place.

This is a cleaner long-term option than keeping one very large YAML file when several modules or domains contribute different media types.

## Processors {#processors}

Processors are tagged automatically through `ProcessorInterface`.

That is the right place to extend the file lifecycle when:

- an extra transformation is needed
- metadata must be extracted before storage
- the application needs another pre-storage or post-copy step

Keep in mind that the processing pipeline is attached to Doctrine lifecycle events.

## Forms And Templates {#forms-and-templates}

Replace or decorate forms when:

- upload fields need different UI or validation
- selectors should expose different preview behavior
- alt text handling or field visibility must change

Override templates when the flow is correct but the presentation needs to match the project UI.

## Admin And Listener Overrides {#admin-and-listener-overrides}

Override admin listeners or route usage when:

- modal search should include private media
- admin create or update behavior needs extra side effects
- migrate or read actions need project-specific policies

The bundle is a good base layer for media management, but not every workflow should remain generic in a real product.

## When To Fork Less {#when-to-fork-less}

Prefer:

- providers over giant config files
- processors over ad hoc file handling
- form replacement over controller forks
- listeners and templates over rewriting the whole admin flow

That keeps your application closer to supported bundle behavior and makes upgrades easier.

## Related Guides {#related-guides}

- [Media Bundle Overview](../media-bundle.md)
- [Concepts](./concepts.md)
- [Using Medias](./using-medias.md)
- [Admin Medias](./admin-medias.md)
- [Name Generators](./name-generators.md)
