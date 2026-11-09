---
title: "Upgrade from 5.5 or 5.6 to 6.0"
description: "Adapt application code, configuration, entities, database migrations, and frontend assets when upgrading Softspring bundles and components to 6.0."
next: "configuration"
related:
    - "requirements"
    - "configure-database"
    - "bundles/account-bundle"
    - "bundles/cms-bundle"
    - "bundles/media-bundle"
---

# Upgrade from 5.5 or 5.6 to 6.0 {#upgrade-to-6-0}

This guide is for an existing Symfony application using Softspring packages from the `5.5` or `5.6` line. It covers the changes your application may need when moving those packages to `6.0`.

Start with the common requirements, then follow the sections for the packages and extension points your application uses. An application that only consumes a component usually needs fewer changes than one that replaces its controllers, implements its interfaces, or extends its Doctrine models.

The two starting lines are not interchangeable: some fixes and features were backported to `5.5` independently of `5.6`. Compare against the versions in your application's lock file. You do not need to pass through `5.6` before upgrading from `5.5`.

## Plan the upgrade {#plan-the-upgrade}

| If your application uses… | Review these changes |
| --- | --- |
| Any PHP bundle or component | [Runtime and dependencies](#runtime-and-dependencies), [custom extensions](#custom-extensions), and [Doctrine mapping](#doctrine-mapping). |
| Accounts or account-owned data | [The membership model](#account-memberships), configuration names, collection initialization, and authorization. |
| User Bundle | [User contracts and authentication](#user-contracts), old role names, and controller overrides. |
| CMS import, export, or fixtures | [CMS Data Plugin](#cms-data-plugin), including namespaces, event constants, and template overrides. |
| Existing CMS definitions and templates | [CMS configuration and rendering](#cms-configuration-and-rendering). |
| Custom form types or pagination | [Forms, events, and collections](#forms-events-and-collections). |
| Media Bundle | [Media configuration and extensions](#media-configuration-and-extensions), plus image driver requirements. |
| Admin JavaScript, Sass, or copied Twig layouts | [Frontend assets](#frontend-assets). |
| CMS, Sections, Blog, or custom foreign keys into CMS tables | [Existing database upgrades](#existing-database-upgrades). |
| Google Cloud Trace | [Tracing instrumentation](#tracing-instrumentation). |

Perform the first upgrade in an isolated copy of the application with representative data. Keep a database and file-storage backup before applying its deployment plan. A Composer update does not migrate stored data or regenerate compiled CMS pages.

This guide follows the maintained `6.0` documentation scope. Experimental features are outside this migration path. Notification and Subscription are not covered by a validated `6.0` upgrade path; applications using them need a separate compatibility review before changing their package constraints.

## Runtime and dependencies {#runtime-and-dependencies}

### PHP and Symfony {#php-and-symfony}

The Armonic `6.0` PHP stack requires **PHP 8.4 or later**. Upgrade both CLI and web runtimes, including workers and scheduled commands. Updating only the Composer container can leave the deployed application running on an incompatible PHP version.

The packages generally target Symfony **6.4, 7.4, and 8.x**, with individual minimum patch versions in their Composer manifests. Symfony 5.4 and early Symfony 6.x installations must move to a supported line first. Updating Softspring packages does not by itself require moving an existing Symfony 6.4 application to Symfony 8. The standalone application's own Symfony requirements are stricter than the reusable packages' requirements.

Review your application's `config.platform.php` as well as its real runtime. Do not use `--ignore-platform-reqs` to make an incompatible application install.

Doctrine ORM 3 and DBAL 4 are not universal prerequisites: several packages also allow older compatible Doctrine versions. Choose a dependency set supported by every installed package, then apply the upgrade guides for any Doctrine or Symfony major version you actually change. See the [requirements](getting-started/requirements.md).

### Update related Softspring packages together {#update-related-packages-together}

Replace your application's direct `5.5`/`5.6` constraints with the selected `6.0` line. Update its related packages together so Composer can resolve the shared interfaces and models.

For example, this is a **fragment** for an application that directly requires the CMS, users, and accounts:

```json
{
    "require": {
        "php": ">=8.4",
        "softspring/account-bundle": "6.0.*",
        "softspring/cms-bundle": "6.0.*",
        "softspring/user-bundle": "6.0.*"
    }
}
```

Use the release or prerelease constraints appropriate to your deployment. `6.0.*@RC` permits a release candidate; `6.0.x-dev` explicitly selects the development branch. Do not copy a development constraint into production just because an example project uses it. Keep the same package line throughout the application.

After changing all applicable direct constraints, inspect the update before installing it:

```bash
composer update 'softspring/*' --with-all-dependencies --dry-run
composer update 'softspring/*' --with-all-dependencies
composer check-platform-reqs
```

Review and commit the resulting application lock file. `composer recipes:update` is not a substitute for adapting existing code: recipe changes can affect your entities, routes, security, and service configuration and need to be reviewed separately.

## Custom extensions {#custom-extensions}

Several classes now declare parameter and return types that were previously implicit. Code that calls these methods with their documented values often needs no change. Code that implements an interface or overrides a parent method must remain signature-compatible.

### Implement new interface methods {#new-interface-methods}

Check direct interface implementations, not only subclasses. The following contracts add required methods:

| Interface | Methods to provide |
| --- | --- |
| CMS `SiteInterface` and Media `MediaInterface` | `getMetadata(): ?array`, `setMetadata(?array $metadata): void`, `setMetadataField(string $field, mixed $value): void`, `getMetadataField(string $field, mixed $default = null): mixed`, and `removeMetadataField(string $field): void`. |
| Blog `ArticleContentInterface` | `getTags(): array` and `setTags(array $tags): void`. |
| Media `NameGeneratorInterface` | `public static function getPriority(): int`. |
| Media `MediaTypeProviderInterface` | `public static function getPriority(): int`. |
| User `LoginFormInterface` | `supportsManualLogin(): bool`. Return whether the form offers manual credential login; a conventional password form normally returns `true`. |
| User `UserPasswordInterface` | `eraseCredentials(): void`, supplied by the current password trait. |

The bundled base models provide the matching metadata/tag implementations. If you already extend one, check its inherited methods before adding duplicate fields. Direct custom models need their own implementations and matching persistence changes; package migrations targeting default tables do not automatically update an unrelated application table.

For a custom media provider or name generator without a special ordering requirement, a neutral priority is:

```php
// Inside the existing provider or name-generator class:
public static function getPriority(): int
{
    return 0;
}
```

When upgrading from `5.5`, direct `SiteInterface` implementations also need `isEnabled(): bool`. Custom `CmsConfig` overrides must accept the current optional enabled-site filtering arguments: `getSites(bool $onlyEnabled = false)`, `getSite(string $id, bool $required = true, bool $onlyEnabled = false)`, and `getSitesForContent(string $contentType, bool $onlyEnabled = false)`. These filtering contracts were already present in the reviewed `5.6` branch.

### Match overridden signatures {#overridden-signatures}

Examples to check in your application:

| Extension point | `6.0` contract or adjustment |
| --- | --- |
| `StreamedCommandRunner::runCommands()` | Returns `void`. |
| `MediaManagerInterface::migrate()` | Adds `bool $force = false` after the output argument. Forward it when delegating to the bundled manager. |
| `ContentVersionCompiler::compileRequest()` | Adds `bool $saveCompiled = true` as its fourth argument. |
| `PaginatedCollection::add()` | Returns `void`; do not depend on the previous implicit return value. |
| `PaginatedCollection::filter()` and `map()` | Return `Doctrine\Common\Collections\Collection`. |
| `PaginatedCollection::indexOf()` | Returns `int|string|false`. Check a missing element with `=== false`. |
| `ChangeLog::create()` | Returns `static`. An override returning a fixed base class is no longer sufficient. |
| Doctrine Templates lifecycle methods | `_generateId()`, `autoSetCreatedAt()`, and `autoSetUpdatedAt()` declare `: void` where provided by their traits. |
| Twig Extra `ExtensibleAppVariable` | `__call(string $method, array $params)`, `__get(string $property): mixed`, and `__isset(string $property)`. |
| Custom authorization voters | Check the new optional fourth `$vote` argument on the specific voter you extend. Its type differs between compatibility implementations. |

For example, update a custom media manager's method signature from:

```diff
- public function migrate(MediaInterface $media, ?OutputInterface $output = null): void
+ public function migrate(MediaInterface $media, ?OutputInterface $output = null, bool $force = false): void
```

The change must also reach decorators, test doubles, and calls to `parent::migrate()`. Forward `$force`; accepting it and silently ignoring it changes the requested operation.

Review manually constructed services and explicit `arguments:` definitions. Examples include the additional Google Identity Platform configuration argument in `LoginController`, `BlockRenderer` in the CMS `BlockController`, and the account context resolver in account listeners. Prefer autowiring the supported interfaces where possible; do not copy every vendor service definition into the application.

Some User Bundle controllers and listeners now use a non-null `LoggerInterface` with a `NullLogger` default. In particular, a `5.5` application that explicitly passed `null` should omit that optional argument or pass a logger object. An optional argument is not necessarily nullable.

## Doctrine mapping {#doctrine-mapping}

Bundled entity traits and lifecycle callbacks use **PHP attributes** in `6.0`. An application's annotation driver does not read those attributes. Convert the affected application mappings and select the attribute driver for the application namespace.

For example:

```diff
- /** @ORM\Entity @ORM\HasLifecycleCallbacks */
+ #[ORM\Entity]
+ #[ORM\HasLifecycleCallbacks]
  class Project
```

Keep `#[ORM\HasLifecycleCallbacks]` on concrete entities using lifecycle traits. Without it, a valid-looking mapping can still omit identifier or timestamp callbacks.

The following is a Symfony configuration fragment for the application's existing `App` mapping. Keep other entity managers and mappings in your project:

<!-- config-tabs:start -->

```yaml
# config/packages/doctrine.yaml
doctrine:
    orm:
        mappings:
            App:
                type: attribute
                is_bundle: false
                dir: '%kernel.project_dir%/src/Entity'
                prefix: 'App\Entity'
                alias: App
```

```php
<?php

// config/packages/doctrine.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('doctrine', [
        'orm' => [
            'mappings' => [
                'App' => [
                    'type' => 'attribute',
                    'is_bundle' => false,
                    'dir' => '%kernel.project_dir%/src/Entity',
                    'prefix' => 'App\Entity',
                    'alias' => 'App',
                ],
            ],
        ],
    ]);
};
```

<!-- config-tabs:end -->

This does **not** mean converting the packages' XML mapping files. Bundles still register XML mappings for several mapped superclasses. Keep the bundle registrations and their target-entity configuration, and avoid adding a second mapping for the same fields in your application.

For Doctrine ChangeLog, prefer `#[Registrable]` and `#[Ignored]` from `Softspring\DoctrineChangeLogBundle\Mapping` over their annotation spellings. Do not remove `doctrine/annotations` indiscriminately: the ChangeLog package still declares it as a dependency for its compatibility reader.

See Doctrine's [attribute mapping reference](https://www.doctrine-project.org/projects/doctrine-orm/en/3.6/reference/attributes-reference.html).

## Account memberships {#account-memberships}

Account Bundle removes the deprecated single-account, multi-account, and account-user relation helpers. Adapt application entities, services, queries, and templates to the membership contracts.

### Rename the contracts and helpers {#rename-account-contracts}

All names in this table are relative to `Softspring\AccountBundle\`:

| `5.5` / `5.6` name | `6.0` replacement |
| --- | --- |
| `Model\AccountUserRelation`, `AccountUserRelationInterface` | `Model\AccountMembership`, `AccountMembershipInterface` |
| `Manager\RelationManager`, `RelationManagerInterface` | `Manager\AccountMembershipManager`, `AccountMembershipManagerInterface` |
| `Entity\AccountManyUserRelationsTrait` | `Entity\AccountMembershipsTrait` |
| `Entity\UserManyAccountRelationsTrait` | `Entity\UserAccountMembershipsTrait` |
| `Entity\AccountMultiAccountedTrait`, `UserMultiAccountedTrait` | `Entity\AccountMembershipsTrait`, `UserAccountMembershipsTrait` |
| `Model\AccountManyUserRelationsInterface` | `Model\AccountMembershipsInterface` |
| `Model\UserManyAccountRelationsInterface` | `Model\UserAccountMembershipsInterface` |
| `Entity\AccountTrait` | `Entity\AccountOwnedTrait` |
| `Model\AccountRelatedInterface` | `Model\AccountOwnedInterface` |
| `Model\AccountFilterInterface` | `Model\AccountScopedInterface` |

The `CompleteSingleAccounted*`, `CompleteMultiAccounted*`, and `CompleteAccountUserRelation` convenience entities are gone, as are the old `SingleAccounted*`/`MultiAccounted*` contracts. Recompose your application's concrete entities from the supported models, capability interfaces, and traits. A mechanical rename of a complete user base class can lose password, role, confirmation, or profile behavior.

See the [current entity examples](bundles/account-bundle/model-and-entities.md). Preserve your application's identifiers and business fields instead of recreating its tables from those examples.

### Keep the existing membership data {#keep-existing-membership-data}

You can keep the application's entity name and table name while replacing its parent model. For example, an existing `App\Entity\AccountUser` can extend `Model\AccountMembership` rather than renaming its database table or losing its data.

The bundled membership mapping still uses the `account_user` table and the account/user associations as a **composite identifier**. Keep an explicit application table mapping where necessary. Do not introduce a new generated ID simply because the class is now called a membership.

`CompleteAccountUserRelation` previously supplied role and grant metadata. If you used that class, keep those fields in your application entity. `AccountMembershipInterface` requires `getRoles(): array` and `setRoles(array $roles): void`; the base `AccountMembership` only supplies the account and user relations.

Replace the configuration key while keeping your existing concrete class. This is an override of the application's Account Bundle configuration:

```diff
 sfs_account:
-    relation_class: App\Entity\AccountUser
+    membership_class: App\Entity\AccountUser
```

The corresponding container parameter changes from `sfs_account.relation.class` to `sfs_account.membership.class`.

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_account.yaml
sfs_account:
    class: App\Entity\Account
    membership_class: App\Entity\AccountUser
```

```php
<?php

// config/packages/sfs_account.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_account', [
        'class' => 'App\Entity\Account',
        'membership_class' => 'App\Entity\AccountUser',
    ]);
};
```

<!-- config-tabs:end -->

### Update collections and queries {#update-membership-collections}

| Old access | New access |
| --- | --- |
| Account `getRelations()`, `addRelation()`, `removeRelation()` | `getMemberships()`, `addMembership()`, `removeMembership()` |
| Account property `userRelations` | `memberships` |
| User `getRelations()`, `addRelation()`, `removeRelation()` | `getAccountMemberships()`, `addAccountMembership()`, `removeAccountMembership()` |
| User property `accountRelations` | `accountMemberships` |
| Account settings Twig variable `relations` | `memberships` |

Update Doctrine `mappedBy`/`inversedBy`, DQL joins, form property paths, serializers, and Twig overrides that use the old names. The current traits expose `getUsers()` and `getAccounts()` as projections of memberships; a projected user is not the membership that contains its roles.

Initialize `$memberships` and `$accountMemberships` with `ArrayCollection` in the appropriate constructors. Keep a call to your user's existing parent constructor and preserve initialization of its other capabilities. The new traits do not initialize these collections for you.

Account-owned entities implement `AccountOwnedInterface`; add `AccountScopedInterface` when they should participate in automatic account filtering. Keep the configured account route parameter, `_account` by default. Custom listeners that used separate route lookup arguments should use `AccountContextResolverInterface` for the shared account resolution behavior.

Before deployment, verify membership counts, account-specific roles, account switching, filtered reads, and writes to account-owned entities against a copy of the existing database. Do not turn an unresolved account scope into unfiltered access.

## User contracts and authentication {#user-contracts}

### Use the supported identity and password capabilities {#identity-and-password-capabilities}

The base `Model\User` no longer supplies the deprecated `getUsername()`, `getPassword()`, `getSalt()`, or `eraseCredentials()` compatibility methods.

- For a login identifier, call `getUserIdentifier()`. An entity that explicitly uses a username capability can still have its own `getUsername()`; do not rename a business username field just because the base compatibility method disappeared.
- For local passwords, implement `UserPasswordInterface` and use the matching entity/model password trait. That capability supplies `getPassword()` and `eraseCredentials()`.
- A passwordless user should not acquire a password capability only to satisfy old code that always called `getPassword()`.
- Replace the removed `Model\UserRolesInterface` alias with the current role capability, normally `RolesInterface` for the old alias's contract. Keep the trait appropriate to the application's role model.

The password trait no longer provides `getSalt()`, although its mapped salt field remains. If an existing password scheme needs a separate salt, preserve the data and implement the legacy password contract needed by your chosen Symfony hasher. Test login with an existing hash before changing or removing that storage. Follow Symfony's [password migration guide](https://symfony.com/doc/6.4/security/passwords.html#password-migration); upgrading the package does not rehash every stored password automatically.

Custom OAuth providers must use `loadUserByIdentifier()`, not the removed `loadUserByUsername()` compatibility method, and keep the typed `supportsClass(string $class): bool`, `connect(...): void`, and `disconnect(...): void` contracts.

Rebase constructor overrides of `LoginController`: its configuration arguments now include `$googleIdentityPlatformConfig` before the event dispatcher. Applications that do not enable this optional feature still need a correct constructor/service definition. Preserve the installed bundle's default configuration instead of enabling the feature as an upgrade workaround.

### Replace deprecated authorization names {#authorization-names}

Deprecated permission voters and old role hierarchy aliases have been removed from User, Account, and Media bundles. Review application `isGranted()`/`is_granted()` calls, access rules, and stored roles.

| Old identifier | Current identifier |
| --- | --- |
| `ROLE_ADMIN_USERS_RO`, `ROLE_ADMIN_USERS_RW` | `ROLE_SFS_USER_ADMIN_USERS_RO`, `ROLE_SFS_USER_ADMIN_USERS_RW` |
| `ROLE_ADMIN_ADMINISTRATORS_RO`, `ROLE_ADMIN_ADMINISTRATORS_RW` | `ROLE_SFS_USER_ADMIN_ADMINISTRATORS_RO`, `ROLE_SFS_USER_ADMIN_ADMINISTRATORS_RW` |
| `ROLE_ADMIN_INVITATION_RO` | `ROLE_SFS_USER_ADMIN_INVITATION_RO` |
| `ROLE_ADMIN_USERS_{LIST,DETAILS,DELETE,UPDATE,PROMOTE}` | `PERMISSION_SFS_USER_ADMIN_USERS_{LIST,DETAILS,DELETE,UPDATE,PROMOTE}` |
| `ROLE_ADMIN_ADMINISTRATORS_{LIST,DETAILS,DEMOTE,UPDATE,DELETE,INVITE}` | `PERMISSION_SFS_USER_ADMIN_ADMINISTRATORS_{LIST,DETAILS,DEMOTE,UPDATE,DELETE,INVITE}` |
| `ROLE_ADMIN_INVITATIONS_{LIST,DETAILS}` | `PERMISSION_SFS_USER_ADMIN_INVITATIONS_{LIST,DETAILS}` |
| `ROLE_ADMIN_ACCESS_HISTORY_LIST` | `PERMISSION_SFS_USER_ADMIN_HISTORY_LIST` |
| `ROLE_ADMIN_ACCOUNTS_RO`, `ROLE_ADMIN_ACCOUNTS_RW` | `ROLE_SFS_ACCOUNT_ADMIN_ACCOUNTS_RO`, `ROLE_SFS_ACCOUNT_ADMIN_ACCOUNTS_RW` |
| `ROLE_ADMIN_ACCOUNTS_{LIST,DETAILS,UPDATE,DELETE}` | `PERMISSION_SFS_ACCOUNT_ADMIN_ACCOUNTS_{LIST,DETAILS,UPDATE,DELETE}` |
| `ROLE_SFS_MEDIA_ADMIN_MEDIAS_{LIST,DETAILS,CREATE,DELETE,UPDATE}` | `PERMISSION_SFS_MEDIA_ADMIN_MEDIAS_{LIST,DETAILS,CREATE,DELETE,UPDATE}` |

Braces abbreviate separate identifiers; they are not literal permission strings. Keep group roles and action permissions distinct. Do not apply a global `ROLE_` to `PERMISSION_` replacement.

Preserve application-specific roles and produce a reviewed data migration for stored legacy roles. Check effective access for a normal user and an administrator; successfully logging in does not prove the old permissions still grant the intended actions.

## CMS Data Plugin {#cms-data-plugin}

CMS import, export, and fixture support moved out of `cms-bundle` into **`softspring/cms-data-plugin`**. This is a maintained `6.0` feature, separate from experimental plugins.

If your application uses those features, require the plugin on the same line:

```bash
composer require 'softspring/cms-data-plugin:6.0.*' --with-all-dependencies
```

Verify that `Softspring\CmsDataPlugin\SfsCmsDataPlugin` is enabled in `config/bundles.php`. Installation of the Composer package alone is not proof that its configuration extensions and routes are active.

Update application references as follows:

| Old reference | Current reference |
| --- | --- |
| `Softspring\CmsBundle\Data\…` | `Softspring\CmsDataPlugin\Data\…` |
| `Softspring\CmsBundle\DataFixtures\…` | `Softspring\CmsDataPlugin\DataFixtures\…` |
| Core `ContentImportForm` and `VersionImportForm` classes/interfaces | The corresponding classes under `Softspring\CmsDataPlugin\Form\Admin\…` |
| Core content/version import and export listeners | `Softspring\CmsDataPlugin\Admin\ActionListener\…` |
| `SfsCmsEvents::ADMIN_CONTENTS_IMPORT_*` | `SfsCmsDataPlugin::ADMIN_CONTENTS_IMPORT_*` |
| `SfsCmsEvents::ADMIN_CONTENT_VERSIONS_IMPORT_*` / `EXPORT_*` | The same constant names on `SfsCmsDataPlugin` |
| Import template overrides under `@SfsCms/admin/content/…` | The corresponding `@SfsCmsDataPlugin/admin/content/…` templates |

For example, change custom transformer imports:

```diff
- use Softspring\CmsBundle\Data\ReferencesRepository;
- use Softspring\CmsBundle\Data\EntityTransformer\ContentEntityTransformer;
+ use Softspring\CmsDataPlugin\Data\ReferencesRepository;
+ use Softspring\CmsDataPlugin\Data\EntityTransformer\ContentEntityTransformer;
```

The published import/export event string values are retained. Code using the old constant owner must still be changed because those constants are no longer declared on `SfsCmsEvents`.

The `content.admin.import`, `version_import`, and `export_version` configuration nodes are supplied by the plugin. Without it, old content definitions containing those nodes are not a complete core-only configuration. Move fixture subclasses, purger references, and service decorations too. See [CMS Data Plugin](bundles/cms-data-plugin.md).

## CMS configuration and rendering {#cms-configuration-and-rendering}

### Replace flattened admin options {#nested-admin-options}

The legacy normalization of flattened content admin options is removed. In native `cms/contents/<type>/config.yaml` definitions, use nested action configuration.

This is an **override fragment** of an existing content definition, not a complete content type:

```diff
 content:
     admin:
-        create_view: '@content/page/admin/create.html.twig'
-        update_type: App\Form\PageUpdateForm
+        create:
+            view: '@content/page/admin/create.html.twig'
+        update:
+            type: App\Form\PageUpdateForm
```

Apply the same nesting to the other old names:

| Under `content.admin` | Current action node |
| --- | --- |
| `list_*`, `create_*`, `read_*`, `preview_*`, `update_*`, `delete_*` | The corresponding `list.*`, `create.*`, `read.*`, `preview.*`, `update.*`, `delete.*` option supported by that action. |
| `versions_is_granted`, `versions_view` | `version_list.is_granted`, `version_list.view` |
| `cleanup_versions_is_granted` | `version_cleanup.is_granted` |
| `keep_version_is_granted` | `version_lock.is_granted` |
| `seo_*` | `version_seo.*` |
| `content_*` | `version_create.*` |
| `import_*` | `import.*`, provided by CMS Data Plugin |
| `version_import_*` | `version_import.*`, provided by CMS Data Plugin |
| `export_version_is_granted` | `export_version.is_granted`, provided by CMS Data Plugin |
| `publish_version_is_granted`, `unpublish_is_granted` | `publish_version.is_granted`, `unpublish.is_granted` |

Do not preserve the old `version_import_*` normalization's intermediate `import.version_*` spelling: the plugin's current node is `admin.version_import`.

### Remove old cache and sitemap options {#cache-and-sitemap-options}

Remove `sfs_cms.content.cache_last_modified`. Configure the current global `sfs_cms.cache` block and the supported content cache options instead. This example selects last-modified caching; keep the strategy appropriate to your application:

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_cms.yaml
sfs_cms:
    cache:
        enabled: true
        type: last_modified
```

```php
<?php

// config/packages/sfs_cms.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_cms', [
        'cache' => ['enabled' => true, 'type' => 'last_modified'],
    ]);
};
```

<!-- config-tabs:end -->

In native site definitions, replace `site.sitemaps.<name>.alternates` with the supported `alternates_locales` and `alternates_sites` options. To preserve the behavior of an old `false` value, set both to `false`.

Enabled sites need a host or path. Review collisions between CMS route paths and configured sitemap, sitemap-index, or robots endpoints; the current site configuration reserves these endpoints. The old `sfs_cms.site.identification` setting is still accepted with a deprecation and should not be used as the basis of new site-resolution code. See [site configuration](configuration/configure-sites.md).

### Use native argument resolvers {#native-argument-resolvers}

The Sensio ParamConverter classes shipped by CMS, Sections, and Account bundles have been removed. Remove explicit references to those converters and adapt controller arguments to the current resolvers.

The CMS content resolver matches the exact `ContentInterface` argument type and reads the request attribute with the argument's name. For example, `{content}` matches `$content`:

```php
<?php

namespace App\Controller;

use Softspring\CmsBundle\Model\ContentInterface;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Routing\Attribute\Route;

final class ContentSummaryController
{
    #[Route('/content/{content}/summary', name: 'content_summary')]
    public function __invoke(ContentInterface $content): JsonResponse
    {
        return new JsonResponse(['name' => $content->getName()]);
    }
}
```

The content-version and section resolvers use their own corresponding interfaces. The account resolver delegates to the account context and its configured route parameter. Do not assume that every removed converter has a matching native replacement: old block/menu converter usage needs explicit application lookup or a custom argument resolver. A concrete application entity type is not automatically the same match as the bundle interface.

### Replace deprecated render helpers {#render-helpers}

| Removed reference | Adjustment |
| --- | --- |
| `Softspring\CmsBundle\Form\Type\MediaModalType` | Use `Softspring\CmsBundle\Form\Type\MediaType`. The Media Bundle's own `Form\MediaModalType` still exists. |
| `Form\Admin\ContentVersion\ContentContentForm` / `ContentContentFormInterface` in CMS | Use `VersionCreateForm` / `VersionCreateFormInterface` in the same namespace. |
| `Softspring\CmsBundle\Form\DynamicFormTrait` | Use the current dynamic form types/extensions, as described below. |
| `Softspring\CmsBundle\Utils\ModuleMigrator` | Use `Softspring\CmsBundle\Utils\DataMigrator`. |
| `Softspring\CmsBundle\Render\RenderErrorException` | Use `Softspring\CmsBundle\Render\Error\RenderErrorException`. |
| Twig `sfs_cms_module()` and `ContentVersionRenderer::renderModuleById()` | Move the call to the supported content/module rendering flow; do not keep a service registration for the removed extension. |

For custom standalone module rendering, `Render\Module\ModuleRenderer::render()` takes module **data**, including its `_module` identifier, together with the profiler collection and optional Twig context. It is not a drop-in Twig function with the old `(moduleId, data)` signature. See [CMS modules](bundles/cms-bundle/modules.md) before replacing that flow.

The deprecated content-level `getSeo()`/`setSeo()` wrappers are removed. Use `setIndexing()` for content indexing settings, and read version SEO from the relevant version. To reproduce the previous **published content** merge, including its null behavior:

```php
$indexing = $content->getIndexing();
$seo = null === $indexing
    ? null
    : $indexing + ($content->getPublishedVersion()?->getSeo() ?? []);
```

For a preview, use the version being previewed rather than silently reading the published version. Do not confuse this existing indexing/version metadata with an optional experimental SEO plugin.

If you extend the CMS URL generator, rebase its protected methods: `getRoutePath()` no longer takes a site argument, while `getSiteSchemeAndHost()` and `getSiteOrLocalePath()` receive a resolved `?SiteInterface`. The old `getEnabledRouteSite()` and `getSiteForRoute()` hooks are gone. Public URL helpers still accept their documented route, locale, and site inputs.

## Forms, events, and collections {#forms-events-and-collections}

### Replace removed form aliases {#form-aliases}

| Removed name | Current name or approach |
| --- | --- |
| `Softspring\Component\DynamicFormType\Form\DynamicFormType` | `Softspring\Component\DynamicFormType\Form\Type\DynamicFormType` |
| `…\Form\DynamicFormCollectionType` | `…\Form\Type\DynamicFormCollectionType` |
| `…\Form\DynamicFormTypeInterface` | `…\Form\Type\DynamicFormTypeInterface` |
| `…\Form\DynamicFormTrait` | Use `DynamicFormType`/`DynamicFormCollectionType`, or implement the current marker interface and register the package extension. Do not keep calls to the removed trait methods. |
| `Softspring\Component\DoctrinePaginator\Form\PaginatorFiltersForm` | `Softspring\Component\DoctrinePaginator\Form\PaginatorForm` |

Update `use` statements, `getParent()`, form service definitions, `entry_type`, and native CMS definitions containing those class names. The similarly named CMS wrapper types under `Softspring\CmsBundle\Form\Type` remain available.

Dynamic constraint configuration now constructs a constraint with `new $constraintClass(...$options)`. Named option keys must match its constructor parameters. Symfony's built-in constraints support their named options, for example `new Length(min: 3, max: 80)`. A custom constraint that still expects one legacy options array must be adapted, or constructed explicitly and passed as a constraint object to the form.

### Review polymorphic form listener overrides {#polymorphic-form-listeners}

`NodesResizeFormListener` now adds existing subforms on **`POST_SET_DATA`**, through `postSetData(FormEvent $event): void`; the old `preSetData()` hook is removed. Update decorators/subclasses and the subscribed event if you customized this listener. Check how your own form listeners are ordered around the change.

The polymorphic node transformer's `reverseTransformInitialValue()` receives an array. A custom discriminator's `getIdFieldForObject()` declares `: mixed`. Preserve existing node discriminators and test adding, removing, reordering, and submitting a collection with existing entries as well as a new empty collection.

### Pass valid view-event data {#view-event-data}

`ViewEvent` in both Events and Crudl Controller now accepts `array|ArrayObject` data. Replace arbitrary scalar/object payloads with a named view context, for example:

```diff
- new ViewEvent($product, $request)
+ new ViewEvent(['product' => $product], $request)
```

This example uses the **Events component's** constructor. Crudl Controller's `ViewEvent` also has a template argument; retain that argument when adapting its calls. An event's context should match the variables used by its template.

Several user/account controllers also pass their `FormInterface` to the render step instead of creating a `FormView` earlier. If a custom view listener inspects values such as `login_form`, `register_form`, or `settings_form`, check which form object it receives. Symfony's controller rendering converts these forms for Twig; code running before that boundary must not assume the value already has `FormView::vars`.

If you override `DispatchTrait::dispatch()`, allow an `object` event as the current trait does; an override restricted to the previous Symfony base event is narrower than its contract.

## Media configuration and extensions {#media-configuration-and-extensions}

### Select an installed image driver {#image-drivers}

Media Bundle validates the configured image driver and supported codecs when building the container. The default is Imagick; having only `ext-gd` is not enough for an image type that inherits that default.

If your application uses GD for ordinary JPEG/PNG versions, select it explicitly. This is a complete small example with a required image type and thumbnail:

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_media.yaml
sfs_media:
    imagine:
        driver: gd
    types:
        article_image:
            type: image
            upload_requirements:
                mimeTypes: ['image/jpeg', 'image/png']
            versions:
                _thumbnail:
                    type: jpeg
                    scale_width: 300
```

```php
<?php

// config/packages/sfs_media.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_media', [
        'imagine' => ['driver' => 'gd'],
        'types' => [
            'article_image' => [
                'type' => 'image',
                'upload_requirements' => ['mimeTypes' => ['image/jpeg', 'image/png']],
                'versions' => [
                    '_thumbnail' => ['type' => 'jpeg', 'scale_width' => 300],
                ],
            ],
        ],
    ]);
};
```

<!-- config-tabs:end -->

Use a per-type or per-version `imagine_driver` override when only some types need a different encoder. Install and enable Imagick when the configured formats require it. Animation processing can require FFmpeg/ffprobe and the appropriate codecs; changing an extension or MIME label does not preserve animation by itself.

The type/version image options now inherit a generation policy, including `allow_upscale`, `resize_tolerance`, and `preserve_original_encoding`. Review the effective configuration before regenerating old media: the default `allow_upscale: false` can change the dimensions of versions that previously enlarged a small source. Keep a copy of existing originals and generated files when testing a new policy.

### Update service references and custom processors {#media-services-and-processors}

| Old extension | `6.0` adjustment |
| --- | --- |
| `Media\NameGenerators` | Inject `Media\NameGeneratorProvider`. Its iterable contains generator services, indexed by their concrete class. Keep type configuration's `generator` class consistent with that service. |
| `EventListener\Admin\…` | The corresponding media action listener classes are under `Admin\ActionListener\…`. Update service decorations and subclasses. |
| Manually supplied `MediaTypesCollection` arguments | The constructor takes `iterable $typeProviders`; update the old `$providers` named service argument. |
| Old provider/name-generator compiler passes | Use the bundle's current autoconfiguration and tagged services. Do not register the removed passes in an application kernel. |
| Custom image encoders implementing only `ProcessorInterface` | Review `ImageProcessorInterface` and processor selection so an image version is encoded once. Keep non-image processors on their appropriate contract. |
| `Tools\Apng` | The old APNG-specific helper is removed. Review `Tools\AnimationDetector` and the supported animation pipeline rather than treating one format as the only animation case. |

`StoreFileProcessor`, version copying, and migration tooling have additional dependencies/options. Rebase explicit constructors and decorators. The normal Symfony service wiring supplies these dependencies; a copied `5.x` definition may not.

The existing `sfs_media_render_*` Twig helpers remain; changes to their callable registration syntax are not a reason to rename them. See [media types](bundles/media-bundle/media-types.md), [name generators](bundles/media-bundle/name-generators.md), and [storage](bundles/media-bundle/storage-options.md).

## Frontend assets {#frontend-assets}

### Update package roots and import paths {#asset-package-roots}

Several packages now place their npm manifest at the package root. Applications with `file:` dependencies pointing to an old `assets/` package must update those entries to the actual `6.0` manifest location.

For example, in an application's root `package.json`:

```diff
- "@softspring/cms-bundle": "file:vendor/softspring/cms-bundle/assets"
+ "@softspring/cms-bundle": "file:vendor/softspring/cms-bundle"
```

The root-manifest convention applies to CMS, Media, Components, Collection Form Type, CMS Module Collection, Sections, Translation, and CMS Sylius packages in this line. Check each installed manifest: Armonic and Polymorphic Form Type still have their frontend manifest under `assets/`. A global removal of `/assets` from every dependency path is incorrect.

Bundler imports use source files under `assets/`; AssetMapper uses the published output. For example, the shared admin shell can be imported into an application bundler entry with:

```js
// assets/admin.js
import '../vendor/softspring/components/assets/layout/admin-shell.js';
```

```scss
// assets/admin.scss
@import '../vendor/softspring/components/assets/styles/admin.bootstrap5';
@import '../vendor/softspring/components/assets/sidebar/sidebar-pills.bootstrap5';
```

That is the **shell**, not the complete CMS editor. Keep the CMS, media, collection, translation, and plugin entries your forms need. Recheck aliases for imports made from inside installed packages and retain explicit `.js` extensions required by browser modules. Do not point a bundler alias at published code when it expects the package's source layout.

For a Webpack Encore application, these are example aliases for installed Composer sources. Merge them into the application's existing Encore configuration; this is not a complete `webpack.config.js`:

```js
const path = require('path');
const Encore = require('@symfony/webpack-encore');

Encore.addAliases({
    '@softspring/components': path.resolve(__dirname, 'vendor/softspring/components/assets'),
    '@softspring/cms-bundle': path.resolve(__dirname, 'vendor/softspring/cms-bundle/assets'),
    '@softspring/media-bundle': path.resolve(__dirname, 'vendor/softspring/media-bundle/assets'),
    '@softspring/collection-form-type': path.resolve(__dirname, 'vendor/softspring/collection-form-type/assets'),
});
```

After adapting the application's dependencies and imports, install its npm dependencies and run its own asset build. Keep and review the application's lock file. Installing Composer packages does not rebuild its Webpack output.

### AssetMapper consumers {#assetmapper-consumers}

Enabled bundles register their published asset paths when AssetMapper is available. Use the importmap entries declared in the installed package's root manifest, for example:

```js
import '@softspring/components/admin-shell.js';
import '@softspring/media-bundle/scripts/media-type.js';
```

Collection Form Type now also supplies `Softspring\Component\CollectionFormType\SfsCollectionFormTypeBundle`; enable it when using its Symfony/AssetMapper integration and Flex has not already done so. Do not invent bundle classes for asset-only packages.

Keep application entrypoints and styles loaded by your Twig layout. Registering an asset path does not execute its JavaScript. Avoid loading both an old copied bundle script and the new module, which can initialize a widget twice.

Rebase copied admin layouts and form themes against their `6.0` versions. Verify the sidebar, collection controls, media picker, translated fields, editor preview, and save action in a browser. See [CMS frontend assets](bundles/cms-bundle/frontend-assets.md) and [UI Components](components/components.md).

## Existing database upgrades {#existing-database-upgrades}

### Preserve the migration history {#migration-history}

Keep the application's existing migration versions table. Do not rerun historical package migrations by dropping that history, and do not replace an upgrade with `doctrine:schema:update --force`.

The packages include platform-specific MySQL and PostgreSQL paths. An upgrade should stay on the application's existing database engine; adding PostgreSQL support does not convert a MySQL database and its data automatically. See [database configuration](getting-started/configure-database.md) for driver and server-version settings.

Historical `5.5`/`5.6` package migrations contain MySQL-specific SQL. If an existing PostgreSQL project used locally adapted migrations, review that application's recorded history and adaptations explicitly; do not replay the old MySQL SQL against PostgreSQL.

After adapting entities and configuration, inspect the migration plan against a copy of the existing database:

```bash
php bin/console doctrine:migrations:status
php bin/console doctrine:migrations:migrate --dry-run
```

Review the generated SQL and any application migrations for custom tables before running the actual upgrade:

```bash
php bin/console doctrine:migrations:migrate --no-interaction
```

### Include every installed plugin {#plugin-migrations}

Check that the installed packages' migration namespaces are registered. These are important `6.0` additions relative to the legacy branches; execute only migrations not already recorded for the application:

| Package migration | Purpose |
| --- | --- |
| CMS `Version20260506104327` | Normalize CMS identifiers and related column definitions for the supported database platforms. |
| CMS `Version20260617120000` | Add site metadata storage. |
| Media `Version20260617123000` | Add media metadata storage. |
| Blog `Version20260525090000` | Add article tags. |
| Blog `Version20260922110000` | Normalize the article content identifier. |
| Sections `Version20261109100000` | Normalize the plugin-owned content-version reference and restore its cascading foreign key. |

Apply core CMS migrations before the dependent Sections correction. That reference-column conversion belongs to **Sections**, not to an application copy of the core migration. If planning a rollback, reverse the dependency order. Test the complete upgrade/rollback plan for the application's actual data and database; do not assume every DDL operation is transactional on MySQL.

### Review custom references and mappings {#custom-database-references}

Core identifiers are normalized from fixed-width definitions to string columns such as `VARCHAR(36)`. This does not mean generating new identifiers for existing content. Keep the values and relationships intact.

Review application foreign keys into CMS tables, custom content subclasses, target-entity mappings, character sets/collations, and column lengths. The core migration preserves external MySQL foreign keys generically, but it cannot redesign application-owned columns or business constraints for you. Keep those changes in reviewed application migrations.

For Account Bundle, verify that renaming the mapped model/property references preserves the existing membership table, composite key, and role/grant fields. An automatically generated migration proposing to drop the relation table is a reason to review the mapping, not an instruction to delete membership data.

After applying the plan, inspect the schema and representative records. Reopen existing content, publish a version, follow its site/locale route, load related media, and check account-scoped access. Recompile affected CMS content when template or frontend URLs embedded in compiled HTML have changed. See [content versions and publication](bundles/cms-bundle/content-versions.md).

## Tracing instrumentation {#tracing-instrumentation}

Google Cloud Trace now exposes explicit instrumentation switches. Kernel instrumentation is enabled by default; event-dispatcher, Twig, Doctrine, and HTTP-cache instrumentation are opt-in. An application expecting all of its old spans must choose the integrations it actually uses.

For example, enable Doctrine and Twig while keeping SQL text out of spans:

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_google_cloud_trace.yaml
sfs_google_cloud_trace:
    instrumentation:
        doctrine: true
        twig: true
    doctrine:
        include_sql: false
```

```php
<?php

// config/packages/sfs_google_cloud_trace.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_google_cloud_trace', [
        'instrumentation' => ['doctrine' => true, 'twig' => true],
        'doctrine' => ['include_sql' => false],
    ]);
};
```

<!-- config-tabs:end -->

With newer DBAL versions, let the bundle register its driver middleware through the configuration. The current `ConnectionTracerMiddleware` is no longer a connection subclass exposing the old `query()`/`exec()`/transaction methods. Update custom decorators and copied service definitions instead of treating it as a DBAL connection.

## Remaining package checks {#remaining-package-checks}

These checks complete the package review without assuming every implementation change requires application edits:

| Package(s) | What to check in an existing application |
| --- | --- |
| `cms-blog-plugin` | Data transformer imports now use CMS Data Plugin; apply the article/tag migrations and preserve custom article fields. |
| `cms-sections-plugin` | Native resolvers, attribute mappings, frontend imports, and the plugin's reference-column migration. |
| `cms-translation-plugin` | Attribute mappings, current Data Plugin integration, and matching frontend package paths. |
| `cms-module-collection` | Rebuild the application's module imports and verify existing module configurations against the selected package version. |
| `cms-sylius-bundle` | Rebase copied Twig/form integrations, load current translations/assets, and check the actual Sylius admin version. A successful Composer solve alone does not prove its editor templates are compatible. |
| `doctrine-templates` | Attribute driver and lifecycle callback return types; preserve existing identifiers and columns. |
| `doctrine-changelog-bundle` | Attribute markers, `static` factory return types in overrides, and the selected Doctrine/annotations dependencies. |
| `events`, `crudl-controller`, `doctrine-paginator` | Typed view contexts, overrides, collection return types, and removal of the deprecated paginator form alias. |
| `dynamic-form-type`, `polymorphic-form-type` | Canonical class names, constraint construction, and form listener timing. |
| `components`, `collection-form-type` | Current package manifests, published/source asset distinction, bundle registration where applicable, and browser initialization. |
| `google-cloud-trace-bundle` | Explicit instrumentation and DBAL middleware wiring. |
| `google-cloud-integration-bundle` | Compatible Cloud SDK dependencies and application logging/error-reporting configuration. |
| `translatable-bundle` | Existing custom DBAL type overrides must accept `mixed` in `convertToPHPValue()` and retain the translation-object contract; also review its frontend imports. |
| `twig-extra-bundle` | Typed magic methods in custom `ExtensibleAppVariable` subclasses. |
| `command-controller`, `mailer-bundle` | Typed overrides, custom template collections, and explicit command names when overriding command service registrations. |
| `crudl-bundle`, `doctrine-query-filters`, `doctrine-target-entity-resolver`, `doctrine-migrations-version-comparator`, `filter-attack-vectors`, `http-cache-store-bundle`, `mime-translatable`, `permissions-bundle`, `time-ago-bundle` | The reviewed differences do not introduce a separate general-purpose application API migration beyond the shared dependency/mapping requirements. Still review custom subclasses and external dependency upgrades used by your application. |

## Before deploying {#before-deploying}

1. Confirm that the CLI, web runtime, workers, and installed PHP extensions satisfy the resolved package requirements.
2. Search application code and configuration for the removed classes, old account relations, flattened CMS options, and deprecated authorization names listed above.
3. Build the application container and production cache, not only the dependency set. Run the application's own tests and rebuild its frontend assets.
4. Review and execute the database plan against representative existing data, preserving migration history, identifiers, memberships, and foreign keys.
5. Exercise existing user login/password flows, account access boundaries, CMS editing/publication, media upload/rendering, and the routes used by the application.
6. Deploy the application code, dependencies, generated assets, and reviewed migrations as one coordinated version. Refresh compiled content where its templates or asset URLs changed.

Keep application-specific integration findings with the application's upgrade work. This guide lists package-level changes; it does not replace checking the project's own extensions and stored data.
