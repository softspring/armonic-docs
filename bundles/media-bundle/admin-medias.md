---
title: "Admin Medias"
description: "Use the built-in admin media library, routes, permissions, and maintenance actions from Media Bundle."
---

# Admin Medias {#admin-medias}

`media-bundle` can expose a full admin media library through `softspring/crudl-controller`.

## Enable Admin Controllers {#enable-admin-controllers}

```yaml
sfs_media:
    media:
        admin_controller: true
```

Import the admin routes:

```yaml
_sfs_media_admin:
    resource: '@SfsMediaBundle/config/routing/admin_media.yaml'
    prefix: /admin/media
```

Import the default role hierarchy when you want the bundled permission names:

```yaml
imports:
    - { resource: '@SfsMediaBundle/config/security/admin_role_hierarchy.yaml' }
```

## Admin Permissions {#admin-permissions}

The bundle ships:

- `ROLE_SFS_MEDIA_ADMIN_MEDIAS_RO`
  - `PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_LIST`
  - `PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_DETAILS`
- `ROLE_SFS_MEDIA_ADMIN_MEDIAS_RW`
  - read-only permissions
  - `PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_CREATE`
  - `PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_UPDATE`
  - `PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_DELETE`
  - `PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_MIGRATE`

## Available Admin Actions {#available-admin-actions}

The admin controller exposes:

- list
- create
- update
- delete
- read
- read_ajax
- search_type
- create_ajax
- migrate

This is more than a simple CRUD. It also supports modal selection and per-media repair flows.

## Search Type Action {#search-type-action}

The `search_type` action is the route used by modal selectors.

By default it forces:

- `private = false`
- `type__in` based on the route argument when the filter is still empty

That means the modal picker is intentionally limited to public media unless you override that flow.

## Migrate Action {#migrate-action}

The `migrate` admin action applies migration logic to one specific media entry.

This is useful when:

- one type changed
- one item is missing generated files
- one entry needs repair without running the global command

## When The Built-In Admin Fits Well {#when-the-built-in-admin-fits-well}

The built-in admin is a good fit when:

- editors need a reusable central media library
- backoffice screens already use Softspring admin conventions
- modal selectors should integrate with library browsing

If your project needs a very custom media workflow, keep the bundle as the backend layer and replace only the admin UI pieces that matter.

## Related Guides {#related-guides}

- [Media Bundle Overview](../media-bundle.md)
- [Install](./install.md)
- [Using Medias](./using-medias.md)
- [Configure Media Types](./media-types.md)
- [Extending Bundle](./extending-bundle.md)
