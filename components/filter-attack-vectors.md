---
title: "Filter Attack Vectors"
description: "Block a very small set of noisy request patterns before the main Symfony application continues processing them."
---

# Filter Attack Vectors {#filter-attack-vectors}

`softspring/filter-attack-vectors` is a very small defensive component.

It is loaded through Composer autoload and stops a short list of obviously unwanted request patterns before the rest of the application continues.

This package is intentionally simple. It is not a firewall, not a WAF, and not a general request validation system.

## What It Does {#what-it-does}

When the package is loaded, it checks `$_SERVER['REQUEST_URI']` and stops the request with a `404 Not Found` response when the URI matches one of its blocked patterns.

Today, it blocks:

- paths starting with `/wp-`
- requests containing `.php`
- requests containing `+--env=`

## Why Use It {#why-use-it}

This package is useful when you want a very small early filter for noisy probe traffic such as:

- WordPress scans hitting a non-WordPress app
- direct `.php` probes against a front-controller application
- simple Symfony env-injection style probes

The value is not deep security logic. The value is stopping a few common requests early and cheaply.

## Installation {#installation}

```bash
composer require softspring/filter-attack-vectors:^6.0
```

No extra Symfony configuration is needed.

## How It Is Applied {#how-it-is-applied}

The component is registered through Composer `autoload.files`.

That means the filter script is loaded automatically when Composer autoload is loaded by the application.

In practice, this makes the package very easy to add, but it also means the behavior is intentionally global for the application process.

## Current Rules {#current-rules}

### WordPress-style probes {#wordpress-style-probes}

Requests starting with `/wp-` are blocked.

This is aimed at common WordPress scanning noise in applications that do not expose WordPress endpoints.

### Direct PHP file probes {#direct-php-file-probes}

Requests containing `.php` are blocked.

This fits front-controller applications where public routes should not expose direct PHP file access.

### Symfony env-injection probes {#symfony-env-injection-probes}

Requests containing `+--env=` are blocked.

This targets a narrow class of hostile probe patterns rather than full command injection analysis.

## Recommended Use {#recommended-use}

Use this package when:

- you want a tiny filter with no configuration
- your application is a normal front-controller PHP app
- you are comfortable with a fixed, explicit rule set

It is a good fit for teams that want a small early noise filter and do not want to maintain a larger custom solution inside the application.

## What It Does Not Try To Do {#what-it-does-not-try-to-do}

This package does not try to:

- replace a reverse proxy or firewall
- understand every malicious request pattern
- classify traffic in detail
- offer configuration, logging, or rule management

That simplicity is intentional.

## Limits {#limits}

The limits are important:

- the rule set is fixed
- matching is intentionally coarse
- some blocked patterns may also match requests that are unusual but not malicious
- the package is best used as a small early filter, not as the only defensive layer

## Summary {#summary}

Choose this component when you want a tiny, explicit, autoload-based filter for a few recurring attack-vector probes and you prefer simplicity over configurability.
