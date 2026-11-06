---
title: "CMS Bundle Modules"
description: "Build editable CMS modules with Symfony forms, Twig templates, and versioned data migrations."
---

# Modules {#modules}

A module is an editable element inside a content layout: a heading, a picture, a call to action, or a more specialized component. Each instance stores its own values in the content version. The module definition provides the form and the templates that interpret those values.

Start with [Configure modules](../../configuration/configure-modules.md) for a minimal hero. This guide adds a reusable rating module and explains how to extend its form and evolve its stored data.

## Create a rating module {#create}

Create this directory in the application's CMS collection:

```text
cms/modules/rating/
├── config.yaml
├── form.html.twig
├── render.html.twig
└── translations/
    └── sfs_cms_modules.en.yaml
```

The example uses a plain title and a score from one to five. It deliberately keeps the heading level fixed, so editors can change the content without changing the page's heading hierarchy.

```yaml
# cms/modules/rating/config.yaml
module:
    revision: 1
    group: content
    compatible_contents: [page]
    form_template: '@module/rating/form.html.twig'
    module_options:
        form_fields:
            title:
                type: text
                type_options:
                    required: true
            score:
                type: choice
                type_options:
                    choices:
                        '1 star': 1
                        '2 stars': 2
                        '3 stars': 3
                        '4 stars': 4
                        '5 stars': 5
```

`render_template` defaults to `@module/rating/render.html.twig`. The dynamic module form resolves `text` and `choice` to Symfony form types and passes `type_options` to them.

```twig
{# cms/modules/rating/form.html.twig #}
{{ form_row(form.title) }}
{{ form_row(form.score) }}
{{ form_rest(form) }}
```

```twig
{# cms/modules/rating/render.html.twig #}
<section class="rating">
    <h2>{{ title|default('') }}</h2>
    {% if score|default(false) %}
        <p class="rating__score">{{ score }} / 5</p>
    {% endif %}
</section>
```

Normal Twig escaping applies to the title. Add the rating styles to your application frontend build.

```yaml
# cms/modules/rating/translations/sfs_cms_modules.en.yaml
rating:
    label: Rating
    prototype_button: Rating
    prototype_button_icon: '<i class="bi bi-star"></i>'
    form:
        title.label: Title
        score.label: Score
```

Clear the Symfony cache, open a page with a container that permits the module, insert **Rating**, and save a draft. Preview the page before publishing. If the module is absent from the picker, check the content's `allowed_modules` and the module's `compatible_contents`.

## Choose field types {#form-types}

Use Symfony field names such as `text`, `textarea`, `choice`, `checkbox`, `integer`, `url`, `collection`, and `entity`. Their usual Symfony options apply. An entity field also needs its entity `class`; a collection needs its `entry_type`.

CMS-specific fields provide editing and reference tracking:

| Field | Purpose |
| --- | --- |
| `id`, `class`, `color`, `css` | HTML presentation values |
| `html` | Rich editorial content |
| `blockInstance`, `blockStatic` | Select an editable block record or a static block type |
| `media`, `mediaVersion` | Select a media object, version, or picture |
| `symfonyRoute` | Select a Symfony route and its parameters |
| `translatable` | Wrap a field with values for each enabled locale |
| `dynamicForm`, `dynamicFormCollection` | Define nested data structures |

See the dedicated guides for [media](module/form-types/media.md), [media modal](module/form-types/media-modal.md), [Symfony routes](module/form-types/symfony-route.md), [links](module/form-types/link.md), and [translatable fields](module/form-types/translatable.md). Their stored values differ from plain strings; use their rendering helpers rather than concatenating database identifiers into URLs.

## Use a custom Symfony form type {#build-your-own-form-types-for-modules}

A form type in `App\Form\Type` can be selected by its short name. For example, `ratingScore` resolves to `App\Form\Type\RatingScoreType`:

```php
<?php
// src/Form/Type/RatingScoreType.php
namespace App\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\ChoiceType;
use Symfony\Component\OptionsResolver\OptionsResolver;

final class RatingScoreType extends AbstractType
{
    public function getParent(): string
    {
        return ChoiceType::class;
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'choices' => array_combine(range(1, 5), range(1, 5)),
        ]);
    }
}
```

Register the type through Symfony's normal `App\` service discovery with autoconfiguration enabled. Then replace the `score` field definition with:

```yaml
module:
    module_options:
        form_fields:
            score:
                type: ratingScore
```

This is an override fragment of the existing module configuration. For custom namespaces or an entirely custom module form, see [Dynamic Form Type](../../components/dynamic-form-type.md). Use `module_type` and `module_options` when the whole module needs a dedicated form type.

## Customize the editor preview {#editor-preview}

Set `edit_template` to a Twig template when the generic editing form is insufficient. The editor template receives a Symfony form view. Render values from `form.<field>.vars.data`; keep form field rendering in `form_template`.

CMS editor assets can synchronize fields with a preview through paired attributes such as `data-edit-content-input` and `data-edit-content-target`. Use the same identifier on both sides. Rich text, translated fields, and media have additional widgets; inspect a bundled module with the same field type before extending their inline behavior.

An editor preview is separate from the public `render_template`. Check both the editing screen and the public preview when changing the module's design.

## Override a bundled module {#overwrite}

Add a directory with the same module identifier to the application's later-loaded CMS collection. Override only the files and configuration values that need to change. For example, a custom `render.html.twig` can change a button's markup without replacing its form.

Collection order matters: later collections override earlier ones. Clear the Symfony cache after changing definitions. Recompile already published content when the public render template changes; clearing the Symfony cache alone does not replace stored compiled HTML.

## Migrate stored values {#migrate-stored-values}

Increase `revision` when changing the saved data shape and add `migrate.php`. The script returns a callable that receives the stored data, its previous revision, and the target revision:

```php
<?php
// cms/modules/rating/migrate.php
return static function (array $data, int $fromRevision, int $toRevision): array {
    if ($fromRevision < 2 && array_key_exists('label', $data)) {
        $data['title'] = $data['label'];
        unset($data['label']);
    }

    return $data;
};
```

Set `revision: 2` for this change. The CMS sets `_revision` after running migration scripts. Preserve unrelated keys, make conversions repeatable, and test data from every revision still present in your database. Increasing the number alone does not convert an old `label` field into `title`.

## Configured media selection migrations {#configured-media-selection-migrations}

Module migration scripts can receive optional fourth and fifth arguments:
`CmsConfig $cmsConfig` and `MediaTypesCollection $mediaTypesCollection`. Older
scripts remain valid. Both editor data mapping and module rendering supply this
context.

`MediaSelectionMigrator::migrate($data, $formFields, $mediaTypesCollection)`
repairs restricted `mediaVersion` fields, including translations and dynamic
collections, using the first available choice in configuration order. Valid
selections and unrelated data remain untouched. A configured picture name is
never replaced with a hard-coded `_default`; unresolved references or empty
allow-lists are preserved for editorial resolution.

Use a matching Media Bundle version that provides `MediaSelectionChoices` alongside these migrations.

## Module site filters {#module-site-filters}

Module site filters accept persisted site entities as well as site identifiers
and boolean maps. Saving a module through a site selection field therefore
keeps the same rendering behaviour when its data is hydrated again.
