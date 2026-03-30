---
title: "Account Bundle"
description: "Add an ownership layer above users, so subscriptions, projects, invoices, and other resources belong to accounts and users operate them through memberships."
---

# Account Bundle {#account-bundle}

The Account Bundle adds an ownership layer above users on top of `softspring/user-bundle`.

Its purpose is simple:

- the account becomes the business owner of the application data;
- users stop being the final owner of that data;
- users operate one or more accounts through memberships or ownership.

That changes the shape of the application in an important way.

Instead of saying:

- a subscription belongs to one user;
- a project belongs to one user;
- an invoice belongs to one user;

you can say:

- a subscription belongs to one account;
- projects belong to one account;
- clients, invoices, or other resources belong to one account;
- users enter the platform to manage those resources on behalf of that account.

Use it when your application needs one or more of these patterns:

- one account owns the subscription or access to the product;
- one account owns business resources such as projects, customers, invoices, or content;
- several users can collaborate inside the same account;
- one user can work for several accounts;
- the application must resolve a current account and scope data to it.

This bundle gives you the base to model that ownership layer and to work with it in routes, controllers, Twig, forms, admin screens, and Doctrine filters.

It is a strong base for account-aware applications, but it is not a full multitenancy product by itself. If your project needs very specific membership rules, account billing logic, or a richer internal permission system, expect to extend some parts.

## Documentation Map {#documentation-map}

Use this page as the full reference and start with the focused guides when you need to implement a concrete part:

- [Install](./account-bundle/install.md)
  - package installation, base configuration, and route imports
- [Model And Entities](./account-bundle/model-and-entities.md)
  - account ownership model, membership relation, and concrete entity patterns
- [Current Account And Routes](./account-bundle/current-account-and-routes.md)
  - `_account`, request resolution, controller arguments, and account-aware route design
- [Register And Settings](./account-bundle/register-and-settings.md)
  - self-service account creation, onboarding, and user-facing account pages
- [Admin And Security](./account-bundle/admin-and-security.md)
  - access checks, admin permissions, CRUDL screens, forms, and events
- [Filter And Scoped Data](./account-bundle/filter-and-scoped-data.md)
  - automatic Doctrine filtering and account assignment for account-owned entities
- [Extend And Customize](./account-bundle/extend-and-customize.md)
  - replacing forms, managers, templates, and behavior safely

If you are integrating the bundle for the first time, the usual reading order is: install, model your entities, wire account-aware routes, then decide whether to keep the built-in registration, settings, and admin flows.

## Installation {#installation}

Install the package:

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

This bundle expects `softspring/user-bundle` to be present and configured.

It also relies on `softspring/twig-extra-bundle` to expose the current account in Twig as `app.account` by default, in the same spirit as `app.user`.

## What The Bundle Adds {#what-the-bundle-adds}

The bundle is built around four practical blocks:

1. An account model, based on `Softspring\AccountBundle\Model\AccountInterface`.
2. Optional account-user relation models, based on `Softspring\AccountBundle\Model\AccountUserRelationInterface`.
3. Runtime account resolution, so routes can work with a current account in the request and in Twig.
4. Ready-made account UI for registration, settings, and admin management.

Most real projects use several of these blocks together.

## Core Functional Model {#core-functional-model}

The most important idea is not technical. It is functional.

This bundle helps you move responsibility from the user to the account.

In an account-based application:

- the account is the owner of the subscription, plan, or business access;
- the account is the owner of the resources managed inside the platform;
- users are the people who access and manage that account.

That is useful when the real customer is not one person, but a company, team, organization, provider, agency, customer group, or any other higher-level business entity.

This is why `account-bundle` is often a better fit than a pure user-based model when:

- several people need to work on the same data;
- one person can work for different customers or organizations;
- billing or access should survive user changes;
- ownership should stay attached to the business entity instead of the current logged user.

## Common Ways To Use It {#common-ways-to-use-it}

You do not need to use the whole bundle at once.

In practice, projects usually use it in one or more of these ways:

- let a user create the first business account after user registration;
- make subscriptions, plans, or entitlements belong to the account instead of the user;
- make projects, customers, invoices, or similar resources belong to the account;
- allow one user to operate several accounts;
- allow several users to collaborate inside one account;
- build account areas such as `/account/{_account}/settings` or `/provider/{_account}/...`;
- scope writes and reads to the current account;
- add a ready-made admin area for account management.

That is the main idea to keep in mind while reading the rest of this guide: this bundle is a toolbox for account-aware applications, not just an entity package.

## Core Model {#core-model}

The minimum shared model is small:

