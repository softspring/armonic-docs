---
title: "Doctrine Migrations Comparator"
description: "Use a custom Doctrine Migrations comparator that sorts migrations by version class name instead of relying on the default comparator behavior."
---

# Doctrine Migrations Comparator {#doctrine-migrations-comparator}

`softspring/doctrine-migrations-version-comparator` provides a custom comparator for Doctrine Migrations.

Its goal is simple: sort migrations by their version class name.

This is useful in projects where migration class names already carry the intended ordering, and you want Doctrine Migrations to follow that naming consistently.

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

<!-- config-tabs:start -->

```yaml
services:
    Softspring\Component\DoctrineMigrationsVersionComparator\VersionNumberComparator: ~

doctrine_migrations:
    services:
        'Doctrine\Migrations\Version\Comparator': 'Softspring\Component\DoctrineMigrationsVersionComparator\VersionNumberComparator'
```

```php
<?php

// config/services.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $services = $container->services();

    $services->set(\Softspring\Component\DoctrineMigrationsVersionComparator\VersionNumberComparator::class);

    $container->extension('doctrine_migrations', [
        'services' => [
            'Doctrine\\Migrations\\Version\\Comparator' => \Softspring\Component\DoctrineMigrationsVersionComparator\VersionNumberComparator::class,
        ],
    ]);
};
```

<!-- config-tabs:end -->

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

## Dependency-aware applications {#dependency-aware-applications}

A chronological comparator is appropriate only when version names encode the required order. It does not understand dependencies between migration namespaces.

Applications such as Sylius can install their own dependency-aware comparator. Keep that comparator and declare the additional migration dependencies instead of replacing it with this component. See [Install Armonic in Sylius](../getting-started/install-sylius-project.md).

Review the SQL produced by every migration before deployment. Changing the comparator does not repair missing migrations, incompatible SQL, or an already partially migrated database.
