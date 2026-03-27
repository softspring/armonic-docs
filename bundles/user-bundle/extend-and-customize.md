---
title: "Extend And Customize User Bundle"
description: "Replace forms, override templates, hook into events, and adapt the user lifecycle safely when integrating the User Bundle in real projects."
---

# Extend And Customize User Bundle {#extend-and-customize-user-bundle}

The bundle is intended to be reused, not treated as a closed product.

It exposes several extension points so projects can adapt the user layer without forking the whole bundle.

The most useful extension points are:

- custom concrete entities
- form replacement through interfaces
- template overrides
- event subscribers
- custom authorization voters
- service overrides for managers or mailers

## Build Your Own Concrete Models {#build-your-own-concrete-models}

The first extension point is your own Doctrine entities.

The bundle gives you mapped superclasses, interfaces, and traits, but your project still decides:

- the final identifier strategy
- which traits to include
- which optional interfaces are supported
- project-specific fields

This is the main reason the bundle adapts well to different application shapes.

## Replace Built-In Forms {#replace-built-in-forms}

Controllers are wired against form interfaces.

That means you can replace form types with service aliases instead of replacing the whole controller.

Example:

```yaml
services:
    App\Form\User\RegisterForm: ~

    Softspring\UserBundle\Form\RegisterFormInterface:
        alias: App\Form\User\RegisterForm
```

The same pattern is available for:

- login
- register
- reset password request
- reset password
- invitation acceptance
- preferences
- change email
- change username
- change password
- admin filter and update forms

This is one of the safest ways to adapt the bundle to a project.

## Override Templates {#override-templates}

The bundle ships templates for public and admin flows.

Typical override targets are:

- `@SfsUser/login/login.html.twig`
- `@SfsUser/register/register.html.twig`
- `@SfsUser/reset_password/request.html.twig`
- `@SfsUser/preferences/preferences.html.twig`
- `@SfsUser/admin/users/list.html.twig`
- `@SfsUser/admin/administrators/details.html.twig`

Override them in your app under:

- `templates/bundles/SfsUserBundle/...`

This is the right place to adapt the UI while keeping the backend flow intact.

## Subscribe To Lifecycle Events {#subscribe-to-lifecycle-events}

The bundle dispatches many events through `Softspring\UserBundle\SfsUserEvents`.

Practical examples:

- `REGISTER_SUCCESS`
- `CONFIRMATION_SUCCESS`
- `INVITATION_ACCEPTED`
- `RESET_SUCCESS`
- `PREFERENCES_UPDATED`
- `CHANGE_EMAIL_UPDATED`
- `ADMIN_USERS_UPDATE_SUCCESS`

Example subscriber:

```php
<?php

namespace App\EventSubscriber;

use Softspring\UserBundle\Event\GetResponseUserEvent;
use Softspring\UserBundle\SfsUserEvents;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;

class UserOnboardingSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents(): array
    {
        return [
            SfsUserEvents::REGISTER_SUCCESS => 'onRegisterSuccess',
        ];
    }

    public function onRegisterSuccess(GetResponseUserEvent $event): void
    {
        // Start your project-specific onboarding
    }
}
```

Use this approach when you need:

- onboarding actions
- analytics
- CRM sync
- account creation after registration
- extra notifications

## Customize Authorization {#customize-authorization}

The bundle already uses voters for sensitive admin actions.

Projects can add more voters when they need extra business rules such as:

- preventing some admins from editing privileged users
- restricting switch user by tenant or account
- blocking destructive actions in specific environments

Keep broad grants in role hierarchy and put subject-specific rules in voters.

## Replace Mailer Behaviour {#replace-mailer-behaviour}

If the bundle mailer integration is too simple for your project, replace or decorate the mailer service.

That is the right place to adapt:

- sender identity
- message content
- delivery channels
- provider-specific metadata

The built-in events are often enough when you only need to trigger extra side effects.

Replace the mailer service only when the email generation itself must change.

## Twig Helper Functions {#twig-helper-functions}

The bundle provides Twig helpers such as:

- `sfs_user_is()`
- `sfs_user_access_is()`
- `sfs_invitation_is()`

These helpers make templates adapt to the actual concrete classes configured in the application.

They are especially useful when:

- the same bundle templates must work with several user capabilities
- a project enables some optional interfaces but not others

## Real Extension Patterns {#real-extension-patterns}

### Add Account Creation After Register

Subscribe to `REGISTER_SUCCESS` and create the first account with `account-bundle`.

### Add Extra Profile Fields

Extend the concrete user entity and replace the preferences or register form.

### Use Your Own Admin Rules

Keep the built-in admin screens and add custom voters for the sensitive actions that need project-specific restrictions.

### Keep The Bundle UI But Rebrand It

Override the public and admin Twig templates while keeping the routes, controllers, and forms.

## Related Guides {#related-guides}

- [Install](install.md) for the base integration points you typically customize later.
- [User settings pages](settings-pages.md) if your first customization target is the self-service forms.
- [Admin area](admin-area.md) when the customization concerns admin UI, permissions, or workflows.
- [Register and reset password](register-and-reset-password.md) when custom onboarding or recovery logic is required.
