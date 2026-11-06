---
title: "Form Schema"
description: "Extract form schemas from Symfony forms and extend field extraction for application-specific types."
---

# Form Schema {#form-schema}

`softspring/form-schema` extracts JSON-schema-like metadata from Symfony forms.

This package is still in active development. Its public API, extracted schema shape, and extension points may change before the first stable release.

## What It Provides {#what-it-provides}

- Schema extraction from a Symfony `FormInterface`, form type class, or form type instance.
- Field extractors for scalar fields, compound fields, collections, choices, and buttons.
- Support for explicit `json_schema` and `json_scheme` form options.
- Basic metadata extraction from labels, help text, defaults, and Symfony validator constraints.
- A Symfony bundle that registers the extractor services and field extractor tags.

## Installation {#installation}

```bash
composer require softspring/form-schema:^6.0@dev
```

If Symfony Flex does not register the bundle automatically, add it manually:

```php
// config/bundles.php
return [
    Softspring\Component\FormSchema\SfsFormSchemaBundle::class => ['all' => true],
];
```

## Basic Usage {#basic-usage}

Inject `SchemaExtractor` and extract a schema from a form type:

```php
use Softspring\Component\FormSchema\Schema\SchemaExtractor;

final class ExampleService
{
    public function __construct(private SchemaExtractor $schemaExtractor)
    {
    }

    public function schema(): array
    {
        return $this->schemaExtractor->extract(ExampleFormType::class);
    }
}
```

You can also pass a resolved `FormInterface` when the form must be created with application-specific options.

## Custom Field Extractors {#custom-field-extractors}

Create a service implementing `FieldSchemaExtractorInterface` and tag it with `softspring.form_schema.field_extractor`.

Extractors are evaluated in service order. Return `true` from `supports()` only for the fields your extractor owns.
