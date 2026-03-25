---
title: "Install Armonic in an Existing Symfony Project"
description: "Quick start for adding Armonic to an existing Symfony application instead of starting from a fresh project."
---

# Install Armonic in an existing Symfony project

>[!IMPORTANT]
> This guide assumes your Symfony project already works and has a configured database connection.
> Before running migrations on a production database, make a backup.

## 1. Prepare your project {#prepare-project}

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
composer require softspring/armonic:6.0.x-dev -W
```

>[!NOTE]
> You can optionally install Softspring packages from source:
> ```bash
> composer config 'preferred-install.softspring/*' source
> ```

## 3. Troubleshooting after install {#troubleshooting-after-install}

>[!NOTE]
> If you get an error like:
> `Cannot autowire service "...DynamicTypesExtension"... TypeResolverInterface ... no such service exists`
> add this service definition to `config/services.yaml`:
> ```yaml
> services:
>     Softspring\Component\DynamicFormType\Form\Resolver\TypeResolverInterface:
>         class: Softspring\Component\DynamicFormType\Form\Resolver\ChainTypeResolver
>         arguments:
>             $resolvers:
>                 - '@Softspring\CmsBundle\Form\Resolver\AppTypeResolver'
>                 - '@Softspring\CmsBundle\Form\Resolver\CmsTypeResolver'
> ```
> and then run:
> ```bash
> bin/console cache:clear
> composer dump-autoload
> ```

>[!NOTE]
> If `bin/console doctrine:migrations:diff --namespace="DoctrineMigrations"` fails with
> `Unknown column type "sfs_translation" requested`, check that these bundles are enabled in `config/bundles.php`:
> ```php
> Softspring\TranslatableBundle\SfsTranslatableBundle::class => ['all' => true],
> Softspring\Component\DynamicFormType\SfsDynamicFormTypeBundle::class => ['all' => true],
> ```
> and ensure the DBAL type is registered in `config/packages/doctrine.yaml`:
> ```yaml
> doctrine:
>     dbal:
>         types:
>             sfs_translation: Softspring\TranslatableBundle\Doctrine\Type\TranslationType
> ```
> then run:
> ```bash
> bin/console cache:clear
> ```

## 4. Run migrations {#run-migrations}

Run Armonic migrations:

```bash
bin/console doctrine:migrations:sync-metadata-storage -n
bin/console doctrine:migrations:migrate -n
```

>[!NOTE]
> If this is a fresh integration environment and migrations are inconsistent, you can reset the database:
> ```bash
> bin/console doctrine:database:drop --if-exists --force
> bin/console doctrine:database:create
> bin/console doctrine:migrations:sync-metadata-storage -n
> bin/console doctrine:migrations:migrate -n
> ```
> Do not do this on environments with data you need to preserve.

## 5. Next steps {#next-steps}

Once installed:

1. Configure security for `/admin` access.
2. Create an admin user.
3. Open `/admin/cms/pages/` and start creating content.
