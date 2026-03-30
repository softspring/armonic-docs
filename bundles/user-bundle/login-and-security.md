---
title: "User Bundle Login And Security"
description: "Configure login, logout, target path, throttling, remember me, switch user, and impersonation features with the User Bundle."
---

# User Bundle Login And Security {#user-bundle-login-and-security}

This guide focuses on the runtime security integration of the bundle.

The login page itself is only one part. A working setup also needs:

- the provider
- the firewall
- public access for the login and recovery routes
- optional remember me and login throttling
- optional switch user and impersonation bar

## Login Routes {#login-routes}

Import the login routes:

```yaml
_sfs_login:
    resource: '@SfsUserBundle/config/routing/login.yaml'
```

This exposes:

- `sfs_user_login`
- `sfs_user_login_check`
- `sfs_user_logout`

The `check` and `logout` controllers are placeholders. They must be handled by Symfony Security through your firewall configuration.

## Provider And Firewall {#provider-and-firewall}

Use the bundle provider:

```yaml
security:
    providers:
        sfs_user:
            id: sfs_user.user_provider
```

Then connect it to your main firewall:

```yaml
security:
    firewalls:
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
```

Without this, the login form can render but authentication will not happen.

## Public Access Rules {#public-access-rules}

The login and recovery pages are public routes.

A practical access control baseline is:

```yaml
security:
    access_control:
        - { path: ^/app/login, roles: PUBLIC_ACCESS }
        - { path: ^/app/reset-password, roles: PUBLIC_ACCESS }
        - { path: ^/app/register, roles: PUBLIC_ACCESS }
        - { path: ^/app/invitation, roles: PUBLIC_ACCESS }
        - { path: ^/admin, roles: ROLE_ADMIN }
```

Only keep the public entries for features you really use.

## Redirect After Login {#redirect-after-login}

The bundle supports a configurable target path parameter:

```yaml
sfs_user:
    login:
        target_path_parameter: _target_path
```

When this option is enabled, login and register pages preserve that parameter and forward it to the security check path.

Use this when:

- users are redirected to login from a protected page
- the application has several post-login landing pages
- you want custom onboarding flows after authentication

## Login Throttling {#login-throttling}

The bundle does not implement throttling by itself. It relies on the standard Symfony feature.

Example:

```yaml
security:
    firewalls:
        main:
            login_throttling:
                max_attempts: 5
                interval: '1 minute'
```

This is the recommended approach for public login forms.

## Remember Me {#remember-me}

Remember me is also standard Symfony configuration:

```yaml
security:
    firewalls:
        main:
            remember_me:
                secret: '%env(APP_SECRET)%'
                lifetime: 2592000
                secure: true
                httponly: true
                always_remember_me: false
```

Use it when your application has a user-facing area and the login UX matters more than short admin-only sessions.

For admin-only backoffices, many teams keep it disabled.

## Switch User {#switch-user}

The bundle supports Symfony `switch_user` and adds extra authorization checks through `SwitchUserVoter`.

Base config:

```yaml
security:
    firewalls:
        main:
            switch_user:
                role: ROLE_ALLOWED_TO_SWITCH
                parameter: _switch_user
```

Typical role setup:

```yaml
security:
    role_hierarchy:
        ROLE_ADMIN:
            - ROLE_ALLOWED_TO_SWITCH
```

The voter adds extra restrictions:

- super admins cannot be impersonated by lower-level admins
- admin-to-admin impersonation is restricted

This makes the feature safer than a plain role check.

## Impersonation Bar {#impersonation-bar}

The bundle can inject a visible bar while an admin is impersonating another user.

Example:

```yaml
sfs_user:
    impersonate_bar:
        enabled: true
        switch_role: ROLE_ALLOWED_TO_SWITCH
        switch_route: app_entrypoint
        switch_parameter: _switch_user
```

This is especially useful in support or customer success tools, because it makes impersonation obvious and gives a quick exit path.

In `armonic-standalone`, only `switch_route` is configured and the bar can be enabled when the project needs it.

## User Identifiers {#user-identifiers}

The provider resolves the user from your concrete entity class.

The actual identifier depends on the interfaces your entity implements:

- `UserIdentifierEmailInterface` for email login
- `UserIdentifierUsernameInterface` for username login

Choose one early. Most modern projects use email as the main identifier.

## Troubleshooting {#troubleshooting}

- The login form renders but authentication never happens:
  Check `form_login.check_path` and `provider`.
- Logout throws the placeholder exception:
  Check that `logout.path` is configured on the firewall.
- Protected pages keep redirecting to login:
  Check firewall pattern and `access_control`.
- Switch user works too broadly:
  Review your role hierarchy and the subject voter behaviour.

## Related Guides {#related-guides}

- [Install](install.md) for the base entity, route imports, and first security baseline.
- [Register and reset password](register-and-reset-password.md) to add public onboarding and recovery flows.
- [Admin area](admin-area.md) if the same firewall must also cover the admin backoffice.
- [OAuth login](oauth.md) when standard login is not enough and the project needs external authentication.
