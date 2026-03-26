---
title: "Doctrine Migrations Comparator"
description: "Use a custom Doctrine Migrations comparator that sorts migrations by version class name instead of relying on the default comparator behavior."
---

# Doctrine Migrations Comparator {#doctrine-migrations-comparator}

`softspring/doctrine-migrations-version-comparator` provides a custom comparator for Doctrine Migrations.

Its goal is simple: sort migrations by their version class name.

This is useful in projects where migration class names already carry the intended ordering, and you want Doctrine Migrations to follow that naming consistently.

## When To Use It {#when-to-use-it}

Use this component when:

- your project uses migration class names with sortable numeric suffixes
- you want a predictable comparator based on the migration version name
- you want the comparator wired once through Symfony configuration

It is a very small component, but it helps keep migration ordering behavior explicit.

## Installation {#installation}

```bash
composer require softspring/doctrine-migrations-version-comparator:^6.0
```

## What It Changes {#what-it-changes}

Doctrine Migrations resolves migration versions and needs a comparator to sort them.

This package replaces the default comparator service with `VersionNumberComparator`, which compares migrations using the migration class short name.

That means namespace differences do not become the main sorting signal when the actual version naming lives in the class name.

## Quick Start {#quick-start}

Register the service and wire it into Doctrine Migrations:

```yaml
services:
    Softspring\Component\DoctrineMigrationsVersionComparator\VersionNumberComparator: ~

doctrine_migrations:
    services:
        'Doctrine\Migrations\Version\Comparator': 'Softspring\Component\DoctrineMigrationsVersionComparator\VersionNumberComparator'
```

That is the full integration used in the recipe shipped with the package.

## Symfony Recipe {#symfony-recipe}

The package recipe adds:

- the comparator service definition
- the `doctrine_migrations.services` override

In practice, the package is meant to be almost configuration-free after installation.

## How It Compares Versions {#how-it-compares-versions}

The comparator tries to resolve the migration class through reflection and compares the short class name.

Example idea:

```php
Version20260325000100
Version20260325000200
```

That keeps the ordering tied to the version naming pattern you already use in migration classes.

If reflection is not possible, the comparator falls back to comparing the raw version strings so the result stays deterministic.

## Typical Usage In Symfony Projects {#typical-usage-in-symfony-projects}

This component is usually not used directly in application code.

Instead, it is part of the project infrastructure:

1. install the package
2. register the comparator service
3. point `Doctrine\Migrations\Version\Comparator` to it
4. let Doctrine Migrations use it during migration ordering

That means the value of the package is not in a public API you call manually, but in keeping migration ordering behavior consistent across the project.

## When It Adds Value {#when-it-adds-value}

This package is a good fit when your team:

- cares about explicit migration ordering rules
- uses a stable migration naming convention
- wants the comparator behavior to be visible in Symfony config instead of implicit

It is especially useful in projects with many bundle migrations where keeping version ordering predictable matters.

## Practical Limits {#practical-limits}

Keep these limits in mind:

- it only changes comparison behavior
- it does not generate migrations
- it does not validate migration contents
- it is only relevant if your project actually benefits from name-based migration ordering

## Summary {#summary}

Choose this component when you want Doctrine Migrations to follow your migration class version naming explicitly through a dedicated comparator service.
