---
title: "Events"
description: "Reuse shared event classes and response-aware event helpers across Symfony bundles and applications."
---

# Events {#events}

`softspring/events` provides a small set of reusable event classes and helper traits for Symfony projects.

This component is useful when several bundles or application modules need the same event shapes, such as:

- a request-aware event
- a form-aware event
- a view event that carries mutable data
- an event that lets listeners return a `Response`

Its value is consistency. Instead of redefining these event objects in each package, you can reuse one shared component.

## What It Solves {#what-it-solves}

In many Symfony codebases, packages end up duplicating the same event patterns:

- a request event with the current request
- a form event with the current form and request
- a hook where listeners may stop the normal flow with a response
- a small helper that dispatches an event and returns the listener response

This component gives you those patterns in a single package.

It is especially useful in reusable bundles, because it helps keep extension points familiar across the ecosystem.

## Installation {#installation}

```bash
composer require softspring/events:^6.0
```

## Main Building Blocks {#main-building-blocks}

The package is small, so the most useful documentation is a direct guide to each part.

### RequestEvent {#requestevent}

`RequestEvent` is the simplest request-aware event.

Use it when you want to notify listeners about a flow that is tied to the current request, but you do not need form context or a response override.

Example:

```php
use Softspring\Component\Events\RequestEvent;

$event = new RequestEvent($request);
$dispatcher->dispatch($event, 'app.account.before_check');
```

Use this for hooks such as:

- before checking a request-dependent workflow
- after reading a route parameter
- around a controller flow that only needs the request context

### GetResponseRequestEvent {#getresponserequestevent}

`GetResponseRequestEvent` extends that pattern by allowing listeners to set a `Response`.

Use it when listeners should be able to stop the normal flow and return a response directly.

Typical cases:

- access checks that may redirect
- pre-controller hooks that may return a maintenance page
- custom guards that may short-circuit a request

Example:

```php
use Softspring\Component\Events\GetResponseRequestEvent;

$event = new GetResponseRequestEvent($request);
$dispatcher->dispatch($event, 'app.account.before_controller');

if (null !== $event->getResponse()) {
    return $event->getResponse();
}
```

### FormEvent {#formevent}

`FormEvent` is for flows built around a Symfony form.

It stores:

- the `FormInterface`
- an optional `Request`

Use it when listeners need to inspect or extend form handling but should not stop the flow with a response.

Typical cases:

- adding side effects after submit
- collecting analytics or audit signals around a form workflow
- enriching a form-based business process

### GetResponseFormEvent {#getresponseformevent}

`GetResponseFormEvent` is the response-aware version of `FormEvent`.

Use it when listeners around a form flow may:

- redirect after a custom condition
- interrupt normal submit handling
- return a custom response before the caller continues

This is a common pattern in reusable bundles that expose pre-submit or post-submit hooks.

## ViewEvent {#viewevent}

`ViewEvent` stores mutable view data in an `ArrayObject`.

You can pass either:

- an array
- an `ArrayObject`

The event always exposes an `ArrayObject`, which means listeners can modify the payload in place.

Use it when you want a clean extension point before rendering, serializing, or returning view data.

Typical cases:

- adding extra template variables
- removing values before rendering
- normalizing output data for a response

Example:

```php
use Softspring\Component\Events\ViewEvent;

$event = new ViewEvent([
    'entity' => $entity,
    'showSidebar' => true,
], $request);

$dispatcher->dispatch($event, 'app.article.view');

$data = $event->getData();
```

## Response-Aware Event Contract {#response-aware-event-contract}

The response-aware events share `GetResponseEventInterface`.

That contract is the common piece behind:

- `GetResponseEvent`
- `GetResponseRequestEvent`
- `GetResponseFormEvent`

Use the interface when your own code only cares about the ability to read or set a `Response`, regardless of the concrete event class.

This is useful in generic helper methods and reusable dispatch helpers.

## DispatchTrait {#dispatchtrait}

`DispatchTrait` is a very small helper for classes that already hold an event dispatcher.

It gives you:

```php
$this->dispatch('event.name', $event);
```

instead of repeating:

```php
$this->eventDispatcher->dispatch($event, 'event.name');
```

This is mainly a convenience trait for controllers, managers, and reusable service classes.

## DispatchGetResponseTrait {#dispatchgetresponsetrait}

`DispatchGetResponseTrait` builds on `DispatchTrait`.

It dispatches a response-aware event and returns the response if a listener has set one.

That is the helper to use when your flow follows this common Symfony pattern:

1. dispatch an event
2. let listeners optionally inject a response
3. return early when a response is present

Example:

```php
use Softspring\Component\Events\DispatchGetResponseTrait;
use Softspring\Component\Events\GetResponseRequestEvent;

$event = new GetResponseRequestEvent($request);

if ($response = $this->dispatchGetResponse('app.article.before_show', $event)) {
    return $response;
}
```

This keeps pre-controller and pre-action hooks compact and easy to read.

## Creating Your Own Events {#creating-your-own-events}

You do not have to limit yourself to the provided concrete classes.

Good extension patterns are:

- extend `RequestEvent` when you need extra request-related context
- extend `FormEvent` when you need extra form-related context
- implement `GetResponseEventInterface` when you want your own event shape but still want the response-short-circuit pattern
- reuse `GetResponseTrait` instead of rewriting response storage

That lets your own packages stay consistent with the rest of the ecosystem without forcing every event into the same class.

## Good Use Cases {#good-use-cases}

This component is a good fit when:

- you maintain reusable Symfony bundles
- you want consistent event APIs across packages
- you often expose pre-action, post-action, or form hooks
- listeners may sometimes interrupt a flow with a response

It is less useful when your project only dispatches domain events with custom payloads and does not need these common Symfony-oriented patterns.

## Recommended Usage Style {#recommended-usage-style}

Use the smallest event that fits the extension point:

- use `RequestEvent` when request context is enough
- use `FormEvent` when the form is the main context
- use `ViewEvent` when listeners should mutate output data
- use a `GetResponse*` event when listeners may short-circuit the flow

This keeps extension points easy to understand for package users.

## Summary {#summary}

Choose `softspring/events` when you want small, reusable, Symfony-oriented event objects and helper traits instead of redefining the same patterns in each package.

Its main value is consistency: the same event shapes, the same response-aware shortcut pattern, and the same dispatch helper style across multiple bundles and applications.
