---
title: "Extend And Customize"
description: "Extend Account Bundle safely by replacing forms, managers, templates, and behavior through events."
---

# Extend And Customize {#extend-and-customize}

`account-bundle` is useful as a base layer, but most real projects customize part of it. The bundle is designed so you can keep the working infrastructure and replace only the pieces that matter.

## Replace Forms First {#replace-forms-first}

The cleanest extension point is usually the form service interfaces.

For example, you can replace the register form:

```yaml
services:
    Softspring\AccountBundle\Form\RegisterFormInterface:
        class: App\Form\Account\RegisterForm
```

The same pattern exists for:

- `SettingsFormInterface`
- `AccountCreateFormInterface`
- `AccountUpdateFormInterface`
- `AccountDeleteFormInterface`
- `AccountListFilterFormInterface`

This is the right option when the flow is still valid but your fields, choices, or validation rules differ.

## Replace The Account Manager When Creation Logic Changes {#replace-the-account-manager-when-creation-logic-changes}

Replace `AccountManagerInterface` when account creation depends on project rules:

- multiple account subtypes such as `Provider` and `Corporate`
- request-driven account creation
- extra creation side effects
- custom repository helpers

This is usually cleaner than forking controllers.

## Replace Behavior With Events {#replace-behavior-with-events}

Prefer subscribers over controller forks when the flow is correct but the business behavior changes.

Good examples are:

- set status or owner on initialize
- create related entities on success
- change redirects
- run onboarding side effects
- block or redirect a flow based on project rules

The bundle exposes useful events for registration, settings, and admin CRUDL actions.

## Replace Templates When The UI Changes But The Flow Stays {#replace-templates-when-the-ui-changes-but-the-flow-stays}

Template overrides are enough when:

- the functional flow stays the same
- the forms are still compatible
- only layout, wording, or presentation changes

Keep in mind that several bundled templates assume fields such as:

- `account.owner`
- `account.owner.displayName`
- `relation.user.username`
- `app.user.relations`

So the default templates are convenient, but not schema-agnostic.

## Keep The Bundle As A Base Layer {#keep-the-bundle-as-a-base-layer}

In practice, the best results usually come from this split:

- the bundle owns account context, route resolution, basic screens, and reusable infrastructure
- your application owns domain rules, account-specific permissions, and product-specific onboarding

That keeps your project close to the supported bundle behavior without forcing you into its exact UI or data model.

## Fixtures And Managers {#fixtures-and-managers}

The bundle also ships:

- `AccountManagerInterface`
- `RelationManagerInterface`
- fixture support through `AccountFixtures` when Doctrine fixtures are installed

These pieces are useful for admin UI development, seeded test data, and project-specific manager replacement.

## Events Reference {#events-reference}

The most relevant event groups are:

- account creation
- account registration
- account settings update
- admin account list, details, create, update, and delete

Most customization work happens through these events and through service replacement of forms, managers, templates, or listeners.

## Related Guides {#related-guides}

- [Account Bundle Overview](../account-bundle.md)
- [Model And Entities](./model-and-entities.md)
- [Register And Settings](./register-and-settings.md)
- [Admin And Security](./admin-and-security.md)
- [Filter And Scoped Data](./filter-and-scoped-data.md)
