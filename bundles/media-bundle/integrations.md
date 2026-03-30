---
title: "Media Bundle Integrations"
description: "Understand the practical integrations around Media Bundle, including migrations, CMS translation fields, and type providers."
---

# Integrations {#integrations}

`media-bundle` is not isolated. Several practical behaviors depend on other parts of the Softspring ecosystem or on Symfony and Doctrine integration points.

## Doctrine Migrations Path {#doctrine-migrations-path}

The bundle prepends its own migrations path into Doctrine Migrations:

- `Softspring\MediaBundle\Migrations` => `@SfsMediaBundle/src/Migrations`

This matters when you rely on bundle-provided schema migrations and want those namespaces available automatically.

## CMS Translation Field Caveat {#cms-translation-field-caveat}

`MediaTypeUploadType` uses `Softspring\CmsBundle\Form\Type\TranslationType` for `altTexts`.

If your project does not expose that form type, disable the field or replace the form:

```php
$builder->add('media', MediaTypeUploadType::class, [
    'media_type' => 'article_image',
    'allow_alt_text_field' => false,
]);
```

This is one of the few places where the bundle assumes an additional Softspring form integration.

## Type Configuration Providers {#type-configuration-providers}

The default provider reads `sfs_media.types`, but the architecture supports extra `MediaTypeProviderInterface` services.

That matters when:

- several modules need to contribute media types
- you want to keep type config close to the feature that uses it
- one large YAML file is becoming hard to maintain

## Permissions And Admin Integration {#permissions-and-admin-integration}

The admin media library is designed to work with:

- `softspring/crudl-controller`
- `softspring/permissions-bundle`

That is why route imports and role hierarchy imports are part of the practical installation path.

## Frontend Integration {#frontend-integration}

The bundle also ships:

- Twig rendering helpers
- form themes
- admin media templates
- frontend assets for media type interactions

It works best when your application treats the bundle as the shared media layer instead of reimplementing upload, versioning, and rendering rules in each feature.

## Related Guides {#related-guides}

- [Media Bundle Overview](../media-bundle.md)
- [Using Medias](./using-medias.md)
- [Admin Medias](./admin-medias.md)
- [Extending Bundle](./extending-bundle.md)
- [Name Generators](./name-generators.md)
