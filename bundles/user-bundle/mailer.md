---
title: "User Bundle Mailer"
description: "Configure the optional mailer integration in the User Bundle for confirmation, invitation, and reset password emails."
---

# User Bundle Mailer {#user-bundle-mailer}

The User Bundle can send user lifecycle emails, but the mailer integration is disabled by default.

Enable it only when the application sends emails from the bundle, such as:

- register confirmation emails
- reset password emails
- invitation emails
- resend confirmation emails from user settings or the admin area

Projects that only use external authentication, for example Google Identity Platform, can keep the mailer disabled.

## Requirements {#requirements}

Install Symfony Mailer before enabling the bundle mailer:

```bash
composer require symfony/mailer
```

Then enable the feature:

```yaml
sfs_user:
    mailer:
        enabled: true
```

The bundle fails during configuration when `sfs_user.mailer.enabled` is `true` but `symfony/mailer` is not installed.

Keep the mailer disabled when the application does not send user emails:

```yaml
sfs_user:
    mailer:
        enabled: false
```

## What The Mailer Sends {#what-the-mailer-sends}

When enabled, the bundle registers `UserMailerInterface` and the automatic email listeners.

The default mailer sends:

- `ConfirmationEmail` after registration when the user implements `ConfirmableInterface`
- `ResetPasswordEmail` after a valid reset password request when the user implements `PasswordRequestInterface`
- `InvitationEmail` when an invitation is created and the invitation feature is enabled

The admin and settings resend actions also use the same mailer when it is available.

## Configure Sender Data {#configure-sender-data}

The mailer configuration accepts a sender address and name:

```yaml
sfs_user:
    mailer:
        enabled: true
        from:
            address: 'no-reply@example.com'
            name: 'Example'
```

Use project configuration or environment variables for real deployments.

## Mail Template Integration {#mail-template-integration}

If `softspring/mailer-bundle` is installed, the User Bundle registers its mail templates through `MailTemplateLoader`.

The provided template ids are:

- `sfs_user.reset_password`
- `sfs_user.register_confirm`
- `sfs_user.invite`

Use this integration when the project manages email content through the Mailer Bundle.

If `softspring/mailer-bundle` is not installed, the User Bundle can still send the default Symfony Mailer messages when `symfony/mailer` is installed and `sfs_user.mailer.enabled` is `true`.

## Replace The Mailer {#replace-the-mailer}

Replace or decorate `Softspring\UserBundle\Mailer\UserMailerInterface` when the project needs a different email generation or delivery strategy.

Example:

```yaml
services:
    App\User\UserMailer:
        decorates: Softspring\UserBundle\Mailer\UserMailerInterface
```

Prefer decorating the service when you only need to add behaviour before or after sending.

Replace it completely when the email content, template engine, or delivery channel must be different.

## Related Guides {#related-guides}

- [Register and reset password](register-and-reset-password.md) for the flows that generate confirmation and reset emails.
- [Invitations and access history](invitations-and-access-history.md) for invitation emails.
- [Google Identity Platform login](google-identity-platform.md) for a mailer-free authentication setup.
- [Extend and customize](extend-and-customize.md) for broader customization patterns.