- `AccountInterface` exposes `getId()`, `getName()`, and `setName()`.
- `Account` is the default mapped superclass for that interface and already stores the `name` field.
- `AccountUserRelationInterface` represents a link between an account and a user.
- `AccountUserRelation` is the mapped superclass for that relation.

The bundle registers Doctrine mappings for the two abstract model classes and then uses target entity resolution to point the interfaces to your application entities.

### Default Mappings {#default-mappings}

The bundle ships these Doctrine building blocks:

- `Softspring\AccountBundle\Model\Account`
  - mapped superclass
  - field `name`
- `Softspring\AccountBundle\Model\AccountUserRelation`
  - mapped superclass
  - composite identifier made of `account` and `user`
  - many-to-one to `AccountInterface`
  - many-to-one to `Softspring\UserBundle\Model\UserInterface`

This means your real entities usually extend the model classes and add their own identifier strategy, owner fields, timestamps, or extra metadata.

## Choosing An Entity Pattern {#choosing-an-entity-pattern}

The bundle supports several account patterns. Pick one early, before you wire routes, forms, and admin screens around it.

### Account With Many User Relations {#account-with-many-user-relations}

This is the most complete pattern. An account has a collection of relation objects, and each relation points to one user.

Use it when the membership itself needs data such as:

- account-specific roles such as owner, admin, manager, or collaborator;
- who granted access;
- invitation or grant timestamps.

Relevant interfaces and traits:

- `AccountManyUserRelationsInterface`
- `UserManyAccountRelationsInterface`
- `AccountManyUserRelationsTrait`
- `UserManyAccountRelationsTrait`

### Account-Scoped Entities {#account-scoped-entities}

Some entities are not accounts, but belong to one account. For example:

- projects;
- invoices;
- content items;
- settings rows.

For that case, use:

- `AccountRelatedInterface`
- `AccountTrait`

If you also want the Doctrine account filter to affect the entity automatically, implement `AccountFilterInterface`.

### Legacy Single And Multi Account APIs {#legacy-single-and-multi-account-apis}

The package still contains older interfaces and helper classes such as:

- `MultiAccountedAccountInterface`
- `MultiAccountedInterface`
- `UserMultiAccountedInterface`
- `SingleAccountedInterface`
- `SingleAccountedAccountInterface`
- `UserSingleAccountedInterface`
- `AccountMultiAccountedTrait`
- `Complete*` sample entities

Most of them are marked as deprecated in the code.

For new code, prefer the non-deprecated relation-based interfaces:

- `AccountManyUserRelationsInterface`
- `UserManyAccountRelationsInterface`
- `AccountRelatedInterface`
- `AccountFilterInterface`

### Membership And Account Roles {#membership-and-account-roles}

This is one of the most important parts of the bundle in real applications.

The account-user relation is where you usually store data such as:

- the role of the user inside that account;
- whether the user is an owner, admin, manager, or member;
- who invited or granted that access;
- extra metadata related to the membership.

This is what lets you model a platform where:

- one company account has several users;
- some users can manage billing or settings;
- some users can only work on operational data;
- one user can switch between different accounts they belong to.

The bundle gives you the relation model and the current-account mechanics. The exact permission model inside the account still belongs to your application, usually through relation roles, custom voters, and account-specific UI.

### Recommended Starting Point {#recommended-starting-point}

If you are starting a new project, the safest baseline is:

1. let `Account` extend the bundled model and implement `AccountManyUserRelationsInterface`;
2. let `User` implement `UserManyAccountRelationsInterface`;
3. create a concrete relation entity extending `AccountUserRelation`;
4. make account-scoped entities implement `AccountRelatedInterface`, and `AccountFilterInterface` when they should be filtered automatically;
5. keep the route parameter as `_account`;
6. import only the routes you really need.

That path matches the current code better than the older single-account and multi-account helper APIs.

## Quick Start {#quick-start}

If you want a practical starting point instead of reading the full reference first, this is the shortest path that matches the bundle well:

1. create your concrete `Account` entity and, if needed, your `AccountUserRelation` entity;
2. configure `sfs_account.class` and `sfs_account.relation_class`;
3. import the bundle routes you really need;
4. start using `_account` in route paths for account-aware areas;
5. read the current account from the request or from Twig;
6. add `AccountRelatedInterface` and `AccountFilterInterface` to entities that should follow the current account automatically.

At that point you already have the base for:

- self-service account creation;
- account-owned subscriptions or plans;
- account-owned business data;
- account settings pages;
- account-aware custom controllers;
- admin account screens;
- automatic account filtering for selected entities.

## Implementing Your Entities {#implementing-your-entities}

Your application owns the concrete Doctrine entities. The bundle gives you base classes, interfaces, and listeners, but your project still defines the real model.

### Account Entity {#account-entity}

