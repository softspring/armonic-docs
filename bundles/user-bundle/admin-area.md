---
title: "User Bundle Admin Area"
description: "Use and configure the built-in admin user management area, including users, administrators, permissions, menu entries, and impersonation support."
---

# User Bundle Admin Area {#user-bundle-admin-area}

The bundle includes a real admin area for managing:

- users
- administrators
- invitations
- optional access history

This is one of the main reasons to document `user-bundle` as a bundle and not as a low-level component: it ships application-facing admin screens, not only models and helpers.

## Admin Route Groups {#admin-route-groups}

Base admin routes:

```yaml
_sfs_user_admin:
    resource: '@SfsUserBundle/config/routing/admin_users.yaml'
    prefix: '/user'

_sfs_user_admins:
    resource: '@SfsUserBundle/config/routing/admin_administrators.yaml'
    prefix: '/admins'
```

Optional:

```yaml
_sfs_user_invitations:
    resource: '@SfsUserBundle/config/routing/admin_invitations.yaml'
    prefix: '/invitations'

_sfs_user_history:
    resource: '@SfsUserBundle/config/routing/admin_access_history.yaml'
    prefix: '/access-history'
```

In many applications these route groups are mounted under `/admin/users`.

## Main Screens {#main-screens}

### Users

The users area includes routes for:

- list
- details
- update
- delete
- promote to admin
- confirm or unconfirm
- enable or disable
- resend confirmation email

This is the screen set you use for normal end users.

### Administrators

The administrators area includes routes for:

- list
- details
- update
- delete
- demote admin
- promote to super admin
- demote super admin

This separation matters because administrator actions have extra authorization rules.

## Permissions Model {#permissions-model}

The bundle publishes permission groups in `config/security/admin_role_hierarchy.yaml`.

The main role groups are:

- `ROLE_SFS_USER_ADMIN_USERS_RO`
- `ROLE_SFS_USER_ADMIN_USERS_RW`
- `ROLE_SFS_USER_ADMIN_ADMINISTRATORS_RO`
- `ROLE_SFS_USER_ADMIN_ADMINISTRATORS_RW`
- `ROLE_SFS_USER_ADMIN_INVITATION_RO`

These roles expand to granular `PERMISSION_*` attributes such as:

- `PERMISSION_SFS_USER_ADMIN_USERS_LIST`
- `PERMISSION_SFS_USER_ADMIN_USERS_UPDATE`
- `PERMISSION_SFS_USER_ADMIN_USERS_PROMOTE`
- `PERMISSION_SFS_USER_ADMIN_ADMINISTRATORS_DETAILS`

Projects should usually:

1. import the bundle role hierarchy
2. aggregate these roles into application roles such as `ROLE_ADMIN`
3. keep controller and template checks at the permission level

## Practical Security Example {#practical-security-example}

```yaml
security:
    role_hierarchy:
        ROLE_ADMIN:
            - ROLE_SFS_USER_ADMIN_USERS_RW
            - ROLE_SFS_USER_ADMIN_ADMINISTRATORS_RW
            - ROLE_SFS_USER_ADMIN_INVITATION_RO
            - ROLE_ALLOWED_TO_SWITCH
        ROLE_SUPER_ADMIN:
            - ROLE_ADMIN
            - PERMISSION_SFS_USER_ADMIN_ADMINISTRATORS_PROMOTE_SUPER
            - PERMISSION_SFS_USER_ADMIN_ADMINISTRATORS_DEMOTE_SUPER
```

This is close to the way `armonic-standalone` integrates the bundle.

## Admin Menu Integration {#admin-menu-integration}

A practical menu setup is:

```yaml
twig:
    globals:
        admin_menu:
            users:
                translation_key: 'sidebar.menu.users.title'
                users:
                    translation_key: 'sidebar.menu.users.users'
                    route: 'sfs_user_admin_users_list'
                    role: PERMISSION_SFS_USER_ADMIN_USERS_LIST
                    active_expression: 'sfs_user_admin_users_'
                    icon: 'bi bi-people me-2'
                administrators:
                    translation_key: 'sidebar.menu.users.administrators'
                    route: 'sfs_user_admin_administrators_list'
                    role: PERMISSION_SFS_USER_ADMIN_ADMINISTRATORS_LIST
                    active_expression: 'sfs_user_admin_administrators_'
                    icon: 'bi bi-people-fill me-2'
```

If you enable invitations or history, add those entries too.

## Filters And Screen Separation {#filters-and-screen-separation}

The admin area uses dedicated Doctrine filters to separate:

- normal users from admins
- admins from normal users

This is why the bundle can expose separate user and administrator screens while still working from the same underlying user entity type.

That separation depends on `RolesAdminInterface` and the admin flags in your concrete user class.

## Impersonation Support {#impersonation-support}

The admin user details screens integrate well with Symfony switch user.

In practical terms, this means support or admin staff can:

- open a user details screen
- impersonate the user when allowed
- see the impersonation bar while operating as that user

This is especially useful in support-heavy products and internal tools.

## Real Use Cases {#real-use-cases}

### Customer Support Backoffice

Use users, administrators, and switch user.

Keep registration optional depending on whether the product is public or staff-managed.

### Editorial Or CMS Admin

Combine user admin with `cms-bundle` and `media-bundle`, and aggregate all the related permissions into one `ROLE_ADMIN`.

### Restricted B2B Administration

Use users and invitations, but keep super-admin promotion only for a very small group.

## Troubleshooting {#troubleshooting}

- The menu entry appears but access is denied:
  Check the final application role hierarchy, not only the imported bundle hierarchy.
- The users screen is empty but the database has users:
  Check that the user entity implements `RolesAdminInterface` correctly.
- Promote or demote actions fail unexpectedly:
  Review the administrator subject voter rules.

## Related Guides {#related-guides}

- [Install](install.md) for the initial admin route imports, user entity, and first admin bootstrap.
- [Login and security](login-and-security.md) for switch user and impersonation behaviour used from the admin screens.
- [Invitations and access history](invitations-and-access-history.md) to enable the optional admin modules around invitations and login audit.
- [Extend and customize](extend-and-customize.md) if the admin forms, templates, or authorization rules need project-specific changes.
