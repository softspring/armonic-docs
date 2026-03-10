# Armonic standalone

## Description

[Armonic Standalone](https://github.com/softspring/armonic-standalone) is a great way to dive into the world of [Armonic](https://github.com/softspring/armonic). With a
quick and easy setup, you can have a fully functional local development environment in minutes.

## Installation

**1. Clone this repository:**

```bash
    git clone https://github.com/softspring/armonic-standalone
```

**2. Install dependencies**

1. [Symfony CLI](https://symfony.com/download):

```bash
curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.deb.sh' | sudo -E bash
sudo apt install symfony-cli
```
***macOS:***
```bash
brew install symfony-cli/tap/symfony-cli
```

2. [Docker](https://docs.docker.com/get-docker/):
   Choose one option.

3. PHP

```bash
sudo apt install php-cli 
```
***macOS:***

First you need Xcode Command Line Tools:
```bash
xcode-select --install
```

```bash
brew install php
brew install mysql
```

4. Composer

```bash
sudo apt install composer
```
***macOS:***
```bash
brew install composer
```

5. NPM
   (Node >= 18)

```bash
sudo apt install npm
```

6. Install Composer packages

In the folder where you downloaded armonic-standalone run. In this case, armonic-standalone:

```bash
  cd armonic-standalone
  composer update
```

7. Start the containers

```bash
  docker compose up -d
```

8. Execute migrations

```bash
  php bin/console doctrine:migrations:migrate -n
```


Troubleshooting:
- If you have an error like:
```bash
  An exception occurred in the driver: SQLSTATE[HY000] [1045] Access denied for user 'app'@'localhost' (using password: YES)
```
Create a new file compose.override.yaml in the root of the project with the following content:

```yaml
services:
###> doctrine/doctrine-bundle ###
  database:
    ports:
      - "${MYSQL_PORT:-33061}:3306"
###< doctrine/doctrine-bundle ###
```
Create a new file .env.local with the following content:

```env
   DATABASE_URL="mysql://app:!ChangeMe!@127.0.0.1:33061/app?serverVersion=8.3.0&charset=utf8mb4"
```
And execute again:

```bash
  docker compose up -d
  php bin/console doctrine:migrations:migrate -n
```

9. Install front-end dependencies

```bash
  npm install
  npm run dev
```

10. Start the Symfony server

```bash
  symfony server:ca:install
  symfony server:start -d
```

11. (Optional) If you want an example page, you have to load fixtures:

```bash
  php bin/console doctrine:fixtures:load
```

12. Create a test user. In this case, the user is “admin”, the email is “email@example.com” and the password is “admin”.

```bash
  php bin/console sfs:user:create admin email@example.com admin
  php bin/console sfs:user:promote email@example.com 
```

## Usage

Open your browser and go to https://127.0.0.1:8000/app/en/login (or https://127.0.0.1:8000/app/es/login if you prefer in Spanish).

1. Log in with the email and password you created in the previous step.
   ![login.png](.files/login.png){.img-fluid}

2. You are done! You can now start working with Armonic Standalone at https://127.0.0.1:8000/admin/en/.
   ![dashboard.png](.files/dashboard.png){.img-fluid}

3. If you have executed point 11 of the installation and you wish to see the example page, you can consult the page at https://127.0.0.1:8000/admin/en/cms/pages/ , its name is 'Home'.
   ![example.png](.files/example.png){.img-fluid}
   You can see the page information and edit it in "View" link.
   ![example-edit.png](.files/example-edit.png){.img-fluid}
   In the list of "Last versions" or in the "Versions" section, you can publish the page and you can see it at https://127.0.0.1:8000/en/.
   ![example-page.png](.files/example-page.png){.img-fluid}
    
