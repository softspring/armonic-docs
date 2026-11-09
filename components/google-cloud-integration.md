---
title: "Google Cloud Integration Bundle"
description: "Send Symfony logs to Google Cloud Logging and attach HTTP request and trace context."
---

# Google Cloud Integration Bundle {#google-cloud-integration-bundle}

Use this bundle to connect Monolog to Google Cloud Logging and attach request context to log records. It also provides optional Google Cloud Error Reporting bootstrap and listeners. Request tracing itself is covered by the separate [Google Cloud Trace Bundle](google-cloud-trace.md).

## Install the bundle {#installation}

```bash
composer require softspring/google-cloud-integration-bundle:6.1.*@dev
```

Enable the bundle if Flex does not register it:

```php
// config/bundles.php
Softspring\Component\GoogleCloudIntegrationBundle\SfsGoogleCloudIntegrationBundle::class => ['all' => true],
```

The logging integration expects MonologBundle. Configure Application Default Credentials in the runtime environment using your deployment platform's service identity. The identity needs permission to write logs to the target Google Cloud project. Keep credential files outside the repository and public directory.

## Configure Cloud Logging {#configure-logging}

Set `GCLOUD_PROJECT` to the target project identifier. Route the desired Monolog records to the handler registered by the bundle:

<!-- config-tabs:start -->

```yaml
# config/packages/monolog.yaml
when@prod:
    monolog:
        handlers:
            cloud:
                type: service
                id: Softspring\Component\GoogleCloudIntegrationBundle\Logging\GoogleCloudLoggingHandler
```

```php
<?php

// config/packages/monolog.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container): void {
    if ($container->env() === 'prod') {

        $container->extension('monolog', [
            'handlers' => [
                'cloud' => [
                    'type' => 'service',
                    'id' => \Softspring\Component\GoogleCloudIntegrationBundle\Logging\GoogleCloudLoggingHandler::class,
                ],
            ],
        ]);

    }
};
```

<!-- config-tabs:end -->

Review the application's existing handlers and channels before adding this handler. Sending the same channel to several handlers can intentionally duplicate its records. Continue using a local stream handler during development.

The handler wraps Google's PSR batch logger. Verify batch delivery in the deployment environment, including its process lifecycle and credentials; local container compilation alone does not prove that logs reached Google Cloud.

## Attach request context {#request-context}

The registered Monolog processor adds `context.stackdriverOptions` when there is a main HTTP request and `GCLOUD_PROJECT` is nonempty. It includes:

- HTTP method, URL, protocol, and available client metadata;
- `X-Request-Id` as a `request_id` label;
- a valid `X-Cloud-Trace-Context` as the Cloud Logging trace identifier, span identifier, and sampled flag.

For example, a request with `X-Cloud-Trace-Context: 105445aa7843bc8bf206b12000100000/1;o=1` is linked to `projects/<project>/traces/105445aa7843bc8bf206b12000100000`. The decimal span identifier is converted to the hexadecimal format expected by Cloud Logging.

The processor enriches a log record; it does not start a trace or a span. CLI records and records without a configured project pass through unchanged. HTTP URLs and referers can contain application data, so apply the application's logging and data-retention policy when deciding which requests to record.

## Enable Error Reporting {#error-reporting}

The optional error-reporting integration checks the process environment:

```dotenv
GCLOUD_ERROR_REPORTING=1
```

Set this before PHP loads Composer's autoloader. The bootstrap and bundle extension use `getenv()`, so a value present only in Symfony's `$_ENV` after Dotenv loading may not activate it. Configure the variable in PHP-FPM, the container, or the process launcher, then rebuild the production cache.

Check the installed Google Cloud Error Reporting dependencies and runtime credentials before enabling it. Test delivery with a synthetic exception in a controlled environment. Do not expose a public route whose only purpose is to trigger errors.

## Verify the integration {#verify-integration}

First confirm that the container resolves the configured handler. Then emit a normal application log message through `Psr\Log\LoggerInterface`, make an HTTP request with a synthetic request identifier, and check the target project's logs for that message and label.

Test CLI and HTTP execution separately: only HTTP requests provide request context. If a log appears without a trace link, verify the project identifier and incoming trace header before changing the application logger.
