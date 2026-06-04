---
title: "User Bundle Google Identity Platform Login"
description: "Enable Google Sign-In with Google Identity Platform in the User Bundle, including entity fields, route import, bundle configuration, and Google client setup."
---

# User Bundle Google Identity Platform Login {#user-bundle-google-identity-platform-login}

The User Bundle can render Google Sign-In on the login page and synchronize the authenticated Google account with a local user.

This feature is optional. Use it when the application delegates authentication to Google Identity Platform, or when Google sign-in is the only login method you want to expose.

The bundle mailer is disabled by default and is not required for this flow.

## Add Identity Platform Fields To The User Entity {#add-identity-platform-fields-to-the-user-entity}

Your concrete user entity must implement `GoogleIdentityPlatformAwareInterface`.

Use the Doctrine trait from `Softspring\UserBundle\Entity` when your user entity is mapped with Doctrine ORM:

```php
use Softspring\UserBundle\Entity\GoogleIdentityPlatformTrait;
use Softspring\UserBundle\Model\GoogleIdentityPlatformAwareInterface;

class User extends UserModel implements GoogleIdentityPlatformAwareInterface
{
    use GoogleIdentityPlatformTrait;
}
```

The trait adds these nullable fields:

- `authSource`
- `identityPlatformUserId`
- `identityProvider`
- `identityProviderUserId`

You will usually also want the user entity to support these existing contracts:

- `UserIdentifierEmailInterface`
- `NameSurnameInterface`
- `UserLastLoginInterface`
- `UserAvatarInterface`
- `ConfirmableInterface`

They are not part of the Identity Platform contract, but they let the synchronizer keep the local profile useful after Google sign-in.

## Import The Routes {#import-the-routes}

Create a dedicated route import in your application:

```yaml
_sfs_user_google_identity_platform:
    resource: '@SfsUserBundle/config/routing/login_google_identity_platform.yaml'
    prefix: /auth/google
```

With that prefix, the callback URL is `/auth/google/callback`.

The route import exposes:

- `sfs_user_login_google_identity_platform`
- `sfs_user_login_google_identity_platform_callback`

Both routes accept `POST` requests. The login page posts the Google credential and CSRF token to the callback route.

## Enable The Feature {#enable-the-feature}

Configure `sfs_user.login.google_identity_platform`:

```yaml
sfs_user:
    login:
        google_identity_platform:
            enabled: true
            client_id: '%env(default::GOOGLE_CLIENT_ID)%'
            api_key: '%env(default::IDENTITY_PLATFORM_API_KEY)%'
            tenant_id: '%env(default::IDENTITY_PLATFORM_TENANT_ID)%'
            success_route: app_home
            failure_route: sfs_user_login
```

Important options:

- `enabled`: renders the Google widget and enables the backend callback.
- `client_id`: Google web client id used by the browser widget.
- `api_key`: Identity Platform API key used by the backend token exchange.
- `tenant_id`: optional Identity Platform tenant id.
- `success_route`: route used after a successful sign-in.
- `failure_route`: route used when sign-in fails.

The feature also supports the Google button options exposed by the bundle configuration:

```yaml
sfs_user:
    login:
        google_identity_platform:
            button:
                type: standard
                theme: outline
                size: large
                shape: rectangular
                text: continue_with
                logo_alignment: left
                width: 320
```

## Configure Google {#configure-google}

In Google Cloud, the OAuth web client must allow:

- the application origin, for example `https://example.test`
- the callback URL from your route import, for example `https://example.test/auth/google/callback`

The `client_id` must be a Google web client id ending with `.apps.googleusercontent.com`.

Use `IDENTITY_PLATFORM_API_KEY` for the backend exchange. This is separate from the browser `client_id`.

## Runtime Behaviour {#runtime-behaviour}

When enabled, the standard login page renders the Google Sign-In widget.

The callback controller validates:

- the Google CSRF cookie token
- the submitted CSRF token
- the submitted Google credential
- the Identity Platform response

Then it synchronizes the Identity Platform user with the local user entity and logs that user into the Symfony firewall.

If the feature is disabled, or if the credential cannot be validated, the controller redirects to the configured `failure_route` and stores an error flash message under `sfs_user_google_identity_platform_error`.

If the route is not imported, the login page shows a warning instead of rendering a broken button.

## Related Guides {#related-guides}

- [Login and security](login-and-security.md) for the standard login page and firewall setup.
- [Install](install.md) for the base user entity and route structure.
- [Mailer](mailer.md) for optional user email setup.
- [Register and reset password](register-and-reset-password.md) for email-based onboarding flows that may require the mailer.
