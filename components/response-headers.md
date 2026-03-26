---
title: "Response Headers"
description: "Centralize HTTP response header rules in Symfony and apply them conditionally when needed."
---

# Response Headers {#response-headers}

`softspring/response-headers` helps you keep HTTP response header rules in one place.

Instead of repeating header logic in controllers or custom listeners, you can register one listener and feed it a reusable set of rules.

This is especially useful for:

- security headers
- API headers
- conditional headers for admin or API areas
- shared header policies across several responses

## Installation {#installation}

```bash
composer require softspring/response-headers:^6.0
```

If you want conditions, also install:

```bash
composer require symfony/expression-language
```

## What This Component Provides {#what-this-component-provides}

This package is a component, not a full Symfony bundle with automatic configuration.

It provides one main building block:

- `Softspring\Component\ResponseHeaders\EventListener\ResponseHeadersListener`

That listener subscribes to the Symfony response event and applies the configured header rules to the current response.

## Basic Setup {#basic-setup}

You must register the listener yourself.

```yaml
# config/packages/response_headers.yaml
parameters:
    response_headers:
        X-Frame-Options: "SAMEORIGIN"
        X-Content-Type-Options: "nosniff"
        Referrer-Policy: "same-origin"

services:
    Softspring\Component\ResponseHeaders\EventListener\ResponseHeadersListener:
        arguments:
            $headers: '%response_headers%'
        tags: ['kernel.event_subscriber']
```

This is enough when all you need is a fixed set of headers applied to every response.

## How Header Definitions Work {#how-header-definitions-work}

The component accepts three practical rule styles.

### Single Value Headers {#single-value-headers}

Use a plain string when the header has one direct value:

```yaml
parameters:
    response_headers:
        X-Frame-Options: "SAMEORIGIN"
        X-Content-Type-Options: "nosniff"
```

This is the simplest form and works well for classic security headers.

### Multiple Directives In One Header {#multiple-directives-in-one-header}

Use an array when you want to build one header from multiple fragments:

```yaml
parameters:
    response_headers:
        Permissions-Policy:
            - "geolocation=()"
            - "camera=()"
        Strict-Transport-Security:
            - "max-age=31536000"
            - "includeSubDomains"
```

The component joins array values with `; `.

That makes it practical for directive-style headers such as:

- `Permissions-Policy`
- `Strict-Transport-Security`
- `Content-Security-Policy`

## Advanced Rule Options {#advanced-rule-options}

Use a structured rule when you need more than a plain value:

```yaml
parameters:
    response_headers:
        Content-Security-Policy:
            value:
                - "default-src 'self'"
                - "img-src 'self' data:"
            replace: true
```

The supported fields are:

- `value`: the header value, as a string or an array
- `replace`: whether the new value replaces an existing header
- `condition`: an ExpressionLanguage expression evaluated for that header

Use this form whenever you need `replace` or `condition`.

## Conditional Headers {#conditional-headers}

Conditions are optional.

If you do not configure conditions, the component works without `symfony/expression-language`.

If you do configure conditions, you must wire an `ExpressionLanguage` instance:

```yaml
# config/packages/response_headers.yaml
parameters:
    response_headers_global_conditions: []
    response_headers:
        X-Robots-Tag:
            value: 'noindex'
            condition: 'request.getPathInfo() matches "^/admin"'

services:
    softspring.response_headers.expression_language:
        class: Symfony\Component\ExpressionLanguage\ExpressionLanguage

    Softspring\Component\ResponseHeaders\EventListener\ResponseHeadersListener:
        arguments:
            $headers: '%response_headers%'
            $expressionLanguage: '@softspring.response_headers.expression_language'
            $globalConditions: '%response_headers_global_conditions%'
        tags: ['kernel.event_subscriber']
```

If you add conditions but do not provide an expression language instance, the listener throws an explicit exception instead of silently ignoring the rule.

## Global Conditions {#global-conditions}

Global conditions run before any individual header rule.

This is useful when you want one gate for the whole header set:

```yaml
parameters:
    response_headers_global_conditions:
        - 'isMainRequest'
```

This pattern is usually a good default because it avoids applying the same header rules to sub-requests.

## Expression Context {#expression-context}

Conditions can use these variables:

- `request`
- `response`
- `isMainRequest`

That lets you build practical rules such as:

```yaml
parameters:
    response_headers:
        X-Robots-Tag:
            value: 'noindex'
            condition: 'request.getPathInfo() matches "^/admin"'
        Access-Control-Allow-Origin:
            value: '*'
            condition: 'request.getPathInfo() matches "^/api"'
```

## Practical Use Cases {#practical-use-cases}

### Security Header Baseline {#security-header-baseline}

```yaml
parameters:
    response_headers_global_conditions:
        - 'isMainRequest'
    response_headers:
        X-Frame-Options: "SAMEORIGIN"
        X-Content-Type-Options: "nosniff"
        Referrer-Policy: "same-origin"
        Strict-Transport-Security:
            - "max-age=31536000"
            - "includeSubDomains"
```

### Conditional Admin Headers {#conditional-admin-headers}

```yaml
parameters:
    response_headers:
        X-Robots-Tag:
            value: 'noindex'
            condition: 'request.getPathInfo() matches "^/admin"'
```

### Policy Headers Built From Multiple Directives {#policy-headers-built-from-multiple-directives}

```yaml
parameters:
    response_headers:
        Permissions-Policy:
            - "geolocation=()"
            - "camera=()"
        Content-Security-Policy:
            - "default-src 'self'"
            - "img-src 'self' data:"
            - "script-src 'self'"
```

## Extending The Component {#extending-the-component}

The component is small on purpose, but it is easy to extend.

### Provide A Custom ExpressionLanguage {#provide-a-custom-expressionlanguage}

You can wire your own `ExpressionLanguage` service with custom providers or functions when your conditions need project-specific helpers.

### Decorate Or Replace The Listener {#decorate-or-replace-the-listener}

If your project needs rules from another source, extra context, or a different application strategy, decorate or replace `ResponseHeadersListener`.

This keeps the public integration simple while still letting your application own the final behavior.

### Register More Than One Listener {#register-more-than-one-listener}

Because this is a plain Symfony service, you can register more than one listener if your application needs separate rule sets managed in different places.

## Limits And Things To Keep In Mind {#limits-and-things-to-keep-in-mind}

- The component does not ship automatic Symfony bundle configuration.
- Applications must wire the listener themselves.
- Array values are always joined with `; `.
- Conditions require `symfony/expression-language`.
- The component applies rules on the response event; it is not a controller helper or a route-level DSL.
