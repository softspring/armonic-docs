---
title: "Account Bundle"
description: "Model account ownership, memberships, current-account routes, and account-scoped data in Symfony."
---

# Account Bundle {#account-bundle}

Account Bundle lets a company, team, or organization own application data while users access that data through memberships. A user can belong to several accounts, and an account can keep its projects, subscriptions, or invoices when its users change.

The bundle provides account and membership model contracts, current-account resolution from routes, a base access voter, optional Doctrine scoping, and registration, settings, member-list, account-list, and administration screens. Your application supplies the concrete entities and the business permissions inside each account.

## Build an account area {#quick-start}

1. [Install the bundle](account-bundle/install.md) after User Bundle and Twig Extra Bundle.
2. [Create the entities](account-bundle/model-and-entities.md): an account, a membership, and an existing user with membership support.
3. [Add account-aware routes](account-bundle/current-account-and-routes.md), such as `/account/{_account}/projects`.
4. [Apply access rules](account-bundle/admin-and-security.md) and test both permitted and denied membership cases.
5. [Scope account-owned entities](account-bundle/filter-and-scoped-data.md) and assign the account explicitly in background work.

For example, two users can share **Example Studio** while one also belongs to **Example Foundation**. The current route selects the account context; a membership grants entry to that account. Application voters decide which member may edit a project or manage billing.

## Add the bundled workflows {#documentation-map}

[Registration and settings](account-bundle/register-and-settings.md) covers self-service account creation, the current account's settings, its members, and the user's account list. [Administration and security](account-bundle/admin-and-security.md) covers central account CRUD and permissions.

Use [Extension points](account-bundle/extend-and-customize.md) to adapt forms, redirects, managers, and templates. Keep membership roles on the membership entity and interpret them in application authorization rules; they are not automatically global Symfony roles.

## Model names in 6.0 {#core-model}

The current API uses `AccountMembership`, `AccountMembershipInterface`, `AccountMembershipsInterface`, and `UserAccountMembershipsInterface`. Account-owned records use `AccountOwnedInterface`; opt into automatic filtering with `AccountScopedInterface`.

Older `AccountUserRelation`, `AccountManyUserRelations*`, `MultiAccounted*`, `SingleAccounted*`, and `AccountFilterInterface` examples do not describe this line. Update application code and mappings together when migrating from those APIs.
