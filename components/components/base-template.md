---
title: "Softspring Base Template"
description: "Reference the base Twig template provided by Softspring Components, including layout blocks, SEO, and app metadata."
---

# Base template

The *base.html.twig* template is aimed to provide a structured template for most purposes. 

It defines the basic HTML structure, includes most common header blocks, and provides basic styles and JS code.

## Basic Usage

If you want to use this base template as default project template, you should extend it from your project *base.html.twig* 
 template, witch is the most common base template in Symfony projects.

```twig
{# templates/base.html.twig #}
{% extends '@SfsComponents/base.html.twig' %}
```

## Three level templates

Symfony recommends using a three level template system: base -> layout -> content. See 
 [template inheritance and layouts](https://symfony.com/doc/current/templates.html#template-inheritance-and-layouts) section.

The **base** template (can be this *@SfsComponents/base.html.twig* template :D) provides the HTML structure,
and the required twig blocks.

You can override this template if you want to define your own base template or include some basic things:

```twig
{# bundles/SfsComponentsBundle/base-bootstrap5.html.twig #}
{% extends '@!SfsComponents/base-bootstrap5.html.twig' %}

{% block seo %}
   {{ parent }}
    <meta name="description" content="{% block description %}{% endblock description %}" />
{% endblock seo %}
```

> It would be the same if you rewrite *base.html.twig* insead of *base-bootstrap5.html.twig*, but it could
> conflict with theming. 

The **layout**, that extends the base template, can include the visual page structure, such as navbars and menus
and provides the layout for the contents.

```twig
{# page-layout.html.twig #}
{% extends '@SfsComponents/base.html.twig' %}

{% block body %}
   <nav>
      ...
   </nav>
   
   <div class="container-fluid">
      {% block content %}
      {% endblock content %}
   </div>
   
   <footer>
      ...
   </footer>
{% endblock body %}
```

The content template is the one that contains the specific page content.

```twig
{# shop/product/details.html.twig #}
{% extends 'page-layout.html.twig' %}

{% block title %}{{ product.name }}{% endblock title %}
{% block description %}{{ product.description }}{% endblock description %}

{% block content %}
     <div class="product">
        ...
     </div>
{% endblock content %}
```


## Header blocks

This template includes most common HTML header parts, defined in different extensible blocks.

For example, the *viewport* block includes this code:

```twig
{% block viewport %}
    <meta name="viewport" content="width=device-width, initial-scale=1"/>
    <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
{% endblock viewport %}
```

You can extend it using *{{ parent() }}*:

```twig
{# templates/base.html.twig #}
{% extends '@SfsComponents/base.html.twig' %}

{% block viewport %}
    {{ parent() }}
    <!-- add your own code -->
{% endblock viewport %}
```

or fully override it, or *disable* it without including any code:

```twig
{# templates/base.html.twig #}
{% extends '@SfsComponents/base.html.twig' %}

{% block viewport %}
    <!-- set your own code or leave it empty -->
{% endblock viewport %}
```

### head_begin

This is the first part of the *<head>* tag, it's useful to include some code you want to be at the 
 really begining of the HTML code.

By default, it does not include any code.

### viewport

This block is aimed to include the *viewport* configuration code. 

By default, it contains:

```html
<meta name="viewport" content="width=device-width, initial-scale=1"/>
<meta http-equiv="X-UA-Compatible" content="IE=edge"/>
```

### seo

The *seo* block is used to include every *SEO* related code, such as title, description, keywords, etc.

By default, it defines the page *title* tag, including another :

```twig
<title>{{ title_prefix|default('') }}{% block title %}Welcome!{% endblock %}</title>
```

In any template, you can set your page title overriding *title* block:

```twig
{% block title %}You own page title{% endblock title %}
```

And usually you will want to include more *SEO* info:

```twig
{% block seo %}
    {{ parent() }} {# this will draw title block #}
    <meta name="description" content="Your page description goes here" />
    <!-- ... -->
    <meta name="og:title" property="og:title" content="You own page title" />
    <!-- ... -->
{% endblock seo %}
```

See above [variables section](#title_prefix) to know about *title_prefix* variable.

### title

Explained before :)

### favicon

You should override the favicon block, because by default it includes a dot.

```twig
{% block favicon %}
<link rel="shortcut icon" href="/favicon.ico" />
<link rel="apple-touch-icon" href="/images/apple-touch-icon.png" />
{% endblock favicon %}
```

### extra_headers

This block is general purpose. You can include here anything you need to include in HTML head.

By default, it includes two configurable parts: *generator* meta tag and *theme color* for some devices.

See [app_version](#app_version) and [app_color](#app_color) variables.

### stylesheets

This block should contain every style related code.

By default it includes CDN base styles (for example bootstrap5).

It also includes *encore* links:

```twig
{{ encore_entry_link_tags('app') }}
```

Of course, you should override it to include your own styles and code:

```twig
{% block stylesheets %}
    {{ encore_entry_link_tags('website') }}
{% endblock %}
```

### header_javascripts

As well as the template defines styles block, it includes some javascript blocks.

This *header_javascripts* block includes the JS that must be included into *<head>* tag.

There is a lot of controversy of where JS code should be included, so keep in mind that this block exists and use
 it if you want.

By default, it sets a cookie called *utz* with the user's timezone, and it includes the *encore_entry_script_tags*.

```twig
{% block header_javascripts %}
    <script type="text/javascript">document.cookie = "utz=" + (new Date()).toString().split(' ')[5] + ';path=/;secure=true'</script>
    {{ encore_entry_script_tags('app') }}
{% endblock %}
```

## Body blocks

### body_begin

Use this block to include some code you want to be located at the really begining of the *<body>* tag.

### body

This block is aimed to be the **main content block** in the template.

### javascripts

This block includes javascript code after content. As said before, use it as you consider.

By default, it includes CDN base javascript (for example bootstrap5).

### body_end

This block is located at the really ending position of the *<body>* tag.

You can include here anything you want to be at the end of the HTML code.

## Variables

There are some variables defined, to configure some parts of the base template.

### html_classes

This variable is used to include *classes* into the *<html>* tag.

```twig
<html lang="{{ app.request.locale }}" class="{{ html_classes|default('') }}">
```

Use it to include some of them:

```twig
{% set html_classes = 'catalog' %}
{% extends '@SfsComponents/base.html.twig' %}
```

### title_prefix

This variable is included before the *title block* in the *seo block*.

See its code:

```twig
<title>{{ title_prefix|default('') }}{% block title %}Welcome!{% endblock %}</title>
```

This is useful to define some common page prefix, for example to discriminate environments:

```yaml
# config/packages/twig.yaml
twig:
    globals:
        title_prefix: '%env(ENVIRONMENT_TITLE_PREFIX)%'
```

### app_version

This *app_version* variable allows to include your application version.

```twig
{% if app_version|default(false) %}
    <meta name="generator" content="{{ app_version }}" />
{% endif %}
```

You can define it in an environment variable:

```yaml
# config/packages/twig.yaml
twig:
    globals:
        app_version: '%env(APP_VERSION)%'
```

### app_color

In some devices, such as mobile phones and tablets, the browser shows a color for your site. This *app_color* variable
 is aimed to define it for this purpose.

```twig
{% if app_color|default(false) %}
    <meta name="msapplication-TileColor" content="#{{ app_color }}"/>
    <meta name="theme-color" content="#{{ app_color }}"/>
{% endif %}
```

```yaml
# config/packages/twig.yaml
twig:
    globals:
        app_color: '2277dd'
```

### body_classes

As well as *html_classes* there is a *body_classes* variable to include *classes* into the *<body>* tag.

```twig
<body class="{{ body_classes|default('') }}">
```

Use it to include some of them:

```twig
{% set body_classes = 'catalog' %}
{% extends '@SfsComponents/base.html.twig' %}
```
