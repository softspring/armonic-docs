---
title: "Doctrine Target Entity Resolver"
description: "Register Doctrine target entity resolution from reusable Symfony bundles that work with interfaces and application entities."
---

# Doctrine Target Entity Resolver {#doctrine-target-entity-resolver}

`softspring/doctrine-target-entity-resolver` is a very small helper for reusable Symfony bundles.

Its job is simple: let a bundle work with interfaces such as `UserInterface` or `AccountInterface`, while the real Doctrine entity class is provided by the application.

This is useful when a bundle wants to stay reusable and does not want to hardcode a concrete entity class.

## When To Use It {#when-to-use-it}

Use this component when your bundle:

- ships interfaces or abstract model contracts
- expects the application to provide the real Doctrine entity class
- needs Doctrine relations pointing to those interfaces
- wants to configure target entity resolution from container parameters

This is the pattern used across several Softspring bundles such as `account-bundle`, `user-bundle`, `mailer-bundle`, and CMS plugins.

## What Problem It Solves {#what-problem-it-solves}

Doctrine relations need a concrete target entity.

Reusable bundles usually should not force the host application to use one specific entity class. Instead, the bundle can:

- define an interface
- let the application choose the implementing entity
- register that mapping during container compilation

This component is the helper that simplifies that last step.

## Main Idea {#main-idea}

The package exposes one base class:

- `AbstractResolveDoctrineTargetEntityPass`

You extend it in your own bundle compiler pass and call `setTargetEntityFromParameter()` for each mapping you need.

## Quick Example {#quick-example}

Example compiler pass:

```php
<?php

namespace App\DependencyInjection\Compiler;

use App\Model\UserInterface;
use Softspring\Component\DoctrineTargetEntityResolver\DependencyInjection\Compiler\AbstractResolveDoctrineTargetEntityPass;
use Symfony\Component\DependencyInjection\ContainerBuilder;

class ResolveDoctrineTargetEntityPass extends AbstractResolveDoctrineTargetEntityPass
{
    protected function getEntityManagerName(ContainerBuilder $container): string
    {
        return 'default';
    }

    public function process(ContainerBuilder $container): void
    {
        $this->setTargetEntityFromParameter(
            'app.user.class',
            UserInterface::class,
            $container,
            true,
        );
    }
}
```

In this example:

- `app.user.class` contains the real entity class
- `UserInterface::class` is the interface used by the bundle
- the compiler pass registers the target entity mapping in Doctrine

## Configure The Entity Class Through Parameters {#configure-the-entity-class-through-parameters}

The common pattern is:

1. expose a bundle parameter such as `sfs_user.user.class`
2. let the application set its concrete class there
3. resolve the interface to that class during compilation

That keeps the public integration point very simple for the application.

## Required And Optional Mappings {#required-and-optional-mappings}

`setTargetEntityFromParameter()` supports both required and optional mappings.

Use a required mapping when the bundle cannot work without that entity:

```php
$this->setTargetEntityFromParameter('app.user.class', UserInterface::class, $container, true);
```

Use an optional mapping when that feature is only enabled in some applications:

```php
$this->setTargetEntityFromParameter('app.history.class', HistoryInterface::class, $container, false);
```

If the parameter is missing and the mapping is optional, the helper skips it.

If the parameter is missing and the mapping is required, compilation fails early.

## What The Helper Validates {#what-the-helper-validates}

Before registering the mapping, the helper checks that the configured class implements the expected interface.

That is important because it turns a broken configuration into an explicit container compilation error instead of a more confusing Doctrine failure later.

## Typical Usage In A Bundle {#typical-usage-in-a-bundle}

This component usually sits next to:

- a bundle configuration parameter holding the entity class
- Doctrine mappings in the bundle that reference interfaces
- a compiler pass registered in the bundle class

That means the full integration flow is usually:

1. bundle defines model interfaces
2. application chooses concrete entities
3. bundle stores those classes in parameters
4. compiler pass resolves target entities
5. Doctrine works with the concrete classes

## Real Softspring Usage Pattern {#real-softspring-usage-pattern}

In Softspring bundles, the pattern is usually this:

- `account-bundle` maps interfaces such as account or relation contracts to application entities
- `user-bundle` does the same for users, invitations, and access history
- `mailer-bundle` uses it for optional email history entities
- CMS plugins use it when plugin model contracts are implemented by the project

That is the main value of this package: keeping that repeated compiler-pass logic in one small reusable helper.

## How To Extend It Safely {#how-to-extend-it-safely}

When you build your own compiler pass on top of this component:

- keep one parameter per target entity mapping
- make required mappings explicit
- keep optional mappings only for genuinely optional features
- ensure your parameter points to a real class implementing the expected interface
- register the compiler pass in the bundle where the Doctrine mapping depends on it

This helper is intentionally small, so the safest extension strategy is to keep your own pass simple too.

## What This Package Does Not Do {#what-this-package-does-not-do}

This component does not:

- create Doctrine mappings for you
- generate entities
- discover target entities automatically
- replace bundle configuration

It only helps wire interface-to-class target entity resolution into Doctrine's existing listener.

## Practical Limits {#practical-limits}

Keep these limits in mind:

- the package assumes Doctrine resolve target entity support is already available in the host application
- it is a compile-time helper, not a runtime lookup service
- it helps only with target entity registration, not with repository, form, or controller integration

## Summary {#summary}

Choose `doctrine-target-entity-resolver` when your bundle wants to stay interface-based, but Doctrine still needs concrete entity classes from the application.
