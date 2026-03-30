---
title: "User Bundle OAuth Login"
description: "Configure the current OAuth login integration provided by the User Bundle, including the Facebook login flow."
---

# User Bundle OAuth Login {#user-bundle-oauth-login}

The bundle includes optional OAuth integration points.

In the current codebase, the documented built-in path is the Facebook login integration based on `hwi/oauth-bundle`.

Use this only if your project really needs third-party authentication. Most applications can start with normal login and add OAuth later.

## Requirements {#requirements}

Install the OAuth dependency:

```bash
composer require hwi/oauth-bundle:^2.0
```

The bundle will reject OAuth configuration if `HWIOAuthBundle` is not installed.

## Configure Facebook OAuth {#configure-facebook-oauth}

Example:

```yaml
sfs_user:
    oauth:
        facebook:
            application_id: '%env(OAUTH_FACEBOOK_APP_ID)%'
            application_secret: '%env(OAUTH_FACEBOOK_APP_SECRET)%'
            login_create: false
```

What these options mean:

- `application_id`: Facebook application identifier
- `application_secret`: Facebook application secret
- `login_create`: whether a user can be created automatically from the OAuth login flow

## Import OAuth Routes {#import-oauth-routes}

```yaml
_sfs_user_oauth_facebook:
    resource: '@SfsUserBundle/config/routing/login_oauth_facebook.yaml'
```

This exposes:

- `sfs_user_login_oauth_facebook`
- `sfs_user_login_oauth_facebook_js`
- `sfs_user_login_oauth_facebook_redirect`

## Login Page Integration {#login-page-integration}

When Facebook OAuth is configured, the standard login template renders:

- the Facebook login button
- the supporting integration script

That means the normal login page can keep both flows:

- standard username or email plus password
- external OAuth login

## Real Use Cases {#real-use-cases}

### Consumer Application

OAuth can reduce friction for first-time users and improve conversion.

### Internal Or B2B Platform

In many B2B projects, standard login is simpler and easier to audit. OAuth should only be added if it clearly matches the customer identity model.

## Caution {#caution}

OAuth integration adds external dependencies and third-party operational requirements.

Before enabling it, decide:

- whether account creation should be automatic
- how OAuth identities map to existing users
- whether support and audit flows remain clear

If those questions are still open, start with the normal login flow first.

## Related Guides {#related-guides}

- [Login and security](login-and-security.md) for the standard login flow that OAuth extends rather than replaces completely.
- [Install](install.md) for the base bundle, user entity, and route structure expected before adding OAuth.
- [Extend and customize](extend-and-customize.md) if the project needs custom user mapping or post-login behaviour.
