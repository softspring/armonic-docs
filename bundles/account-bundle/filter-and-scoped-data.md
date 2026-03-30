---
title: "Filter And Scoped Data"
description: "Scope account-owned data automatically with the Account Bundle Doctrine filter and account-aware write helpers."
---

# Filter And Scoped Data {#filter-and-scoped-data}

This is one of the most practical parts of `account-bundle`: turning the current account route context into automatic read and write scoping for account-owned entities.

## The Usual Account-Scoped Pattern {#the-usual-account-scoped-pattern}

The normal flow is:

1. the route contains `/{_account}`
2. the bundle resolves that value to the current account
3. Doctrine enables the `account` filter for the request
4. entities implementing `AccountFilterInterface` are limited to the current account automatically

That means your controllers can often load data normally and still remain scoped to the current account.

## Requirements For Automatic Filtering {#requirements-for-automatic-filtering}

For automatic filtering to work:

- the route must carry the current account, usually as `/{_account}`
- the entity must implement `AccountFilterInterface`
- the entity must expose an account relation compatible with the bundled filter
- `sfs_account.filter.enabled` must be true

## SQL Filter Assumption {#sql-filter-assumption}

The bundled SQL filter adds a condition like:

```sql
<table_alias>.account_id = :_account
```

So the default implementation assumes:

- the entity has an `account_id` column
- the current account should be matched against that column

If your mapping uses another column name or another ownership strategy, treat the bundled filter as a starting point and replace it.

## Automatic Account On PrePersist {#automatic-account-on-prepersist}

`AccountFilteredEventListener` also helps on writes.

When an entity:

- implements `AccountFilterInterface`
- does not already have an account
- also implements `SingleAccountedInterface` or `AccountRelatedInterface`

the listener copies the current request account into the entity during Doctrine `prePersist`.

This is useful for account-aware create flows:

```php
$project = new Project();
$project->setName('Intranet redesign');

$entityManager->persist($project);
$entityManager->flush();
```

If the request already resolved `/_account`, the entity can receive that account automatically.

## Good Use Cases {#good-use-cases}

This pattern works especially well for:

- projects
- invoices
- subscriptions
- customer records
- provider resources
- account-specific settings

These are entities that belong clearly to one account and should never leak across account contexts.

## Limitations To Keep In Mind {#limitations-to-keep-in-mind}

- the bundled filter expects `account_id`
- some runtime helpers still read `_account` directly
- complex ownership models may need a custom filter or custom persistence logic

The bundle removes a lot of repeated `andWhere('entity.account = :account')`, but it is still a base implementation, not a universal multitenancy engine.

## Related Guides {#related-guides}

- [Account Bundle Overview](../account-bundle.md)
- [Model And Entities](./model-and-entities.md)
- [Current Account And Routes](./current-account-and-routes.md)
- [Admin And Security](./admin-and-security.md)
- [Extend And Customize](./extend-and-customize.md)
