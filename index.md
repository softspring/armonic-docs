---
title: "Armonic Documentation"
description: "Documentation hub for Armonic, covering installation, CMS usage, configuration, bundles, and Symfony integration."
---

# Armonic

<video autoplay muted loop> 
  <source src="https://raw.githubusercontent.com/softspring/armonic-docs/5.4/.files/armonic.webm" type="video/webm"/> 
</video>

## Introduction {#introduction}

- [Why Armonic?](introduction/why-armonic.md)

## Getting started {#getting-started}

- [Requirements](getting-started/requirements.md)
- [Install a new Symfony project](getting-started/install-new-symfony-project.md)
- [Install an existing Symfony project](getting-started/install-existing-symfony-project.md)
- [Use Armonic standalone edition](standalone/install-armonic-standalone.md)
- [Install in a Sylius project](getting-started/install-sylius-project.md)

## Armonic configuration {#armonic-configuration}

- [Configure sites](configuration/configure-sites.md)
- [Configure content types](configuration/configure-content-types.md)
- [Configure layouts](configuration/configure-layouts.md)
- [Configure modules](configuration/configure-modules.md)
- [Configure sections](configuration/configure-sections.md)
- [Configure medias](configuration/configure-medias.md)
- [Configure blocks](configuration/configure-blocks.md)
- [Configure menus](configuration/configure-menu.md)

## User manual {#user-manual}

- [Create a new page](user-manual/create-a-new-page.md)
- [Add modules or blocks to a page](user-manual/adding-modules-or-blocks.md)
- [Preview and publish a page](user-manual/preview-and-publish-a-page.md)
- [Create a new block](user-manual/create-a-new-block.md)
- [Create a new menu](user-manual/create-a-new-menu.md)
- [Create a new media](user-manual/create-a-new-media.md)
- [Create routes](user-manual/create-routes.md)

## Bundles {#bundles}

### CMS bundle {#cms-bundle}

- [Install](bundles/cms-bundle/install.md)
- [Getting started](bundles/cms-bundle/getting-started.md)
- [Concepts](bundles/cms-bundle/concepts.md)
- [Configuration](bundles/cms-bundle/configuration.md)
- [Sites](bundles/cms-bundle/sites.md)
- [Sections](bundles/cms-bundle/sections.md)
- [Modules](bundles/cms-bundle/modules.md)
- [Module preview texts](bundles/cms-bundle/module/preview-texts.md)
- CMS module form types
  - [Media](bundles/cms-bundle/module/form-types/media.md)
  - [Media modal](bundles/cms-bundle/module/form-types/media-modal.md)
  - [Symfony route](bundles/cms-bundle/module/form-types/symfony-route.md)
  - [Translatable](bundles/cms-bundle/module/form-types/translatable.md)
- [Extend with collections](bundles/cms-bundle/collections.md)
- [Troubleshooting](bundles/cms-bundle/troubleshooting.md)

### Media bundle {#media-bundle}

- [Media bundle](bundles/media-bundle.md)
- [Install](bundles/media-bundle/install.md)
- [Getting started](bundles/media-bundle/getting-started.md)
- [Concepts](bundles/media-bundle/concepts.md)
- [Configure media types](bundles/media-bundle/media-types.md)
- [Using medias](bundles/media-bundle/using-medias.md)
- [Admin medias](bundles/media-bundle/admin-medias.md)
- [Integrations](bundles/media-bundle/integrations.md)
- [Storage options](bundles/media-bundle/storage-options.md)
- [Name generators](bundles/media-bundle/name-generators.md)
- [Extending bundle](bundles/media-bundle/extending-bundle.md)

### User bundle {#user-bundle}

- [User bundle](bundles/user-bundle.md)
- [Install](bundles/user-bundle/install.md)
- [Login and security](bundles/user-bundle/login-and-security.md)
- [Register and reset password](bundles/user-bundle/register-and-reset-password.md)
- [Mailer](bundles/user-bundle/mailer.md)
- [User settings pages](bundles/user-bundle/settings-pages.md)
- [Admin area](bundles/user-bundle/admin-area.md)
- [Invitations and access history](bundles/user-bundle/invitations-and-access-history.md)
- [OAuth login](bundles/user-bundle/oauth.md)
- [Google Identity Platform login](bundles/user-bundle/google-identity-platform.md)
- [Extend and customize](bundles/user-bundle/extend-and-customize.md)

### Account bundle {#account-bundle}

- [Account bundle](bundles/account-bundle.md)
- [Install](bundles/account-bundle/install.md)
- [Model and entities](bundles/account-bundle/model-and-entities.md)
- [Current account and routes](bundles/account-bundle/current-account-and-routes.md)
- [Register and settings](bundles/account-bundle/register-and-settings.md)
- [Admin and security](bundles/account-bundle/admin-and-security.md)
- [Filter and scoped data](bundles/account-bundle/filter-and-scoped-data.md)
- [Extend and customize](bundles/account-bundle/extend-and-customize.md)

### Other bundles {#other-bundles}

- [Mailer bundle](bundles/mailer-bundle.md)
- [Notification bundle](bundles/notification-bundle.md)

## Components {#components}

- [Overview](components/components.md)

### Forms {#forms}

- [Collection form type](components/collection-form-type.md)
- [Dynamic form type](components/dynamic-form-type.md)
- [Polymorphic form type](components/polymorphic-form-type.md)

### Doctrine {#doctrine}

- [Doctrine paginator](components/doctrine-paginator.md)
- [Doctrine query filters](components/doctrine-query-filters.md)
- [Doctrine target entity resolver](components/doctrine-target-entity-resolver.md)
- [Doctrine migrations comparator](components/doctrine-migrations-comparator.md)
- [Doctrine templates](components/doctrine-templates.md)
- [Doctrine changelog](components/doctrine-changelog-bundle.md)

### HTTP and runtime {#http-and-runtime}

- [Command controller](components/command-controller.md)
- [Crudl controller](components/crudl-controller.md)
- [Response headers](components/response-headers.md)
- [HTTP cache store bundle](components/http-cache-store-bundle.md)
- [Events](components/events.md)

### Translations and content {#translations-and-content}

- [Mime translatable](components/mime-translatable.md)
- [Translatable bundle](components/translatable-bundle.md)
- [Twig extra bundle](components/twig-extra-bundle.md)

### Security and permissions {#security-and-permissions}

- [Permissions bundle](components/permissions-bundle.md)
- [Filter attack vectors](components/filter-attack-vectors.md)

### Google Cloud {#google-cloud}

- [Google Cloud integration](components/google-cloud-integration.md)
- [Google Cloud trace](components/google-cloud-trace.md)

### Other packages {#other-packages}

- [Crudl bundle](components/crudl-bundle.md)
- [Time ago bundle](components/time-ago-bundle.md)