A typical account entity extends the bundled model and adds its own identifier and ownership:

```php
<?php

namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;
use Softspring\AccountBundle\Entity\AccountManyUserRelationsTrait;
use Softspring\AccountBundle\Entity\SlugIdTrait;
use Softspring\AccountBundle\Model\Account as AccountModel;
use Softspring\AccountBundle\Model\AccountManyUserRelationsInterface;
use Softspring\UserBundle\Entity\OwnerTrait;
use Softspring\UserBundle\Model\OwnerInterface;

#[ORM\Entity]
class Account extends AccountModel implements AccountManyUserRelationsInterface, OwnerInterface
{
    use SlugIdTrait;
    use OwnerTrait;
    use AccountManyUserRelationsTrait;

    public function __construct()
    {
        $this->userRelations = new ArrayCollection();
    }
}
```

Two details matter here:

- `AccountManyUserRelationsTrait` expects a `userRelations` collection and does not initialize it for you.
- The bundled admin forms and templates assume the account has an `owner` field. If you want to reuse the built-in admin UI, using `OwnerInterface` is the safest path.

### User Entity {#user-entity}

If users can belong to multiple accounts through relation objects, the user usually exposes a collection of account relations:

```php
<?php

namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;
use Softspring\AccountBundle\Entity\UserManyAccountRelationsTrait;
use Softspring\AccountBundle\Model\UserManyAccountRelationsInterface;
use Softspring\UserBundle\Model\User as UserModel;

#[ORM\Entity]
class User extends UserModel implements UserManyAccountRelationsInterface
{
    use UserManyAccountRelationsTrait;

    public function __construct()
    {
        parent::__construct();
        $this->accountRelations = new ArrayCollection();
    }
}
```

The default templates also assume your user exposes a readable display name, usually through `getDisplayName()` and `username`.

### Account-User Relation Entity {#account-user-relation-entity}

When you need metadata on memberships, create a relation entity that extends the abstract model:

```php
<?php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Softspring\AccountBundle\Model\AccountUserRelation as AccountUserRelationModel;
use Softspring\AccountBundle\Model\AccountUserRelationInterface;
use Softspring\UserBundle\Model\UserInterface;

#[ORM\Entity]
class AccountUserRelation extends AccountUserRelationModel implements AccountUserRelationInterface
{
    #[ORM\ManyToOne(targetEntity: UserInterface::class)]
    #[ORM\JoinColumn(name: 'granted_by_id', referencedColumnName: 'id', onDelete: 'SET NULL')]
    protected ?UserInterface $grantedBy = null;

    #[ORM\Column(type: 'json')]
    protected array $roles = [];

    public function getGrantedBy(): ?UserInterface
    {
        return $this->grantedBy;
    }

    public function setGrantedBy(?UserInterface $grantedBy): void
    {
        $this->grantedBy = $grantedBy;
    }

    public function getRoles(): array
    {
        return $this->roles;
    }

    public function setRoles(array $roles): void
    {
        $this->roles = $roles;
    }
}
```

This gives you the same idea as the bundled `CompleteAccountUserRelation` sample, but without building new code on a deprecated convenience class.

In practice, this relation entity is often the right place to store account-level permissions and collaboration metadata.

### Account-Scoped Entity {#account-scoped-entity}

If an entity belongs to one account and should be filtered automatically in account routes, keep the model small:

```php
<?php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Softspring\AccountBundle\Entity\AccountTrait;
use Softspring\AccountBundle\Model\AccountFilterInterface;
use Softspring\AccountBundle\Model\AccountRelatedInterface;

#[ORM\Entity]
class Project implements AccountRelatedInterface, AccountFilterInterface
{
    use AccountTrait;

    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 180)]
    private string $name;
}
```

This is the pattern expected by the Doctrine account filter and by the automatic account assignment on `prePersist`.

## Bundle Configuration {#bundle-configuration}

Configure the bundle under `sfs_account`:

```yaml
# config/packages/sfs_account.yaml
sfs_account:
    class: App\Entity\Account
    relation_class: App\Entity\AccountUserRelation
    admin: true
    entity_manager: default
    twig_app_var_name: account
    route_param_name: _account
    find_field_name: id
    filter:
        enabled: true
```

### Configuration Options {#configuration-options}

The available options are:

- `class`
  - concrete account class used for `AccountInterface`
- `relation_class`
  - concrete relation class used for `AccountUserRelationInterface`
  - keep it `null` only if your project does not use relation entities
- `admin`
  - loads the built-in admin controllers and forms
- `entity_manager`
  - entity manager alias used by the bundle managers and target entity resolver
- `twig_app_var_name`
  - name injected into the Twig app variable extension
