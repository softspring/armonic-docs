---
title: "Admin And Security"
description: "Understand account access checks, bundled admin screens, and the security model used by Account Bundle."
---

# Admin And Security {#admin-and-security}

`account-bundle` includes base access checks for account-aware routes and an admin CRUDL area for central account management.

## Base Account Access Check {#base-account-access-check}

When the current-account request attribute exists, `AccountAccessPermissionListener` checks:

```php
is_granted('CHECK_ACCOUNT_ACCESS', $account)
```

This answers a basic question: can this user enter this account context at all?

It is not the full permission model inside the account. Finer rules such as who can invite users, manage billing, or edit projects usually belong to your relation roles and custom voters.

## Account Access Voter {#account-access-voter}

`AccountAccessVoter` grants access in these base cases:

- the authenticated user is a platform admin through `RolesAdminInterface`
- the account implements `OwnerInterface` and the user is the owner
- the account implements `MultiAccountedAccountInterface` and the user belongs to the account users
- the account implements `SingleAccountedAccountInterface` and the user belongs to the account users

This is a practical starting point, but the bundled voter still reflects some legacy interfaces. Many real applications will add more precise rules on top.

## Admin Role Hierarchy {#admin-role-hierarchy}

The bundle ships admin permissions for account management:

- `ROLE_SFS_ACCOUNT_ADMIN_ACCOUNTS_RO`
  - `PERMISSION_SFS_ACCOUNT_ADMIN_ACCOUNTS_LIST`
  - `PERMISSION_SFS_ACCOUNT_ADMIN_ACCOUNTS_DETAILS`
- `ROLE_SFS_ACCOUNT_ADMIN_ACCOUNTS_RW`
  - read-only permissions
  - `PERMISSION_SFS_ACCOUNT_ADMIN_ACCOUNTS_CREATE`
  - `PERMISSION_SFS_ACCOUNT_ADMIN_ACCOUNTS_UPDATE`
  - `PERMISSION_SFS_ACCOUNT_ADMIN_ACCOUNTS_DELETE`

Your application decides where those roles are assigned.

## Bundled Admin Screens {#bundled-admin-screens}

When `admin: true`, the bundle loads a CRUDL-based admin area with these actions:

- list
- create
- details
- update
- delete
- count widget

This is useful when internal staff need to inspect and manage accounts centrally.

## Admin Forms {#admin-forms}

The built-in admin forms are:

- `AccountListFilterForm`
- `AccountCreateForm`
- `AccountUpdateForm`
- `AccountDeleteForm`

`AccountCreateForm` and `AccountUpdateForm` always include:

- `name`
- `owner`

If your account entity does not expose an `owner` field, replace the bundled form services before using the default admin create and update screens.

## Admin Events {#admin-events}

Each CRUDL action exposes events through `SfsAccountEvents`.

Useful groups are:

- list
- details
- create
- update
- delete

`AdminAccountListener` adds default behavior such as redirecting to the details page after create or update success, and optionally deleting related single-account users during delete flows.

## Security In Real Projects {#security-in-real-projects}

Use the bundled security as a base layer:

- keep `CHECK_ACCOUNT_ACCESS` for "can enter this account area?"
- store relation-specific roles in your membership entity
- add custom voters or listeners for product rules such as billing, invitations, or project administration

This keeps the bundle responsible for account context and lets your application own the business permission model.

## Related Guides {#related-guides}

- [Account Bundle Overview](../account-bundle.md)
- [Current Account And Routes](./current-account-and-routes.md)
- [Register And Settings](./register-and-settings.md)
- [Filter And Scoped Data](./filter-and-scoped-data.md)
- [Extend And Customize](./extend-and-customize.md)
