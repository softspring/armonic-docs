---
title: "Install User Bundle"
description: "Install and configure the User Bundle in a Symfony application, including the base user entity, routes, security, and the first administrator."
---

# Install User Bundle {#install-user-bundle}

This guide covers the practical minimum to get the bundle running in a real application.

At the end of this guide you will have:

- a concrete `App\Entity\User`
- the bundle enabled
- routes imported
- Symfony Security configured
- a first admin user created from CLI

## Install The Package {#install-the-package}

```bash
composer require softspring/user-bundle:^6.0
```

If your application does not use Symfony Flex, enable the bundle manually:

```php
<?php

return [
    // ...
    Softspring\UserBundle\SfsUserBundle::class => ['all' => true],
];
```

## What The Flex Recipe Adds {#what-the-flex-recipe-adds}

With Symfony Flex, the recipe gives you a practical starting point:

- `src/Entity/User.php`
- `config/routes/sfs_user.yaml`
- `config/routes/admin/sfs_user.yaml`
- `config/packages/sfs_user.yaml`
- `config/packages/security.yaml.dist`

Some features stay disabled or commented on purpose:

- public register
- invitations
- access history

That default is intentional. Invitations and access history need extra concrete entities and should not be enabled accidentally in a fresh install.

## Create Your User Entity {#create-your-user-entity}

The bundle is designed to work with your own concrete `User` class.

The usual pattern is:

- extend `Softspring\UserBundle\Model\User`
- implement only the interfaces you need
- compose the matching traits

A practical baseline for many projects is:

```php
<?php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Softspring\UserBundle\Entity\ConfirmableTrait;
use Softspring\UserBundle\Entity\NameSurnameTrait;
use Softspring\UserBundle\Entity\PasswordRequestTrait;
use Softspring\UserBundle\Entity\RolesAdminTrait;
use Softspring\UserBundle\Entity\UserHasLocalePreferenceTrait;
use Softspring\UserBundle\Entity\UserIdentifierEmailTrait;
use Softspring\UserBundle\Entity\UserLastLoginTrait;
use Softspring\UserBundle\Entity\UserPasswordTrait;
use Softspring\UserBundle\Model\ConfirmableInterface;
use Softspring\UserBundle\Model\NameSurnameInterface;
use Softspring\UserBundle\Model\PasswordRequestInterface;
use Softspring\UserBundle\Model\RolesAdminInterface;
use Softspring\UserBundle\Model\User as UserModel;
use Softspring\UserBundle\Model\UserHasLocalePreferenceInterface;
use Softspring\UserBundle\Model\UserIdentifierEmailInterface;
use Softspring\UserBundle\Model\UserLastLoginInterface;
use Softspring\UserBundle\Model\UserPasswordInterface;

#[ORM\Entity]
#[ORM\HasLifecycleCallbacks]
class User extends UserModel implements
    NameSurnameInterface,
    UserPasswordInterface,
    PasswordRequestInterface,
    UserIdentifierEmailInterface,
    ConfirmableInterface,
    RolesAdminInterface,
    UserLastLoginInterface,
    UserHasLocalePreferenceInterface
{
    use UserIdentifierEmailTrait;
    use NameSurnameTrait;
    use UserPasswordTrait;
    use PasswordRequestTrait;
    use ConfirmableTrait;
    use RolesAdminTrait;
    use UserLastLoginTrait;
    use UserHasLocalePreferenceTrait;

    #[ORM\Id]
    #[ORM\Column(length: 13, options: ['fixed' => true])]
    #[ORM\GeneratedValue(strategy: 'NONE')]
    protected ?string $id = null;

    #[ORM\PrePersist]
    public function generateId(): void
    {
        $this->id = uniqid();
    }

    public function getId(): ?string
    {
        return $this->id;
    }

    public function getDisplayName(): string
    {
        return trim(($this->getName() ?? '').' '.($this->getSurname() ?? ''));
    }
}
```

This baseline gives you:

- email-based login
- encoded passwords
- admin and super-admin flags
- email confirmation
- reset password
- last login storage
- locale preference

## Configure The Bundle {#configure-the-bundle}

Minimal config:

```yaml
# config/packages/sfs_user.yaml
sfs_user:
    class: App\Entity\User
```

Recommended starter config:

```yaml
# config/packages/sfs_user.yaml
imports:
    - { resource: '@SfsUserBundle/config/security/admin_role_hierarchy.yaml' }

sfs_user:
    class: App\Entity\User
    confirm_email:
        enabled: true
    reset_password:
        token_ttl: 86400
    login:
        target_path_parameter: _target_path
    invite:
        enabled: false
    history:
        enabled: false
```

Important options:

- `class`: your concrete user entity
- `entity_manager`: Doctrine entity manager name
- `confirm_email.enabled`: enable or disable the confirmation email flow
- `reset_password.token_ttl`: password reset token lifetime in seconds
- `login.target_path_parameter`: query or form parameter used for redirect-after-login
- `invite.enabled`: enables invitation services and routes
- `history.enabled`: enables access history services and routes

