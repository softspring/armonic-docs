---
title: "Translatable Bundle"
description: "Store translated values in JSON fields, edit them in Symfony forms, and add optional automatic translation tools."
---

# Translatable Bundle {#translatable-bundle}

`softspring/translatable-bundle` helps you work with translated field values inside regular Symfony entities and forms.

It is a good fit when your application needs fields like names, labels, excerpts, or alt texts in multiple languages, but you do not want to build a separate translation table for each entity.

The bundle gives you:

- a `Translation` value object
- a Doctrine DBAL type for JSON-backed translated fields
- Symfony form types for editing translated values
- Twig helpers to read the translated value for the current request
- an optional translation API layer for admin interfaces

## Installation {#installation}

Install the package:

```bash
composer require softspring/translatable-bundle:^6.0
```

## What It Solves {#what-it-solves}

This bundle adds one practical pattern:

- store all translations for one field in one JSON value
- keep one default locale inside that value
- edit all locales together in one form widget
- read the best translation for the current request locale

That makes it useful for content fields such as:

- menu item labels
- media alt texts
- SEO fields
- short descriptions
- any other text that belongs to one entity field and should be edited per locale

It is less appropriate when translations need their own relations, lifecycle, or advanced querying as first-class records.

## Basic Configuration {#basic-configuration}

The bundle configuration lives under `sfs_translatable`.

The base form and Doctrine features work without extra configuration. The optional translation API layer is configured here:

```yaml
# config/packages/sfs_translatable.yaml
sfs_translatable:
    api:
        driver: google
```

In practice, Armonic standalone enables the Google driver this way and imports the auto-translate frontend script in the admin asset build.

## Store A Translated Field In Doctrine {#store-a-translated-field-in-doctrine}

The bundle registers the `sfs_translation` DBAL type automatically through `prepend()`.

That lets you declare a translated field directly in Doctrine mapping:

```php
#[ORM\Column(type: 'sfs_translation', nullable: true)]
private ?Translation $title = null;
```

Or in XML:

```xml
<field name="altTexts" column="alt_texts" type="sfs_translation" nullable="true" />
```

This is how other bundles use it. For example, `media-bundle` uses it for `altTexts`.

### How Values Behave {#how-values-behave}

The stored value is a `Translation` object that contains:

- locale => text pairs
- `_default` for the fallback locale
- `_trans_id` as an internal identifier used by the form layer

When you ask for a translation:

1. the requested locale is used if it has a value
2. otherwise the default locale is used
3. otherwise the bundle returns an empty string

If the entity was loaded through Doctrine, the bundle injects the current `RequestStack` into the `Translation` object on `postLoad`, so calling `translate()` without arguments can follow the current request locale.

## Use TranslationType For Common Text Fields {#use-translationtype-for-common-text-fields}

`TranslationType` is the main form type for translated text fields.

It extends `TranslatableType` and adds `_trans_id` automatically. The default inner field type is `TextType`.

Typical usage:

```php
use Softspring\TranslatableBundle\Form\Type\TranslationType;

$builder->add('title', TranslationType::class, [
    'required' => false,
]);
```

This is the easiest option when you want one translated text input per locale.

### Automatic Guessing For Doctrine Fields {#automatic-guessing-for-doctrine-fields}

If a Doctrine field uses the `sfs_translation` type, the bundle can guess `TranslationType` automatically through its form type guesser.

That means you can often rely on your usual form builder or admin tooling without configuring the field type manually.

## Use TranslatableType For Custom Field Types {#use-translatabletype-for-custom-field-types}

`TranslatableType` is the lower-level building block.

Use it when you want the same translated structure but with another inner field type, for example a textarea, a rich text editor, or a custom form type.

Example:

```php
use Softspring\TranslatableBundle\Form\Type\TranslatableType;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;

$builder->add('summary', TranslatableType::class, [
    'type' => TextareaType::class,
    'type_options' => [
        'required' => false,
        'attr' => [
            'rows' => 5,
        ],
    ],
]);
```

### What The Form Type Adds {#what-the-form-type-adds}

For each configured locale, the form creates one child field.

It also adds:

- `_default` with the default locale
- `data-input-lang` on each locale field
- `data-fallback-lang` on non-default locale fields
- `localeFields` in the form view for easier theming

