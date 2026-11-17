---
title: "CMS MCP Plugin (Experimental)"
description: "Expose read-only CMS tools to MCP consumers using published content and shared serializers."
related: ["components/cms-api-plugin", "bundles/cms-ai-plugin"]
---

# CMS MCP Plugin {#cms-mcp-plugin}

> **Experimental — 6.1-dev:** configuration, APIs and output may change before a stable release. Use this package in development and validate the workflows enabled in your application.

The MCP plugin registers read-only tools for site context, configuration, published content, internal links, menus, images, and optional traffic metrics. Use it when an agent should query CMS context through a structured tool interface.

## Install and register {#install-and-register}

```bash
composer require softspring/cms-mcp-plugin:6.1.*@dev
```

Register the bundle when Flex does not do it automatically:

```php
// config/bundles.php
Softspring\CmsApiPlugin\SfsCmsApiPlugin::class => ['all' => true],
Softspring\CmsMcpPlugin\SfsCmsMcpPlugin::class => ['all' => true],
```

The host application also needs Symfony MCP Bundle enabled and its server/transport configured. Registering these tool services does not create a universally named public endpoint. Choose transport, discovery, and access policies in that server configuration.

## Select tools by task {#select-tools-by-task}

| Task | Tool |
| --- | --- |
| Find sites | `sfs_cms_sites_list` |
| Inspect one site's context | `sfs_cms_sites_get_context` |
| Inspect content/layout/module definitions | `sfs_cms_configuration_get_context` |
| Find published pages | `sfs_cms_contents_search_published` |
| Read a published item | `sfs_cms_contents_get_published` |
| Find internal link destinations | `sfs_cms_routes_find_internal_links` |
| Read navigation | `sfs_cms_menus_get_context` |
| Find configured image types and media | `sfs_cms_media_images_list_types`, `sfs_cms_media_images_search`, `sfs_cms_media_images_get_context` |
| Read traffic metrics | `sfs_cms_analytics_get_site_metrics`, `sfs_cms_analytics_query_pages` |

## Search before reading a payload {#search-before-reading-a-payload}

For `sfs_cms_contents_search_published`, use an argument object such as:

```json
{"query":"accessibility","contentType":"page","site":"default","locale":"en","limit":10}
```

The tool returns summaries and caps the limit at 50. Use the returned content identifier with `sfs_cms_contents_get_published`; set `includeData: true` only when the payload is needed. Internal-link discovery can use the same query, site, locale, and limit context.

## Understand the boundary {#understand-the-boundary}

Published content tools select the published version rather than unsaved editor data. Configuration tools can expose definitions and, optionally, raw configuration with shared redaction. These are different kinds of context; read-only does not mean every payload is appropriate for every consumer.

Analytics tools consume the [Analytics Plugin](../bundles/cms-analytics-plugin.md) provider chain when available. The MCP package does not configure Plausible or GA4 credentials itself.

The original context tools do not save, import, export, or publish content. See the experimental operation services below for controlled mutations. For an assistant that proposes changes in an open form, use the [CMS AI Plugin](../bundles/cms-ai-plugin.md) and retain its manual save/review workflow.

## Consume the shared API services {#consume-shared-api-services}