- `route_param_name`
  - request attribute used as the current-account key
- `find_field_name`
  - field used by the request listener when it resolves the current account
- `filter.enabled`
  - enables the Doctrine filter listener and the `prePersist` helper listener

For most projects, `class`, `relation_class`, and `filter.enabled` are the options that change the behaviour of the bundle in a meaningful way. The rest are mainly integration details.

### Target Entity Resolution {#target-entity-resolution}

At container build time, the bundle maps:

- `AccountInterface` to `sfs_account.class`
- `AccountUserRelationInterface` to `sfs_account.relation.class`

That lets bundled forms, managers, and Doctrine mappings work against interfaces instead of hard-coded application classes.

### Route Parameter Advice {#route-parameter-advice}

The default `route_param_name` is `_account`, and that is the safest value.

In the current implementation, some internals still read `_account` directly:

- `AccountValueResolver`
- `AccountFilteredEventListener`
- `AccountFilter`

So yes, you can change `route_param_name`, but in practice you should only do it if you also control the affected routes and the account-aware code around them.

## Importing Routes {#importing-routes}

The bundle ships route files for separate features. Import only the ones you actually use.

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

The route files included by the bundle define these actions:

- `register.yaml`
  - account registration form
  - registration success page
- `settings.yaml`
  - account settings form
- `settings_users.yaml`
  - list of users for the current account
- `user_accounts.yaml`
  - list of accounts for the current user
- `admin_accounts.yaml`
  - list, create, details, update, delete, and count widget

The settings routes are where the `_account` route parameter matters most, because those flows depend on the current-account request listeners.

The admin routes are different. They use `{account}` in the path and let the CRUDL controller load the entity for admin actions. They do not depend on the `_account` request-listener flow.

## Account In The URL {#account-in-the-url}

The account-aware flow starts when the account is part of the route.

For example:

```yaml
project_list:
    path: /account/{_account}/projects
    controller: App\Controller\ProjectController::list
```

or:

```yaml
_provider:
    resource: 'routes/provider/*'
    prefix: '/{_locale}/provider/{_account}'
```

When `_account` is present in the URL, the bundle resolves that route value into the real account entity and replaces the raw value in the request attributes.

That means your application code can work with the resolved account directly:

```php
$account = $request->attributes->get('_account');
```

At that point the request already carries the current account context, and that same context is reused by Twig integration, access checks, and the Doctrine account filter.

## Building Account Areas {#building-account-areas}

One of the most useful parts of the bundle is the current-account flow around `_account`.

Once your routes include that parameter, your own controllers, Twig templates, and action listeners can work inside the current account context without repeating account-loading code everywhere.

For example, a real project can organize separate account areas like this:

```yaml
_provider:
    resource: 'routes/provider/*'
    prefix: '/{_locale}/provider/{_account}'

_corporate:
    resource: 'routes/corporate/*'
    prefix: '/{_locale}/corporate/{_account}'
```

With that structure in place, custom controllers can simply use the resolved account from the request:

```php
/** @var Provider $provider */
$provider = $request->attributes->get('_account');
```

That pattern is often more valuable than the built-in pages themselves, because it lets you build your own account-specific product areas on top of the current-account resolution already provided by the bundle.

## Automatic Filtering From The Route Account {#automatic-filtering-from-the-route-account}

This is the usual pattern for account-owned data:

1. the route contains `/{_account}`;
2. the bundle resolves that value to the current account and stores it in the request attributes;
3. Doctrine enables the `account` filter for that request;
4. entities implementing `AccountFilterInterface` are limited to the current account automatically.

For example, if `Project` belongs to one account and implements `AccountFilterInterface`, a route such as:

```yaml
project_list:
    path: /account/{_account}/projects
    controller: App\Controller\ProjectController::list
```

lets your controller load projects normally:

```php
$projects = $projectRepository->findBy([], ['name' => 'ASC']);
```

and, on that request, Doctrine will automatically keep only the rows for the current account.

This is one of the biggest practical wins of the bundle, because it removes a lot of repeated `andWhere('entity.account = :account')` code from account-scoped areas.

### Requirements For Automatic Filtering {#requirements-for-automatic-filtering}

For that automatic filtering to work as expected:

- the route must carry the current account, usually as `/{_account}`;
- the entity must implement `AccountFilterInterface`;
- the entity must expose an account relation compatible with the bundled filter;
- the filter must be enabled in `sfs_account.filter.enabled`.

In the current bundled filter, the account foreign key is expected under `account_id`. If your mapping uses a different shape, you should treat the bundled filter as a starting point and replace it.

## Registration Flow {#registration-flow}

`RegisterController` creates a new account through `AccountManagerInterface`, builds `RegisterForm`, dispatches account events, saves the entity, and then redirects to the success page.

