---
title: "Crudl Controller"
description: "Build reusable CRUD and list controllers for Symfony backoffice screens with managers, action configuration, events, and pagination-aware lists."
---

# Crudl Controller {#crudl-controller}

`softspring/crudl-controller` is a reusable controller toolkit for classic Symfony backoffice screens.

It is designed for cases where many entities need the same kind of actions:

- create
- read
- update
- delete
- list
- apply
- transition

Instead of rewriting the controller flow for every entity, you wire one controller service, one manager, some action configuration, and then customize behavior with forms and events.

## When To Use It {#when-to-use-it}

This component is a good fit when your application has backoffice or admin screens such as:

- products
- users
- orders
- invoices
- blog posts
- CMS content

and those screens all need the same kind of server-rendered CRUD flow.

It is especially useful when you want:

- one consistent action flow across entities
- reusable redirects, events, and form hooks
- list pages with filters and pagination
- the option to replace default persistence with custom apply logic

## Main Idea {#main-idea}

The package is built around three pieces:

- `CrudlController`
- a manager implementing `CrudlEntityManagerInterface`
- action configuration passed through `$configs`

The controller owns the generic action flow.

The manager owns entity creation, repository access, persistence, and deletion.

The configuration decides which forms, templates, routes, and events each action should use.

## Quick Start {#quick-start}

The common pattern is:

1. create a manager for your entity
2. register `CrudlController` as a Symfony service
3. configure the actions you need
4. add routes that point to `::create`, `::read`, `::update`, `::delete`, or `::list`

## Create A Manager {#create-a-manager}

The manager is the bridge between the generic controller flow and the concrete Doctrine entity.

You can implement the interface directly or extend the default manager.

Simple custom manager:

```php
<?php

namespace App\Manager;

use Doctrine\ORM\EntityManagerInterface;
use Softspring\Component\CrudlController\Manager\CrudlEntityManagerInterface;
use Softspring\Component\CrudlController\Manager\CrudlEntityManagerTrait;
use App\Entity\Product;

class ProductManager implements CrudlEntityManagerInterface
{
    use CrudlEntityManagerTrait;

    public function __construct(
        protected EntityManagerInterface $em,
    ) {
    }

    public function getTargetClass(): string
    {
        return Product::class;
    }
}
```

Or extend the default manager:

```php
<?php

namespace App\Manager;

use Softspring\Component\CrudlController\Manager\DefaultCrudlEntityManager;

class ProductManager extends DefaultCrudlEntityManager
{
}
```

Service example:

```yaml
services:
    App\Manager\ProductManagerInterface:
        class: App\Manager\ProductManager
        arguments:
            $targetClass: App\Entity\Product
```

## Use Doctrine Target Entities {#use-doctrine-target-entities}

The manager also works well when your real entity is resolved from an interface.

That is useful in reusable bundles where the package wants to work with a model contract and the application provides the final entity class.

In that case:

- `getTargetClass()` can return the interface
- `getEntityClass()` resolves the concrete Doctrine entity class

This is one of the reasons `crudl-controller` is useful in reusable bundles, not only in plain applications.

## Register The Controller Service {#register-the-controller-service}

`CrudlController` is meant to be registered as a normal Symfony service.

Typical service wiring:

```yaml
services:
    _defaults:
        autowire: true

    product.controller:
        class: Softspring\Component\CrudlController\Controller\CrudlController
        public: true
        tags: [ 'controller.service_arguments' ]
        arguments:
            $manager: '@App\Manager\ProductManagerInterface'
            $configs:
                create:
                    entity_attribute: 'product'
                    form: App\Form\Admin\ProductCreateForm
                    view: 'admin/products/create.html.twig'

                read:
                    entity_attribute: 'product'
                    param_converter_key: 'id'
                    view: 'admin/products/read.html.twig'

                update:
                    entity_attribute: 'product'
                    param_converter_key: 'id'
                    form: App\Form\Admin\ProductUpdateForm
                    view: 'admin/products/update.html.twig'

                delete:
                    entity_attribute: 'product'
                    param_converter_key: 'id'
                    form: App\Form\Admin\ProductDeleteForm
                    view: 'admin/products/delete.html.twig'

                list:
                    entities_attribute: 'products'
                    filter_form: App\Form\Admin\ProductListFilterForm
                    view: 'admin/products/list.html.twig'
                    read_route: 'app_admin_product_read'
```

