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

A practical Armonic project can keep the Monolog configuration in one file and change the handlers for each Symfony environment.

Its baseline is:

- write development logs to `stderr`
- keep console output useful by excluding noisy channels
- buffer test logs and write them only when an error occurs
- write production logs to `stderr` for collection by the hosting platform
- declare application channels separately from their handlers

### Configuration By Environment {#configuration-by-environment}

```yaml
# config/packages/monolog.yaml
monolog:
    channels:
        - deprecation
        - cms

when@dev:
    monolog:
        handlers:
            main:
                type: stream
                path: 'php://stderr'
                level: debug
                channels: ['!event', '!doctrine']
            console:
                type: console
                process_psr_3_messages: false
                channels: ['!event', '!doctrine', '!console']

when@test:
    monolog:
        handlers:
            main:
                type: fingers_crossed
                action_level: error
                handler: nested
                excluded_http_codes: [404, 405]
                channels: ['!event']
            nested:
                type: stream
                path: '%kernel.logs_dir%/%kernel.environment%.log'
                level: debug

when@prod:
    monolog:
        handlers:
            main:
                type: stream
                path: 'php://stderr'
                level: info
                channels: ['!deprecation']
            console:
                type: console
                process_psr_3_messages: false
                channels: ['!event', '!doctrine']
```

This configuration follows a container-oriented deployment model:

- containers collect development logs from `stderr`
- tests keep normal output quiet but preserve the complete log when an error triggers the `fingers_crossed` handler
- expected `404` and `405` responses do not trigger test log files
- production entries at `info` level or above are written to `stderr` for collection by the hosting platform
- production deprecations are excluded from the main handler

Replace the production `stream` handler only when the hosting platform requires a specific Monolog handler or transport.

### Optional Browser Logging In Development {#optional-browser-logging-in-development}

You can also add `firephp` and `chromephp` handlers as development options:

```yaml
when@dev:
    monolog:
        handlers:
            firephp:
                type: firephp
                level: info
            chromephp:
                type: chromephp
                level: info
```

Enable them only when browser-based inspection is useful. These handlers add log data to response headers, so the web server may need a larger header-size limit.

### Channels And Handlers {#channels-and-handlers}

Declaring a channel creates a named logger such as `monolog.logger.cms`. It does not create a separate destination by itself. The environment handlers still decide where records from that channel are sent.

Add channels only when a component needs a named logger. For example, the HttpCache channel can be declared next to the component configuration:

```yaml
# config/packages/http_cache.yaml
monolog:
    channels: ['http_cache']

sfs_http_cache_store:
    logger: 'monolog.logger.http_cache'
```

This keeps optional component configuration together while the main Monolog handlers continue to control the destination in each environment.

### How To Connect Armonic Components {#how-to-connect-armonic-components}

When a component accepts a logger service id, point it to a declared channel.

For example, declare an `operations` channel only if command-controller output needs its own logger service:

```yaml
# config/packages/monolog.yaml
monolog:
    channels:
        - deprecation
        - cms
        - operations

# config/routes/admin_tools.yaml
admin_tools_run_import:
    path: /admin/tools/run-import/{source}
    controller: Softspring\Component\CommandController\Controller\CommandController
    defaults:
        command: 'app:import'
        arguments: ['source']
        options: ['env']
        loggerOutputService: 'monolog.logger.operations'
```

The named channel makes routing and filtering possible, but it still reaches the normal `main` handler unless you explicitly add a more specific handler.

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