The default form is intentionally small:

- `name`

The controller dispatches these events:

- `sfs_account.register.initialize`
- `sfs_account.register.form_valid`
- `sfs_account.register.form_invalid`
- `sfs_account.register.success`

### What Happens On Successful Registration {#what-happens-on-successful-registration}

The built-in `AccountCreateListener` extends the registration flow:

- if the authenticated user exists and the account implements `OwnerInterface`, the user becomes the owner;
- if the account implements `MultiAccountedAccountInterface`, the listener creates a membership relation for the current user;
- if that relation has `roles`, it adds `ROLE_OWNER`;
- if that relation has `grantedBy`, it stores the current user there too.

This means the registration route works best in projects where:

- users can create their own accounts;
- an account has an owner;
- memberships use the legacy multi-account relation API exposed by `MultiAccountedAccountInterface`.

### Redirect After User Registration {#redirect-after-user-registration}

`UserRegisterListener` listens to user bundle events:

- `sfs_user.register.success`
- `sfs_user.invitation.accepted`

If the registered user implements `UserMultiAccountedInterface` and still has no accounts, the listener redirects that user to `sfs_account_register`.

This creates a simple 2-step onboarding flow:

1. create or accept the user account;
2. create the first business account.

## Custom Registration Journeys {#custom-registration-journeys}

The built-in registration flow is intentionally small, which makes it a good base for project-specific onboarding.

In practice, applications often extend it in 2 directions:

- enrich the account before the form is shown or before it is saved;
- replace the default success redirect with a business-specific destination.

For example, a project with multiple account types can:

1. expose the registration route under `/{accountType}/register`;
2. set owner or status during `sfs_account.register.initialize`;
3. authenticate the owner or redirect to a custom dashboard during `sfs_account.register.success`.

That is exactly the kind of extension point the event system is there for. You keep the bundle controller, but move project rules into subscribers.

## Multiple Account Types {#multiple-account-types}

The bundle does not force you to use a single account class for every business case.

A practical pattern is to configure one abstract account root class and then use Doctrine inheritance for concrete account types such as:

- `Provider`
- `Corporate`
- any other domain-specific subtype

This works well when:

- all account types share the same current-account flow;
- each type needs its own routes, dashboard, settings, or status rules;
- registration should branch depending on the chosen account type.

If account registration must instantiate different concrete classes, replacing `AccountManagerInterface` is usually the cleanest approach. A real project can keep `AbstractAccount` as the configured root class and override `createEntity()` so the manager returns `Provider`, `Corporate`, or another subtype depending on the request.

This is a good example of the bundle adding value through shared account infrastructure while still letting the application keep control of its domain model.

## Current Account Resolution {#current-account-resolution}

The bundle can turn a route parameter into the current account object before your controller runs.

### Request Listener {#request-listener}

`AccountRequestListener` subscribes to `kernel.request` and runs after routing.

When the request has the configured account route attribute:

1. it reads the raw route value;
2. it loads the account through the repository of `AccountInterface`;
3. it searches with the configured `find_field_name`;
4. it writes the resolved entity back into the same request attribute;
5. it stores the entity in the router context;
6. it injects the entity into Twig through the configured `twig_app_var_name`.

With the default configuration, a route like this:

```yaml
account_settings:
    path: /account/{_account}/settings
```

makes the current account available in three places:

- `Request::$attributes['_account']`
- router context parameter `_account`
- `app.account` in Twig

That Twig integration is possible because `softspring/twig-extra-bundle` extends Symfony's `app` variable. `account-bundle` then injects the resolved account into that extensible app variable under the configured name, which is `account` by default.

In practice, this is what lets templates do things such as:

```twig
{{ app.account.name }}
```

or compare the current account while rendering account-aware navigation.

### Controller Value Resolver {#controller-value-resolver}

On Symfony versions with `ValueResolverInterface`, the bundle also registers `AccountValueResolver`.

It resolves controller arguments typed as `AccountInterface` from the `_account` request attribute:

```php
use Softspring\AccountBundle\Model\AccountInterface;
use Symfony\Component\HttpFoundation\Response;

public function settings(AccountInterface $account): Response
{
    // ...
}
```

In the current implementation, this resolver looks up the account by `id` and reads `_account` directly. It does not use `find_field_name`.

That is why the request listener is still the main and most flexible account resolution mechanism.

In day-to-day usage, this feature matters because it lets your own application code stay simple. Most custom controllers only need to trust that `_account` is already resolved and access-checked.

## Access Control {#access-control}

Account-aware routes are protected by `AccountAccessPermissionListener`.

When the current-account request attribute exists, the listener checks:

