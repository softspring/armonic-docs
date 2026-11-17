---
title: "Config Bundle (Experimental)"
description: "Declare shared configuration defaults and inject values into Symfony services."
related:
    - "components"
    - "configuration"
---

# Config Bundle {#config-bundle}

> **Experimental — 6.1-dev:** the configuration and administration APIs may change before a stable release.

`softspring/config-bundle` gives projects and bundles a shared configuration provider.
You can declare defaults, inject a value into an existing service, or pass a value object
that asks the provider for a value on each operation.

The provider reads configured defaults and, when administration is enabled, non-sensitive database
overrides. Administration uses CRUDL actions and configurable permissions. Encryption, auditing
and secret-manager adapters are planned; the administration screen is not a credential store.

## Requirements {#requirements}

- PHP 8.4 or newer.
- Symfony 6.4, 7.4, or 8.x, within the constraints declared by the package.
- The defaults-only configuration does not require a database connection. Enable administration only after configuring Doctrine and CRUDL.

This guide belongs to Git branch `6.1`, published as `6.1-dev`.

## Installation {#installation}

```bash
composer require softspring/config-bundle:6.1.*@dev
```

If the package is not yet available on Packagist, add its VCS repository to the project's
`composer.json` before running that command:

```json
{
    "repositories": [
        {"type": "vcs", "url": "https://github.com/softspring/config-bundle"}
    ]
}
```

Merge this fragment with any existing repositories. The package does not yet have a Flex
recipe. Add this entry to the existing array in `config/bundles.php`:

```php
Softspring\Bundle\ConfigBundle\SfsConfigBundle::class => ['all' => true],
```

No package configuration is needed just to boot the bundle.

## Declare defaults {#declare-defaults}

Define application defaults under `sfs_config.defaults`. Keys are case-sensitive and may
contain dots. Null, false, zero, empty strings, and arrays are values, not missing keys.

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_config.yaml
sfs_config:
    defaults:
        mailchimp.api_key: '%env(MAILCHIMP_API_KEY)%'
        ai.system_instructions: 'Write clear and useful answers.'
        feature.enabled: false
        feature.limit: 0
        feature.options: ['summary', 'translation']
        feature.optional: null
```

```php
<?php

// config/packages/sfs_config.php
use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_config', [
        'defaults' => [
            'mailchimp.api_key' => '%env(MAILCHIMP_API_KEY)%',
            'ai.system_instructions' => 'Write clear and useful answers.',
            'feature.enabled' => false,
            'feature.limit' => 0,
            'feature.options' => ['summary', 'translation'],
            'feature.optional' => null,
        ],
    ]);
};
```

<!-- config-tabs:end -->

Choose one format. Set `MAILCHIMP_API_KEY` in the application's environment or Symfony
secrets when using this example. Never put a real credential in a literal default: literals
can be included in the compiled container. Defaults referencing `%env(...)%` are resolved
when the defaults provider is constructed, not on every call to `get()`.

Other bundles can supply defaults and definitions through the standard Symfony extension configuration
mechanism. Applications can override this configuration using their normal Symfony package configuration.

## Read values through the provider {#read-values-through-the-provider}

With normal Symfony autowiring enabled, inject the provider interface:

```php
<?php

namespace App\Ai;

use Softspring\Bundle\ConfigBundle\Config\ConfigProviderInterface;

final class PromptInstructions
{
    public function __construct(private ConfigProviderInterface $config)
    {
    }

    public function get(): string
    {
        $value = $this->config->get('ai.system_instructions');
        if (!is_string($value)) {
            throw new \UnexpectedValueException('AI instructions must be a string.');
        }

        return $value;
    }
}
```

`get()` returns the effective value without casting it. Consumers must check the expected type where needed.
Registered definitions select the administration form type; undeclared keys remain available through defaults
but are not exposed as editable settings.
A missing key throws `Softspring\Bundle\ConfigBundle\Exception\ConfigNotFoundException`.
A declared null returns null.

## Inject a direct value {#inject-a-direct-value}

Use the env processor for an existing service that accepts a scalar or array. This example
assumes `App\Mailchimp\Client` already has a constructor argument named `$apiKey`:

<!-- config-tabs:start -->

```yaml
# config/services.yaml (additional service definition)
services:
    App\Mailchimp\Client:
        arguments:
            $apiKey: '%env(sfs_config:mailchimp:api_key)%'
