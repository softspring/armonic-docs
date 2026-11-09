---
title: "CMS Module Media Modal Field Type"
description: "Use the current modal media picker and understand the media and mediaVersion aliases in Armonic 6.0."
---

# Media modal {#media-modal-type}

The standard CMS `media` field already opens a media selection modal in 6.0. Use `mediaVersion` to select an output version alongside the media. See [Media fields](media.md) for complete configuration and rendering examples.

## Replace an older field alias {#example-usage}

The CMS no longer defines `Softspring\CmsBundle\Form\Type\MediaModalType`. Replace an old `type: mediaModal` with `type: media` when the saved value is a media object:

```yaml
# Fragment of cms/modules/feature/config.yaml
module:
    module_options:
        form_fields:
            image:
                type: media
                type_options:
                    media_types:
                        background:
                            image: [sm]
                    show_thumbnail: true
```

Changing to `mediaVersion` also changes the data structure to `{media, version}`. Provide a module data migration for existing content before making that change.

## Requirements {#requirements}

The modal needs the Media Bundle form theme, admin JavaScript, and media search routes. Editors must have permission to access the search endpoint. Upload new files in the media administration area, then return to the picker to select them; this field is not an upload form.

## Restrict selectable media {#field-options}

Set `media_types` to an explicit allow-list that matches the frontend renderer. `null` allows all configured media types; an empty map allows none. For `mediaVersion`, the submitted output is also checked against the versions, pictures, or video sets allowed for the selected type.
