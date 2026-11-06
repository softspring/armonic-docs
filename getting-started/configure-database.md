---
title: "Configure MySQL or PostgreSQL"
description: "Choose MySQL or PostgreSQL for Armonic 6.0, start a local database, and configure the matching Doctrine connection."
next: "install-new-symfony-project"
related:
    - "install-existing-symfony-project"
    - "use-armonic-standalone-edition"
    - "install-sylius-project"
---

# Configure MySQL or PostgreSQL {#configure-database}

Choose one database engine for your application. These local examples use **MySQL 8.4** or **PostgreSQL 17**. Use the same engine for development, tests, and production so your application migrations target the correct platform.

| Engine | PHP extension | Local port | Database |
| --- | --- | --- | --- |
| MySQL | `pdo_mysql` | `33061` | `app` |
| PostgreSQL | `pdo_pgsql` | `54321` | `app` |

Install the matching extension for both the PHP CLI and the PHP runtime serving your application. Check the drivers available to the CLI:

```bash
php -r 'print_r(PDO::getAvailableDrivers());'
```

The list must include `mysql` or `pgsql`, depending on your choice. Restart the web runtime after installing a PHP extension.

## Start a local database {#start-a-local-database}

If you already have a database server, skip to [Connect Doctrine](#connect-doctrine) and use its credentials and port.

The examples below are alternatives. Replace the `database` service in your Compose files with the one you choose. Keep unrelated services. Credentials below are for local development.

### MySQL 8.4 {#mysql}

In `compose.yaml`:

```yaml
services:
    database:
        image: mysql:8.4
        environment:
            MYSQL_DATABASE: app
            MYSQL_USER: app
            MYSQL_PASSWORD: app_password
            MYSQL_ROOT_PASSWORD: root_password
        healthcheck:
            test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "--silent"]
            interval: 5s
            timeout: 5s
            retries: 20
        volumes:
            - mysql_database_data:/var/lib/mysql

volumes:
    mysql_database_data:
```

In `compose.override.yaml`, replace the existing database port mapping with:

```yaml
services:
    database:
        ports:
            - "127.0.0.1:33061:3306"
```

### PostgreSQL 17 {#postgresql}

In `compose.yaml`:

```yaml
services:
    database:
        image: postgres:17-alpine
        environment:
            POSTGRES_DB: app
            POSTGRES_USER: app
            POSTGRES_PASSWORD: app_password
        healthcheck:
            test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB"]
            interval: 5s
            timeout: 5s
            retries: 20
        volumes:
            - postgres_database_data:/var/lib/postgresql/data

volumes:
    postgres_database_data:
```

In `compose.override.yaml`, replace the existing database port mapping with:

```yaml
services:
    database:
        ports:
            - "127.0.0.1:54321:5432"
```

This data directory is for PostgreSQL 17. Check the [official image documentation](https://hub.docker.com/_/postgres) before changing the major version; newer images can use a different volume layout.

### Start the chosen service {#start-database}

```bash
docker compose up -d --wait database
docker compose ps database
```

Each engine uses its own named volume. Changing an image or a connection URL does not convert existing data to another engine. Keep existing volumes when you need their data. Both images apply the initial database credentials only when their data directory is empty.

## Connect Doctrine {#connect-doctrine}

Set one `DATABASE_URL` in `.env.local`.

**MySQL:**

```dotenv
DATABASE_URL="mysql://app:app_password@127.0.0.1:33061/app?serverVersion=8.4&charset=utf8mb4"
```

**PostgreSQL:**

```dotenv
DATABASE_URL="postgresql://app:app_password@127.0.0.1:54321/app?serverVersion=17&charset=utf8"
```

These URLs assume PHP runs on the host. If PHP runs in the same Compose network, use `database:3306` for MySQL or `database:5432` for PostgreSQL instead of the host and published port.

For an existing server, adjust the credentials, database name, port, and `serverVersion`. URL-encode special characters in usernames and passwords. See [Symfony's database connection guide](https://symfony.com/doc/8.0/doctrine.html#configuring-the-database).

Keep the connection URL in your existing Doctrine configuration. Either YAML or PHP works with both engines:

<!-- config-tabs:start -->

```yaml
# config/packages/doctrine.yaml
doctrine:
    dbal:
        url: '%env(resolve:DATABASE_URL)%'
```

```php
<?php

// config/packages/doctrine.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('doctrine', [
        'dbal' => ['url' => '%env(resolve:DATABASE_URL)%'],
    ]);
};
```

<!-- config-tabs:end -->

Merge this into your current configuration and keep the ORM mappings and other application settings. When selecting PostgreSQL, remove MySQL-only settings from the DBAL connection: a fixed MySQL `driver`, `charset: utf8mb4`, and `default_table_options.charset` or `default_table_options.collate`. PostgreSQL uses `utf8` in the connection URL above. Also remove any fixed `server_version` that conflicts with the chosen server.

Clear the cache and check the connection before installing the application tables:

```bash
php bin/console cache:clear
php bin/console dbal:run-sql 'SELECT 1'
```

Then return to your installation guide and run its migrations. Migrations containing platform-specific SQL must match the selected engine; check any application and third-party migrations as well as Armonic's.

## Connection problems {#connection-problems}

- **Could not find driver:** check the PHP extension used by the CLI and web runtime.
- **Connection refused:** check the service health, host, and published port. A PHP container uses the Compose service name and internal port.
- **Authentication failed:** compare `.env.local` with the credentials used when the data volume was first initialized.
- **Invalid encoding on PostgreSQL:** remove MySQL's `utf8mb4` connection setting and use `utf8`.
