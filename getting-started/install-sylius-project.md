# Install Armonic in a Sylius project

>[!IMPORTANT]
> This guide assumes you have Composer, PHP, Node.js/Yarn (or npm), and MySQL database.

## Install Sylius project {#install-sylius-project}

If you have an existing Sylius project, you can skip this section and go to the [Install Armonic in the Sylius project](#install-armonic) section.
If you don't have a Sylius project yet, you can create one using the Sylius Standard Edition. 
You can follow the official Sylius installation guide to set up a new Sylius project: https://docs.sylius.com.
These are the basic steps to create a new Sylius project:

## 1. Requirements {#requirements}

Make sure you have the following requirements installed on your system:
https://docs.sylius.com/getting-started-with-sylius/before-you-begin

## 1.Create a new Sylius project {#create-new-sylius-project}

We will install Sylius Community Edition. The document of this is in https://docs.sylius.com/the-book/sylius-ce-installation. This is a resume:

```bash
$ composer create-project sylius/sylius-standard ArmonicSyliusProject
$ cd ArmonicSyliusProject
$ npm install
$ npm run build
```

## 2.Configure database {#configure-database}

We will configure MySQL database and, for this, we will create (if it doesn't exist) a .env.local file with the following content.
In this configuration file, we will put:
```
DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/sylius_%kernel.environment%?serverVersion=8.0"
```
Replace db_user, db_password, and other values with your credentials

## 3.Install Sylius {#sylius-install}

````bash
$ php bin/console sylius:install
````
## 4. Install frontend assets {#install-assets}

You can install frontend assets using Yarn or npm. With yarn:
```bash
$ yarn install
$ yarn build
```
With npm:
```bash
$ npm install
$ npm run build
```
## 5. Load fixtures {#load-fixtures}

Sylius comes with a set of fixtures that you can load to populate your database with sample data.

```bash
$ php bin/console sylius:fixtures:load
```

## 6. Start the local development server {#start-server}

Once Sylius is installed and the assets are compiled, you can start a local web server:

```bash
symfony serve
```

Then open your browser at http://127.0.0.1:8000 to view the shop. 
The admin dashboard at http://127.0.0.1:8000/admin is accessible with the following credentials:
- Username: sylius@example.com
- Password: sylius

![sylius-admin.png](.files/sylius-admin.png){.img-fluid}

## Install Armonic in the Sylius project {#install-armonic}

Now that you have a Sylius project up and running, you can install Armonic in it.

## 1. Install packages {#install-packages}

Get the packages with composer. 
```bash
$ composer require softspring/cms-bundle
```