```

```php
<?php

// config/services.php (additional service definition)
use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;

return static function (ContainerConfigurator $container): void {
    $container->services()->set(App\Mailchimp\Client::class)
        ->arg('$apiKey', '%env(sfs_config:mailchimp:api_key)%');
};
```

<!-- config-tabs:end -->

The processor maps `mailchimp:api_key` to the provider key `mailchimp.api_key`.
Use colons between key segments in this expression; a dotted final env name is not valid.
The injected value keeps its original type. No actual environment variable named
`mailchimp:api_key` is required.

This injection is a snapshot for the lifetime of the service. It does not update a scalar
already held by a client or worker. It also cannot make options needed during container
compilation dynamic. Do not use this store to bootstrap its own database connection or
encryption key when those capabilities are added.

## Read through a value object {#read-through-a-value-object}

Use `#[SfsConfig('key')]` on a constructor parameter to inject a `ConfigValueInterface` bound
to one setting. Enable Symfony `autoconfigure: true` on the consumer; applications normally
set this in their service defaults. The attribute supports constructor parameters typed as
`ConfigValueInterface` or `ConfigValue`. It rejects scalar parameters and method parameters.
For a direct scalar snapshot, use the env processor described above.

```php
<?php

namespace App\Ai;

use Softspring\Bundle\ConfigBundle\Attribute\SfsConfig;
use Softspring\Bundle\ConfigBundle\Config\ConfigValueInterface;

final class PromptBuilder
{
    public function __construct(
        #[SfsConfig('ai.system_instructions')]
        private readonly ConfigValueInterface $instructions,
    ) {
    }

    public function build(string $input): string
    {
        $instructions = $this->instructions->get();
        if (!is_string($instructions)) {
            throw new \UnexpectedValueException('AI instructions must be a string.');
        }

        return $instructions."\n\n".$input;
    }
}
```

The equivalent explicit configuration registers a normal Symfony value service. This also
works for consumers without attributes or autoconfiguration:

<!-- config-tabs:start -->

```yaml
# config/services.yaml (additional service definition)
services:
    app.ai.system_instructions:
        class: Softspring\Bundle\ConfigBundle\Config\ConfigValue
        arguments:
            $provider: '@sfs_config.provider'
            $key: 'ai.system_instructions'

    App\Ai\PromptBuilder:
        arguments:
            $instructions: '@app.ai.system_instructions'
```

```php
<?php

// config/services.php (additional service definition)
use Softspring\Bundle\ConfigBundle\Config\ConfigValue;
use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;
use function Symfony\Component\DependencyInjection\Loader\Configurator\service;

return static function (ContainerConfigurator $container): void {
    $services = $container->services();
    $services->set('app.ai.system_instructions', ConfigValue::class)
        ->arg('$provider', service('sfs_config.provider'))
        ->arg('$key', 'ai.system_instructions');
    $services->set(App\Ai\PromptBuilder::class)
        ->arg('$instructions', service('app.ai.system_instructions'));
};
```

<!-- config-tabs:end -->

Both forms construct a value object without reading the setting. Each `get()` calls the
configured provider again and preserves its original value, including null, false, zero and
empty arrays. An unknown key fails when read, according to the provider's normal behavior.
A custom provider can supply keys that are absent from the defaults registry.

The initial `DefaultsConfigProvider` is immutable; wrapping it does not make its defaults
change. The administration provider reads the current database override on each call.
Replacement providers retain their own cache and refresh policies. A downstream client that
copies the result of `get()` still needs its own refresh strategy, particularly in workers.

## Replace or decorate the provider {#replace-or-decorate-the-provider}

`ConfigProviderInterface` is aliased to `sfs_config.provider`. Applications can replace
that service with another implementation or decorate it to add a source while retaining
defaults. For a decorator, inject `.inner` rather than `ConfigProviderInterface` into the
decorator itself to avoid a circular dependency.

