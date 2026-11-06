---
title: "Subscription Bundle"
description: "Integrate subscription models, customer workflows, platform adapters, and subscription administration."
related: ["bundles/account-bundle", "bundles/user-bundle"]
---

# Subscription Bundle {#subscription-bundle}

Subscription Bundle provides plan and subscription models, customer-facing subscription screens, administration, and lifecycle events. Use it when a customer's access follows a subscription rather than a one-time CMS permission grant.

It depends on the customer and platform packages as well as the configured payment adapter. This is an application integration, not a complete billing setup supplied by a single bundle.

## Install and register {#install-and-register}

```bash
composer require softspring/subscription-bundle:6.0.x-dev
```

Register the bundle when Flex does not do it automatically:

```php
// config/bundles.php
Softspring\SubscriptionBundle\SfsSubscriptionBundle::class => ['all' => true],
```

## Configure the model contracts {#configure-the-model-contracts}

<!-- config-tabs:start -->

```yaml
# config/packages/sfs_subscription.yaml
sfs_subscription:
    entity_manager: default
    model:
        plan: App\Entity\Plan
        subscription: App\Entity\Subscription
        subscription_transition: App\Entity\SubscriptionTransition
```

```php
<?php

// config/packages/sfs_subscription.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    $container->extension('sfs_subscription', [
        'entity_manager' => 'default',
        'model' => [
            'plan' => \App\Entity\Plan::class,
            'subscription' => \App\Entity\Subscription::class,
            'subscription_transition' => \App\Entity\SubscriptionTransition::class,
        ],
    ]);
};
```

<!-- config-tabs:end -->

These are application entities implementing the corresponding model contracts. The bundle also accepts optional `product`, `subscription_item`, and `invoice` classes. Configure and map a subscription item when using the multi-plan model; do not assume an optional `null` class creates a usable entity automatically.

Choose the single-plan or multi-plan contract before wiring forms and routes. The customer must implement `SubscriptionCustomerInterface` and maintain its subscription relation. Review the installed customer/platform package configuration and generated Doctrine migration together.

## Connect the platform adapter {#connect-the-platform-adapter}

The subscription manager creates or updates local entities and dispatches lifecycle events. Cancellation, synchronization, and external billing effects depend on the application's adapter/listeners. Merely calling a method named `cancel()` does not prove a provider has canceled an external subscription.

For example, a service with `SubscriptionManagerInterface` can subscribe a compatible customer to a configured plan:

```php
$subscription = $subscriptionManager->subscribe($customer, $plan);
```

Use test-mode provider credentials and synthetic customers when validating the integration. Check local persistence, emitted events, and the provider state separately.

## Add customer and administration routes {#add-customer-and-administration-routes}

Route resources include `subscribe.yaml`, `customer_subscription.yaml`, and the admin files for plans, products, and subscriptions under `@SfsSubscriptionBundle/config/routing/`. Mount them beneath application paths with the required authenticated customer context.

Import `@SfsSubscriptionBundle/config/config/admin_role_hierarchy.yaml` for the supplied admin roles and assign them through the host application's hierarchy. Customer ownership checks remain necessary for detail, upgrade, cancellation, and reactivation actions.

## Current implementation limits {#current-implementation-limits}

The current `SubscriptionManager::trial()` body is unfinished, multi-plan `upgrade()` has an unimplemented branch, and `removePlan()` does not itself change an item quantity or remove an item. Do not present those methods as a complete ready-to-use lifecycle without application behavior and tests.

For a supported single-plan path, validate subscribe, upgrade, cancellation, reactivation, synchronization, and transition history against the configured adapter. Add listeners or replace managers for the domain rules that are not supplied by the default implementation.
