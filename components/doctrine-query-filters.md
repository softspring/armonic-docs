---
title: "Doctrine Query Filters"
description: "Apply reusable filter and sorting rules to Doctrine query builders and use a lightweight Symfony form base for filter screens."
---

# Doctrine Query Filters {#doctrine-query-filters}

`softspring/doctrine-query-filters` helps you apply filtering and sorting rules to Doctrine query builders without rewriting the same repository conditions for every listing screen.

It is useful when your application has many admin listings, backoffice searches, or filter forms that follow the same patterns.

## Why Use It {#why-use-it}

Use this component when:

- filter inputs already map naturally to a small filter grammar
- many screens need the same kinds of conditions such as `like`, `in`, `between`, or null checks
- you want a thin bridge between a Symfony form and a Doctrine `QueryBuilder`
- you want reusable sorting support, including joined properties

This package does not try to become a full search engine or a full specification framework. It stays small and query-builder oriented.

## Installation {#installation}

```bash
composer require softspring/doctrine-query-filters:^6.0
```

## The Core Idea {#the-core-idea}

The package mainly revolves around:

- `Filters::apply()`
- `Filters::sortBy()`
- `FiltersForm`

The idea is simple:

1. represent filters as an array
2. apply them to a Doctrine `QueryBuilder`
3. optionally drive that array from a Symfony GET form

## Applying Filters Manually {#applying-filters-manually}

The lowest-level API is `Filters::apply()`:

```php
<?php

use Softspring\Component\DoctrineQueryFilters\Filters;

$qb = $repository->createQueryBuilder('p');

Filters::apply($qb, [
    'name__like' => 'john',
    'status__in' => ['active', 'pending'],
]);
```

This lets you keep the filtering rules close to your controller or service without manually building every expression.

## Filter Naming Grammar {#filter-naming-grammar}

The filter name includes the field and, optionally, the operator.

Examples:

- `name`
- `name__like`
- `status__in`
- `price__between`
- `deletedAt__null`

If there is no suffix operator, the component uses simple equality.

So:

```php
[
    'status' => 'active',
]
```

means:

```sql
status = :value
```

## Supported Operators {#supported-operators}

The current implementation supports:

- equality by default
- `like`
- `in`
- `notIn`
- `between`
- `lt`
- `lte`
- `gt`
- `gte`
- `null`
- `is`

### `like` {#like}

```php
[
    'name__like' => 'john',
]
```

This becomes a `%value%` search.

### `in` And `notIn` {#in-and-notin}

```php
[
    'status__in' => ['draft', 'published'],
]
```

Use these when the UI already gives you a set of accepted values.

### `between` {#between}

```php
[
    'createdAt__between' => ['2024-01-01', '2024-12-31'],
]
```

This is useful for date ranges and numeric ranges.

### Null Checks {#null-checks}

```php
[
    'deletedAt__null' => true,
]
```

This means `IS NULL`.

If the value is `false`, it becomes `IS NOT NULL`.

The `is` operator accepts:

- `null`
- `'null'`
- `'not_null'`

## OR Filters {#or-filters}

The package supports OR groups by combining field definitions with `___or___`.

Example:

```php
[
    'name__like___or___surname__like' => 'john',
]
```

This is useful for quick search fields where one input should search multiple columns.

## Filtering Joined Fields {#filtering-joined-fields}

You can filter joined properties by using dot notation:

```php
[
    'owner.name__like' => 'john',
]
```

The component will add a left join automatically when needed.

That makes it practical for admin screens that need small joined filters without custom join boilerplate in every controller.

## Sorting Results {#sorting-results}

`Filters::sortBy()` applies order definitions:

```php
<?php

Filters::sortBy($qb, [
    'name' => 'asc',
]);
```

Joined-field sorting is also supported:

```php
<?php

Filters::sortBy($qb, [
    'owner.name' => 'asc',
]);
```

Just like filtering, the helper adds the join automatically when necessary.

## Using FiltersForm {#using-filtersform}

`FiltersForm` is a lightweight base type for GET filter forms.

It already sets useful defaults:

- `method: GET`
- `csrf_protection: false`
- `required: false`
- `allow_extra_fields: true`

It also normalizes a `query_builder` option so the form can carry its own base query builder definition.

### Example Filter Form {#example-filter-form}

```php
<?php

use Softspring\Component\DoctrineQueryFilters\FiltersForm;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;

class CustomerFilterForm extends FiltersForm
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder->add('search', TextType::class, [
            'property_path' => '[name__like___or___email__like]',
        ]);
    }
}
```

This is a simple and effective pattern for listing pages:

- the form field name can stay friendly
- the submitted data shape still maps directly to filter rules

## Defining The Base Query Builder {#defining-the-base-query-builder}

`FiltersForm` expects a `class` option and can resolve the query builder in three ways:

- from a provided `QueryBuilder`
- from a callable that receives the repository
- from the entity repository of the given class

That means you can keep default query setup close to the form if that improves reuse.

## Recommended Usage Patterns {#recommended-usage-patterns}

This component works best when:

- filter names are stable and well documented
- listing screens follow repeatable rules
- the project wants a small utility, not a large query abstraction layer
- filter forms are GET forms and naturally map to query string parameters

It is a good fit for:

- admin listings
- moderate backoffice search screens
- CRUDL filters
- reusable filters shared across several controllers

## Practical Limits {#practical-limits}

Keep these limits in mind:

- the filter grammar is string-based
- complex nested logic still belongs in custom query code
- the package is good for repeatable filtering rules, not for every possible query shape
- some older form-integration ideas are still visible in the codebase, but the stable public value today is the query-builder helper and the base form type

## Summary {#summary}

Choose `doctrine-query-filters` when you want small, repeatable Doctrine filtering and sorting rules driven by arrays or GET forms, especially for admin and backoffice listing screens.