A custom provider must preserve the distinction between an absent key and explicit null,
false, zero, or empty values. Define failure behavior as well as precedence: a failed
secret lookup or unreadable ciphertext must not silently become a different credential.
Do not claim that a custom integration is provided by the bundle until it is implemented.

## Register editable values {#register-editable-values}

The optional administration uses the shared CRUDL actions. Applications and bundles register known keys;
editors cannot create arbitrary configuration keys. Each editable key must have a default and a key of at most
190 letters, digits, dots, underscores or hyphens, starting with a letter. Definitions support `string`, `text`, `boolean`, `integer`, `float`, and `json` form types.
A multiline instruction field is a `text` setting.

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_config.yaml
sfs_config:
    defaults:
        app.editorial_instructions: ''
    groups:
        cms:
            label: config.cms.label
            description: config.cms.description
            translation_domain: app_configuration
            priority: 100
    definitions:
        app.editorial_instructions:
            type: text
            label: 'Editorial instructions'
            group: cms
            help: 'Additional guidance for generated content.'
            max_length: 20000
            permissions:
                read: ROLE_ADMIN
                update: ROLE_SUPER_ADMIN
                delete: ROLE_SUPER_ADMIN
    admin:
        enabled: true
        permissions:
            list: ROLE_ADMIN
            read: ROLE_ADMIN
            update: ROLE_ADMIN
            delete: ROLE_ADMIN
```

```php
<?php

// config/packages/sfs_config.php
use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_config', [
        'defaults' => ['app.editorial_instructions' => ''],
        'groups' => ['cms' => [
            'label' => 'config.cms.label',
            'description' => 'config.cms.description',
            'translation_domain' => 'app_configuration',
            'priority' => 100,
        ]],
        'definitions' => ['app.editorial_instructions' => [
            'type' => 'text',
            'label' => 'Editorial instructions',
            'group' => 'cms',
            'help' => 'Additional guidance for generated content.',
            'max_length' => 20000,
            'permissions' => [
                'read' => 'ROLE_ADMIN', 'update' => 'ROLE_SUPER_ADMIN', 'delete' => 'ROLE_SUPER_ADMIN',
            ],
        ]],
        'admin' => ['enabled' => true, 'permissions' => [
            'list' => 'ROLE_ADMIN', 'read' => 'ROLE_ADMIN', 'update' => 'ROLE_ADMIN', 'delete' => 'ROLE_ADMIN',
        ]],
    ]);
};
```

<!-- config-tabs:end -->

Global action permissions default to `ROLE_ADMIN`. A per-key permission is an additional requirement and
cannot bypass the global permission. Update and reset also require read access to the key. Permissions are
Symfony authorization attributes: applications can supply their own roles or voters without a particular user bundle.
Form submissions are protected by CSRF and their real form types; numeric values reject empty submissions unless
`nullable: true` is configured. JSON fields validate their input before saving. Setting labels and help use the `sfs_config` translation domain. A setting's `group` is a stable key, not a translated title.

### Organize settings into blocks {#configuration-groups}

Register configuration blocks under `sfs_config.groups`. Each key identifies a block independently of its
translated title. A setting joins a block by referencing its key in `definitions.<key>.group`.
Applications and bundles can contribute groups through the same Symfony extension configuration used for defaults.

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_config.yaml (additional configuration)
sfs_config:
    groups:
        cms:
            label: config.cms.label
            description: config.cms.description
            translation_domain: app_configuration
            priority: 100
        mailer:
            label: config.mailer.label
            description: config.mailer.description
            translation_domain: app_configuration
            priority: 50
    definitions:
        app.editorial_instructions:
            group: cms
```

```php
<?php

// config/packages/sfs_config.php (additional configuration)
use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_config', [
        'groups' => [
            'cms' => [
                'label' => 'config.cms.label', 'description' => 'config.cms.description',
                'translation_domain' => 'app_configuration', 'priority' => 100,
            ],
            'mailer' => [
                'label' => 'config.mailer.label', 'description' => 'config.mailer.description',
                'translation_domain' => 'app_configuration', 'priority' => 50,
            ],
        ],
        'definitions' => ['app.editorial_instructions' => ['group' => 'cms']],
    ]);
};
```

