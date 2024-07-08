## Requirements {#requirements}

A Symfony > 5.4 project is required to use this bundle, with a Doctrine database configured.

## Installation {#installation}

```bash
$ composer require softspring/user-bundle:^5.2
```

## Configuration {#configuration}

### Enable the bundle {#enable-the-bundle}

Add the bundle (and the required bundles) to your *config/bundles.php*:

```php
<?php

return [
    ...
    Softspring\PermissionsBundle\SfsPermissionsBundle::class => ['all' => true],
    Softspring\TwigExtraBundle\SfsTwigExtraBundle::class => ['all' => true],
    Softspring\UserBundle\SfsUserBundle::class => ['all' => true],
];
```

### Create user class {#create-user-class}

Create a new User class in *src/Entity/User.php*:

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
use Softspring\UserBundle\Model\UserAvatarInterface;
use Softspring\UserBundle\Model\UserAvatarTrait;
use Softspring\UserBundle\Model\UserHasLocalePreferenceInterface;
use Softspring\UserBundle\Model\UserIdentifierEmailInterface;
use Softspring\UserBundle\Model\UserLastLoginInterface;
use Softspring\UserBundle\Model\UserPasswordInterface;

#[ORM\Entity]
#[ORM\HasLifecycleCallbacks]
class User extends UserModel implements NameSurnameInterface, UserPasswordInterface, PasswordRequestInterface, UserIdentifierEmailInterface, UserAvatarInterface, ConfirmableInterface, RolesAdminInterface, UserLastLoginInterface, UserHasLocalePreferenceInterface
{
    use UserIdentifierEmailTrait;
    use NameSurnameTrait;
    use UserPasswordTrait;
    use PasswordRequestTrait;
    use UserAvatarTrait;
    use ConfirmableTrait;
    use RolesAdminTrait;
    use UserLastLoginTrait;
    use UserHasLocalePreferenceTrait;

    #[ORM\Id]
    #[ORM\Column(length: 13, options: ['fixed' => true])]
    #[ORM\GeneratedValue(strategy: 'NONE')]
    protected ?string $id = null;

    public function getId(): ?string
    {
        return $this->id;
    }

    #[ORM\PrePersist]
    public function _generateId(): void
    {
        $this->id = uniqid();
    }

    public function getDisplayName(): string
    {
        return $this->getName() . ' ' . $this->getSurname();
    }
}
```

### Database migration {#database-migration}

```bash
$ php bin/console doctrine:migrations:diff --namespace="DoctrineMigrations"
$ php bin/console doctrine:migrations:migrate -n
```

### Configure routes {#configure-routes}

Configure routes in *config/routes/sfs_user.yaml*:

```yaml
# config/routes/sfs_user.yaml
_sfs_login:
    resource: '@SfsUserBundle/config/routing/login.yaml'

# _sfs_invitation:
#     resource: '@SfsUserBundle/config/routing/invitation.yaml'
#     prefix: "/invitation"

_sfs_reset_password:
    resource: '@SfsUserBundle/config/routing/reset_password.yaml'
    prefix: "/reset-password"

_sfs_change_password:
    resource: '@SfsUserBundle/config/routing/change_password.yaml'
    prefix: "/user/change-password"

#_sfs_register:
#    resource: '@SfsUserBundle/config/routing/register.yaml'
#    prefix: "/register"

_sfs_preferences:
    resource: '@SfsUserBundle/config/routing/preferences.yaml'
    prefix: "/user/preferences"
```

We recommend to subpath the routes, but it depends on your project.

```yaml
# config/routes.yaml
_sfs_user:
    resource: 'routes/sfs_user.yaml'
    prefix: /app
```

### Configure security {#configure-security}

Configure security in *config/packages/security.yaml*:

```yaml
# config/packages/security.yaml
imports:
    - { resource: '@SfsUserBundle/config/security/admin_role_hierarchy.yaml' }

security:
    # https://symfony.com/doc/current/security.html#registering-the-user-hashing-passwords
    password_hashers:
        Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface: 'auto'
    # https://symfony.com/doc/current/security.html#loading-the-user-the-user-provider
    providers:
        sfs_user:
            id: sfs_user.user_provider

    access_decision_manager:
        strategy: unanimous
        allow_if_all_abstain: false

    role_hierarchy:
        ROLE_ADMIN:
            - ROLE_ADMIN_USERS_RW
            # - ROLE_ADMIN_INVITATION_RO
            # - ROLE_ALLOWED_TO_SWITCH
            - ROLE_ADMIN_ADMINISTRATORS_RW
        ROLE_SUPER_ADMIN:
            - ROLE_ADMIN
            - ROLE_ADMIN_ADMINISTRATORS_PROMOTE_SUPER
            - ROLE_ADMIN_ADMINISTRATORS_DEMOTE_SUPER

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

            # login_throttling:
            #     max_attempts: 5
            #     interval: '1 minute'

            # remember_me:
            #     secret: \"%env(APP_SECRET)%\"
            #     lifetime: 2592000 # 1 month
            #     secure: true
            #     httponly: true
            #     always_remember_me: false

            logout:
                path: sfs_user_logout

            # switch_user:
            #     role: ROLE_ALLOWED_TO_SWITCH
            #     parameter: _switch_user

    access_control:
        - { path: ^/app/login, roles: PUBLIC_ACCESS }
        # - { path: ^/app/invitation, roles: PUBLIC_ACCESS }
        - { path: ^/app/reset-password, roles: PUBLIC_ACCESS }
        - { path: ^/admin, roles: ROLE_ADMIN }

when@test:
    security:
        password_hashers:
            # By default, password hashers are resource intensive and take time. This is
            # important to generate secure password hashes. In tests however, secure hashes
            # are not important, waste resources and increase test times. The following
            # reduces the work factor to the lowest possible values.
            Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface:
                algorithm: auto
                cost: 4 # Lowest possible value for bcrypt
                time_cost: 3 # Lowest possible value for argon
                memory_cost: 10 # Lowest possible value for argo
```

### Create a user {#create-a-user}

Create a user with the command:

```bash
$ php bin/console sfs:user:create username user@example.com 123456
```

Promote the user to admin:

```bash
$ php bin/console sfs:user:promote user@example.com
```

### Check login {#check-login}

Now you can go to login page at /app/login and login with the user you just created.


