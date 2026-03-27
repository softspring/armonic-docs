---
title: "User Bundle Invitations And Access History"
description: "Enable invitation-only onboarding and user access history with the extra entities, routes, and configuration required by the User Bundle."
---

# User Bundle Invitations And Access History {#user-bundle-invitations-and-access-history}

Invitations and access history are optional features.

They are not enabled by default because they require extra concrete entities and they are not needed in every project.

Use them when:

- users should be invited instead of self-registering
- administrators need to see who logged in and when
- support or audit requirements matter

## Enable Invitations {#enable-invitations}

Enable invitation services like this:

```yaml
sfs_user:
    invite:
        enabled: true
        class: App\Entity\UserInvitation
```

Then import the public invitation routes:

```yaml
_sfs_invitation:
    resource: '@SfsUserBundle/config/routing/invitation.yaml'
    prefix: '/invitation'
```

And the admin invitation routes:

```yaml
_sfs_user_invitations:
    resource: '@SfsUserBundle/config/routing/admin_invitations.yaml'
    prefix: '/invitations'
```

Finally, make the public invitation path accessible:

```yaml
security:
    access_control:
        - { path: ^/app/invitation, roles: PUBLIC_ACCESS }
```

## Invitation Entity {#invitation-entity}

Your project must provide a concrete invitation entity extending the bundle model:

```php
<?php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Softspring\UserBundle\Model\UserInvitation as UserInvitationModel;

#[ORM\Entity]
class UserInvitation extends UserInvitationModel
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    public function getId(): ?int
    {
        return $this->id;
    }
}
```

You can extend it further with:

- roles to grant on acceptance
- inviter metadata
- business fields such as company or team information

## Invitation Flow {#invitation-flow}

The built-in invitation flow is:

1. an admin or CLI command creates the invitation
2. the invitation email is sent
3. the invited user opens the acceptance URL
4. the bundle creates or completes the user
5. the invitation is marked as accepted

This is a good default for B2B tools, partner portals, and restricted products.

## Invitation Commands {#invitation-commands}

The bundle ships:

```bash
php bin/console sfs:user:invite user@example.com
```

This is useful for:

- initial environment bootstrapping
- ops scripts
- one-off account creation

## Enable Access History {#enable-access-history}

Enable access history services:

```yaml
sfs_user:
    history:
        enabled: true
        class: App\Entity\UserAccess
```

Then import the admin route if you want the history list screen:

```yaml
_sfs_user_history:
    resource: '@SfsUserBundle/config/routing/admin_access_history.yaml'
    prefix: '/access-history'
```

## Access History Entity {#access-history-entity}

Your project must provide a concrete access entity extending the bundle model:

```php
<?php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Softspring\UserBundle\Model\UserAccess as UserAccessModel;

#[ORM\Entity]
class UserAccess extends UserAccessModel
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    public function getId(): ?int
    {
        return $this->id;
    }
}
```

The access model stores:

- user
- login time
- IP
- user agent

That gives you a useful audit baseline without building a custom system first.

## How Access History Is Recorded {#how-access-history-is-recorded}

When history is enabled, the bundle subscribes to login events and creates access records through the `UserAccessManipulator`.

This covers:

- standard interactive login
- implicit login after flows such as register, confirmation, invitation acceptance, or reset success

That is important because it keeps the audit trail consistent across several entry points.

## Real Use Cases {#real-use-cases}

### Invitation-Only B2B SaaS

Enable invitations, disable public register, and let admins create new users for each customer.

### Internal Corporate Tool

Enable access history to help support or security teams review the last accesses of a user account.

### High-Touch Support Team

Combine invitations, access history, and switch user to manage restricted accounts with a better audit trail.

## Related Guides {#related-guides}

- [Install](install.md) for the base user integration before enabling these optional modules.
- [Register and reset password](register-and-reset-password.md) if you need to compare self-registration against invitation-only onboarding.
- [Admin area](admin-area.md) for the backoffice screens that manage invitations and, optionally, access history.
- [Login and security](login-and-security.md) for the authentication events that interact with access history registration.