MCP depends on `cms-api-plugin` and calls its [application queries](cms-api-plugin.md#reuse-cms-operations)
directly. It does not make HTTP requests to the project. Enable the API bundle so Symfony can register those
services; no REST route import or Swagger/Nelmio installation is needed for MCP.

Site and menu tools use `SiteQuery` and `MenuQuery`. Content tools use `PublishedContentQuery` and the same
`ContentProvider` used by REST content queries. Media tools use `MediaImageQuery` and the shared `MediaProvider`.
Configuration tools use `ConfigurationContextQuery`. These services are all in
`Softspring\CmsApiPlugin\Application\Query`.

When the Analytics Plugin is enabled, the API layer registers `AnalyticsQuery` against its statistics provider chain.
Without that plugin, analytics tools return an unavailable-provider result. Provider integrations belong to the
Analytics Plugin; the obsolete Plausible driver classes formerly shipped in MCP are no longer provided.

Keep transport authentication in the host application. A project may protect MCP HTTP with an OAuth resource-server
integration or another Symfony authenticator. No OAuth server, user entity, or user bundle is required by this package.
STDIO uses the process's configured access policy and does not inherit an HTTP login automatically.

## Protect HTTP with OAuth {#protect-http-with-oauth}

OAuth belongs to the host application. The CMS packages do not require a particular authorization server or user
bundle. Reuse an existing OAuth provider and Symfony authenticator when available. For an application that needs
its own authorization server, [League OAuth2 Server Bundle](https://github.com/thephpleague/oauth2-server-bundle)
is an optional integration.

Install it in the host application, register `LeagueOAuth2ServerBundle`, and follow its
[setup guide](https://github.com/thephpleague/oauth2-server-bundle/blob/2.x/docs/index.md) to configure signing keys,
an encryption key, persistence, and migrations. Keep private keys and encryption secrets outside version control.
Both the console process and the HTTP worker must be able to read their configured keys.

Use authorization code with PKCE (`S256`) for interactive clients. Enable refresh tokens if needed, disable password,
implicit, and client credentials grants for this flow, and require a challenge for public clients. Register each
client's exact redirect URI and grant only `cms:read`. A public client has no client secret.

The following example assumes an existing user provider named `app_users`, an admin/login firewall on the same host,
and an MCP route at `/_mcp`. Replace the provider and host parameter with the application's own values.
Insert the MCP firewall before the existing admin firewall. Keep the authorization endpoint behind the existing
login and administrator policy; only the token endpoint accepts anonymous requests.

<!-- config-tabs:start -->

```yaml
# config/packages/security.yaml (merge into the existing configuration)
security:
    firewalls:
        mcp:
            host: '%admin_domain%'
            pattern: '^/_mcp$'
            stateless: true
            provider: app_users
            oauth2: true
        # Existing admin/login firewall follows here.
    access_control:
        - { host: '%admin_domain%', path: '^/oauth2/token$', roles: PUBLIC_ACCESS }
        # Existing login exceptions follow here, then the protected admin policy.
        - { host: '%admin_domain%', path: '^/', roles: ROLE_ADMIN }
```

```php
<?php

// config/packages/security.php (merge into the existing configuration)
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('security', [
        'firewalls' => [
            'mcp' => [
                'host' => '%admin_domain%',
                'pattern' => '^/_mcp$',
                'stateless' => true,
                'provider' => 'app_users',
                'oauth2' => true,
            ],
            // Existing admin/login firewall follows here.
        ],
        'access_control' => [
            ['host' => '%admin_domain%', 'path' => '^/oauth2/token$', 'roles' => 'PUBLIC_ACCESS'],
            // Existing login exceptions follow here, then the protected admin policy.
            ['host' => '%admin_domain%', 'path' => '^/', 'roles' => 'ROLE_ADMIN'],
        ],
    ]);
};
```

<!-- config-tabs:end -->

Expose one language-independent MCP endpoint at `/_mcp`. Pass `locale` as a tool argument when a query or operation
needs a content language; the connection URL does not select it. Keep any load balancer path exceptions and the
Symfony firewall aligned with this exact endpoint.

Import the OAuth routes on the admin host, and require the read scope on the MCP route import:

<!-- config-tabs:start -->

```yaml
# config/routes/oauth_mcp.yaml
league_oauth2_server:
    resource: '@LeagueOAuth2ServerBundle/config/routes.php'
    type: php
    prefix: /oauth2
    host: '%admin_domain%'

_mcp:
    resource: .
    type: mcp
    host: '%admin_domain%'
    defaults:
        oauth2_scopes: ['cms:read']
```

```php
<?php

// config/routes/oauth_mcp.php
namespace Symfony\Component\Routing\Loader\Configurator;

return static function (RoutingConfigurator $routes): void {
    $routes->import('@LeagueOAuth2ServerBundle/config/routes.php', 'php')
        ->prefix('/oauth2')->host('%admin_domain%');
    $routes->import('.', 'mcp')->host('%admin_domain%')
        ->defaults(['oauth2_scopes' => ['cms:read']]);
};
```

<!-- config-tabs:end -->


Keep Symfony MCP's host allowlist aligned with the route host. A valid token can still receive HTTP 403 when DNS
rebinding protection rejects the host header. With Symfony MCP Bundle versions that support `allowed_hosts`, use:

<!-- config-tabs:start -->

```yaml
# config/packages/mcp.yaml
mcp:
    http:
        allowed_hosts: ['%admin_domain%']
```

```php
<?php

// config/packages/mcp.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('mcp', ['http' => ['allowed_hosts' => ['%admin_domain%']]]);
};
```

<!-- config-tabs:end -->

The host application must handle League's `OAuth2Events::AUTHORIZATION_REQUEST_RESOLVE` event. Render a consent
form for the authenticated user, protect it with Symfony CSRF, and bind consent to the client, user, redirect URI,
state, PKCE challenge, and scopes. Resolve authorization only after a valid explicit allow/deny submission.
Do not automatically approve authorization on a GET request.

Register a public development client, adapting its name and redirect URI to the consumer:

```bash
php bin/console league:oauth2-server:create-client --public \
    --redirect-uri=http://127.0.0.1:8765/callback \
    --grant-type=authorization_code --grant-type=refresh_token \
    --scope=cms:read 'Local MCP test' mcp-local-test
```

The consumer generates a random verifier and state, sends the `S256` challenge to `/oauth2/authorize`, verifies the
returned state, then exchanges the code and verifier at `/oauth2/token`. Send the resulting token as
`Authorization: Bearer <access_token>` on every HTTP MCP request. The resource server must validate the token,
required scope, and current user permissions. With the example policy, that user must still have `ROLE_ADMIN`.
Browser login cookies alone do not authenticate the stateless MCP firewall.

This configuration provides OAuth endpoints and bearer validation. Automatic MCP OAuth discovery, protected-resource
metadata, and dynamic client registration are separate integrations; registering CMS tools does not implement them.

## Manage CMS elements {#manage-cms-elements}

Experimental operation services are being tested in the 6.1 development line. They reuse the configured CMS
forms, entity managers, permissions, and action events. REST and MCP call the same API application service;
MCP does not call HTTP controllers. The operation catalog reflects the handlers installed in the application.
It is not a guarantee that every CMS or third-party plugin feature is supported.

The operation adapters currently require the corresponding CMS admin action definitions and form services to be
registered. They are a compatibility layer over that workflow, not an independent replacement for it.

Use `sfs_cms_resources_read` to inspect `contents`, `content_versions`, `routes`, `media`, `menus`, `blocks`,
`sections`, `section_versions`, or `sites`. Content detail requires `filters.type`; version lists require
`filters.content` or `filters.section`. This includes drafts and requires the host permission `CMS_API_READ`.
The original published-content tools remain available.

To preview a draft, call `sfs_cms_resources_read` with `resource: "content_preview"`, the content `id`, and
`filters.type`. Optional filters select `version`, `locale`, and `site`:

```json
{"resource":"content_preview","id":"CONTENT_ID","filters":{"type":"article","version":"VERSION_ID","locale":"es","site":"blog"}}
```

This uses the API's [content preview query](cms-api-plugin.md#preview-content-version) and the CMS compiler.
Inspect `ok`, `errors` and `containerErrors`, then review the returned `html` with its assets in a browser.
A successful compile is not proof of a correct layout. Preview requires `CMS_API_READ`, does not persist compiled
data and does not publish. Publishing remains a separate operation requiring the host publication permission.

Use this workflow for mutations:

1. Call `sfs_cms_operations_list` to discover installed operations and their required scopes.
2. Call `sfs_cms_operation_schema` with `resource`, `action`, and `context`.
3. Inspect the actual form fields, choice values, current values, and collection prototypes.
4. Call `sfs_cms_operation_execute` with matching `context` and `data`. Validation failures return
   `ok: false` with field errors; a successful operation returns its element identifier.
5. Read the result and verify its state. Creating a content element does not publish it.

For example, inspect the creation form for a project-defined content type:

```json
{"resource":"contents","action":"create","context":{"type":"article"}}
```

The fields depend on the host application's definitions and form extensions. Supply the requested sites, locales,
route identifiers and paths, and any project-specific fields. Do not assume every application has an article type,
blog site, author field, or the same modules. Create/edit a version using `content_versions` and `version_create`,
with the parent `content` and its content `type` in context. Inspect that version's form before submitting module data.

Operation context accepts identifiers: `id`, `type`, `content`, `section`, `version`, `prevVersion`, `locale`, and
`layout`. Arbitrary internal request attributes and service names are rejected. Submit form data as an object.
Collections, localized values and module containers retain the structure described by their Symfony form.
File fields accept `{"_file":{"name":"example.png","base64":"..."}}`; the normal media validators and storage
pipeline still apply. File operations can have external storage effects that a database rollback cannot undo.

The HTTP equivalents, relative to the configured API prefix, are:

| Method | Path | Input |
| --- | --- | --- |
| GET | `/resources/{resource}` | Query filters |
| GET | `/resources/{resource}/{id}` | Query filters |
| GET | `/operations` | None |
| POST | `/operations/{resource}/{action}/schema` | `context` object |
| POST | `/operations/{resource}/{action}` | `context`, `data`, optional `confirmed` |

The host must grant the Symfony authorization attributes `CMS_API_READ`, `CMS_API_WRITE`, `CMS_API_PUBLISH`, and
`CMS_API_DELETE` as appropriate. No OAuth bundle or user implementation is required by these services. An OAuth
host can map them to `cms:read`, `cms:write`, `cms:publish`, and `cms:delete` while also checking current user roles.
Keep the default grant read-only and require a new consent flow to obtain additional scopes. Existing read tokens
must never acquire write access just because the user is an administrator.

Publishing, unpublishing, recompilation, and clearing compiled versions require publication permission.
Deletion and cleanup require deletion permission. These operations also require `confirmed: true`; a request to
create a draft does not imply confirmation to publish or delete it. Native CMS action permissions still apply.

The current operation bridge does not provide unrestricted configuration-file editing, site-definition creation,
user administration, or a general plugin import/export gateway. Menu deletion and full media lifecycle coverage
remain outside the validated operation set. Discover actual operations rather than inventing action names.

## Inspect version SEO statistics {#version-seo-statistics}

Use `sfs_cms_seo_version_statistics` with an explicit content type, content id and version id:

```json
{"contentType":"article","contentId":"CONTENT_ID","versionId":"VERSION_ID","locale":"en","site":"main","limit":20}
```

Locale, site and limit are optional. The tool uses the API's `SeoStatisticsQuery` and the
[SEO Plugin's deterministic analyzer](../bundles/cms-seo-plugin.md#version-word-statistics).
It requires the host `CMS_API_READ` permission, verifies version ownership and can inspect drafts.
It returns the same frequency, density, emphasis, tag-context and parameter data used by Twig and the admin screens.

Check `ok` before reading rankings. Without the SEO Plugin the tool returns `installed: false` and `ok: false`.
Authorization and invalid-context failures return `ok: false` with a structured error status. Rendering failures also
return `ok: false`; they must not be interpreted as zero words. The tool never publishes or updates SEO metadata.
No AI provider is involved in calculating the statistics.
