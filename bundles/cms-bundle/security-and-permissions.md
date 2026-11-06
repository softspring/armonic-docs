---
title: "CMS administration permissions"
description: "Connect CMS roles to application authentication and grant editorial access by feature and action."
related: ["components/permissions-bundle", "bundles/user-bundle/login-and-security"]
---

# Administration permissions {#administration-permissions}

The CMS integrates with the application's Symfony Security setup. It does not create an authentication policy merely by importing admin routes. Protect the admin prefix, then grant the granular permissions needed by each editor.

## Import the role hierarchy {#import-the-role-hierarchy}

<!-- config-tabs:start -->

```yaml
# config/packages/security.yaml
imports:
    - { resource: '@SfsCmsBundle/config/security/admin_role_hierarchy.yaml' }
    - { resource: '@SfsMediaBundle/config/security/admin_role_hierarchy.yaml' }
security:
    role_hierarchy:
        ROLE_EDITOR:
            - ROLE_SFS_CMS_ADMIN_CONTENTS_RW
            - ROLE_SFS_CMS_ADMIN_ROUTES_RW
            - ROLE_SFS_CMS_ADMIN_MENUS_RW
            - ROLE_SFS_CMS_ADMIN_BLOCKS_RW
            - ROLE_SFS_MEDIA_ADMIN_MEDIAS_RW
```

```php
<?php

// config/packages/security.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->import('@SfsCmsBundle/config/security/admin_role_hierarchy.yaml');

    $container->import('@SfsMediaBundle/config/security/admin_role_hierarchy.yaml');

    $container->extension('security', [
        'role_hierarchy' => [
            'ROLE_EDITOR' => [
                'ROLE_SFS_CMS_ADMIN_CONTENTS_RW',
                'ROLE_SFS_CMS_ADMIN_ROUTES_RW',
                'ROLE_SFS_CMS_ADMIN_MENUS_RW',
                'ROLE_SFS_CMS_ADMIN_BLOCKS_RW',
                'ROLE_SFS_MEDIA_ADMIN_MEDIAS_RW',
            ],
        ],
    ]);
};
```

<!-- config-tabs:end -->

Merge this hierarchy with existing security configuration. Assign `ROLE_EDITOR` through the application's user model and adapt the firewall's access rule to admit it to the editorial area. If that rule only permits `ROLE_ADMIN`, the new role alone will not pass it.

This example grants broad write access to those features. Choose narrower read, creator, or publisher roles from the installed hierarchy when an editor must not publish or delete content. Sites and optional plugins have separate roles.

## Distinguish menus from authorization {#distinguish-menus-from-authorization}

A hidden sidebar item is a navigation choice. Controller/action permission checks must still enforce access. Test a restricted user by requesting the route directly, not only by looking at which buttons are visible.

Content definitions can override an action's `is_granted` attribute. Use application voters for record-specific restrictions, such as an editorial team being allowed to edit only its own content. Choose an access-decision strategy that preserves those denials when several voters participate.

## Grant plugin capabilities explicitly {#grant-plugin-capabilities-explicitly}

[Translation](../cms-translation-plugin.md), [sections](sections.md), and [data import/export](../cms-data-plugin.md) add permissions beyond core content editing. Import the matching hierarchy and assign only the intended capabilities.

Public content routing and admin preview also have different purposes. A preview link is not a replacement for publishing, and `noIndex` is not an access-control setting.
