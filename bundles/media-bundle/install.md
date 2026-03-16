---
title: "Install the Media Bundle"
description: "Install the Softspring Media Bundle, review its image and storage requirements, and enable it in Symfony."
---

# Installation

Make sure Composer is installed globally, as explained in the
[installation chapter](https://getcomposer.org/doc/00-intro.md)
of the Composer documentation.

## Requirements {#requirements}

### PHP requirements {#php-requirements}

This bundle uses **Imagine library** to manage images with **GD implementation**.

So you will require to install PHP **gd extension**.

### Storage requirements {#storage-requirements}

At the moment, the only supported storage option is Google Cloud Storage bucket.

See more storage options in chapter [Storage Options](storage-options.md).



## Applications that use Symfony Flex {#applications-that-use-symfony-flex}

Open a command console, enter your project directory and execute:

```bash
$ composer require softspring\media-bundle:^5.4
```

## Applications that don't use Symfony Flex {#applications-that-dont-use-symfony-flex}

### Step 1: Download the Bundle {#step-1-download-the-bundle}

Open a command console, enter your project directory and execute the
following command to download the latest stable version of this bundle:

```console
$ composer require softspring\media-bundle:^5.4
```

### Step 2: Enable the Bundle {#step-2-enable-the-bundle}

Then, enable the bundle by adding it to the list of registered bundles
in the `config/bundles.php` file of your project:

```php
// config/bundles.php

return [
    // ...
    Softspring\MediaBundle\SfsMediaBundle::class => ['all' => true],
];
```