In practice, most projects use one controller service per backoffice section.

## Add Routes {#add-routes}

Once the service is registered, routes are straightforward:

```yaml
app_admin_product_list:
    controller: product.controller::list
    path: /

app_admin_product_create:
    controller: product.controller::create
    path: /create

app_admin_product_update:
    controller: product.controller::update
    path: /{id}/update

app_admin_product_delete:
    controller: product.controller::delete
    path: /{id}/delete

app_admin_product_read:
    controller: product.controller::read
    path: /{id}
```

Then import them under a common prefix if needed:

```yaml
app_admin_product_routes:
    resource: 'routes/admin_product.yaml'
    prefix: '/admin/product'
```

## Action Overview {#action-overview}

The package supports several action types.

### Create {#create}

Use `create` when a new entity must be built, shown in a form, and then persisted.

Typical usage:

- admin product creation
- creating a new CMS block
- onboarding a new resource from a backoffice screen

Main configuration points:

- `form`
- `view`
- `entity_attribute`
- `is_granted`
- `success_redirect_to`

Main event hooks:

- `initialize_event_name`
- `create_entity_event_name`
- `form_prepare_event_name`
- `form_init_event_name`
- `form_valid_event_name`
- `apply_event_name`
- `success_event_name`
- `failure_event_name`
- `form_invalid_event_name`
- `view_event_name`
- `exception_event_name`

This is the most extensible action when you need to initialize a new entity before the form or replace persistence with a custom apply step.

### Read {#read}

Use `read` when you need a detail page for an existing entity.

Typical usage:

- detail page in an admin section
- preview of a single record
- read-only screen before update or transition actions

Main configuration points:

- `param_converter_key`
- `view`
- `entity_attribute`
- `is_granted`

Main event hooks:

- `not_found_event_name`
- `initialize_event_name`
- `view_event_name`

This action is intentionally simple and works well for admin detail pages.

### Update {#update}

Use `update` when an entity must be loaded, displayed in a form, and saved again.

Typical usage:

- edit product
- edit user
- edit invoice or blog post

It combines the entity-loading pattern of `read` with the form workflow of `create`.

Main configuration points:

- `param_converter_key`
- `form`
- `view`
- `entity_attribute`
- `success_redirect_to`

Main event hooks:

- `load_entity_event_name`
- `found_event_name`
- `not_found_event_name`
- `form_prepare_event_name`
- `form_init_event_name`
- `form_valid_event_name`
- `apply_event_name`
- `success_event_name`
- `failure_event_name`
- `form_invalid_event_name`
- `view_event_name`
- `exception_event_name`

This is the standard choice for classic edit screens.

### Delete {#delete}

Use `delete` when deletion should go through a form and a controlled workflow.

Typical usage:

- confirmation page before delete
- delete action with a custom form field or extra confirmation
- delete flow that dispatches side effects after removal

It follows the same structure as `update`, but the default apply step calls `deleteEntity()` on the manager.

### List {#list}

Use `list` when a backoffice screen needs filters, sorting, pagination, and a reusable result flow.

This action builds on:

- `softspring/doctrine-query-filters`
- `softspring/doctrine-paginator`

Main configuration points:

- `filter_form`
- `entities_attribute`
- `view`
- `view_page`
- `read_route`

Main event hooks:

- `initialize_event_name`
- `filter_form_prepare_event_name`
- `filter_form_init_event_name`
- `filter_event_name`
- `view_event_name`
- `exception_event_name`

