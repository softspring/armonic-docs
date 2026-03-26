---
title: "Doctrine Templates"
description: "Reuse common Doctrine embeddables, entity traits, and matching Symfony form types for addresses, contact cards, ids, slugs, and timestamps."
---

# Doctrine Templates {#doctrine-templates}

`softspring/doctrine-templates` is a small toolbox of reusable Doctrine model pieces.

It is useful when many entities in an application repeat the same kinds of fields: ids, names, slugs, timestamps, addresses, or simple contact-card data.

Instead of rewriting those pieces in each project, this component gives you embeddables, traits, interfaces, and form types that you can compose into your own entities.

## Why Use It {#why-use-it}

Use this package when:

- several entities need the same small field sets
- you want a reusable `Address` or `HCard` embeddable
- you want matching Symfony form types for those embeddables
- you prefer lightweight traits over a bigger shared base entity hierarchy

This package does not try to define your full domain model. It gives you reusable building blocks.

## Installation {#installation}

```bash
composer require softspring/doctrine-templates:^6.0
```

## What The Package Provides {#what-the-package-provides}

The component has three main parts:

- embeddables for structured address and contact data
- traits for recurring Doctrine fields and ids
- Symfony form types for editing those embeddables

## Embeddables {#embeddables}

The package includes these embeddables:

- `Address`
- `HAddress`
- `HCard`

They implement matching interfaces and are designed to be embedded into your own entities.

### Address {#address}

`Address` stores:

- `postOfficeBox`
- `streetAddress`
- `extendedAddress`
- `locality`
- `region`
- `postalCode`
- `countryCode`

Typical usage:

```php
<?php

use Doctrine\ORM\Mapping as ORM;
use Softspring\Component\DoctrineTemplates\Entity\Embeddable\Address;

#[ORM\Entity]
class Customer
{
    #[ORM\Embedded(class: Address::class)]
    protected Address $billingAddress;
}
```

### HAddress And HCard {#haddress-and-hcard}

`HAddress` extends the address contract and `HCard` adds:

- `name`
- `surname`
- `tel`

Use `HCard` when you want contact-style data stored together with the address.

## Form Types {#form-types}

The package includes:

- `AddressType`
- `HAddressType`
- `HCardType`

These form types are useful when your entity embeds one of the provided value objects and you want a ready-to-use form layer.

### AddressType {#addresstype}

`AddressType` builds a practical address form and adds useful browser autocomplete hints.

Typical usage:

```php
<?php

use Softspring\Component\DoctrineTemplates\Form\AddressType;

$builder->add('billingAddress', AddressType::class);
```

By default it includes:

- `streetAddress`
- `extendedAddress`
- `postalCode`
- `locality`
- `region`
- `countryCode`

It hides `postOfficeBox` unless you enable it.

### Useful AddressType Options {#useful-addresstype-options}

`AddressType` includes a few practical options:

- `address_lines`
- `show_post_office_box`
- `show_country`
- `country_choices`
- `autocomplete_section`

Example:

```php
<?php

$builder->add('shippingAddress', AddressType::class, [
    'address_lines' => 1,
    'show_post_office_box' => true,
    'country_choices' => ['ES', 'FR', 'PT'],
    'autocomplete_section' => 'shipping',
]);
```

This is useful when:

- you want a single street-address line
- you only support a small set of countries
- you want clean browser autocomplete for multiple address groups in the same page

### HCardType {#hcardtype}

`HCardType` builds on top of `AddressType` and adds:

- `name`
- `surname`
- `tel`

That makes it a good fit for contact persons, shipping contacts, or business contact data stored as an embeddable inside a larger entity.

## Traits For Entity Fields {#traits-for-entity-fields}

The package also provides many reusable traits for common entity fields.

The important point is that these traits represent different modeling choices. You should pick only the ones that fit your entity.

### Identity Traits {#identity-traits}

Examples:

- `AutoId`
- `Guid`
- `KeyIdTrait`
- `UniqId`
- `UniqIdString`
- `ToStringIdTrait`

Use them when you want a ready-made id strategy without rewriting the same field and accessor in many entities.

Do not combine multiple id traits in the same entity.

### Naming And Slug Traits {#naming-and-slug-traits}

Examples:

- `Named`
- `NamedString`
- `SlugTrait`
- `CurrencyTrait`

These are simple traits for small recurring fields.

They are useful in admin-heavy applications where many entities need the same handful of columns.

### Timestamp Traits {#timestamp-traits}

The package offers two timestamp styles:

- `TimestampMarks`: stores `DateTime` values
- `Timestamps`: stores Unix timestamps and exposes them as `DateTime`

That means the package supports both styles, but you should choose one approach per entity model.

Do not mix both timestamp systems blindly in the same entity tree.

## Recommended Usage Pattern {#recommended-usage-pattern}

This component works best when you treat it as a toolbox.

A good pattern is:

1. choose one id strategy
2. choose whether the entity needs a reusable embeddable such as `Address` or `HCard`
3. add only the small field traits that really belong to the entity
4. use the matching Symfony form type where it saves time

That keeps your entity readable and avoids trait combinations that fight each other.

## Extending The Package {#extending-the-package}

The safest way to extend this package is composition, not modification.

Good extension patterns are:

- create your own embeddable that reuses one of the traits
- extend `AddressType` or `HCardType` when you need extra fields or project-specific options
- implement your own interfaces or validation rules on top of the provided model contracts

This is usually better than forking the package just to add a few project-specific fields.

## Practical Limits {#practical-limits}

Keep these limits in mind:

- the package gives you templates, not a complete persistence architecture
- some traits are alternatives to each other, not pieces to stack together blindly
- validation rules are mostly your responsibility
- lifecycle strategy still belongs to your application and Doctrine mapping decisions

In short: it saves repetition, but it does not remove the need to design your own domain model carefully.

## Summary {#summary}

Choose `doctrine-templates` when your application repeatedly needs the same Doctrine embeddables, id styles, timestamp traits, and matching form types, and you want to reuse those pieces without building a bigger abstract model layer.
