---
title: "Account Bundle Model And Entities"
description: "Choose the right account model and implement concrete entities that fit Account Bundle."
---

# Model And Entities {#model-and-entities}

The main design decision in `account-bundle` is not technical. It is deciding that the account, not the user, is the owner of the business data.

That is usually the right model when:

- one company or team owns the subscription
- several users work on the same data
- one user can belong to several organizations
- access must survive user changes

## Core Shared Model {#core-shared-model}

The bundle keeps the minimum common model small:

- `AccountInterface`
  - exposes the account identity and name
- `Account`
  - mapped superclass with the `name` field
- `AccountUserRelationInterface`
  - represents a link between an account and a user
- `AccountUserRelation`
  - mapped superclass for that relation

Your application owns the concrete Doctrine entities and extends these model classes.

## Recommended Pattern For New Projects {#recommended-pattern-for-new-projects}

For new code, the safest baseline is:

1. let `Account` extend the bundled model
2. let `Account` implement `AccountManyUserRelationsInterface`
3. let `User` implement `UserManyAccountRelationsInterface`
4. create a concrete relation entity extending `AccountUserRelation`
5. make account-owned business entities implement `AccountRelatedInterface`
6. add `AccountFilterInterface` when those entities should be filtered automatically

This is a better long-term fit than the older `SingleAccounted*`, `MultiAccounted*`, and `Complete*` helpers, many of which are already deprecated.

## Account Entity {#account-entity}

A typical account entity extends the bundled model and adds ownership and memberships:

```php
<?php

namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;
use Softspring\AccountBundle\Entity\AccountManyUserRelationsTrait;
use Softspring\AccountBundle\Model\Account as AccountModel;
use Softspring\AccountBundle\Model\AccountManyUserRelationsInterface;
use Softspring\UserBundle\Entity\OwnerTrait;
use Softspring\UserBundle\Model\OwnerInterface;

#[ORM\Entity]
class Account extends AccountModel implements AccountManyUserRelationsInterface, OwnerInterface
{
    use OwnerTrait;
    use AccountManyUserRelationsTrait;

    public function __construct()
    {
        $this->userRelations = new ArrayCollection();
    }
}
```

If you want to reuse the bundled admin forms and templates, keeping an `owner` field is the safest choice.

## User Entity {#user-entity}

If users can belong to several accounts, the user usually exposes account relations:

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

## Membership Relation Entity {#membership-relation-entity}

The account-user relation is usually where real product rules live. It is a good place for:

- account-specific roles such as owner, admin, manager, or member
- who granted access
- invitation metadata
- timestamps or status

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
    protected ?UserInterface $grantedBy = null;

    #[ORM\Column(type: 'json')]
    protected array $roles = [];
}
```

This is usually the right place to model who can manage billing, users, projects, or other account-specific actions.

## Account-Owned Business Entities {#account-owned-business-entities}

Any entity that belongs to one account should expose that relation explicitly:

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
}
```

Use `AccountRelatedInterface` when the entity belongs to one account. Add `AccountFilterInterface` when reads should be scoped automatically on account-aware routes.

## Choosing Between Ownership Styles {#choosing-between-ownership-styles}

Use the bundle when the real business owner is the account:

- subscriptions belong to the company account
- projects belong to the provider account
- invoices belong to the customer account
- several people act on behalf of that account

Do not force the bundle if your application is fundamentally single-user and ownership never needs to move above the user.

## Related Guides {#related-guides}

- [Account Bundle Overview](../account-bundle.md)
- [Install](./install.md)
- [Current Account And Routes](./current-account-and-routes.md)
- [Filter And Scoped Data](./filter-and-scoped-data.md)
- [Extend And Customize](./extend-and-customize.md)
