---
title: "Polymorphic Form Type"
description: "Build Symfony collections where each item can use a different form type and a different PHP class."
---

# Polymorphic Form Type {#polymorphic-form-type}

`softspring/polymorphic-form-type` solves one specific Symfony form problem: a collection where each item may use a different form type and a different PHP class.

Use it when one list can contain several node types, such as:

- product properties with different structures
- block or module editors
- rule builders
- workflows made of different step types

This component is the base building block that later packages such as `cms-bundle` reuse to build richer editors.

## Installation {#installation}

```bash
composer require softspring/polymorphic-form-type:^6.0
```

## The Main Idea {#the-main-idea}

Symfony `CollectionType` works well when every item uses the same form type.

This component is for the opposite case:

```text
Entity form
  -> properties: PolymorphicCollectionType
       -> size node     -> SizePropertyType
       -> weight node   -> WeightPropertyType
       -> category node -> CategoryPropertyType
```

Each submitted item carries a discriminator field. The component reads that discriminator and decides:

- which child form type to build
- which PHP class to instantiate
- which prototype to expose to the frontend

## The Two Collection Types {#the-two-collection-types}

The package provides two collection types.

### PolymorphicCollectionType {#polymorphiccollectiontype}

Use `Softspring\Component\PolymorphicFormType\Form\Type\PolymorphicCollectionType` when you want full control through explicit maps.

This is the right choice for:

- plain PHP objects
- arrays
- custom models not backed by Doctrine inheritance

### DoctrinePolymorphicCollectionType {#doctrinepolymorphiccollectiontype}

Use `Softspring\Component\PolymorphicFormType\Form\Type\DoctrinePolymorphicCollectionType` when the collection items are Doctrine entities in an inheritance hierarchy.

This variant resolves classes through Doctrine metadata and can reload existing entities by id during submit.

## Create Node Types {#create-node-types}

Every node form type must extend:

`Softspring\Component\PolymorphicFormType\Form\Type\Node\AbstractNodeType`

That base class adds the internal hidden fields needed by the collection:

- the discriminator field, `_node_discr` by default
- the id field, when the Doctrine variant needs it

Your node type should focus on the real business fields:

```php
use Softspring\Component\PolymorphicFormType\Form\Type\Node\AbstractNodeType;
use Symfony\Component\Form\FormBuilderInterface;

class SizePropertyType extends AbstractNodeType
{
    protected function buildChildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder->add('length');
        $builder->add('width');
        $builder->add('height');
    }
}
```

## Use PolymorphicCollectionType {#use-polymorphiccollectiontype}

The non-Doctrine collection type needs three pieces of information:

- `types_map`: discriminator => form type
- `discriminator_map`: discriminator => PHP class
- `form_factory`: the Symfony form factory

Example:

```php
use App\Form\Type\CategoryPropertyType;
use App\Form\Type\SizePropertyType;
use App\Form\Type\WeightPropertyType;
use App\Model\Properties\Category;
use App\Model\Properties\Size;
use App\Model\Properties\Weight;
use Softspring\Component\PolymorphicFormType\Form\Type\PolymorphicCollectionType;

$builder->add('properties', PolymorphicCollectionType::class, [
    'form_factory' => $options['form_factory'],
    'types_map' => [
        'size' => SizePropertyType::class,
        'weight' => WeightPropertyType::class,
        'category' => CategoryPropertyType::class,
    ],
    'discriminator_map' => [
        'size' => Size::class,
        'weight' => Weight::class,
        'category' => Category::class,
    ],
]);
```

### What Happens On Submit {#what-happens-on-submit}

For each submitted node:

1. the listener reads `_node_discr`
2. the matching child form type is added
3. the transformer creates the right object class
4. the submitted fields are mapped back into that object

That means the collection can return a mixed list of objects after one normal form submit.

## Use The Doctrine Variant {#use-the-doctrine-variant}

When nodes are Doctrine entities using inheritance, use `DoctrinePolymorphicCollectionType`.

In this case you configure:

- `abstract_class`
- `types_map`
- `entity_manager`

Example:

```php
use App\Entity\Property\AbstractProperty;
use App\Form\Type\CategoryPropertyType;
use App\Form\Type\SizePropertyType;
use App\Form\Type\WeightPropertyType;
use Softspring\Component\PolymorphicFormType\Form\Type\DoctrinePolymorphicCollectionType;

$builder->add('properties', DoctrinePolymorphicCollectionType::class, [
    'abstract_class' => AbstractProperty::class,
    'entity_manager' => $entityManager,
    'types_map' => [
        'size' => SizePropertyType::class,
        'weight' => WeightPropertyType::class,
        'category' => CategoryPropertyType::class,
    ],
]);
```

The Doctrine variant uses `_node_id` by default so existing entities can be found again during submit.

## Important Options {#important-options}

Common options:

- `types_map`
- `types_options`
- `discriminator_map`
- `form_factory`
- `discriminator_field`
- `id_field`
- `allow_add`
- `allow_delete`
- `prototype_name`

Doctrine-only options:

- `abstract_class`
- `entity_manager`

### Use Types Options Per Node Type {#use-types-options-per-node-type}

`types_options` lets you pass different options to each node form type.

Example:

```php
'types_options' => [
    'size' => [
        'prototype_button_label' => 'Add size field',
    ],
    'weight' => [
        'prototype_button_label' => 'Add weight field',
    ],
]
```

This is especially useful when:

- prototypes need different button labels
- one node type needs more options than another
- you want to pass UI metadata per discriminator

This is exactly the pattern used by `cms-bundle` to build module collections.

## Prototypes And Frontend Integration {#prototypes-and-frontend-integration}

When `allow_add` and `prototype` are enabled, the collection view exposes one prototype per discriminator in `form.vars.prototypes`.

That gives the frontend enough information to render different “add node” buttons and insert the correct subform structure for each node type.

The package also ships a Twig theme:

`@polymorphic-form-theme.html.twig`

This theme renders:

- the collection wrapper
- one button per prototype
- the encoded prototype markup used by frontend add actions

## How Other Packages Extend It {#how-other-packages-extend-it}

The best real example in this repository is `cms-bundle`.

`ModuleCollectionType` extends `PolymorphicCollectionType` and adds:

- module-specific discriminator maps
- grouped prototypes
- translated prototype button labels
- per-module options
- custom data mapping

That shows the intended extension pattern:

1. keep the base polymorphic mechanics here
2. build domain-specific collection types on top

## Recommended Usage Patterns {#recommended-usage-patterns}

This component is a good fit when:

- each item type has different fields
- you want one collection field in one form
- the discriminator is stable and explicit
- your application already has some frontend logic to add items dynamically

Good examples:

- content blocks
- rule nodes
- heterogeneous settings lists
- configurable product properties

## Limits And Current Notes {#limits-and-current-notes}

A few limits matter in practice:

- the plain variant needs both explicit class and form type maps
- the Doctrine variant only supports one identifier field
- the component resolves backend form structure, but your application still needs frontend behavior to insert prototypes dynamically
- every node type must extend `AbstractNodeType`, otherwise the internal discriminator and id workflow is not present
