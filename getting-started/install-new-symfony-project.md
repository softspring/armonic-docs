# Install Symfonic in a new Symfony project

>[!IMPORTANT]
> This guide assumes you have Composer and Symfony CLI installed. 
> If you don't have them installed, you can follow the instructions in the [Requirements](requirements.md) section.

## 1.Create a new symfony project {#create-new-symfony-project}

Symfonic is a Symfony project, so first of all we need to create a new Symfony project. 
You can create a new Symfony project using the Symfony CLI or Composer as shown in the official <a href="https://symfony.com/doc/current/setup.html">Symfony documentation</a>. 

Let's create a new Symfony project using the Symfony CLI:

```bash
$ symfony new cms-new-project --webapp
```

By default, Symfony 7.0 version will be used, but you can specify the version of Symfony to use:

```bash
$ symfony new cms-new-project --version="6.4.*" --webapp
```

* If it's your first time running symfony-cli, it might complain that you lack some needed packages (ie: ext-xml). You can install them with
and try again, but you will need to delete the project folder "cms-new-project" and start over.

After creating the project, you can start the Symfony local web server:

```bash
$ cd cms-new-project
$ symfony server:start -d
```

![welcome-to-symfony7.png](.files/welcome-to-symfony7.png){.img-fluid}

## 2.Configure database {#configure-database}

Symfonic uses Doctrine to manage the database schema. You can use any database supported by Doctrine, but for this example we will use MySQL with Docker.

First of all, you need to modify the composer.yaml file provided by the new Symfony project.

```yaml
# compose.yaml
services:
    ###> doctrine/doctrine-bundle ###
    database:
        image: mysql:${MYSQL_VERSION:-8.3.0}
        environment:
            MYSQL_DATABASE: ${MYSQL_DB:-app}
            # You should definitely change the password in production
            MYSQL_PASSWORD: ${MYSQL_PASSWORD:-!ChangeMe!}
            MYSQL_USER: ${MYSQL_USER:-app}
            MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:-root}
        healthcheck:
            test: [ "CMD", "mysqladmin" ,"ping", "-h", "localhost" ]
            timeout: 20s
            retries: 10
        volumes:
            - database_data:/var/lib/mysqlql/data:rw
    ###< doctrine/doctrine-bundle ###

volumes:
    ###> doctrine/doctrine-bundle ###
    database_data:
    ###< doctrine/doctrine-bundle ###
```

```yaml
# compose.override.yaml
services:
    ###> doctrine/doctrine-bundle ###
    database:
        ports:
            - "${MYSQL_PORT:-33061}:3306"
    ###< doctrine/doctrine-bundle ###
```
## 3.Start the database

Then you can start the database using Docker Compose:

```bash
$ docker-compose up -d
```
>[!NOTE]
> If you restart your computer at some point, you will need to start the database again with `docker-compose up -d`.


You will need to modify the .env file to configure the database connection:

```
# .env
###> doctrine/doctrine-bundle ###
DATABASE_URL="mysql://app:!ChangeMe!@127.0.0.1:33061/app?serverVersion=8.3.0&charset=utf8mb4"
###< doctrine/doctrine-bundle ###
```

## 4.Install symfonic {#install-symfonic}

We will use Symfony Flex to install Symfonic in the new Symfony project. 

>[!INFO]
> By the moment, configure recipes manually
> ```bash
> $ composer config --json extra.symfony.endpoint '["https://api.github.com/repos/softspring/recipes/contents/index.json",  "flex://defaults"]'
> ```

Install **symfonic** package with composer (say Yes or Yes for all packages to install the recipes):

```bash
$ composer require softspring/symfonic:^5.2

 Do you want to execute this recipe?
    [y] Yes
    [n] No
    [a] Yes for all packages, only for the current installation session
    [p] Yes permanently, never ask again for this project
    (defaults to n): a
```

This will currently install Symfonic 5.3, the latest version available.

>[!INFO]
> Also, you can configure the preferred install type for softspring packages as source:
> ```bash
> $ composer config 'preferred-install.softspring/*' source
> ```

After installing the package, you must run the Doctrine migrations to create the database schema:

```bash
$ bin/console doctrine:migrations:migrate -n
```

Now you can see the start page of the Symfonic project:

![welcome-to-symfonic.png](.files/welcome-to-symfonic.png){.img-fluid}

## 5.Configure security {#configure-security}

Before you can create your first page, you need to configure security to access the admin area.

You can use any Symfony <a href="https://symfony.com/doc/current/security.html">security configuration</a> or bundle you want, but for this example we will use the Softspring User Bundle.

### 6.Configure Softspring User Bundle {#configure-sfs-user-bundle}

Install the Softspring User Bundle with composer:

```bash
$ composer require softspring/user-bundle:^5.2
```

We recommend to use the Symfony flex recipes to configure the bundle:

```bash
Do you want to execute this recipe?
    [y] Yes
    [n] No
    [a] Yes for all packages, only for the current installation session
    [p] Yes permanently, never ask again for this project
    (defaults to n): y
```

>[!INFO]
> Also you can do it manually, see the [installation instructions](../bundles/user-bundle/install.md).

Then a new User entity has been created, and routes to login, register, and reset password have been added.

But still some manual steps are needed to configure the security.

Run the following commands to configure the security and routes:

```bash
$ mv config/packages/security.yaml.dist config/packages/security.yaml
$ cat config/routes.yaml.dist >> config/routes.yaml
$ rm config/routes.yaml.dist 
```

Create the database schema with a new Doctrine migration:

```bash
$ bin/console doctrine:migrations:diff --namespace="DoctrineMigrations"
$ bin/console doctrine:migrations:migrate -n
```

Before entering the admin area, you need to create a user and promote it to the admin role.

```bash
$ bin/console sfs:user:create username user@example.com 123456
$ bin/console sfs:user:promote user@example.com
```

Now you can go to admin area and login with the user you just created:

![symfonic-create-page.png](.files/symfonic-create-page.png){.img-fluid}
