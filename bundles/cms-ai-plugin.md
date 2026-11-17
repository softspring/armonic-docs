---
title: "CMS AI Plugin (Experimental)"
description: "Use the experimental CMS assistant to inspect published context and review generated draft changes before saving."
related: ["components/form-schema", "components/cms-mcp-plugin", "bundles/cms-bundle/content-versions"]
---

# CMS AI Plugin {#cms-ai-plugin}

> **Experimental — 6.1-dev:** configuration, APIs and output may change before a stable release. Use this package in development and validate the workflows enabled in your application.

The AI plugin adds an admin chatbot, a content-editor agent panel, site instructions, and media generation helpers. It is experimental and requires a configured Symfony AI platform.

## Install and register {#install-and-register}

```bash
composer require softspring/cms-ai-plugin:6.1.*@dev
```

Register the bundle when Flex does not do it automatically:

```php
// config/bundles.php
Softspring\CmsApiPlugin\SfsCmsApiPlugin::class => ['all' => true],
Softspring\CmsAiPlugin\SfsCmsAiPlugin::class => ['all' => true],
```

Enable the API Plugin for its shared query services and register Form Schema as well. No REST routes or Swagger are needed. MCP is optional; enable it only when the assistant needs CMS tools. For an OpenAI-backed installation, install the Symfony AI OpenAI platform package and configure the platform through Symfony AI:
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

## Choose assistant models {#choose-assistant-models}

The content editor and the Armonic AI chat use an ordered list of allowed models per Symfony AI platform.
The default selection contains the current OpenAI family (`gpt-6.1-sol`, `gpt-6.1-terra`, `gpt-6-astra`,
`gpt-6-luna`) and Gemini 4 (`gemini-4-hydrogen`, `gemini-4-argon`, `gemini-4-carbon`).
The first compatible model in each list is the initial selection. These are curated defaults, not a live provider
model feed. New catalog entries do not automatically become selectable.

Use `assistant_models` to replace a platform's list, change its order, or explicitly enable an older model:

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_cms_ai.yaml
sfs_cms_ai:
    assistant_models:
        openai: ['gpt-6.1-sol', 'gpt-4.1']
```

```php
<?php

// config/packages/sfs_cms_ai.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_cms_ai', [
        'assistant_models' => ['openai' => ['gpt-6.1-sol', 'gpt-4.1']],
    ]);
};
```

<!-- config-tabs:end -->

This override leaves the Gemini defaults unchanged. Multiple configuration files replace the list for the same
platform instead of appending models. Use an empty list to disable model choices for a platform. Other platforms
and custom platform names need an explicit list. The same allowlist is checked on the server; sending an unlisted
model directly to an assistant endpoint is rejected before calling the provider.

Every selected model must also exist in the platform's Symfony AI model catalog and support message input,
text output and tool calling. The content editor additionally requires structured output. Configuring an image,
embedding or audio-only model here does not make it compatible. Missing catalog entries are hidden; an empty
selection never falls back to an old model. Reloading the editor replaces an old session selection with the first
allowed compatible model.

If the installed Symfony AI catalog predates a model, register its metadata with Symfony AI. For example:

<!-- config-tabs:start -->

```yaml
# config/packages/ai.yaml
ai:
    model:
        gemini:
            gemini-4-hydrogen:
                class: Symfony\AI\Platform\Bridge\Gemini\Gemini
                capabilities: [input-messages, input-image, output-text, output-streaming, output-structured, tool-calling, thinking]
```

```php
<?php