The default locale field is required by default. Other locales are optional.

## Form Themes And Rendering {#form-themes-and-rendering}

The bundle ships three form theme entry points:

- `@SfsTranslatable/form/translatable_type.html.twig`
- `@SfsTranslatable/form/translatable_type.bootstrap5.html.twig`
- `@SfsTranslatable/form/translatable_type.semantic-ui.html.twig`

If your app already uses Bootstrap 5 or Semantic UI, import the matching theme directly. Other bundles do this the same way.

Example:

```twig
{% form_theme form
    '@SfsTranslatable/form/translatable_type.bootstrap5.html.twig'
%}
```

The default theme groups one input per locale and shows the locale code as the label.

## Read Translations In Twig {#read-translations-in-twig}

The bundle provides the `sfs_translate` Twig filter:

```twig
{{ entity.title|sfs_translate }}
```

This reads the current request locale and falls back to the default locale stored in the value.

Use it when the template receives a `Translation` object directly and you want the display value without handling locale logic manually.

## Add Automatic Translation Buttons {#add-automatic-translation-buttons}

The bundle can add translate buttons to `TranslationType` fields when the API layer is enabled.

This is especially useful in admin forms where one editor writes the source text in the default locale and then asks the UI to prefill the other locales.

When an API driver is configured:

- the form extension adds extra metadata to non-default locale fields
- the form theme renders a translate button next to those fields
- the frontend script sends a POST request to the translation endpoint
- the translated text is written back into the target field

### Import The Route {#import-the-route}

The translation endpoint is not enabled automatically. Import the route in your application:

```yaml
# config/routes/sfs_translatable.yaml
sfs_translatable:
    resource: '@SfsTranslatableBundle/config/routing/api_translate.yaml'
```

The endpoint is:

- `POST /api/translate`

It expects:

- `text`
- `source`
- `target`

### Import The Frontend Script {#import-the-frontend-script}

The form theme only renders the button markup. The button becomes interactive when you import the bundle script:

```js
import '@softspring/translatable-bundle/scripts/auto-translate';
```

This is how `armonic-standalone` enables the feature in its admin entrypoint.

### How The UI Works {#how-the-ui-works}

The shipped JS does three practical things:

1. listen for click events on translate buttons
2. send the source text and locales to the API endpoint
3. write the translated text into the target input

If the source field is empty, the script skips the request.

## Google Driver {#google-driver}

The bundle currently ships one built-in driver: `google`.

When enabled, the bundle wires `Google\Cloud\Translate\V2\TranslateClient` behind `TranslatorDriverInterface`.

That makes it enough for many back-office translation helpers, but it is still an application decision:

- whether automatic translation is enabled
- which route is exposed
- how editors review generated text before publishing

## Extend The Bundle {#extend-the-bundle}

The bundle is designed to be extended in a few practical ways.

### Provide Another Translation Driver {#provide-another-translation-driver}

Implement `TranslatorDriverInterface` if you want to use another provider or an internal translation service.

That is the main extension point for the API layer.

### Reuse The Base Form Types {#reuse-the-base-form-types}

Other bundles can wrap `TranslationType` or `TranslatableType` and set their own defaults.

That is how CMS-related packages build their own translation-aware form types while keeping the same core behavior.

### Reuse Or Override The Form Theme {#reuse-or-override-the-form-theme}

You can reuse the shipped Twig blocks directly, or wrap them from another bundle template and adapt the markup to your UI kit.

This is already done in other packages, especially in CMS-related bundles.

## Recommended Usage Patterns {#recommended-usage-patterns}

Use this bundle when:

- one entity owns a translated field directly
- you want editors to see every locale in one form
- a JSON column is enough for persistence
- fallback to one default locale is the expected behavior

Good examples:

- media alt texts
- menu labels
- small content fragments
- SEO strings stored on the same record

## Limits And Current Notes {#limits-and-current-notes}

A few limits are important to know before adopting it broadly:

- the bundle is built around JSON-backed translated values, not separate translation entities
- the translation API is optional and requires both route import and frontend asset import
- only the Google driver is shipped out of the box
- automatic translation is a helper for editors, not a full content workflow
- the Twig filter and `Translation::translate()` follow the current request locale when available, then fall back to the stored default locale