## Import Routes {#import-routes}

Import the route groups in your main `config/routes.yaml`:

```yaml
_sfs_user:
    resource: 'routes/sfs_user.yaml'
    prefix: '/app'

_sfs_admin_user:
    resource: 'routes/admin/sfs_user.yaml'
    prefix: '/admin/users'
```

Then create the public routes file:

```yaml
# config/routes/sfs_user.yaml
_sfs_login:
    resource: '@SfsUserBundle/config/routing/login.yaml'

_sfs_reset_password:
    resource: '@SfsUserBundle/config/routing/reset_password.yaml'
    prefix: '/reset-password'

_sfs_change_password:
    resource: '@SfsUserBundle/config/routing/change_password.yaml'
    prefix: '/user/change-password'

_sfs_preferences:
    resource: '@SfsUserBundle/config/routing/preferences.yaml'
    prefix: '/user/preferences'

#_sfs_register:
#    resource: '@SfsUserBundle/config/routing/register.yaml'
#    prefix: '/register'
```

This matches the default Flex recipe.

Start with login, reset password, preferences, and change password. Add the optional route groups only when the project needs them:

- `register.yaml` for self-service signup
- `invitation.yaml` for invitation-only onboarding
- `change_email.yaml`, `change_username.yaml`, and `settings_confirmation.yaml` for extra self-service pages

And the admin routes file:

```yaml
# config/routes/admin/sfs_user.yaml
_sfs_user_admin:
    resource: '@SfsUserBundle/config/routing/admin_users.yaml'
    prefix: '/user'

_sfs_user_admins:
    resource: '@SfsUserBundle/config/routing/admin_administrators.yaml'
    prefix: '/admins'

#_sfs_user_invitations:
#    resource: '@SfsUserBundle/config/routing/admin_invitations.yaml'
#    prefix: '/invitations'

#_sfs_user_history:
#    resource: '@SfsUserBundle/config/routing/admin_access_history.yaml'
#    prefix: '/access-history'
```

Keep invitations and access history commented until the related `sfs_user` feature flags and concrete entities exist.

The bundle is modular. Import only the route groups you actually want to expose.

## Configure Symfony Security {#configure-symfony-security}

The bundle expects a provider based on `sfs_user.user_provider`.

Use a base security config like this:

```yaml
# config/packages/security.yaml
security:
    password_hashers:
        Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface: 'auto'

    providers:
        sfs_user:
            id: sfs_user.user_provider

    access_decision_manager:
        strategy: unanimous
        allow_if_all_abstain: false

    role_hierarchy:
        ROLE_ADMIN:
            - ROLE_SFS_USER_ADMIN_USERS_RW
            - ROLE_SFS_USER_ADMIN_ADMINISTRATORS_RW
        ROLE_SUPER_ADMIN:
            - ROLE_ADMIN
            - PERMISSION_SFS_USER_ADMIN_ADMINISTRATORS_PROMOTE_SUPER
            - PERMISSION_SFS_USER_ADMIN_ADMINISTRATORS_DEMOTE_SUPER

    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false

        main:
            pattern: ^/(admin|app)/
            lazy: true
            provider: sfs_user
            form_login:
                provider: sfs_user
                login_path: sfs_user_login
                check_path: sfs_user_login_check
            logout:
                path: sfs_user_logout

    access_control:
        - { path: ^/app/login, roles: PUBLIC_ACCESS }
        - { path: ^/app/reset-password, roles: PUBLIC_ACCESS }
        - { path: ^/admin, roles: ROLE_ADMIN }
```

If you enable registration or invitations later, open those public routes explicitly in `access_control`.

## Run Doctrine Migrations {#run-doctrine-migrations}

Once your `User` entity exists and the bundle config is in place, create and run migrations:

```bash
php bin/console doctrine:migrations:diff
php bin/console doctrine:migrations:migrate -n
```

## Create The First Admin User {#create-the-first-admin-user}

The bundle ships CLI commands for the first bootstrap:

```bash
php bin/console sfs:user:create admin admin@example.com ChangeMe --enabled --admin
php bin/console sfs:user:promote admin --super-admin
```

Main commands:

- `sfs:user:create`
- `sfs:user:promote`
- `sfs:user:invite`

## Real Integration Pattern {#real-integration-pattern}

In `armonic-standalone`, the bundle is usually integrated with:

- public app routes under `/app`
- admin routes under `/admin/users`
- imported admin role hierarchy
- a single `main` firewall for `/app` and `/admin`

That is a good reference when you want a real project example instead of an isolated snippet.

## Related Guides {#related-guides}

- [Login and security](login-and-security.md) for the full firewall, target path, remember me, and switch user setup.
- [Register and reset password](register-and-reset-password.md) if your application is public and users can create their own accounts.
- [Admin area](admin-area.md) to wire the backoffice screens after the base install is done.
- [Invitations and access history](invitations-and-access-history.md) if you need invitation-only onboarding or login audit trails.
