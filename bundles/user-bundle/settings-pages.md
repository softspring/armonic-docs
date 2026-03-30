---
title: "User Bundle Settings Pages"
description: "Configure and use the built-in user settings pages for preferences, email, username, password, and confirmation email actions."
---

# User Bundle Settings Pages {#user-bundle-settings-pages}

The User Bundle includes several built-in routes for authenticated users to manage their own account data.

These pages are useful because they save the project from rebuilding common account self-service forms.

## Available Settings Routes {#available-settings-routes}

The bundle ships these route groups:

- `preferences.yaml`
- `change_email.yaml`
- `change_username.yaml`
- `change_password.yaml`
- `settings_confirmation.yaml`

A practical public routes file looks like this:

```yaml
_sfs_preferences:
    resource: '@SfsUserBundle/config/routing/preferences.yaml'
    prefix: '/user/preferences'

_sfs_change_email:
    resource: '@SfsUserBundle/config/routing/change_email.yaml'
    prefix: '/user/change-email'

_sfs_change_username:
    resource: '@SfsUserBundle/config/routing/change_username.yaml'
    prefix: '/user/change-username'

_sfs_change_password:
    resource: '@SfsUserBundle/config/routing/change_password.yaml'
    prefix: '/user/change-password'

_sfs_settings_confirmation:
    resource: '@SfsUserBundle/config/routing/settings_confirmation.yaml'
    prefix: '/user'
```

These routes are not public. They are intended for authenticated users.

## Preferences Page {#preferences-page}

The preferences page route is:

- `sfs_user_preferences`

This page is intended for general profile settings.

In practice, it is often used for:

- display name data
- locale selection
- avatar-related fields
- other user-owned profile fields present in your custom form

The template adapts to optional user capabilities such as avatar support.

## Change Email {#change-email}

The change email page route is:

- `sfs_user_change_email`

Use it when:

- email is the main user identifier
- users should manage their own login email
- the project wants a dedicated flow instead of mixing email into the generic preferences form

If your project has approval or verification rules around email changes, this is a good flow to customize through events or form replacement.

## Change Username {#change-username}

The change username page route is:

- `sfs_user_change_username`

This route is only useful when your project uses username-based identifiers or exposes usernames as editable public handles.

If your application is email-first and usernames do not exist, simply do not import this route.

## Change Password {#change-password}

The change password page route is:

- `sfs_user_change_password`

This is usually one of the most valuable built-in pages in admin-created user systems, because it gives users a normal way to rotate credentials without involving support.

Your entity must support passwords through `UserPasswordInterface`.

## Resend Confirmation Email {#resend-confirmation-email}

The confirmation helper route is:

- `sfs_user_settings_confirmation`

It resends the confirmation email for the logged-in user when:

- the user implements `ConfirmableInterface`
- the user is still not confirmed
- mailer integration is available

This is useful in applications where users miss the original email and need a simple retry action from their settings area.

## Real Navigation Pattern {#real-navigation-pattern}

The bundle gives you the pages, but your application should provide the surrounding navigation.

A common layout is a profile or account menu with entries such as:

- Profile
- Change email
- Change password
- Security
- Notifications

The bundle covers the first account-level user settings pages, and your project can add the rest around them.

## When To Use Separate Routes {#when-to-use-separate-routes}

Separate settings routes are usually better than one big profile form when:

- email changes have extra validation or audit requirements
- username changes should be optional
- password updates should remain isolated
- the project wants clear URLs and clear permissions

That is the pattern followed by the bundle.

## Customizing The Forms {#customizing-the-forms}

Each settings controller depends on a form interface.

That means you can replace the form type service without replacing the controller:

```yaml
services:
    App\Form\User\PreferencesForm: ~

    Softspring\UserBundle\Form\Settings\PreferencesFormInterface:
        alias: App\Form\User\PreferencesForm
```

The same approach works for:

- `ChangeEmailFormInterface`
- `ChangeUsernameFormInterface`
- `ChangePasswordFormInterface`

## Troubleshooting {#troubleshooting}

- The page exists but fields are missing:
  Check which interfaces your user entity implements and which custom form is registered.
- Changing password does not update the stored password:
  Check `UserPasswordInterface` and password hashing configuration.
- Resend confirmation does nothing:
  Check `ConfirmableInterface` and mailer integration.

## Related Guides {#related-guides}

- [Install](install.md) for the base entity and the route imports needed before these pages can work.
- [Register and reset password](register-and-reset-password.md) for the public onboarding flows that usually lead into these pages.
- [Extend and customize](extend-and-customize.md) if you want to replace the built-in settings forms or templates.
