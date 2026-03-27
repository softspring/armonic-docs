---
title: "User Bundle Register And Reset Password"
description: "Enable self-registration, email confirmation, reset password, and related public account onboarding flows with the User Bundle."
---

# User Bundle Register And Reset Password {#user-bundle-register-and-reset-password}

This guide covers the public account onboarding flows:

- self-registration
- email confirmation
- reset password request
- reset password completion

These are the routes you enable when your application is not admin-only.

## Enable Registration Routes {#enable-registration-routes}

Import the register routes:

```yaml
_sfs_register:
    resource: '@SfsUserBundle/config/routing/register.yaml'
    prefix: '/register'
```

This exposes:

- `sfs_user_register`
- `sfs_user_register_success`
- `sfs_user_register_confirm`

Then make the route public:

```yaml
security:
    access_control:
        - { path: ^/app/register, roles: PUBLIC_ACCESS }
```

## Entity Requirements For Registration {#entity-requirements-for-registration}

Registration works best when your user entity implements:

- `UserPasswordInterface`
- one identifier interface
  - usually `UserIdentifierEmailInterface`
- `ConfirmableInterface` if you want confirmation emails

Optional but common:

- `NameSurnameInterface`
- `UserHasLocalePreferenceInterface`

The register template already adapts to optional fields such as `name`, `surname`, `username`, and `email`.

## Email Confirmation Flow {#email-confirmation-flow}

The confirmation flow is enabled by default:

```yaml
sfs_user:
    confirm_email:
        enabled: true
```

When a user registers and your entity implements `ConfirmableInterface`:

1. the user is persisted
2. a confirmation token is generated
3. the confirmation email listener sends the email
4. the user confirms through `sfs_user_register_confirm`

That means you usually want:

- a configured mailer
- a concrete `User` entity with confirmation fields
- public access to the confirmation URL

If your project should allow direct login immediately after register, disable confirmation or customize the registration event flow.

## Reset Password Routes {#reset-password-routes}

Import the reset password routes:

```yaml
_sfs_reset_password:
    resource: '@SfsUserBundle/config/routing/reset_password.yaml'
    prefix: '/reset-password'
```

This exposes:

- `sfs_user_reset_password_request`
- `sfs_user_reset_password_requested`
- `sfs_user_reset_password`
- `sfs_user_reset_password_success`

And it must stay public:

```yaml
security:
    access_control:
        - { path: ^/app/reset-password, roles: PUBLIC_ACCESS }
```

## Entity Requirements For Reset Password {#entity-requirements-for-reset-password}

Your user entity must implement `PasswordRequestInterface`.

That adds:

- password request token
- password requested timestamp

Without that interface, the reset email can be requested but the bundle has no place to store the token correctly.

## Token Lifetime {#token-lifetime}

Configure the token TTL like this:

```yaml
sfs_user:
    reset_password:
        token_ttl: 86400
```

This value is used by the bundle templates and flow configuration.

For most applications, one day is a reasonable default.

## Real Project Patterns {#real-project-patterns}

### Self-Service SaaS

Enable register, confirmation email, and reset password.

This is the usual setup for public products where users create their own accounts.

### Internal Tool With Manual Provisioning

Disable registration and keep only reset password.

This is common when users are created by staff, but still need a standard way to recover their password.

### Invitation-Only B2B Product

Disable registration and use invitations instead.

That keeps onboarding controlled while reusing the same authentication and settings layers.

## Interaction With Account Bundle {#interaction-with-account-bundle}

Some projects combine `user-bundle` with `account-bundle`.

A common pattern is:

1. register the user with `user-bundle`
2. create the first business account with `account-bundle`
3. redirect to the account onboarding flow

In `armonic-standalone`, the user register route is currently disabled because the project can route sign-up into a more specific application flow.

## Troubleshooting {#troubleshooting}

- Registration page is missing:
  Check that the register route group is imported.
- Register works but no email is sent:
  Check mailer configuration and `ConfirmableInterface`.
- Reset password page loads but token reset never completes:
  Check `PasswordRequestInterface` on the user entity.
- Register route is public but still redirects:
  Check the firewall pattern and `access_control` order.

## Related Guides {#related-guides}

- [Install](install.md) for the base entity, routes, and initial `sfs_user` configuration.
- [Login and security](login-and-security.md) for the firewall and public access rules this flow depends on.
- [User settings pages](settings-pages.md) to continue with authenticated self-service actions after onboarding.
- [Invitations and access history](invitations-and-access-history.md) if the project should not allow public registration.