```php
is_granted('CHECK_ACCOUNT_ACCESS', $account)
```

If access is denied, the request fails with an unauthorized response.

At this level, the question is simple:

- can this user enter this account context or not?

That is not the same as the full permission model inside the account.

### Account Access Voter {#account-access-voter}

The `AccountAccessVoter` grants access in these cases:

- the authenticated user implements `RolesAdminInterface` and `isAdmin()` returns `true`;
- the account implements `OwnerInterface` and the user is the owner;
- the account implements `MultiAccountedAccountInterface` and the user belongs to `getUsers()`;
- the account implements `SingleAccountedAccountInterface` and the user belongs to `getUsers()`.

This is another area where the bundle still reflects some legacy interfaces. Even if your project is moving toward the newer relation-based APIs, the built-in access rules still depend on account ownership or user collections exposed by the account.

In other words, the built-in voter is a base access check. It answers whether the user can access the account area at all.

Finer rules such as:

- who can invite other users;
- who can edit billing settings;
- who can manage projects but not invoices;
- who is an owner, admin, or manager inside the account;

usually belong to relation roles and application-specific permission checks built on top of this base.

### Admin Role Hierarchy {#admin-role-hierarchy}

The bundle also ships a role hierarchy file for account admin screens:

- `ROLE_SFS_ACCOUNT_ADMIN_ACCOUNTS_RO`
  - `PERMISSION_SFS_ACCOUNT_ADMIN_ACCOUNTS_LIST`
  - `PERMISSION_SFS_ACCOUNT_ADMIN_ACCOUNTS_DETAILS`
- `ROLE_SFS_ACCOUNT_ADMIN_ACCOUNTS_RW`
  - read-only permissions
  - `PERMISSION_SFS_ACCOUNT_ADMIN_ACCOUNTS_CREATE`
  - `PERMISSION_SFS_ACCOUNT_ADMIN_ACCOUNTS_UPDATE`
  - `PERMISSION_SFS_ACCOUNT_ADMIN_ACCOUNTS_DELETE`

Your application still decides where these roles are assigned.

## Doctrine Filter And Automatic Account Assignment {#doctrine-filter-and-automatic-account-assignment}

If `filter.enabled` is true, the bundle loads two runtime helpers:

- `AccountDoctrineFilterListener`
- `AccountFilteredEventListener`

### SQL Filter {#sql-filter}

On account-aware requests, `AccountDoctrineFilterListener` enables a Doctrine SQL filter named `account`.

That filter only affects entities implementing `AccountFilterInterface`, and it adds this SQL condition:

```sql
<table_alias>.account_id = :_account
```

This has 2 practical consequences:

- the entity must expose an `account_id` column;
- the filter assumes the account foreign key is stored under that exact name.

If your entity uses another column name or another account relation strategy, the bundled filter is not enough on its own. In that case, treat it as a base implementation, not as a finished solution.

### Automatic Account On PrePersist {#automatic-account-on-prepersist}

`AccountFilteredEventListener` listens to Doctrine `prePersist`.

When an entity:

- implements `AccountFilterInterface`, and
- does not already have an account, and
- also implements `SingleAccountedInterface` or `AccountRelatedInterface`,

the listener copies the current request account into the entity before persistence.

This is convenient for account-scoped write operations inside an `_account` route:

```php
$project = new Project();
$project->setName('Intranet redesign');

$entityManager->persist($project);
$entityManager->flush();
```

If the route already resolved `_account`, the project receives that account automatically.

In the current implementation, this listener reads `_account` directly from the request stack.

## Using The Bundle For Account-Scoped Data {#using-the-bundle-for-account-scoped-data}

This is the part of the bundle that helps when your real business entities belong to an account.

Typical examples are:

- subscriptions or plans;
- projects;
- orders or invoices;
- customers;
- provider resources;
- account-specific settings;
- internal records that should never leak between accounts.

The practical pattern is:

1. add an account relation to the entity;
2. implement `AccountRelatedInterface`;
3. also implement `AccountFilterInterface` when reads should be scoped automatically on `_account` routes.

This gives you two useful behaviours:

- the entity can receive the current account automatically on `prePersist`;
- Doctrine can automatically restrict reads to the current account for supported entities.

That is where the bundle starts feeling like an account-aware application layer instead of just a set of account forms.

## Admin Screens {#admin-screens}

When `admin: true`, the bundle loads a CRUDL controller configuration for account administration.

The available actions are:

- list
- create
- details
- update
- delete
- count widget

### Admin Controller Model {#admin-controller-model}

The admin routes are powered by `softspring/crudl-controller`.

The service `sfs_account.admin.account.controller` defines, per action:

- required permission;
- event names;
- view template;
- form service;
- redirect target.

