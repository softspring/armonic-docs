---
title: "Notification Bundle"
description: "Store, manage, render, and deliver user notifications in Symfony applications."
---

# Notification Bundle {#notification-bundle}

The Notification Bundle provides a notification model for Symfony applications.

It includes entities, forms, controllers, notifier services, Twig integration, and console commands.

## Installation {#installation}

```bash
composer require softspring/notification-bundle:^6.0
```

## What It Provides {#what-it-provides}

- notification entity and model abstractions
- controllers and forms for notification workflows
- notifier services
- Twig integration to render notifications
- commands to work with stored notifications

## Configuration {#configuration}

Main configuration keys under `sfs_notification`:

- `notification_class`
- `user_class`
- `notify_user_command`
- `db_driver`

The bundle currently supports `orm` and `custom` drivers.

## Runtime Features {#runtime-features}

The package includes:

- a notifier service to create notifications
- a controller for notification-related UI flows
- a form type for notification preferences
- a Twig extension for notification rendering
- a command to notify users from the console

## Typical Usage {#typical-usage}

Use this bundle when your project needs first-class user notifications instead of ad hoc flash messages or custom tables for each feature.
