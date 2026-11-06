---
title: "Softspring Components"
description: "Find technical building blocks for Symfony forms, Doctrine, HTTP, translations, permissions, and Google Cloud integrations."
related:
    - "bundles"
    - "configuration"
    - "getting-started"
---

# Components {#components}

This section groups technical building blocks by the problem they help solve. Use each package guide to check its setup and integration points. For application features such as the CMS, media library, or user management, see [Bundles](../bundles/overview.md).

## Shared UI {#shared-ui}

The **[UI components package](../components/components.md)** provides shared Twig layouts, an admin shell, paginators, flash messages, and form themes.

## Forms {#forms}

- [Form schema](../components/form-schema.md)
- [Collection form type](../components/collection-form-type.md)
- [Dynamic form type](../components/dynamic-form-type.md)
- [Polymorphic form type](../components/polymorphic-form-type.md)

## Doctrine {#doctrine}

- [Paginator](../components/doctrine-paginator.md)
- [Query filters](../components/doctrine-query-filters.md)
- [Target entity resolver](../components/doctrine-target-entity-resolver.md)
- [Migrations comparator](../components/doctrine-migrations-comparator.md)
- [Templates](../components/doctrine-templates.md)
- [Changelog](../components/doctrine-changelog-bundle.md)

## HTTP and runtime {#http-and-runtime}

- [Cache configuration](../components/cache.md)
- [Logging](../components/logging.md)
- [Command controller](../components/command-controller.md)
- [Crudl controller](../components/crudl-controller.md)
- [Response headers](../components/response-headers.md)
- [HTTP cache store](../components/http-cache-store-bundle.md)
- [Events](../components/events.md)

## Translations and content {#translations-and-content}

- [MIME translations](../components/mime-translatable.md)
- [Translatable fields](../components/translatable-bundle.md)
- [Twig helpers](../components/twig-extra-bundle.md)

## Security and permissions {#security-and-permissions}

- [Permissions](../components/permissions-bundle.md)
- [Filter attack vectors](../components/filter-attack-vectors.md)

## Google Cloud {#google-cloud}

- [Google Cloud integration](../components/google-cloud-integration.md)
- [Google Cloud tracing](../components/google-cloud-trace.md)

## Other packages {#other-packages}

- [Crudl bundle](../components/crudl-bundle.md)
- [Time ago bundle](../components/time-ago-bundle.md)

## CMS integrations {#cms-integrations}

- [CMS API](cms-api-plugin.md): read-only HTTP JSON endpoints.
- [CMS MCP](cms-mcp-plugin.md): structured tools for MCP consumers.
- [CMS Mate](cms-mate-extension.md): local inspection through the host Symfony kernel.
