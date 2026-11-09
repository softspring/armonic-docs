---
title: "CMS Mate Extension (Experimental)"
description: "Give local AI Mate tools access to the host Symfony application for read-only CMS inspection."
related: ["components/cms-api-plugin", "components/cms-mcp-plugin"]
---

# CMS Mate Extension {#cms-mate-extension}

> **Experimental — 6.1@dev:** configuration, APIs and output may change before a stable release. Use this package in development and validate the workflows enabled in your application.

The Mate extension lets a local AI Mate process inspect CMS content through the host Symfony application's kernel and Doctrine setup. It is experimental and distinct from the HTTP API and MCP server plugins.

## Install in the host project {#install-in-the-host-project}

```bash
composer require softspring/cms-mate-extension:6.1.*@dev
```

The project needs CMS core, Doctrine ORM, and Symfony AI Mate. The package's Composer `extra.ai-mate` metadata declares `src/Capability`, `config/config.php`, and its instruction file for Mate discovery. It is not a Symfony bundle to add to `config/bundles.php`.

Run Mate with the intended Symfony application as its root. The bootstrap helper expects an autoloadable `App\Kernel`, loads that project's `.env` through Dotenv when available, and uses `APP_ENV` / `APP_DEBUG` to boot its container.

## Inspect content {#inspect-content}

The available tools are:

| Tool | Purpose |
| --- | --- |
| `cms-content-types` | List configured content types |
| `cms-content-search` | Search by type, text, site, language, and publication state |
| `cms-content-get` | Read one item by type and identifier |

Begin by listing content types, then search within the required type, site, and language before reading a specific item. This avoids assuming a project's content model from generic names.

## Choose the correct environment {#choose-the-correct-environment}

The tools use the application's real connection and services. Read-only tools can still inspect data available to that environment; use a sanitized development database when working with an external model or an untrusted client.

This extension does not import, export, load fixtures, edit content, or offer a public HTTP endpoint. Use [CMS API](cms-api-plugin.md) for HTTP clients and [CMS MCP](cms-mcp-plugin.md) for MCP consumers.

## Extend a local capability {#extend-a-local-capability}

Place additional tools under `Capability`, reusable readers under `Service`, and register their services in the extension's configuration. Keep inputs scoped and use the CMS model/serializer APIs instead of exposing arbitrary database queries.
