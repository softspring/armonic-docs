---
title: "Armonic Requirements"
description: "Check the PHP, Symfony, database, and development tools needed for Armonic 6.0."
next: "configure-database"
related:
    - "upgrade-to-6.0"
    - "install-new-symfony-project"
    - "install-existing-symfony-project"
    - "use-armonic-standalone-edition"
    - "install-sylius-project"
---

# Requirements {#requirements}

Armonic runs inside a Symfony application. Before installing it, check that your environment has the following.

## Application requirements {#application-requirements}

| Requirement | For Armonic 6.0 |
| --- | --- |
| PHP | **8.4 or later**, for both the command line and the web server. |
| Symfony | **6.4, 7.4, or 8.x**. The standalone project currently uses Symfony 8.1. |
| Composer | To install Armonic and its PHP dependencies. |
| Database | MySQL or PostgreSQL, configured through Doctrine. The examples use MySQL 8.4 or PostgreSQL 17. |

### PHP extensions {#php-extensions}

The CMS and media packages require `libxml`, `zip`, and `gd`. You also need the PHP driver for your database, `pdo_mysql` for MySQL or `pdo_pgsql` for PostgreSQL.

The standalone project also declares `ctype`, `iconv`, `intl`, and `mysqli` in its Composer requirements. Its `mysqli` requirement still applies when using PostgreSQL; it does not replace `pdo_pgsql`. Composer checks the PHP version and extensions required by your selected packages during installation.

## Local development tools {#local-development-tools}

- **Symfony CLI** is used in the guides to create projects and run the local web server.
- **Docker** is used in the guides to run MySQL or PostgreSQL. You can use a database installed on your machine instead.
- **Node.js and npm or Yarn** are needed if your project builds frontend assets. The standalone project uses Webpack Encore.

See [Configure MySQL or PostgreSQL](../getting-started/configure-database.md) for local services, connection URLs, and Doctrine settings.

## Start installing {#start-installing}

Choose the guide that matches your project:

- [Create a new Symfony project](../getting-started/install-new-symfony-project.md).
- [Add Armonic to an existing Symfony project](../getting-started/install-existing-symfony-project.md).
- [Use the standalone project](../standalone/install-armonic-standalone.md).