That is why the admin part feels like a ready-made Symfony backoffice instead of a single monolithic controller class.

### Admin Forms {#admin-forms}

The built-in forms are:

- `AccountListFilterForm`
- `AccountCreateForm`
- `AccountUpdateForm`
- `AccountDeleteForm`

The forms make some assumptions about your model. This is useful when your project follows the standard Softspring conventions, and limiting when it does not.

#### Create And Update Forms {#create-and-update-forms}

Both forms always include:

- `name`
- `owner`

This means the default admin create and update screens expect the account entity to expose an `owner` property compatible with Symfony Form type guessing.

If your account entity does not have an owner, replace the bundled form service interfaces with your own forms before using the admin UI.

#### List Filter Form {#list-filter-form}

The list filter form extends `PaginatorForm` from `softspring/doctrine-paginator`.

It always supports filtering by account name.

If the account entity implements `OwnerInterface`, it also adds an owner search field. The exact fields searched depend on the user class:

- `owner.name`
- `owner.surname`
- `owner.email`

The form uses `softspring/doctrine-query-filters` style property paths such as `name__like` and `owner.name__like___or___owner.surname__like`.

#### Delete Form {#delete-form}

`AccountDeleteForm` can optionally expose a `deleteSingleAccountedUsers` field.

That happens when:

- the account implements `MultiAccountedAccountInterface`, and
- a related user only belongs to that account.

The paired `AdminAccountListener` then removes those users from the account and deletes the user entities through `UserManagerInterface`.

### Admin Events {#admin-events}

Each CRUDL action has its own event sequence. The constants live in `SfsAccountEvents`.

Useful groups are:

- list
  - `sfs_account.admin.accounts.list_initialize`
  - `sfs_account.admin.accounts.list_view`
- details
  - `sfs_account.admin.accounts.details_initialize`
  - `sfs_account.admin.accounts.details_view`
- create
  - initialize, form valid, form invalid, success, view
- update
  - initialize, form valid, form invalid, success, view
- delete
  - initialize, form valid, form invalid, success, view

`AdminAccountListener` adds two default behaviors:

- after create and update success, redirect to the details page of the account;
- on delete confirmation, optionally delete single-account users tied only to that account.

## Extending The Built-In UI {#extending-the-built-in-ui}

The built-in UI is useful, but it is not all-or-nothing.

The bundle is designed so you can keep the controllers and replace just the pieces that matter for your project.

### Replacing Forms {#replacing-forms}

The most direct extension point is the form interfaces used by the controllers.

You can replace the default register form with your own implementation:

```yaml
services:
    Softspring\AccountBundle\Form\RegisterFormInterface:
        class: App\Form\Account\RegisterForm
```

The same pattern exists for:

- `SettingsFormInterface`
- `AccountCreateFormInterface`
- `AccountUpdateFormInterface`
- `AccountDeleteFormInterface`
- `AccountListFilterFormInterface`

This is usually the cleanest way to add fields, validation rules, or project-specific choices without replacing the controllers.

### Replacing The Account Manager {#replacing-the-account-manager}

When account creation itself is project-specific, replace `AccountManagerInterface`.

This is especially useful when:

- registration must create different account subtypes;
- the project needs custom repository helpers;
- entity creation depends on request data or business rules.

That pattern is already used successfully in real applications where one registration flow can create different account classes depending on a route parameter such as `accountType`.

### Replacing Templates {#replacing-templates}

If your model follows the same functional flow but needs different presentation, overriding templates is often enough.

That is a good fit when:

- the fields are still compatible with the built-in forms;
- the account lifecycle is the same;
- only the UI, layout, or wording changes.

### Replacing Behaviour With Events {#replacing-behaviour-with-events}

When the project rules are not just visual, prefer event subscribers over controller forks.

This is the right place to:

- set default owner or status;
- create extra relations;
- change redirects;
- run onboarding side effects;
- block or redirect a flow under project-specific conditions.

The account registration flow and the admin CRUDL flow both expose useful events for that.

## Settings And User-Facing Pages {#settings-and-user-facing-pages}

The package also includes non-admin screens, so you can cover both self-service account flows and backoffice flows with the same bundle.

### Account Settings {#account-settings}

`SettingsController` edits the current account resolved from the request attribute.

The bundled `SettingsForm` contains one field:

- `name`

It dispatches these events:

- `sfs_account.settings.form_valid`
- `sfs_account.settings.form_invalid`
- `sfs_account.settings.updated`

### Settings Users List {#settings-users-list}

`Settings\UsersController` renders the account members of the current account.

This controller only loads relations when the account implements `MultiAccountedAccountInterface`.

