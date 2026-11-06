---
title: "Command Controller"
description: "Expose selected Symfony console commands through HTTP controllers with buffered or streamed output."
---

# Command Controller {#command-controller}

`softspring/command-controller` lets a Symfony application execute console commands from an HTTP controller.

This is useful for internal tools, admin endpoints, maintenance actions, or controlled integrations where a command already exists and you want an HTTP entry point on top of it.

It is not meant to turn every command into a public web API. The value of the component is giving you a small bridge from request data to console execution when that bridge is really needed.

## Installation {#installation}

```bash
composer require softspring/command-controller:^6.0
```

## When To Use It {#when-to-use-it}

Use this component when:

- you already have a Symfony command that solves the job
- the action should be triggered from an admin route or an internal endpoint
- a streamed plain-text response is acceptable
- you want to reuse command code instead of duplicating the logic in a controller

Typical examples:

- cache warmers or maintenance commands triggered from a protected admin area
- bounded operations started from back office tools
- operational endpoints used by trusted internal systems

## Main Building Blocks {#main-building-blocks}

The component gives you:

- `CommandController`
- `CommandRunner`
- `StreamedCommandRunner`
- `LoggerCommandOutput`
- `StreamedCommandOutput`

In practice, most applications start with the controller and only touch the runners or output classes when they need more control.

## Quick Start {#quick-start}

The controller expects:

- a command name
- a Symfony `Request`
- a list of request keys that are allowed as command arguments
- a list of request keys that are allowed as command options

Assume the application already defines `app:import` with a `source` argument and a `--dry-run` option. Register `CommandController` as a public controller service, protect the route with the application's administrator authorization, and add CSRF or equivalent request authentication for state-changing operations. The component does not provide those controls automatically.

The route can then pass only the selected argument and option:

<!-- config-tabs:start -->

```yaml
admin_tools_run_import:
    path: /admin/tools/run-import/{source}
    controller: Softspring\Component\CommandController\Controller\CommandController
    methods: [POST]
    defaults:
        command: 'app:import'
        arguments: [ 'source' ]
        options: [ 'dry-run' ]
```

```php
<?php

// config/routes.php
namespace Symfony\Component\Routing\Loader\Configurator;

return static function (RoutingConfigurator $routes): void {
    $routes->add('admin_tools_run_import', '/admin/tools/run-import/{source}')
        ->controller(\Softspring\Component\CommandController\Controller\CommandController::class)
        ->methods(['POST'])
        ->defaults(['command' => 'app:import', 'arguments' => ['source'], 'options' => ['dry-run']]);
};
```

<!-- config-tabs:end -->

Send an authenticated POST request, for example to:

```text
/admin/tools/run-import/products?dry-run=1
```

the component builds command input from the allowed route attributes and query values. It does not read ordinary POST body fields for these arguments. Execution remains synchronous inside the HTTP request; use Messenger or another queue for background work.

## Arguments And Options {#arguments-and-options}

The controller does not forward every request value automatically.

That is the core safety feature of the package.

- keys listed in `arguments` become command arguments
- keys listed in `options` become command options prefixed with `--`

This means you decide explicitly what the HTTP layer is allowed to pass to the command.

That keeps the integration predictable and reduces accidental command exposure.

## Buffered Vs Streamed Output {#buffered-vs-streamed-output}

By default, the controller uses streamed output. Its response starts with HTTP 200 and does not translate the command's exit code into an error status. Use buffered mode when the caller needs a reliable 200/500 status for success or failure.

That is usually the right choice for commands that may take time or should show progress as they run.

If you set the request attribute `stream` to `false`, the component uses the buffered runner instead and returns a normal plain-text response after the command ends.

Use streamed mode when:

- the command can take more than a few seconds
- seeing incremental output helps operators
- you want to avoid buffering a large command response in memory

Use buffered mode when:

- the command is short
- you only care about the final output
- the endpoint is used more like a simple action than a live console stream

## Logging Output {#logging-output}

If the request provides `loggerOutputService`, the controller resolves that service from the container and routes command output into a PSR-3 logger.

This is useful when:

- command output should be stored in logs
- operators do not need live streamed output
- you want command lines to be recorded as application log entries

The logger must be accessible through the injected container. Registering the controller as a service alone does not inject that container automatically: configure its constructor explicitly, or use an application controller with a narrow service locator. Keep the logger id in trusted route defaults, not user-supplied input.

## Using The Runners Directly {#using-the-runners-directly}

You do not need to go through the controller in every case.

You can use:

- `CommandRunner` for buffered responses
- `StreamedCommandRunner` for streamed responses

That is useful when:

- you want your own controller around the execution flow
- you need extra access checks
- you want to build a more specific admin action on top of the same command bridge

## Recommended Safety Rules {#recommended-safety-rules}

Running commands from HTTP is powerful, so keep the integration narrow.

Recommended rules:

- protect the route with strong access control
- expose only explicitly selected commands
- whitelist only the request values you really need
- prefer internal or admin-only routes
- log usage when the endpoint has operational impact

If the action is business-facing or user-facing, it is usually better to build a proper service/controller flow instead of bridging to a console command.

## Extension Patterns {#extension-patterns}

Common extension patterns are:

- wrapping the base controller in your own controller class
- adding your own authorization rules before calling the runner
- replacing output handling with a custom logger or stream strategy
- defining application-specific routes that expose only one operational command each

This keeps the generic component small while still letting the application own the risky parts.

## Current Limits {#current-limits}

- The runners are tightly coupled to the application kernel boot process.
- The component is best for internal tooling, not for public API style endpoints.
- Long-running or high-volume operational work may still be better solved with queues or workers instead of direct HTTP-triggered commands.
