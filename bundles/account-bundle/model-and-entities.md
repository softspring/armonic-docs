---
title: "Account Bundle Model And Entities"
description: "Choose the right account model and implement concrete entities that fit Account Bundle."
---

# Model And Entities {#model-and-entities}

Use an account as the owner of shared business data and memberships to connect it to the users who work with it.

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
- `AccountMembershipInterface`
  - represents a link between an account and a user
- `AccountMembership`
  - mapped superclass for that relation

Your application owns the concrete Doctrine entities and extends these model classes.

## Recommended Pattern For New Projects {#recommended-pattern-for-new-projects}

For new code, the safest baseline is:

1. let `Account` extend the bundled model
2. let `Account` implement `AccountMembershipsInterface`
3. let `User` implement `UserAccountMembershipsInterface`
4. create a concrete relation entity extending `AccountMembership`
5. make account-owned business entities implement `AccountOwnedInterface`
6. add `AccountScopedInterface` when those entities should be filtered automatically

These are the 6.0 model contracts. The older single-account, multi-account, and relation helper APIs are no longer present in this line.

## Account Entity {#account-entity}

A typical account entity extends the bundled model and adds ownership and memberships:

```php
<?php

namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;
use Softspring\AccountBundle\Entity\AccountMembershipsTrait;
use Softspring\AccountBundle\Model\Account as AccountModel;
use Softspring\AccountBundle\Model\AccountMembershipsInterface;
use Softspring\UserBundle\Entity\OwnerTrait;
use Softspring\UserBundle\Model\OwnerInterface;

#[ORM\Entity]
class Account extends AccountModel implements AccountMembershipsInterface, OwnerInterface
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    public function getId(): ?int
    {
        return $this->id;
    }

    use OwnerTrait;
    use AccountMembershipsTrait;

    public function __construct()
    {
        $this->memberships = new ArrayCollection();
    }
}
```

If you want to reuse the bundled admin forms and templates, keeping an `owner` field is the safest choice.

## User Entity {#user-entity}

If users can belong to several accounts, the user usually exposes account relations:

Extend the concrete `App\Entity\User` from [User Bundle installation](../user-bundle/install.md), preserving its identifier, password, and other capabilities. Add `UserAccountMembershipsInterface` and `UserAccountMembershipsTrait`, then initialize the collection in its constructor:

```php
// Add these imports to src/Entity/User.php:
use Doctrine\Common\Collections\ArrayCollection;
use Softspring\AccountBundle\Entity\UserAccountMembershipsTrait;
use Softspring\AccountBundle\Model\UserAccountMembershipsInterface;

// Add UserAccountMembershipsInterface to the existing implements list.
// Inside the existing User class:
use UserAccountMembershipsTrait;

public function __construct()
{
    parent::__construct();
    $this->accountMemberships = new ArrayCollection();
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
use Softspring\AccountBundle\Model\AccountMembership as AccountMembershipModel;
use Softspring\AccountBundle\Model\AccountMembershipInterface;
use Softspring\UserBundle\Model\UserInterface;

#[ORM\Entity]
class AccountMembership extends AccountMembershipModel implements AccountMembershipInterface
{
    #[ORM\ManyToOne(targetEntity: UserInterface::class)]
    protected ?UserInterface $grantedBy = null;

    #[ORM\Column(type: 'json')]
    protected array $roles = [];

    public function getRoles(): array
    {
        return $this->roles;
    }

    public function setRoles(array $roles): void
    {
        $this->roles = array_values(array_unique($roles));
    }
}
```

The base mapped membership uses the account and user associations as a composite identifier. Do not add a second generated identifier unless you deliberately replace that mapping. `getRoles()` and `setRoles()` are required by its interface.

This is where the application can store who may manage billing, users, or projects. Add accessors for `grantedBy` if your application uses that optional metadata.

## Account-Owned Business Entities {#account-owned-business-entities}

Any entity that belongs to one account should expose that relation explicitly:

```php
<?php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Softspring\AccountBundle\Entity\AccountOwnedTrait;
use Softspring\AccountBundle\Model\AccountScopedInterface;
use Softspring\AccountBundle\Model\AccountOwnedInterface;

#[ORM\Entity]
class Project implements AccountOwnedInterface, AccountScopedInterface
{
    use AccountOwnedTrait;

    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 180)]
    private string $name = '';

    public function getId(): ?int
    {
        return $this->id;
    }

    public function setName(string $name): void
    {
        $this->name = $name;
    }
}
```

Use `AccountOwnedInterface` when the entity belongs to one account. Add `AccountScopedInterface` when reads should be scoped automatically on account-aware routes.

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

## Keep both sides of a membership consistent {#membership-associations}

The collection helpers add or remove items; they do not set every owning association for you. When creating a membership explicitly, set the account and user as well as updating both collections:

```php
$membership = new AccountMembership();
$membership->setAccount($account);
$membership->setUser($user);
$membership->setRoles(['ROLE_MEMBER']);
$account->addMembership($membership);
$user->addAccountMembership($membership);
$entityManager->persist($membership);
$entityManager->flush();
```

The automatic registration listener performs these steps for the first owner membership. Review the generated migration and Doctrine mappings before using the model with existing data.
