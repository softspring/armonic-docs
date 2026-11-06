---
title: "CMS Module Symfony Route Field Type"
description: "Configure the symfonyRoute form type to select Symfony routes and route parameters inside CMS modules."
---

# Symfony Route type {#symfony-route-type}

This **symfonyRoute** type makes reference to *Softspring\CmsBundle\Form\Type\SymfonyRouteType* form type.

This form type provides a way to select a route.

## Fields {#fields}

**Route name**

A list of Symfony routes' names. This list includes the CMS defined routes.

**Route parameters**

This field accepts a JSON containing attributes for the URL generation.

For example, this Symfony route:

<!-- config-tabs:start -->

```yaml
# config/routes.yaml
product_big_view:
    path: /product-big/{color}/view
```

```php
<?php

// config/routes.php
namespace Symfony\Component\Routing\Loader\Configurator;

return static function (RoutingConfigurator $routes): void {
    $routes->add('product_big_view', '/product-big/{color}/view');
};
```

<!-- config-tabs:end -->

requires a "color" parameter. We can define its value with *{"color": "yellow"}*.

## Options {#options}

### Attribute allow-list {#restrictpatterns}

Limit the picker to routes that define a particular default attribute. The implementation checks whether the key exists, not whether its value is true. When several keys are supplied, a route is included if it defines any one of them.

Every CMS has a *_sfs_cms_reference* default value defined to be used as filter.

You can flag some routes with a default value to use as filter, for example

<!-- config-tabs:start -->

```yaml
# config/routes.yaml
my_route:
     path: /example
     defaults:
         use_for_some_content: true
```

```php
<?php

// config/routes.php
namespace Symfony\Component\Routing\Loader\Configurator;

return static function (RoutingConfigurator $routes): void {
    $routes->add('my_route', '/example')->defaults(['use_for_some_content' => true]);
};
```

<!-- config-tabs:end -->

Then you can use this value as option in your modules:

```yaml
module:
     module_options:
         form_fields:
             button_link:
                 type: "symfonyRoute"
                 type_options:
                     restrict_default_attribute: "use_for_some_content"
```

Or combine with some others:

```yaml
module:
     module_options:
         form_fields:
             button_link:
                 type: "symfonyRoute"
                 type_options:
                     restrict_default_attribute:
                         - "_sfs_cms_reference"
                         - "use_for_some_content"
                         - "use_for_alternative_content"
```

### Exclude route names {#restrictdefaultattribute}

Also routes can be filtered by patterns using restrict_patterns option.

The default exclusions are `^_profiler`, `^_wdt`, `^admin_`, and `^sfs_.*admin`. Setting `restrict_patterns` replaces that list. Patterns are matched against route names and are wrapped in `/` delimiters by the form type.

You can set your own patterns:

```yaml
module:
     module_options:
         form_fields:
             button_link:
                 type: "symfonyRoute"
                 type_options:
                     restrict_patterns:
                         - "^_profiler"
                         - "^_wdt"
                         - "admin"
                         - "^skip_route$"
```

Of course, you can combine both restriction methods ("restrict_default_attribute" and "restrict_patterns").

## Use the value {#use-the-value}

This type returns an array with both fields:

```php
    $form->get('route')->getData() => [
        'route_name' => 'product_big_view',
        'route_params' => [
            'color' => 'yellow'
        ]       
    ]   
```

In a module render template, use the CMS URL helper so CMS references are resolved as well as application routes:

```twig
{% if button_link.route_name|default(false) %}
    {{ sfs_cms_url(button_link) }}
{% endif %}
```

or with PHP:

```php
    $this->router->generate($route['route_name'], $route['route_params']);
```
