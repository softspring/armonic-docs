---
title: "Permissions Component"
description: "Enable PERMISSION_* attributes as first-class Symfony security checks and combine them with role hierarchy and custom voters."
---

# Permissions Component {#permissions-bundle}

`softspring/permissions-bundle` is a small security component packaged as a Symfony bundle.

Its purpose is to make attributes that start with `PERMISSION_` work like hierarchical security checks in Symfony.

That sounds small, but it creates a consistent authorization model across the Softspring ecosystem:

- application roles stay as `ROLE_*`
- action-level permissions use `PERMISSION_*`
- reusable packages can publish stable permission names in `role_hierarchy`
- applications can aggregate those permissions into business roles
- custom voters can still deny or refine access for concrete subjects

In practice, this is what makes checks such as `is_granted('PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_UPDATE', $media)` behave as expected.

## Why Treat It As A Component {#why-treat-it-as-a-component}

This package is better understood as a component than as an application-facing bundle because it does not provide:

- screens
- routes
- controllers
- persistence
- admin UI
- end-user workflows

It contributes one infrastructure capability to Symfony Security and is meant to be reused by higher-level bundles and applications.

## Installation {#installation}

```bash
composer require softspring/permissions-bundle:^6.0
```

If your application does not use Symfony Flex, enable it manually:

```php
<?php

return [
    // ...
    Softspring\PermissionsBundle\SfsPermissionsBundle::class => ['all' => true],
];
```

## What The Component Adds {#what-the-component-adds}

The component registers one service:

- a `RoleHierarchyVoter`
- configured with the prefix `PERMISSION_`
- under the service id `sfs_permissions.role_hierarchy.permission`

That is the whole core.

It does not add:

- a permission database
- user-to-permission persistence
- ACL tables
- a UI to assign permissions
- a custom configuration tree

## How It Works {#how-it-works}

Symfony already has a role hierarchy voter for `ROLE_*`.

This component registers a second hierarchy voter for the `PERMISSION_` prefix.

That means a check like:

```php
$this->denyAccessUnlessGranted('PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_UPDATE');
```

can be granted through `role_hierarchy`, even if the user does not literally have that string in the stored roles array.

Example:

```yaml
security:
    role_hierarchy:
        ROLE_SFS_MEDIA_ADMIN_MEDIAS_RW:
            - ROLE_SFS_MEDIA_ADMIN_MEDIAS_RO
            - PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_CREATE
            - PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_DELETE
            - PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_UPDATE
            - PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_MIGRATE
```

With the component enabled, a user with `ROLE_SFS_MEDIA_ADMIN_MEDIAS_RW` is also granted `PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_UPDATE`.

## No Custom Configuration {#no-custom-configuration}

There is no `sfs_permissions:` config block to define.

The component works through standard Symfony Security configuration:

- `security.role_hierarchy`
- `is_granted()`
- `denyAccessUnlessGranted()`
- custom voters

## Why Use `PERMISSION_*` Instead Of `ROLE_*` {#why-use-permission-instead-of-role}

Using `PERMISSION_*` creates a clean separation between:

- business or user roles
  - `ROLE_ADMIN`
  - `ROLE_EDITOR`
  - `ROLE_SFS_MEDIA_ADMIN_MEDIAS_RW`
- atomic allowed actions
  - `PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_LIST`
  - `PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_UPDATE`
  - `PERMISSION_SFS_USER_ADMIN_USERS_PROMOTE`

That separation gives practical benefits:

- reusable packages can ship stable permission names without deciding your final user roles
- applications can compose roles from permissions
- controllers and templates can check the exact action they need
- subject-specific voters can deny one permission without redefining the whole role model

## Main Usage Pattern {#main-usage-pattern}

The standard Softspring pattern has three layers:

1. each package publishes atomic `PERMISSION_*` names
2. each package groups them into convenience `ROLE_*` roles when it makes sense
3. the application grants those roles to users or aggregates them into broader business roles

### Package-Level Permission Groups {#package-level-permission-groups}

For example, `media-bundle` defines:

