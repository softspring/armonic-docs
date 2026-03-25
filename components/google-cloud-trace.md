---
title: "Google Cloud Trace Bundle"
description: "Trace Symfony request flow, Twig rendering, event dispatching, Doctrine DBAL activity, and HttpCache work in Google Cloud Trace."
---

# Google Cloud Trace Bundle {#google-cloud-trace-bundle}

`softspring/google-cloud-trace-bundle` adds Google Cloud Trace spans to the main runtime layers of a Symfony application.

The bundle is designed for projects that already run behind distributed tracing infrastructure and receive a `traceparent` header. When that context is available, the bundle creates spans for the request flow and sends them to Google Cloud Trace at the end of the request.

This is mainly an integration component. Its value is not a user-facing feature. Its value is making performance and request flow easier to inspect in production.

## What It Solves {#what-it-solves}

Without this bundle, trace data usually has to be added by hand in controllers and services.

With this bundle, you get tracing around common Symfony layers with almost no application code:

- request handling in the kernel
- kernel termination
- dispatched events
- Twig template rendering
- Symfony HttpCache execution
- Doctrine DBAL query and transaction activity

That gives you a fast way to answer practical questions such as:

- where is this request spending time?
- is the slowdown in Twig, events, or database access?
- is HttpCache involved?
- which route or response characteristics are attached to the traced request?

## Installation {#installation}

```bash
composer require softspring/google-cloud-trace-bundle:^6.0
```

Register the bundle in your application if needed.

## Quick Start {#quick-start}

The normal setup is:

1. install the bundle
2. enable the bundle in your Symfony application
3. make sure your application receives a valid `traceparent` header from upstream infrastructure
4. use the kernel trait if you want the kernel itself to own the top-level server span and trace submission

There is no custom bundle configuration to add in the common case.

## Important Requirement: Trace Context {#important-requirement-trace-context}

The bundle only creates spans when an incoming trace context exists.

In practice, that means the request must include a `traceparent` header. If that header is missing, the tracing helper returns `null` spans and the decorators behave as no-ops.

This is important for adoption:

- if your local environment does not send `traceparent`, the bundle will look inactive
- if your reverse proxy or platform strips that header, no trace will be created
- if your application already receives distributed tracing context from Google Cloud infrastructure, the bundle can attach its spans to that request flow

## What Gets Traced Automatically {#what-gets-traced-automatically}

The bundle hooks into the container at compile time and instruments the services that are present in the application.

### Kernel Handling {#kernel-handling}

The HTTP kernel is decorated to trace:

- the request path itself
- `kernel.handle`
- `kernel.terminate`

This gives you the main application flow for each traced request.

### Event Dispatching {#event-dispatching}

The Symfony event dispatcher is decorated so each dispatched event can create its own span.

This is useful when your application or Symfony itself relies heavily on events. It helps reveal whether request time is spent in event-driven workflows instead of controllers alone.

### Twig Rendering {#twig-rendering}

If Twig is enabled, the bundle replaces the Twig environment class with a traced one.

That means calls to `render()` create a span named after the template being rendered.

This is useful when:

- pages render many nested templates
- email or back office pages are Twig-heavy
- you want to see whether rendering is part of a slow request

### HttpCache {#httpcache}

If Symfony HttpCache is enabled, the bundle replaces the HttpCache class with a traced one.

This helps when you need visibility into:

- cache request handling
- cache termination
- the difference between application time and HttpCache time

### Doctrine DBAL {#doctrine-dbal}

If Doctrine is present, the bundle instruments DBAL activity.

It supports two styles depending on the DBAL version in use:

- middleware-based tracing on newer DBAL versions
- SQL logger decoration on older DBAL versions

This gives you spans for:

- queries
- `exec`
- transaction begin
- commit
- rollback

## Using The Kernel Trait {#using-the-kernel-trait}

The package also ships a `TraceKernelTrait`.

Use it when you want your kernel to:

- create a top-level server span during boot
- stop that span on terminate
- send the collected trace at the end of the request

This is the simplest integration point when you control the kernel and want request-level tracing without wiring custom services yourself.

Typical use:

```php
use Softspring\GoogleCloudTraceBundle\TraceKernelTrait;
use Symfony\Bundle\FrameworkBundle\Kernel\MicroKernelTrait;
use Symfony\Component\HttpKernel\Kernel as BaseKernel;

class Kernel extends BaseKernel
{
    use MicroKernelTrait;
    use TraceKernelTrait;
}
```

Use this carefully if your kernel already customizes `boot()` or `terminate()`, because the trait becomes part of that lifecycle.

## Creating Custom Spans {#creating-custom-spans}

The bundle is not limited to automatic tracing.

You can also use the `Tracer` helper directly in your own code when you want business-level spans around work that matters to your application.

Typical cases:

- wrapping an external API call
- tracing a pricing or billing calculation
- measuring a document generation step
- tracing a domain workflow that is more useful than a low-level event name

Basic pattern:

```php
use Softspring\GoogleCloudTraceBundle\Trace\Tracer;

$span = Tracer::createSpan('billing.generate_invoice');
Tracer::start($span);

// do work

Tracer::stop($span);
```

This is especially useful when the automatic layers are not enough and you need spans that match your domain language.

## What Attributes Are Added {#what-attributes-are-added}

When request or response information is available, the tracer adds attributes such as:

- request URL and method
- request scheme
- Symfony route name
- response status code
- response protocol version
- response content length
- selected cache-related response values such as TTL and cache control directives

This makes the trace more useful when comparing behavior across routes or responses.

## Good Use Cases {#good-use-cases}

This component is a good fit when:

- your application already runs in Google Cloud
- distributed trace context reaches PHP requests
- you want quick visibility into Symfony runtime layers
- you need more observability without manually instrumenting every feature

It is especially useful in projects that combine:

- heavy Twig rendering
- event-driven architecture
- Doctrine-heavy request flow
- Symfony HttpCache

## Extension And Integration Notes {#extension-and-integration-notes}

The bundle is intentionally light on configuration. Most extension happens through integration choices:

- decide whether to use the kernel trait
- add your own custom spans around domain workflows
- keep or replace tracing around the services that matter most in your application

If you need more control over naming, filtering, or span strategy, the usual approach is to build that around `Tracer` in your own application layer.

## Troubleshooting {#troubleshooting}

### I installed the bundle but no traces appear {#i-installed-the-bundle-but-no-traces-appear}

The first thing to check is whether the request includes `traceparent`.

Without that header, the tracer stays inactive by design.

### Twig or Doctrine are not traced {#twig-or-doctrine-are-not-traced}

The bundle only instruments the related subsystem when it is present in the container. If Twig, Doctrine, or HttpCache are not enabled in the application, there is nothing to decorate.

### My application needs domain-level trace names {#my-application-needs-domain-level-trace-names}

Use `Tracer` directly in your own services. The automatic spans are a baseline, not a replacement for all application-specific instrumentation.

## Current Limits {#current-limits}

The main limits to keep in mind are:

- tracing depends on incoming trace context
- there is no rich configuration layer for filtering or renaming spans
- the bundle focuses on request lifecycle instrumentation, not full background worker tracing
- automatic tracing gives infrastructure visibility first; domain-level spans still need to be added by the application when needed

## Summary {#summary}

Choose this bundle when you want Google Cloud Trace visibility across the main Symfony runtime layers with very little setup.

Its main value is practical observability: faster debugging of slow requests, clearer understanding of where time is spent, and a simple base for adding your own application-level spans.
