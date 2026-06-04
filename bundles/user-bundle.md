---
title: "User Bundle"
description: "Overview of the Softspring User Bundle and the practical guides available for installation, security, registration, settings, admin management, and extension."
---

# User Bundle {#user-bundle}

The User Bundle provides the user layer for Symfony applications built with Softspring packages.

It covers the common parts most projects need:

- a reusable `User` model base
- login and logout
- optional self-registration
- email confirmation
- reset password
- user settings pages
- admin user management
- optional invitations, access history, impersonation bar, and OAuth integration
- optional Google Identity Platform login

This bundle is not only an entity package. It also ships routes, controllers, forms, templates, events, managers, and security integration.

That matters because real projects usually need more than a `User` entity:

- a public login page
- a real firewall and provider configuration
- a way to register or invite users
- profile and password settings pages
- an admin area to manage users and administrators
- extension points to adapt forms, templates, mailers, and business rules

## Start Here {#start-here}

Read the guides in this order when you are integrating the bundle for the first time:

1. `Install`
2. `Login and security`
3. `Register and reset password`
4. `User settings pages`
5. `Admin area`

Then add the optional guides you need:

- `Invitations and access history`
- `OAuth login`
- `Google Identity Platform login`
- `Extend and customize`

## Common Project Shapes {#common-project-shapes}

This bundle fits several common application shapes.

### Admin-Only Backoffice

Use login, reset password, admin routes, and CLI commands.

Keep public registration disabled and create users manually or from the admin area.

### Public Self-Service Application

Enable login, register, email confirmation, reset password, and settings pages.

This is the typical setup for SaaS products, client portals, and user-facing web applications.

### Invitation-Only Platform

Disable public registration.

Enable invitations and let admins create access from the backoffice or from CLI commands.

### Multi-Bundle Application

The bundle integrates well with `account-bundle`, `permissions-bundle`, and admin-oriented bundles such as `cms-bundle` and `media-bundle`.

This is the usual pattern in Armonic-based applications.

## What To Read Next {#what-to-read-next}

- [Install](user-bundle/install.md) explains the base entity, bundle configuration, routes, security, and first admin user.
- [Login and security](user-bundle/login-and-security.md) explains provider, firewall, throttling, remember me, target path, switch user, and impersonation bar.
- [Register and reset password](user-bundle/register-and-reset-password.md) explains self-registration, confirmation emails, and password recovery.
- [Mailer](user-bundle/mailer.md) explains the optional email integration for confirmation, invitation, and reset password messages.
- [User settings pages](user-bundle/settings-pages.md) explains preferences, change email, change username, change password, and resend confirmation email.
- [Admin area](user-bundle/admin-area.md) explains routes, permissions, menu integration, and the user and administrator screens.
- [Invitations and access history](user-bundle/invitations-and-access-history.md) explains the extra entities and routes required for invitation-only or audited systems.
- [OAuth login](user-bundle/oauth.md) explains the current Facebook integration path.
- [Google Identity Platform login](user-bundle/google-identity-platform.md) explains how to enable Google Sign-In through Google Identity Platform.
- [Extend and customize](user-bundle/extend-and-customize.md) explains how to replace forms, override templates, and use lifecycle events.
