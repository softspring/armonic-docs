---
title: "CMS Module Media Field Type"
description: "Select media and generated versions from the CMS editor and render them with the Media Bundle helpers."
---

# Media fields {#media-type}

In 6.0, the CMS `media` field extends `Softspring\MediaBundle\Form\MediaModalType`. Editors select an existing media item in a modal. The stored model value is a media object; it does not include a selected output version.

Use `mediaVersion` when editors must choose both the media and a generated image version, responsive picture, or video set. It extends `MediaVersionModalType` and stores an array with `media` and `version` keys.

The following examples are fragments of an existing module definition. See [Configure modules](../../../../configuration/configure-modules.md) to create the complete module.

## Select a media item {#example-usage}

```yaml
# cms/modules/feature/config.yaml
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

The media type `background` and its `sm` version must exist in [Media Bundle configuration](../../../../configuration/configure-medias.md). This allow-list limits selectable media types. For a `media` field, choose the output version in the render template:

```twig
{# cms/modules/feature/render.html.twig #}
{% if image|default(false) %}
    {{ image|sfs_media_render_image('sm', {class: 'img-fluid'}) }}
{% endif %}
```

`media_types: null` permits all configured types. An empty map permits none. Avoid leaving the field unrestricted when its render template expects only images.

## Let editors choose the output {#select-output}

```yaml
module:
    module_options:
        form_fields:
            visual:
                type: mediaVersion
                type_options:
                    media_types:
                        background:
                            image: [sm, lg]
                            picture: [responsive]
                    show_thumbnail: true
```

Create `sm`, `lg`, and the `responsive` picture in the Media Bundle first. The submitted version must belong to the configured allow-list for the selected media type.

```twig
{% if visual.media|default(false) and visual.version|default(false) %}
    {{ visual.media|sfs_media_render(visual.version, {class: 'img-fluid'}) }}
{% endif %}
```

The generic render helper selects the appropriate image, picture, video, or video-set renderer from the selected output name.

## Configure the editor preview {#preview-values}

Pass a shared preview identifier through the field attributes and the editor template:

```yaml
module:
    module_options:
        form_fields:
            image:
                type: media
                type_options:
                    media_types:
                        background:
                            image: [sm]
                    attr:
                        data-media-preview-input: featureImage
```

```twig
{# cms/modules/feature/edit.html.twig #}
<div data-media-preview-target="featureImage">
    {% if form.image.vars.data|default(false) %}
        {{ form.image.vars.data|sfs_media_render_image('sm', {class: 'img-fluid'}) }}
    {% endif %}
</div>
```

Load the Media Bundle and CMS admin JavaScript and form themes. The modal also needs the media search admin routes. If the picker is absent, inspect the browser console and the search response before changing the stored field data.

## HTML attributes {#media-attributes}

`media_attr`, `image_attr`, `picture_attr`, and `video_attr` configure HTML attributes exposed to the form view and modal. The current implementation merges them with PHP's array union: a key present in `media_attr` takes precedence over the same key in a specific attribute map. Keep shared and specific keys distinct.

Public rendering uses the attributes passed to the Twig render helper. It does not automatically read the module form configuration. For example:

```twig
{% if video|default(false) and video.isVideo() %}
    {{ video|sfs_media_render_video('_original', {
        controls: true,
        preload: 'metadata'
    }) }}
{% endif %}
```

## Use the legacy select widget {#legacy-select}

If an application still needs a plain select rather than a modal, use the explicit Symfony form class `Softspring\MediaBundle\Form\MediaChoiceType`. Its `media_types` map uses a single output name per mode, such as `background: {image: sm}`. An empty map produces no choices.

This is a different widget from the CMS `media` alias in 6.0. Do not assume that examples for the old select widget describe the current modal's options or submitted structure.
