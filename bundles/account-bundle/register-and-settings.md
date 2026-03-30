---
title: "Register And Settings"
description: "Use the built-in registration and settings flows from Account Bundle and know where to extend them."
---

# Register And Settings {#register-and-settings}

`account-bundle` includes user-facing flows for account creation and account settings. They are a good starting point when you want self-service account creation without building the whole flow from scratch.

## Account Registration Flow {#account-registration-flow}

`RegisterController` creates a new account through `AccountManagerInterface`, builds `RegisterForm`, dispatches events, saves the entity, and redirects to the success page.

The default registration form is intentionally small:

- `name`

The main registration events are:

- `sfs_account.register.initialize`
- `sfs_account.register.form_valid`
- `sfs_account.register.form_invalid`
- `sfs_account.register.success`

## What Happens On Successful Registration {#what-happens-on-successful-registration}

`AccountCreateListener` extends the default flow:

- if the current user exists and the account implements `OwnerInterface`, that user becomes the owner
- if the account implements `MultiAccountedAccountInterface`, the listener creates a membership relation for the current user
- if that relation supports roles, it adds `ROLE_OWNER`
- if that relation supports `grantedBy`, it stores the current user there too

This makes the built-in flow a practical starting point for SaaS onboarding, but it also shows that some default behavior still relies on legacy multi-account interfaces.

## Redirect After User Registration {#redirect-after-user-registration}

`UserRegisterListener` listens to:

- `sfs_user.register.success`
- `sfs_user.invitation.accepted`

When the registered user implements `UserMultiAccountedInterface` and still has no accounts, the listener redirects the user to `sfs_account_register`.

That creates a simple onboarding flow:

1. create the user
2. create the first business account

## Custom Registration Journeys {#custom-registration-journeys}

The default registration flow is intentionally small so you can adapt it without replacing everything.

Common customizations are:

- set status, type, or owner during `sfs_account.register.initialize`
- create related data during `sfs_account.register.success`
- redirect to a domain-specific dashboard instead of the default success page
- branch by account type such as `Provider` or `Corporate`

For multiple account types, replacing `AccountManagerInterface` is often the cleanest solution.

## Account Settings {#account-settings}

`SettingsController` edits the current account resolved from the request. The default form contains one field:

- `name`

It dispatches:

- `sfs_account.settings.form_valid`
- `sfs_account.settings.form_invalid`
- `sfs_account.settings.updated`

This is a good base when you want a simple editable account profile and then need to add your own fields later.

## Settings Users List {#settings-users-list}

The bundle also includes a members page for the current account. It works when the account implements `MultiAccountedAccountInterface`.

Use it as a starting point, not as a universal membership UI. Projects using the newer relation-based APIs often need a custom page or a replaced controller/view.

## User Accounts List {#user-accounts-list}

`User\AccountsController` shows the accounts for the current user. In the current implementation it expects the authenticated user to implement `UserMultiAccountedInterface`.

This is useful for classic "switch between accounts" experiences, but it is another area where older multi-account APIs still shape the built-in page.

## When To Keep The Built-In UI {#when-to-keep-the-built-in-ui}

The bundled pages are a good fit when:

- your account model follows standard Softspring conventions
- you want a fast starting point
- you are happy to extend forms or templates later

If your onboarding, membership rules, or account types are very specific, keep the controllers only as a reference and move sooner to custom services and pages.

## Related Guides {#related-guides}

- [Account Bundle Overview](../account-bundle.md)
- [Install](./install.md)
- [Current Account And Routes](./current-account-and-routes.md)
- [Admin And Security](./admin-and-security.md)
- [Extend And Customize](./extend-and-customize.md)
