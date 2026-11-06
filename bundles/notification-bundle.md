---
title: "Notification Bundle"
description: "Create persistent in-application notifications, render translated messages, and track read state."
---

# Notification Bundle {#notification-bundle}

Notification Bundle stores messages addressed to individual users. Use it for an in-application notification list with new and read states, rather than a flash message that disappears after a request. The bundled notifier persists notifications; it does not send email, push messages, or SMS.

## Install and configure {#installation}

```bash
composer require softspring/notification-bundle:6.0.x-dev
```

Enable `Softspring\NotificationBundle\SfsNotificationBundle` in `config/bundles.php` when Flex does not register it. Supply a concrete notification entity implementing `NotificationInterface` and a configured user entity:

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_notification.yaml
sfs_notification:
    notification_class: App\Entity\Notification
    user_class: App\Entity\User
    db_driver: orm
    notify_user_command: true
```

```php
<?php

// config/packages/sfs_notification.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_notification', [
        'notification_class' => \App\Entity\Notification::class,
        'user_class' => \App\Entity\User::class,
        'db_driver' => 'orm',
        'notify_user_command' => true,
    ]);
};
```

<!-- config-tabs:end -->

The abstract `Softspring\NotificationBundle\Model\Notification` implements the message and state methods. A concrete entity must supply its identifier and user association. Review the mapping before creating a migration: the bundled ORM superclass maps `message` with the legacy DBAL `array` type. Applications using DBAL 4 need a compatible mapping, such as JSON, rather than assuming this type is available. `db_driver: custom` disables the bundled ORM mapping so the application can own the complete mapping.

## Create a notification {#create-notification}

Inject `Softspring\NotificationBundle\Notifier\Notifier` into an application service. For a translated message, use:

```php
$notification = $notifier->createNotification($user, [
    'id' => 'project.ready',
    'domain' => 'notification',
    'data' => ['%name%' => 'Documentation demo'],
]);
$entityManager->persist($notification);
$entityManager->flush();
```

```yaml
# translations/notification.en.yaml
project.ready: 'Project %name% is ready.'
```

`createNotification()` returns an unsaved model, which lets your service persist it in the same transaction as the business change. `notifyUser()` persists and flushes immediately. Convenience methods such as `noticeUserRaw()` and `warningUser()` select a message level for you.

## Render the current user's messages {#render-notifications}

The Twig extension exposes `getUserNotifications(limit, onlyUnread)` and `notificationMessage(notification)`:

```twig
{% for notification in getUserNotifications(10, true) %}
    <p>{{ notificationMessage(notification) }}</p>
{% endfor %}
```

The message helper is marked HTML-safe. Keep raw messages and translation markup under application control, and escape user-supplied values before allowing them into HTML notifications. The helper translates using the current translator locale; do not assume the stored `locale` key changes the translator automatically.

## Read state and routes {#read-state}

`markRead()` stores the read timestamp. `markUnread()` clears it. `new` is a separate flag used when the user has seen the notification list.

The bundled route resource is `@SfsNotificationBundle/config/routing/notifications.yaml`. It provides a list and read/unread actions. Its current state-changing routes do not define HTTP method restrictions or CSRF checks. For a public-facing application, provide authenticated POST endpoints with CSRF validation and ownership checks before exposing these actions. Avoid rendering `notificationMarkAsRead()` as a display-only helper: it writes to the database.

## Notify from the console {#console}

With `notify_user_command: true`, the bundle registers:

```bash
php bin/console sfs:notification:notify --email=reader@example.com 'Your export is ready.'
```

The recipient must exist in the configured user repository. This command creates an in-application notification; it does not deliver an email. Verify a stored notification and the receiving user's list separately from any external delivery feature your application adds.