```yaml
security:
    role_hierarchy:
        ROLE_SFS_MEDIA_ADMIN_MEDIAS_RO:
            - PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_LIST
            - PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_DETAILS
        ROLE_SFS_MEDIA_ADMIN_MEDIAS_RW:
            - ROLE_SFS_MEDIA_ADMIN_MEDIAS_RO
            - PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_CREATE
            - PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_DELETE
            - PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_UPDATE
            - PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_MIGRATE
```

### Application-Level Aggregation {#application-level-aggregation}

Projects such as `armonic-standalone` then aggregate package roles into final application roles:

```yaml
security:
    role_hierarchy:
        ROLE_ADMIN:
            - ROLE_SFS_USER_ADMIN_USERS_RW
            - ROLE_SFS_MEDIA_ADMIN_MEDIAS_RW
            - ROLE_SFS_CMS_ADMIN_BLOCKS_RW
            - ROLE_SFS_CMS_ADMIN_CONTENTS_RW
```

This lets packages stay decoupled from your final business role model.

## Real Usage Cases {#real-usage-cases}

### Declarative CRUD Configuration {#declarative-crud-configuration}

Reusable controller config often checks permissions directly:

```yaml
list:
    is_granted: 'PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_LIST'
create:
    is_granted: 'PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_CREATE'
update:
    is_granted: 'PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_UPDATE'
delete:
    is_granted: 'PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_DELETE'
```

This is a good default pattern for reusable packages because it keeps permissions stable and lets each application decide how to grant them.

### Direct Controller Checks {#direct-controller-checks}

When one action does not fit a generic CRUD flow, check the permission directly:

```php
$this->denyAccessUnlessGranted('PERMISSION_SFS_USER_ADMIN_USERS_PROMOTE', $user);
```

This keeps the controller explicit while still allowing hierarchy-based grants and subject-specific denials.

### Twig Templates {#twig-templates}

Templates can use the same permission names:

```twig
{% if is_granted('PERMISSION_SFS_USER_ADMIN_USERS_UPDATE', user) %}
    <a href="...">Update</a>
{% endif %}
```

The template does not need to know whether the grant came from:

- a direct role
- role hierarchy
- a custom voter

### Declarative Menus {#declarative-menus}

The same permissions work well in config-driven menus:

```yaml
users:
    route: 'sfs_user_admin_users_list'
    role: PERMISSION_SFS_USER_ADMIN_USERS_LIST
```

This is useful when visibility should automatically follow authorization rules.

## Combining Permissions With Custom Voters {#combining-permissions-with-custom-voters}

This component grants permission attributes through hierarchy, but it does not stop you from adding more specific voters.

That is how several Softspring packages work.

For example, one voter can say the user is broadly allowed to recompile content, while another voter denies recompilation for one specific subject because it is disabled in the current state.

This gives you a layered model:

1. role hierarchy says the user is broadly allowed
2. a domain voter says this concrete subject is still not allowed right now

## Why `unanimous` Works Well {#why-unanimous-works-well}

Several Softspring applications use:

```yaml
security:
    access_decision_manager:
        strategy: unanimous
```

That fits well with `PERMISSION_*` checks because:

- the hierarchy voter can grant the permission
- a subject-specific voter can deny it
- under `unanimous`, the deny wins

## Naming Convention {#naming-convention}

The common naming pattern is:

```text
PERMISSION_<VENDOR OR APP>_<AREA>_<RESOURCE>_<ACTION>
```

Examples:

- `PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_LIST`
- `PERMISSION_SFS_ACCOUNT_ADMIN_ACCOUNTS_UPDATE`
- `PERMISSION_SFS_USER_ADMIN_USERS_PROMOTE`
- `PERMISSION_SFS_CMS_ADMIN_SECTION_VERSION_DELETE`

This matters because permission names become part of the reusable API exposed by each package.

## Limitations {#limitations}

These limits are deliberate:

- there is no persistence layer
- there is no admin UI
- there is no object-level logic by itself
- only the `PERMISSION_` prefix is handled

If you need dynamic permission assignment or object-specific rules, build that in your application and combine it with Symfony voters.

## What To Read Next {#what-to-read-next}

- [Crudl controller](../components/crudl-controller.md)
- [User bundle](../bundles/user-bundle.md)
- [Doctrine query filters](../components/doctrine-query-filters.md)
