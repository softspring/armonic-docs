---
title: "Logging"
description: "Choose the right Armonic component for runtime logs, cache logs, cloud logging, tracing, and audit trails."
---

# Logging {#logging}

Armonic does not provide one central logging bundle for every case.

Instead, logging is exposed through several focused packages, each one covering a different operational need.

Use this guide to choose the right one and avoid mixing normal application logs, tracing, and audit history.

## What Logging Means In Armonic {#what-logging-means-in-armonic}

In practice, Armonic projects usually need one or more of these concerns:

- runtime logs for operational actions and command execution
- cache logs for hits, misses, and cache writes
- cloud logging and error reporting integrations
- tracing for request timing and span visibility
- audit trails for business or data changes

These concerns overlap in production, but they are not the same thing and Armonic documents them separately.

## Generic Project Configuration {#generic-project-configuration}

For a normal Armonic project, the generic logging baseline should stay close to standard Symfony and Monolog conventions.

Start with:

- one default application log
- one channel for deprecations
- one dedicated channel only for subsystems that need separate operational visibility

In many projects, that means:

- `app` or the default unnamed application log
- `deprecation`
- `http_cache`
- `operations` for command-controller or sensitive admin actions

### Minimal Monolog Baseline {#minimal-monolog-baseline}

```yaml
# config/packages/monolog.yaml
monolog:
    channels: ['deprecation', 'http_cache', 'operations']

    handlers:
        main:
            type: stream
            path: '%kernel.logs_dir%/%kernel.environment%.log'
            level: info
            channels: ['!event', '!deprecation']

        deprecation:
            type: stream
            path: '%kernel.logs_dir%/%kernel.environment%.deprecations.log'
            level: notice
            channels: ['deprecation']

        http_cache:
            type: stream
            path: '%kernel.logs_dir%/%kernel.environment%.http_cache.log'
            level: info
            channels: ['http_cache']

        operations:
            type: stream
            path: '%kernel.logs_dir%/%kernel.environment%.operations.log'
            level: info
            channels: ['operations']
```

This structure is simple on purpose:

- normal application logs stay together
- noisy or operationally important subsystems get their own files
- Armonic-specific integrations can target a named logger cleanly

### Development vs Production {#development-vs-production}

In development, `debug` level can be acceptable for temporary diagnosis.

In production, start with `info` or `notice` for most handlers and only widen to `debug` for a specific channel when you are investigating a real issue.

That keeps log volume under control and makes the useful entries easier to find.

### How To Connect Armonic Components {#how-to-connect-armonic-components}

Once those channels exist, the Armonic-side wiring is straightforward.

Use the `operations` channel for command-controller endpoints:

```yaml
admin_tools_run_import:
    path: /admin/tools/run-import/{source}
    controller: Softspring\Component\CommandController\Controller\CommandController
    defaults:
        command: 'app:import'
        arguments: ['source']
        options: ['env']
        loggerOutputService: 'monolog.logger.operations'
```

Use the `http_cache` channel for HttpCache storage logging:

```yaml
sfs_http_cache_store:
    logger: 'monolog.logger.http_cache'
```

This gives you a generic project-level setup without tying every log concern to one global file.

## Choose The Right Package {#choose-the-right-package}

### Log Command Output From HTTP Endpoints {#log-command-output-from-http-endpoints}

Use [Command controller](command-controller.md) when a protected HTTP endpoint runs a Symfony command and you want the command output written to a PSR-3 logger.

This is the documented use of `loggerOutputService`.

Use it when:

- an admin or internal route triggers a command
- operators do not need streamed output in the browser
- you want command lines recorded in application logs

Typical shape:

```yaml
admin_tools_run_import:
    path: /admin/tools/run-import/{source}
    controller: Softspring\Component\CommandController\Controller\CommandController
    defaults:
        command: 'app:import'
        arguments: ['source']
        options: ['env']
        loggerOutputService: 'monolog.logger.operations'
```

This is a good fit for maintenance endpoints, imports, or back office actions that should leave an operational log trail.

### Log Symfony HttpCache Activity {#log-symfony-httpcache-activity}

Use [HTTP cache store bundle](http-cache-store-bundle.md) when the main thing you need is visibility into Symfony HttpCache behavior.

The bundle supports an optional logger service id for:

- cache hits
- cache misses
- cache writes
- fragment cache activity

Typical configuration:

```yaml
monolog:
    channels: ['http_cache']

sfs_http_cache_store:
    logger: 'monolog.logger.http_cache'
```

This is the right tool when you are debugging cache behavior, `Vary` handling, or fragment-heavy pages.

### Send Logs To Google Cloud {#send-logs-to-google-cloud}

Use [Google Cloud integration](google-cloud-integration.md) when the project needs Google Cloud logging, error reporting, or tracing integration.

This package is the documented entry point for:

- Google Cloud logging integration
- Google Cloud error reporting integration
- Google Cloud trace integration support

Use it when your application already runs on Google Cloud or your operations model is centered there.

### Use Tracing When Logs Are Not Enough {#use-tracing-when-logs-are-not-enough}

Use [Google Cloud trace](google-cloud-trace.md) when the problem is request timing and runtime visibility, not just textual log output.

Tracing helps answer questions such as:

- where request time is spent
- whether Twig, events, Doctrine, or HttpCache are involved
- which part of the request flow is slow

That is observability, not classic logging. In real projects you often use both:

- logs for events and operator-facing records
- traces for timing and request flow analysis

## Logging vs Audit History {#logging-vs-audit-history}

Some Armonic packages expose history or change tracking, but that does not make them general logging tools.

### Doctrine Entity Change History {#doctrine-entity-change-history}

Use [Doctrine changelog](doctrine-changelog-bundle.md) when you need an audit trail or change stream for Doctrine entities.

This component tracks entity insertions, updates, and deletions, and can enrich them with request and user context.

Use it when the main question is:

- who changed this entity
- what fields changed
- when the change happened

Do not treat it as a replacement for runtime logs.

### Mail History {#mail-history}

[Mailer bundle](../bundles/mailer-bundle.md) includes an optional email history model and admin UI.

The current documentation is explicit: this history model is not a full logging pipeline.

Use it when you need:

- a record of email messages in your application model
- admin visibility into sent or failed mail records

Do not use it as the only operational logging strategy for mail delivery.

## Recommended Baseline {#recommended-baseline}

For most production Armonic projects, a practical baseline is:

1. keep standard Symfony and Monolog application logging as the base
2. add a dedicated channel for HttpCache if cache behavior matters
3. log command-controller endpoints that trigger operational work
4. add tracing when request performance or runtime flow becomes hard to inspect
5. add audit tools such as Doctrine changelog only where business history matters

That keeps each signal focused and easier to operate.

## Common Mistakes {#common-mistakes}

- using audit history as if it were application logging
- expecting tracing to replace normal logs
- sending every command through HTTP without route protection and logging
- debugging HttpCache without a dedicated cache logger channel
- using mail history as the only evidence of delivery or failure handling

## Related Pages {#related-pages}

- [Command controller](command-controller.md)
- [HTTP cache store bundle](http-cache-store-bundle.md)
- [Google Cloud integration](google-cloud-integration.md)
- [Google Cloud trace](google-cloud-trace.md)
- [Doctrine changelog](doctrine-changelog-bundle.md)
- [Mailer bundle](../bundles/mailer-bundle.md)
