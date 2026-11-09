---
title: "Install Armonic in an Existing Symfony Project"
description: "Quick start for adding Armonic to an existing Symfony application instead of starting from a fresh project."
next: "configuration"
related:
    - "configure-database"
    - "bundles/user-bundle/install"
    - "build-a-small-website"
---

# Install Armonic in an existing Symfony project {#install-armonic-in-an-existing-symfony-project}

>[!IMPORTANT]
> This guide assumes your Symfony project already works and has a configured database connection.
> Before running migrations on a production database, make a backup.

## 1. Prepare your project {#prepare-project}

Keep the database engine already used by your application. Armonic can connect through Doctrine to MySQL or PostgreSQL. Check the matching PHP driver and `DATABASE_URL` in [Configure MySQL or PostgreSQL](../getting-started/configure-database.md).

For example, choose one connection in `.env.local`, adjusting the database and credentials to your project:

**MySQL:**

```dotenv
DATABASE_URL="mysql://app:app_password@127.0.0.1:33061/app?serverVersion=8.4&charset=utf8mb4"
```

**PostgreSQL:**

```dotenv
DATABASE_URL="postgresql://app:app_password@127.0.0.1:54321/app?serverVersion=17&charset=utf8"
```

Changing the URL does not migrate existing application data between engines. Keep your existing migrations and verify their platform compatibility.

Move to your existing project root:

```bash
cd /path/to/your-symfony-project
```

If your project does not use Symfony Flex endpoints for Softspring recipes, configure them:

```bash
composer config --json extra.symfony.endpoint '["https://api.github.com/repos/softspring/recipes/contents/index.json",  "flex://defaults"]'
```

Because Armonic 6 currently uses development branches, configure Composer stability:

```bash
composer config minimum-stability dev
composer config prefer-stable true
```

## 2. Install Armonic {#install-armonic}

Install Armonic:

```bash
composer require softspring/armonic:6.1.*@dev -W
```

>[!NOTE]
> You can optionally install Softspring packages from source:
> ```bash
> composer config 'preferred-install.softspring/*' source
> ```

## 3. Troubleshooting after install {#troubleshooting-after-install}

Check that `Softspring\Component\DynamicFormType\SfsDynamicFormTypeBundle` is enabled first: it registers the default resolver services. If custom service wiring still produces an error like:
`Cannot autowire service "...DynamicTypesExtension"... TypeResolverInterface ... no such service exists`
add this service definition to `config/services.yaml`:

<!-- config-tabs:start -->

```yaml
services:
    Softspring\Component\DynamicFormType\Form\Resolver\TypeResolverInterface:
        class: Softspring\Component\DynamicFormType\Form\Resolver\ChainTypeResolver
        arguments:
            $resolvers:
                - '@Softspring\CmsBundle\Form\Resolver\AppTypeResolver'
                - '@Softspring\CmsBundle\Form\Resolver\CmsTypeResolver'
                - '@Softspring\Component\DynamicFormType\Form\Resolver\DefaultTypeResolver'
```

```php
<?php

// config/services.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $services = $container->services();

    $services->set(\Softspring\Component\DynamicFormType\Form\Resolver\TypeResolverInterface::class, \Softspring\Component\DynamicFormType\Form\Resolver\ChainTypeResolver::class)
        ->args([
            '$resolvers' => [
                service(\Softspring\CmsBundle\Form\Resolver\AppTypeResolver::class),
                service(\Softspring\CmsBundle\Form\Resolver\CmsTypeResolver::class),
                service(\Softspring\Component\DynamicFormType\Form\Resolver\DefaultTypeResolver::class),
            ],
        ]);
};
```

<!-- config-tabs:end -->

and then run:
```bash
bin/console cache:clear
composer dump-autoload
```

If `bin/console doctrine:migrations:diff --namespace="DoctrineMigrations"` fails with
`Unknown column type "sfs_translation" requested`, check that these bundles are enabled in `config/bundles.php`:
```php
Softspring\TranslatableBundle\SfsTranslatableBundle::class => ['all' => true],
Softspring\Component\DynamicFormType\SfsDynamicFormTypeBundle::class => ['all' => true],
```
and ensure the DBAL type is registered in `config/packages/doctrine.yaml`:

<!-- config-tabs:start -->

```yaml
doctrine:
    dbal:
        types:
            sfs_translation: Softspring\TranslatableBundle\Doctrine\Type\TranslationType
```

```php
<?php

// config/packages/doctrine.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('doctrine', [
        'dbal' => [
            'types' => ['sfs_translation' => \Softspring\TranslatableBundle\Doctrine\Type\TranslationType::class],
        ],
    ]);
};
```

<!-- config-tabs:end -->

then run:
```bash
bin/console cache:clear
```

## 4. Run migrations {#run-migrations}

Run Armonic migrations:

```bash
bin/console doctrine:migrations:sync-metadata-storage -n
bin/console doctrine:migrations:migrate -n
```

If a migration fails, inspect `doctrine:migrations:status`, the failed SQL, and the active database platform. Test the integration against a copy of the application's schema and retain its migration history. Resetting a database is not a recovery step for an existing application.

## 5. Next steps {#next-steps}

Connect the CMS to the application's existing authentication and admin role hierarchy. If using User Bundle, follow its [installation guide](../bundles/user-bundle/install.md), then generate the application entity migration after the bundle migrations have been applied.

Check that the admin route import exists, the required form themes and assets load, and the user has the CMS permissions. Open `/admin/cms/pages/`, create a draft, and preview it before publishing. Continue with [CMS configuration](../configuration/overview.md) to define the site's content model.
