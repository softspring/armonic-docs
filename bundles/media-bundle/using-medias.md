---
title: "Using Medias"
description: "Upload, select, render, and reuse media entries in forms, Twig, and application flows with Media Bundle."
---

# Using Medias {#using-medias}

Once media types exist, the next question is how to use the bundle in real application flows.

## Upload Forms {#upload-forms}

`MediaTypeUploadType` is the main application-facing upload form.

Important options are:

- `media_type`
- `required_uploads`
- `allow_name_field`
- `allow_description_field`
- `allow_alt_text_field`

The form adds:

- `name`
- `description`
- `_original`
- one field for each manual-upload version
- `altTexts`

At submit time, it also sets the media type key and the `private` flag from the configured type.

## Version Upload Fields {#version-upload-fields}

`MediaVersionUploadType` is the inner form used for version uploads. It applies the custom image validation derived from the configured upload requirements.

## Choosing Existing Media {#choosing-existing-media}

Use `MediaChoiceType` when you need to select existing media entities instead of uploading new ones.

It can:

- filter by allowed media type keys
- preload preview HTML in the choice attributes
- support UI flows where editors reuse existing library items

## Modal Selectors {#modal-selectors}

`MediaModalType` and `MediaVersionModalType` support admin-style media pickers and modal search flows.

They are useful when:

- editors need to browse the media library from another form
- previews should be shown in the selector
- media choices should be restricted to specific types

The default modal search flow only exposes public media.

## Twig Rendering {#twig-rendering}

The bundle provides filters and functions such as:

- `sfs_media_render_image`
- `sfs_media_render_picture`
- `sfs_media_render_video`
- `sfs_media_render_video_set`
- `sfs_media_render`
- `sfs_media_image_url`
- `sfs_media_type_config`

Examples:

```twig
{{ media|sfs_media_render_image('article') }}
{{ media|sfs_media_render_picture('default') }}
{{ media|sfs_media_render_video('mp4', { controls: true }) }}
{{ media|sfs_media_render('image#article') }}
```

## Alt Text Fallback {#alt-text-fallback}

For rendered images, the `alt` attribute is resolved in this order:

1. translated `altTexts`
2. media `description`
3. media `name`

This is a useful fallback chain to keep in mind when designing editor workflows.

## Real Usage Patterns {#real-usage-patterns}

Common real use cases are:

- article hero images with responsive picture sets
- shared media library items reused in several pages
- product or catalog images with manual thumbnails
- videos rendered through multi-source `<video>` sets

The bundle is strongest when the media behavior is repeated across the application and benefits from one configured contract.

## Related Guides {#related-guides}

- [Media Bundle Overview](../media-bundle.md)
- [Getting Started](./getting-started.md)
- [Configure Media Types](./media-types.md)
- [Admin Medias](./admin-medias.md)
- [Integrations](./integrations.md)