So the default users list page is still designed around the legacy multi-account relation API. Use it as a starting point, not as a universal account users UI.

### User Accounts List {#user-accounts-list}

`User\AccountsController` renders the list of accounts for the current user.

In the current implementation, it requires the authenticated user to implement `UserMultiAccountedInterface`. Otherwise it throws an exception.

This is one of the clearest signs that the user-facing account list still depends on the legacy multi-account API.

## Twig Templates {#twig-templates}

The bundle ships templates for:

- register pages
- settings pages
- admin pages
- the user accounts page

When you reuse them, keep in mind that several templates assume these fields exist:

- `account.owner`
- `account.owner.displayName`
- `relation.user.username`
- `app.user.relations`

So the default templates are useful when your user and account entities follow the conventions of `softspring/user-bundle`, but they are not schema-agnostic.

## Managers {#managers}

The bundle provides two managers:

- `AccountManagerInterface`
  - backed by `AccountManager`
  - extends the default CRUDL entity manager behavior
- `RelationManagerInterface`
  - backed by `RelationManager`
  - creates relation entities and exposes the relation repository

The account manager is used by the registration flow, settings flow, admin CRUDL screens, and fixtures.

The relation manager is used mainly when the bundle needs to create membership links automatically.

## Fixtures {#fixtures}

If `doctrine/doctrine-fixtures-bundle` is installed, the bundle registers `AccountFixtures`.

The fixture:

- depends on `Softspring\UserBundle\DataFixtures\UserFixtures`;
- creates 300 accounts;
- assigns a random owner when the account implements `OwnerInterface`;
- belongs to the fixture group `sfs_account`.

That makes it useful for admin UI development and pagination checks.

## Practical Integration Notes {#practical-integration-notes}

These points are not generic advice. They come directly from the current implementation, and they are the parts most likely to save you time.

### Keep `_account` As The Current Account Key {#keep-account-as-the-current-account-key}

Several classes still read `_account` directly. If you move away from that name, review the listeners and resolver carefully.

### Prefer Relation-Based Memberships For New Code {#prefer-relation-based-memberships-for-new-code}

The newer interfaces and traits are the safest long-term API:

- `AccountManyUserRelationsInterface`
- `UserManyAccountRelationsInterface`
- `AccountRelatedInterface`

The older `MultiAccounted*`, `SingleAccounted*`, and `Complete*` helpers are still present, but many are already deprecated. If you are starting fresh, do not build new code around them unless you also plan to maintain those legacy integration points.

### Treat Bundled UI As A Base Layer {#treat-bundled-ui-as-a-base-layer}

The forms and templates assume:

- an account owner exists;
- user display information follows user-bundle conventions;
- some pages work only with legacy interfaces.

That is fine for standard Softspring applications, but projects with a different account model should expect to override forms, listeners, or templates.

### Use Real Product Areas To Validate The Design {#use-real-product-areas-to-validate-the-design}

The fastest way to know whether your account model is working well is to build one real account area early.

For example:

- a provider dashboard;
- an account settings area;
- an account-scoped list of business entities;
- an admin page for account management.

If building that first area feels awkward, the problem is usually not in routing. It is often a sign that the account model or membership pattern should be simplified first.

## Events Reference {#events-reference}

The event constants exposed by the bundle are:

- `sfs_account.account.created`
- `sfs_account.register.initialize`
- `sfs_account.register.form_valid`
- `sfs_account.register.form_invalid`
- `sfs_account.register.success`
- `sfs_account.settings.form_valid`
- `sfs_account.settings.form_invalid`
- `sfs_account.settings.updated`
- `sfs_account.admin.accounts.list_initialize`
- `sfs_account.admin.accounts.list_view`
- `sfs_account.admin.accounts.details_initialize`
- `sfs_account.admin.accounts.details_view`
- `sfs_account.admin.accounts.create_initialize`
- `sfs_account.admin.accounts.create_form_valid`
- `sfs_account.admin.accounts.create_form_invalid`
- `sfs_account.admin.accounts.create_success`
- `sfs_account.admin.accounts.create_view`
- `sfs_account.admin.accounts.update_initialize`
- `sfs_account.admin.accounts.update_form_valid`
- `sfs_account.admin.accounts.update_form_invalid`
- `sfs_account.admin.accounts.update_success`
- `sfs_account.admin.accounts.update_view`
- `sfs_account.admin.accounts.delete_initialize`
- `sfs_account.admin.accounts.delete_form_valid`
- `sfs_account.admin.accounts.delete_form_invalid`
- `sfs_account.admin.accounts.delete_success`
- `sfs_account.admin.accounts.delete_view`

Most extension work in this bundle happens through these events and through service replacement of forms, templates, or listeners.
