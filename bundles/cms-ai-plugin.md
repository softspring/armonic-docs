---
title: "CMS AI Plugin (Experimental)"
description: "Use the experimental CMS assistant to inspect published context and review generated draft changes before saving."
related: ["components/form-schema", "components/cms-mcp-plugin", "bundles/cms-bundle/content-versions"]
---

# CMS AI Plugin {#cms-ai-plugin}

> **Experimental — 6.1@dev:** configuration, APIs and output may change before a stable release. Use this package in development and validate the workflows enabled in your application.

The AI plugin adds an admin chatbot, a content-editor agent panel, site instructions, and media generation helpers. It is experimental and requires a configured Symfony AI platform.

## Install and register {#install-and-register}

```bash
composer require softspring/cms-ai-plugin:6.1.*@dev
```

Register the bundle when Flex does not do it automatically:

```php
// config/bundles.php
Softspring\CmsAiPlugin\SfsCmsAiPlugin::class => ['all' => true],
```

Register the dependent MCP and Form Schema bundles as well. For an OpenAI-backed installation, install the Symfony AI OpenAI platform package and configure the platform through Symfony AI:
<!-- config-tabs:start -->

```yaml
# config/packages/ai.yaml
ai:
    platform:
        openai:
            api_key: '%env(OPENAI_API_KEY)%'
```

```php
<?php

// config/packages/ai.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('ai', ['platform' => ['openai' => ['api_key' => '%env(OPENAI_API_KEY)%']]]);
};
```

<!-- config-tabs:end -->

Provide the key through local or deployment secrets. The plugin discovers the application's available platforms and model catalogs; select a model available to that configured provider. No provider calls are possible without that setup.

## Register the admin endpoints {#register-the-admin-endpoints}

<!-- config-tabs:start -->

```yaml
# config/routes/sfs_cms_ai.yaml
_sfs_cms_ai:
    resource: '@SfsCmsAiPlugin/config/routing/admin.yaml'
    prefix: /admin/{_locale}/cms-ai
```

```php
<?php

// config/routes/sfs_cms_ai.php
namespace Symfony\Component\Routing\Loader\Configurator;

return static function (RoutingConfigurator $routes): void {
    $routes->import('@SfsCmsAiPlugin/config/routing/admin.yaml')->prefix('/admin/{_locale}/cms-ai');
};
```

<!-- config-tabs:end -->

Keep this prefix behind the admin firewall. The content agent also checks the content type's version-create permission. Site AI settings are contributed through the normal CMS site admin routing provider.

## Work on a draft {#work-on-a-draft}

Open the content editor and describe the change to the agent. It receives the current unsaved form payload, schema, selected site/language, and conversation history. It can consult the read-only [CMS MCP tools](../components/cms-mcp-plugin.md) for published context.

The response is merged with the browser payload and validated through Symfony forms. It updates the open form; it does not create a saved content version. Review the proposed values, preview the result, and use the normal CMS save and publish actions.

Conversation history is kept in the Symfony session for the content, base version, and layout. A debug panel can show returned and merged payloads, tool calls, raw response, and validation errors.

## Provide site instructions {#provide-site-instructions}

Use the site's **AI** tab for audience, tone, editorial rules, SEO guidance, and other site-level instructions. They are stored under `site.metadata.sfs_cms_ai`. Treat them as information passed to the configured provider; do not include confidential material that should not leave the application.

## Align runtime limits {#align-runtime-limits}

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_cms_ai.yaml
sfs_cms_ai:
    http_client:
        timeout: 180.0
        max_duration: 240.0
    content_editor:
        max_execution_time: 240
```

```php
<?php

// config/packages/sfs_cms_ai.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_cms_ai', [
        'http_client' => ['timeout' => 180.0, 'max_duration' => 240.0],
        'content_editor' => ['max_execution_time' => 240],
    ]);
};
```

<!-- config-tabs:end -->

Provider requests can outlast normal admin requests. Align PHP and proxy limits with the intended workflow. An admin panel being visible does not prove the provider is configured, the selected model is usable, or generated content is appropriate for publication.

The media helpers and optional SEO integration have their own provider/model requirements. Test each enabled workflow independently with synthetic input before using editorial material.