<!-- config-tabs:end -->

This fragment extends the setting and default in the previous example. It declares a Mailer block to which an
application or a bundle can add its own settings; it does not register Mailer values. Add the labels to your catalog:

```yaml
# translations/app_configuration.en.yaml
config.cms.label: CMS
config.cms.description: Content management settings.
config.mailer.label: Mailer
config.mailer.description: Email delivery settings.
```

Provide another catalog, such as `app_configuration.es.yaml`, with the same translation keys for Spanish.
`label` is required; `description` is optional. The default block translation domain is `sfs_config`.
Higher priority blocks appear first; equal priorities use their stable keys as a deterministic tie-breaker.
Blocks and their settings keep the existing per-key permissions. Empty blocks, including blocks whose settings
are all inaccessible or sensitive, are not rendered. A registered Mailer block appears when it has readable settings.

Settings without an explicit group use the built-in `general` block. Applications can override its label,
translation domain and priority. Unknown group references are rejected while configuring the container.
When updating earlier 6.1 development definitions, replace translated group text with the registered group key.
Groups and translation metadata are Symfony configuration; changing a stored setting value does not rewrite them.

### Enable administration {#enable-administration}

Register `SfsConfigBundle`, DoctrineBundle, TwigBundle, SecurityBundle, the shared
`Softspring\CrudlBundle\SfsCrudlBundle`, and its
`Softspring\Component\DynamicFormType\SfsDynamicFormTypeBundle` dependency. Use normal Symfony
framework form, CSRF, translation and validation support. Doctrine ORM needs a configured database connection.
Admin routes are opt-in and must be imported behind the application's admin firewall:

<!-- config-tabs:start -->

```yaml
# config/routes/config_admin.yaml
sfs_config_admin:
    resource: '@SfsConfigBundle/config/routes/admin.yaml'
    prefix: /admin/configuration
```

```php
<?php

// config/routes/config_admin.php
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return static function (RoutingConfigurator $routes): void {
    $routes->import('@SfsConfigBundle/config/routes/admin.yaml')->prefix('/admin/configuration');
};
```

<!-- config-tabs:end -->

Enabling administration registers the Doctrine entity mapping and, when DoctrineMigrationsBundle is enabled,
the bundle's migration namespace from `src/Migrations`. Run the usual application migration command before using the routes:

```bash
php bin/console doctrine:migrations:migrate
```

Review migrations normally before running them in a deployed environment. The bundle uses Doctrine's schema
API for its own `sfs_config_value` table. Administration flows are tested with SQLite and the local MySQL application. PostgreSQL runtime validation
remains pending.

The routes `sfs_config_admin_list`, `sfs_config_admin_read`, `sfs_config_admin_update`, and
`sfs_config_admin_delete` use the configured CRUDL controller. Update creates the first stored override or edits
an existing one. Delete is a reset action with its own confirmation form: it removes the override, preserving the
registered key and its default. There is no separate action for creating or deleting definitions.
The list includes defaults even when no override exists, filters inaccessible keys, and identifies the active source. Each setting shows its translated label and help text. Technical configuration keys are not displayed in the list.
Views and action permissions can be customized through the standard `sfs_crudl.controllers.sfs_config` configuration.
Override `@SfsConfig/admin/layout.html.twig` in the application's bundle templates to use its admin layout.

The list, read, edit and reset screens provide translated breadcrumbs. The list shows Configuration;
setting screens link back to the list, and edit/reset screens also link to the setting's read view.
The current item has `aria-current="page"`. The default layout renders its own breadcrumb navigation.
When integrating an application admin layout, render the shared `breadcrums_content` block (the spelling
matches the other Softspring bundles). The screen templates call `parent()` to preserve the application's
navigation, such as its CMS entry, before adding the configuration items.


### Create administration pages for groups {#group-administration-pages}

Keep the general administration and add focused pages for registered configuration groups. Import the same
route resource again, using a unique `name_prefix` and a fixed `_sfs_config_group` route default. For example,
this extends the earlier `cms` group registration:

