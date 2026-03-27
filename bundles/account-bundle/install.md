---
title: "Install Account Bundle"
description: "Install and configure Account Bundle with the minimum setup needed for real account-aware applications."
---

# Install Account Bundle {#install-account-bundle}

`softspring/account-bundle` adds an ownership layer above users. In most applications, the account becomes the business owner of subscriptions, projects, invoices, or other data, and users operate those resources through memberships or ownership.

This bundle expects `softspring/user-bundle` to be installed first. It also works best with `softspring/twig-extra-bundle`, because the current account can then be exposed in Twig as `app.account`.

## Install The Package {#install-the-package}

```bash
composer require softspring/account-bundle:^6.0
```

If your application does not use Symfony Flex, enable the bundle manually:

```php
<?php

return [
    // ...
    Softspring\AccountBundle\SfsAccountBundle::class => ['all' => true],
];
```

## Add Base Configuration {#add-base-configuration}

Start with a minimal configuration and then enable only the parts you really need:

```yaml
# config/packages/sfs_account.yaml
sfs_account:
    class: App\Entity\Account
    relation_class: App\Entity\AccountUserRelation
    admin: true
    twig_app_var_name: account
    route_param_name: _account
    find_field_name: id
    filter:
        enabled: true
```

The most important options are:

- `class`
  - your concrete account entity used for `AccountInterface`
- `relation_class`
  - your concrete membership relation used for `AccountUserRelationInterface`
- `admin`
  - loads the built-in admin controllers and forms
- `filter.enabled`
  - enables automatic account filtering and automatic account assignment on `prePersist`

Keep `route_param_name` as `_account` unless you also plan to review the bundle internals. Several runtime classes still read `_account` directly.

## Import Only The Routes You Need {#import-only-the-routes-you-need}

The bundle ships route files by feature. Import only the ones that match your application:

```yaml
# config/routes/sfs_account.yaml
_sfs_account_register:
    resource: '@SfsAccountBundle/config/routing/register.yaml'
    prefix: /register/account

_sfs_account_settings:
    resource: '@SfsAccountBundle/config/routing/settings.yaml'
    prefix: /account/{_account}/settings

_sfs_account_settings_users:
    resource: '@SfsAccountBundle/config/routing/settings_users.yaml'
    prefix: /account/{_account}/settings/users

_sfs_account_user_accounts:
    resource: '@SfsAccountBundle/config/routing/user_accounts.yaml'
    prefix: /user/accounts

_sfs_account_admin_accounts:
    resource: '@SfsAccountBundle/config/routing/admin_accounts.yaml'
    prefix: /admin/accounts
```

Practical rule:

- import `register.yaml` when users can create their own account
- import `settings.yaml` when users can rename or manage the current account
- import `settings_users.yaml` when you want a bundled starting page for account members
- import `user_accounts.yaml` when users can belong to several accounts
- import `admin_accounts.yaml` when backoffice staff manage accounts centrally

## Typical First Working Setup {#typical-first-working-setup}

For a new SaaS-style application, a practical first milestone is:

1. install `user-bundle`
2. create `Account` and `AccountUserRelation`
3. enable `filter.enabled`
4. import `register.yaml` and `settings.yaml`
5. build one real account area under `/account/{_account}/...`

That gives you enough to test the real model early: account registration, current account resolution, and account-scoped data.

## What The Bundle Adds At Runtime {#what-the-bundle-adds-at-runtime}

After installation, the bundle can provide:

- target entity resolution for account and relation interfaces
- request-time resolution of the current account from `/{_account}`
- `app.account` in Twig
- built-in registration, settings, user, and admin pages
- Doctrine account filtering for entities that opt in
- automatic account assignment on account-aware write requests

The bundle does not replace your project model. You still own the real account entity, relation entity, and business entities.

## Related Guides {#related-guides}

- [Account Bundle Overview](../account-bundle.md)
- [Model And Entities](./model-and-entities.md)
- [Current Account And Routes](./current-account-and-routes.md)
- [Register And Settings](./register-and-settings.md)
- [Admin And Security](./admin-and-security.md)
