---
title: "Dynamic Form Type"
description: "Build Symfony forms from array-based field definitions and extend type resolution when your application needs dynamic form structures."
---

# Dynamic Form Type {#dynamic-form-type}

`softspring/dynamic-form-type` helps you build Symfony forms from array configuration.

It is useful when the form structure does not live in one fixed PHP form class. A common case is a CMS, a configurable admin area, or any feature where fields come from YAML, database records, or business rules built at runtime.

## Why Use It {#why-use-it}

Use this component when:

- the set of fields changes depending on configuration
- you want to store form definitions outside PHP form classes
- you need to reuse the same dynamic form logic in different parts of the application
- you want to keep using normal Symfony form types and validator constraints

This package does not try to replace Symfony forms. It builds on top of them.

## Installation {#installation}

```bash
composer require softspring/dynamic-form-type:^6.0
```

In a Symfony application, enable the bundle if it is not loaded automatically:

```php
<?php

return [
    Softspring\Component\DynamicFormType\SfsDynamicFormTypeBundle::class => ['all' => true],
];
```

## Quick Start {#quick-start}

The main entry point is `DynamicFormType`.

You pass the field definition through the `form_fields` option:

```php
<?php

use Softspring\Component\DynamicFormType\Form\Type\DynamicFormType;
use Symfony\Component\Form\FormBuilderInterface;

final class ContactPayloadForm extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder->add('payload', DynamicFormType::class, [
            'form_fields' => [
                'name' => [
                    'type' => 'text',
                    'type_options' => [
                        'required' => true,
                    ],
                ],
                'message' => [
                    'type' => 'textarea',
                    'type_options' => [
                        'required' => true,
                    ],
                ],
            ],
        ]);
    }
}
```

Each entry in `form_fields` becomes a normal Symfony field.

## The `form_fields` Structure {#the-form-fields-structure}

Each field accepts a small definition object:

```yaml
form_fields:
    title:
        type: text
        type_options:
            required: true
            help: 'Shown in the page header'
```

The usual keys are:

- `type`: the form type to use
- `type_options`: the options passed to that form type

If `type` is omitted, the component falls back to `text`.

## Using Symfony Form Types {#using-symfony-form-types}

Short aliases such as `text`, `textarea`, `checkbox`, or `choice` are resolved to normal Symfony core form types.

That means you can keep using standard Symfony options:

```yaml
form_fields:
    published:
        type: checkbox
        type_options:
            required: false

    category:
        type: choice
        type_options:
            choices:
                News: news
                Blog: blog
                Docs: docs
```

You can also use a fully qualified form type class:

```yaml
form_fields:
    custom:
        type: App\Form\Type\SeoMetaType
```

## Adding Constraints {#adding-constraints}

Validator constraints are added inside `type_options.constraints`.

You can declare them in three useful forms.

### Short Constraint Names {#short-constraint-names}

```yaml
form_fields:
    title:
        type: text
        type_options:
            constraints:
                - notBlank
```

### Constraint With Options {#constraint-with-options}

```yaml
form_fields:
    score:
        type: text
        type_options:
            constraints:
                - constraint: range
                  options:
                      min: 1
                      max: 100
```

### Fully Qualified Constraint Classes {#fully-qualified-constraint-classes}

```yaml
form_fields:
    slug:
        type: text
        type_options:
            constraints:
                - App\Validator\Constraints\ValidSlug
```

If the constraint configuration is invalid, the form fails with a clear configuration error instead of silently ignoring it.

## Collections Of Dynamic Entries {#collections-of-dynamic-entries}

Use `DynamicFormCollectionType` when you need a collection where each entry is itself a dynamic form.

```php
<?php

use Softspring\Component\DynamicFormType\Form\Type\DynamicFormCollectionType;

$builder->add('items', DynamicFormCollectionType::class, [
    'entry_options' => [
        'form_fields' => [
            'label' => [
                'type' => 'text',
            ],
            'value' => [
                'type' => 'text',
            ],
        ],
    ],
]);
```

The collection type enables these defaults:

- `allow_add: true`
- `allow_delete: true`
- `prototype: true`
- `entry_type: DynamicFormType`
- `prototype_initial_elements: 1`

This makes it useful for repeated dynamic blocks, rows, or configurable lists inside the same form.

## Extending Type Resolution {#extending-type-resolution}

This is one of the most valuable parts of the component.

The package does not only resolve Symfony core types. It also supports a chain of custom resolvers.

That lets your application map short names such as `media`, `translation`, `link`, or `color` to your own form types.

Create a resolver that implements `TypeResolverInterface`:

```php
<?php

namespace App\Form\Resolver;

use Softspring\Component\DynamicFormType\Form\Resolver\TypeResolverInterface;

final class AppTypeResolver implements TypeResolverInterface
{
    public function resolveTypeClass(?string $type): ?string
    {
        $class = 'App\\Form\\Type\\'.ucfirst((string) $type).'Type';

        return class_exists($class) ? $class : null;
    }

    public function getPossibleFormClasses(string $type): array
    {
        return [
            'App\\Form\\Type\\'.ucfirst($type).'Type',
        ];
    }
}
```

Then tag it:

```yaml
services:
    App\Form\Resolver\AppTypeResolver:
        tags:
            - { name: 'softspring.dynamic_form_type.type_resolver', priority: 100 }
```

The higher-priority resolvers run first.

## What The Default Resolver Looks For {#what-the-default-resolver-looks-for}

If you do not add your own resolver, the built-in resolver looks in these namespaces:

- `App\Form\Type\...Type`
- `Softspring\Component\DynamicFormType\Form\Type\...Type`
- `Symfony\Component\Form\Extension\Core\Type\...Type`
- `Symfony\Bridge\Doctrine\Form\Type\...Type`

That is enough for many projects, but real applications often add one or more custom resolvers.

## Real Usage In Softspring Packages {#real-usage-in-softspring-packages}

This component is not only theoretical in the Softspring ecosystem.

`cms-bundle` uses it to build configurable admin forms from package and module configuration. For example, content forms and module forms pass `extra_fields`, `indexing`, or `module_options.form_fields` directly into `DynamicFormType`.

`cms-module-collection` shows the pattern clearly. Module YAML files define fields such as:

```yaml
module_options:
    form_fields:
        title:
            type: translation
            type_options:
                type: text

        media:
            type: translatable
            type_options:
                type: mediaVersion
```

Those short names only work because other packages register extra type resolvers on top of this component.

That is the main integration pattern to keep in mind:

1. store field definitions in config
2. pass them to `DynamicFormType`
3. teach the resolver chain how to resolve your application-specific field names

## Recommended Usage Patterns {#recommended-usage-patterns}

This component works best when:

- the configuration source is trusted and controlled by developers or admins
- you keep the field schema predictable
- custom field aliases are documented in your application
- you use resolvers to keep configuration short and readable

It is especially useful in:

- CMS block and module editors
- configurable settings screens
- metadata forms
- plugin-driven admin forms

## Limits {#limits}

Keep these limits in mind:

- the component expects valid array configuration
- it does not provide a visual form builder UI
- it does not validate your business schema before building the Symfony form
- advanced behavior still depends on the form types and constraints available in the application

In other words, it is a strong building block for dynamic forms, not a no-code form platform.

## Summary {#summary}

Choose `dynamic-form-type` when you want Symfony forms driven by configuration instead of one fixed PHP class, and especially when your application benefits from custom type aliases resolved through tagged resolvers.