<!-- config-tabs:start -->

```yaml
# config/routes/config_admin.yaml
cms_configuration:
    resource: '@SfsConfigBundle/config/routes/admin.yaml'
    prefix: /admin/configuration/cms
    name_prefix: cms_
    defaults:
        _sfs_config_group: cms

configuration:
    resource: '@SfsConfigBundle/config/routes/admin.yaml'
    prefix: /admin/configuration
```

```php
<?php

// config/routes/config_admin.php
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return static function (RoutingConfigurator $routes): void {
    $routes->import('@SfsConfigBundle/config/routes/admin.yaml')
        ->prefix('/admin/configuration/cms')
        ->namePrefix('cms_')
        ->defaults(['_sfs_config_group' => 'cms']);

    $routes->import('@SfsConfigBundle/config/routes/admin.yaml')->prefix('/admin/configuration');
};
```

<!-- config-tabs:end -->

Replace the previous general import with this file; do not import the general routes twice. Import fixed
subpages before the general resource so its `/{id}` route does not capture their paths.
The CMS page is `/admin/configuration/cms/`, with routes `cms_sfs_config_admin_list`,
`cms_sfs_config_admin_read`, `cms_sfs_config_admin_update` and `cms_sfs_config_admin_delete`.
The general page keeps its original `sfs_config_admin_*` names and shows all readable groups.
Normal application host, locale prefixes and firewall protection still apply to both imports.

The group page shows only readable, non-sensitive settings from that group. Its translated heading uses
the group's label and translation domain. Links, breadcrumbs, back buttons and successful save/reset
redirects retain the imported route prefix. Keep the action suffixes from the resource when reusing these
routes: the bundle derives sibling route names from the matched route.

An unknown group or a key from another group returns HTTP 404. A registered group without readable settings
renders the normal empty state. The group comes from route attributes; query parameters do not select or
replace it. Group selection does not grant access: the existing global action and per-key permissions still
apply to every request, including submitted forms. The general page remains available to its authorized users.

To add Mailer or another group later, register its definitions and repeat the scoped import with another
path and name prefix. Link `cms_sfs_config_admin_list` or the corresponding list route from the application's
sidebar. The bundle does not require a particular menu implementation.

### Resolution and refresh {#resolution-and-refresh}

With administration disabled, the provider continues to use declared defaults without a database query.
When enabled, a database override wins over the default, including explicit null, false, zero, empty strings and
empty arrays. A reset restores the fallback. Database lookup failures are errors rather than silent fallback.

Each provider call reads the current database value, avoiding a retained in-process value across worker requests.
There is no distributed cache to invalidate. A scalar previously injected into another service is still a snapshot:
consumers that need changes immediately should call `ConfigProviderInterface` or `ConfigValueInterface` at use time.

### Sensitive values {#sensitive-values}

Definitions marked `sensitive: true` are excluded from administration, including direct read, edit and reset URLs.
This initial administration supports non-sensitive settings. It does not provide encrypted secret editing. Keep
credentials in environment variables or the application's secret manager. Configuration reads are still internal
application operations and must not be exposed through an unrestricted endpoint.

## Remaining planned scope {#planned-functional-scope}

Encrypted sensitive overrides, key rotation, an actor audit, optional secret-manager providers and richer
per-definition validation remain planned work. Database credentials and encryption bootstrap must stay independent
from the configuration store.

## Contributing {#contributing}

Development targets `6.1`. Read the package's
[CONTRIBUTING.md](https://github.com/softspring/config-bundle/blob/6.1/CONTRIBUTING.md)
for maintenance commands and validation guidance. The functional contract remains in
[FEATURES.md](https://github.com/softspring/config-bundle/blob/6.1/FEATURES.md);
keep implemented behavior and planned work distinct in both that contract and this guide.

Package working rules are included in the package's own
[AGENTS.md](https://github.com/softspring/config-bundle/blob/6.1/AGENTS.md).
No access to a private template repository is required.

[Report issues](https://github.com/softspring/config-bundle/issues) or
[send a pull request](https://github.com/softspring/config-bundle/pulls).