This is the action that gives the package its `CRUDL` shape instead of only `CRUD`.

### Apply {#apply}

Use `apply` when you want the package to load the entity and run the action lifecycle, but the real side effect must come from an `apply_event_name`.

Typical usage:

- trigger an external API action on an entity
- run a command-like operation from a backoffice button
- perform a custom action that is not a standard update or delete

Important detail:

- this action is intentionally event-driven
- if no `apply_event_name` marks the action as applied, the controller throws

So `apply` is the right tool when the bundle should orchestrate the flow, but your application owns the real side effect.

### Transition {#transition}

Use `transition` when the entity is attached to a Symfony Workflow and the backoffice must execute one transition.

Typical usage:

- publish / unpublish
- approve / reject
- archive / restore

Main configuration points:

- `transition_attribute`
- `workflow_name`
- `form`
- `view`
- `success_redirect_to`

This action is valuable when a project wants the same eventful CRUDL flow around a workflow transition instead of writing a custom controller per transition.

## Events As The Main Extension Point {#events-as-the-main-extension-point}

Events are one of the strongest parts of the bundle.

They let you customize the action flow without subclassing the controller and without rewriting the whole action.

Typical event use cases:

- stop the flow early and return a custom response
- build the entity manually
- change form options
- add dynamic fields
- replace persistence with API calls
- add view data
- handle failure or exception cases

This is what makes the bundle useful in reusable bundles and larger applications: the controller flow stays standard, but the business rules remain open.

## Common Customization Patterns {#common-customization-patterns}

Some common patterns work especially well with this package.

### Custom Entity Loading {#custom-entity-loading}

Use `load_entity_event_name` when loading by route parameter is not enough.

Examples:

- load by slug instead of id
- load by composite criteria
- load through a bundle-specific repository method

### Replace Default Persistence {#replace-default-persistence}

Use `apply_event_name` and mark the event as applied when persistence must not go through the default Doctrine manager.

Examples:

- save to an external API
- dispatch a domain command instead of persisting directly
- perform extra side effects before considering the action successful

### Add Form Context {#add-form-context}

Use `form_prepare_event_name` and `form_init_event_name` when the form needs dynamic options or dynamic fields.

Examples:

- change validation groups
- pass current user or request context
- add conditional fields only for some routes or states

### Add View Data {#add-view-data}

Use `view_event_name` when templates need extra data but the controller should stay generic.

Examples:

- sidebar data
- breadcrumbs
- extra action buttons
- statistics or warnings tied to the entity

## A Practical Product Admin Example {#a-practical-product-admin-example}

This is a typical case for the bundle:

- `list` shows paginated products with filters
- `create` opens a form for a new product
- `read` shows product details
- `update` edits an existing product
- `delete` asks for confirmation before removal
- `transition` publishes or archives the product

The manager handles the entity.

The controller service owns the generic flow.

Events inject project-specific rules where needed.

That is the sweet spot of this package.

## Recommended Usage Style {#recommended-usage-style}

This package works best when:

- controllers are registered as services
- each admin section has one manager
- actions are configured explicitly
- events are used for business-specific branching
- list screens use the paginator and filter components consistently

It works less well when:

- the action flow is extremely custom and barely resembles CRUD or list behavior
- the project is API-first and does not need Twig or server-rendered forms

## Practical Limits {#practical-limits}

Keep these limits in mind:

- list action assumes a filter form and the Softspring paginator/filter stack
- apply action is not a generic “do something” shortcut; it must be driven by the apply event
- transition action assumes Symfony Workflow is already modeled in the application
- this is a controller workflow component, not a full admin bundle with templates included

## Summary {#summary}

Choose `crudl-controller` when your Symfony project needs many backoffice CRUD and list screens with a consistent structure, but still wants enough hooks to adapt each action to real business rules.