// config/packages/ai.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('ai', ['model' => ['gemini' => [
        'gemini-4-hydrogen' => [
            'class' => \Symfony\AI\Platform\Bridge\Gemini\Gemini::class,
            'capabilities' => ['input-messages', 'input-image', 'output-text', 'output-streaming', 'output-structured', 'tool-calling', 'thinking'],
        ],
    ]]]);
};
```

<!-- config-tabs:end -->

This fragment extends an already configured Gemini platform and requires its Symfony AI platform package.
Declare capabilities from the provider documentation and verify provider support before using a new model.
Catalog compatibility does not prove account access: the API key must have access to the selected model.
Image generation keeps its separate model selection.

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

## Run AI without MCP {#run-ai-without-mcp}

The AI plugin does not require `cms-mcp-plugin` in production. Image helpers and form-based content editing can run
without an MCP server. The content editor reads the selected site's context through `Softspring\CmsApiPlugin\Application\Query\SiteQuery`.

By default, `Softspring\CmsAiPlugin\Tool\ToolProviderInterface` resolves to `NullToolProvider`: the assistant receives
no tools. It can still propose changes to the open form from the supplied editing context. It cannot query additional
published context or traffic metrics unless a tool provider is configured.

When Symfony MCP Bundle is enabled, the plugin registers `Bridge\Mcp\McpToolProvider` as an optional bridge. It adapts
registered `sfs_cms_` tools to Symfony AI tool definitions and executes them through the MCP SDK. Install and enable
`cms-mcp-plugin` to provide those CMS tools. The bridge does not start an HTTP connection to the application's MCP endpoint.

Applications may provide another `ToolProviderInterface` implementation. Its `getDefinitions()` method returns
named tool descriptions and input schemas, `getTools()` returns Symfony AI `Tool` objects, and `execute()` performs
the selected operation. The provider must enforce its own operation/resource authorization; tool discovery alone
does not grant access.

<!-- config-tabs:start -->

```yaml
# config/services.yaml
services:
    App\Ai\ProjectToolProvider:
        autowire: true
    Softspring\CmsAiPlugin\Tool\ToolProviderInterface:
        alias: App\Ai\ProjectToolProvider
```

```php
<?php

// config/services.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

use App\Ai\ProjectToolProvider;
use Softspring\CmsAiPlugin\Tool\ToolProviderInterface;

return static function (ContainerConfigurator $container): void {
    $services = $container->services();
    $services->set(ProjectToolProvider::class)->autowire();
    $services->alias(ToolProviderInterface::class, ProjectToolProvider::class);
};
```

<!-- config-tabs:end -->

The existing chatbot class and routes keep their names for compatibility. Their agents consume the tool-provider
contract rather than depending directly on MCP registry or session classes.

## Add a newer provider model {#add-a-newer-provider-model}

The Symfony AI catalog shipped with an installed bridge can predate a provider model. Register a verified model
through Symfony AI's standard `ai.model` configuration in the host application. The CMS assistants will discover
it through the same capability filters; the CMS plugin does not need provider-specific model names in its code.

For example, merge this configuration with the OpenAI platform configuration above to add
[GPT-6.1 Sol](https://developers.openai.com/api/docs/models/gpt-6.1-sol). Its API identifier is `gpt-6.1-sol`.
The OpenAI `Gpt` model class uses the Responses API, which is required for this model's tool calls.

<!-- config-tabs:start -->

```yaml
# config/packages/ai_open_ai_platform.yaml (additional model)
ai:
    model:
        openai:
            gpt-6.1-sol:
                class: Symfony\AI\Platform\Bridge\OpenAi\Gpt
                capabilities:
                    - input-messages
                    - input-image
                    - output-text
                    - output-streaming
                    - output-structured
                    - tool-calling
                    - thinking
```

```php
<?php

// config/packages/ai_open_ai_platform.php (additional model)
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

use Symfony\AI\Platform\Bridge\OpenAi\Gpt;

return static function (ContainerConfigurator $container): void {
    $container->extension('ai', ['model' => ['openai' => ['gpt-6.1-sol' => [
        'class' => Gpt::class,
        'capabilities' => [
            'input-messages', 'input-image', 'output-text', 'output-streaming',
            'output-structured', 'tool-calling', 'thinking',
        ],
    ]]]]);
};
```

<!-- config-tabs:end -->

Reload the editor and select `gpt-6.1-sol` under the OpenAI platform. Registering a model adds it to the catalog;
it does not change an existing session's compatible selection. Provider account access and available API credits
are still required. A model appearing in the selector or a successful model metadata lookup does not prove that
generation, tool calls, or structured output have completed successfully.
