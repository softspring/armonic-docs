---
title: "Doctrine Changelog Component"
description: "Capture Doctrine entity changes, enrich them with request and user context, and persist or forward them through Doctrine or BigQuery."
---

# Doctrine Changelog Component {#doctrine-changelog-component}

`softspring/doctrine-changelog-bundle` is a technical component for Symfony applications that need an audit trail or a change stream based on Doctrine entity lifecycle changes.

It listens to Doctrine `onFlush`, creates change entries for tracked entities, enriches them with context, and then either stores them with a driver or leaves them available for your own listeners.

## What It Provides {#what-it-provides}

- change detection for entity insertions, updates, and deletions
- registrable and ignored markers for entity classes and fields
- change events that application code can subscribe to
- optional collectors for request, user, and action metadata
- an in-memory `ChangesStack`
- storage drivers for Doctrine and BigQuery

This package is infrastructure, not UI. It does not ship routes, screens, or backoffice pages.

## Install {#install}

```bash
composer require softspring/doctrine-changelog-bundle:6.0.x-dev
```

If your application does not use Symfony Flex, enable the bundle manually:

```php
<?php

return [
    // ...
    Softspring\DoctrineChangeLogBundle\SfsDoctrineChangeLogBundle::class => ['all' => true],
];
```

## Basic Configuration {#basic-configuration}

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_doctrine_change_log.yaml
sfs_doctrine_change_log:
    collect:
        request: true
        user: true
        action: true

    storage:
        enabled: false
        driver: doctrine
```

```php
<?php

// config/packages/sfs_doctrine_change_log.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_doctrine_change_log', [
        'collect' => ['request' => true, 'user' => true, 'action' => true],
        'storage' => ['enabled' => false, 'driver' => 'doctrine'],
    ]);
};
```

<!-- config-tabs:end -->

The current schema requires a `driver` even when storage is disabled. The value is not used until storage is enabled.

That setup already gives you change events and an in-memory stack. It does not persist anything yet.

## Which Entities Are Tracked {#which-entities-are-tracked}

Only entities marked as registrable are tracked.

Use PHP attributes in 6.0:

```php
<?php

namespace App\Entity;

use Softspring\DoctrineChangeLogBundle\Mapping as ChangeLog;

#[ChangeLog\Registrable([])]
class Product
{
    #[ChangeLog\Ignored([])]
    private string $updatedByCache;
}
```

Ignored fields are removed from the Doctrine change set before the changelog event is dispatched.

## Runtime Flow {#runtime-flow}

At runtime, the component works in four steps:

1. `DoctrineChangesListener` reads scheduled entity insertions, updates, and deletions in `onFlush`
2. it builds one `ChangeEntry` per tracked entity change
3. collector listeners enrich that entry with metadata
4. if storage is enabled, `ChangesPersistListener` persists the stacked entries on `kernel.terminate`

This means you can use the component in two ways:

- as a complete changelog persistence layer
- as an internal event source that your own application code consumes

## Collected Metadata {#collected-metadata}

The optional collectors add:

- request data
  - IP, user agent, method, path
- user data
  - current authenticated user identifier
- action
  - `insertion`, `update`, or `deletion`

These are controlled by the `collect` block and are independent from the actual Doctrine change detection.

## Storage Modes {#storage-modes}

### Disabled Storage {#disabled-storage}

When `storage.enabled` is `false`, the component still dispatches change events and fills the stack, but it does not persist them.

This is useful when your application wants to forward changes to its own queue, stream, or audit service.

### Doctrine Storage {#doctrine-storage}

<!-- config-tabs:start -->

```yaml
sfs_doctrine_change_log:
    storage:
        enabled: true
        driver: doctrine
```

```php
<?php

// config/packages/sfs_doctrine_change_log.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_doctrine_change_log', ['storage' => ['enabled' => true, 'driver' => 'doctrine']]);
};
```

<!-- config-tabs:end -->

The default Doctrine driver expects `App\Entity\ChangeLog`. Define a mapped entity extending its mapped superclass:

```php
<?php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Softspring\DoctrineChangeLogBundle\Entity\ChangeLog as ChangeLogModel;

#[ORM\Entity]
class ChangeLog extends ChangeLogModel
{
}
```

Generate and review the application's migration for this table. Override the storage driver service argument if you use another entity name. Do not mark the changelog entity itself as registrable.

This is a good fit when:

- you want a local audit table
- audit queries stay in the same relational database
- operational simplicity matters more than warehouse-style analytics

### BigQuery Storage {#bigquery-storage}

<!-- config-tabs:start -->

```yaml
sfs_doctrine_change_log:
    storage:
        enabled: true
        driver: big-query
        big_query:
            project: '%env(BIGQUERY_PROJECT)%'
            dataset: '%env(BIGQUERY_DATASET)%'
            table:
                mode: fixed
                fixed:
                    name: changelog
                attribute:
                    attribute_name: changelog_table
```

```php
<?php

// config/packages/sfs_doctrine_change_log.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_doctrine_change_log', [
        'storage' => [
            'enabled' => true,
            'driver' => 'big-query',
            'big_query' => [
                'project' => '%env(BIGQUERY_PROJECT)%',
                'dataset' => '%env(BIGQUERY_DATASET)%',
                'table' => [
                    'mode' => 'fixed',
                    'fixed' => ['name' => 'changelog'],
                    'attribute' => ['attribute_name' => 'changelog_table'],
                ],
            ],
        ],
    ]);
};
```

<!-- config-tabs:end -->

The current schema also requires `table.attribute.attribute_name` in fixed mode; it only affects table selection when `mode: attribute` is used. Install `google/cloud-bigquery` and configure runtime credentials before enabling this driver. The example supplies no credential file or client data.

BigQuery is the better fit when:

- changes should be exported to a warehouse
- analytics and reporting matter more than local ORM access
- the audit stream belongs to a broader data platform

## Practical Integration Advice {#practical-integration-advice}

- use registrable markers only on entities whose changes have business value
- ignore noisy cache or denormalized fields
- start with storage disabled if you first want to validate the event stream
- use Doctrine storage for audit tables and BigQuery for analytics pipelines
- keep collection changes explicit through relation entities, because this component tracks entity changes, not generic collection diffs

## Extension Points {#extension-points}

The main extension points are:

- custom listeners for `InsertionChangeEvent`, `UpdateChangeEvent`, and `DeletionChangeEvent`
- custom storage drivers implementing `StorageDriverInterface`
- custom collector listeners that enrich `ChangeEntry`
- custom changelog entity classes when using Doctrine storage

This is where the component adds value: it standardizes the change capture layer and leaves persistence and downstream usage open.
