## Layout Configuration (`layout` section)

This configuration defines the layout structure of a page or content type.  
You can use it to control which types of modules can be inserted into different parts of the page.

There are several areas to configure: 
- Config file: `app/cms/layouts/my_layout/config.yaml` where "my_layout" is the name of the layout.
- edit.html.twig: The template that renders the layout when editing in the admin panel.
- render.html.twig: The template that displays the layout on the frontend.
- translations: Folder with translations for strings of text that might appear in the layout.

The config file can be very brief, as the one we use for articles:

```yaml
layout:
    revision: 1
    containers:
        main: ~
        header: ~
```

or can have some definition of allowed modules in the containers, like this:

```yaml
        headers:
            allowed_modules:
                - html_headers
                - css
```
see that in the first example we call the container "header" and in the second example we call it "headers", this is completely flexible, up to you. 
The important thing is that, afterwards, in the templates, you use the same name to render the container.

## Config reference:

---
## Field Reference

### `layout`

- **`revision`**  
  Used to force cache or layout rebuilds when the layout structure changes.

- **`containers`**  
  Defines named layout areas where content modules can be placed.

---

### `containers`

Each key under `containers` defines a named area in the layout.  
These areas correspond to sections in the HTML output (e.g. `headers`, `main`, `sidebar`, etc.).

For each container, you can specify:

- **`allowed_modules`**: list of module types that can be added to this container.  
  If you want to allow any type of module, set this to `~` (YAML null), as seen above.

---

## Template files

The twig layout files can be as complex as you like, but in general you will be adding some seo information in the header
and then rendering the containers in the body of the page, like this:

```
{% extends '@SfsComponents/base.html.twig' %}
{% block stylesheets %}
    {{ encore_entry_link_tags('sfs24') }}
{% endblock %}

{% block extra_headers %}
    {{ parent() }}
    {{ containers.headers|default('')|raw }}
{% endblock %}

{% block title %}{{ version.seo.metaTitle|default([])|sfs_cms_trans }}{% endblock %}

{% block seo %}
    {{ parent() }}
    <meta name="description" content="{{ version.seo.metaDescription|default([])|sfs_cms_trans }}"/>
    <meta name="keywords" content="{{ version.seo.metaKeywords|default([])|sfs_cms_trans }}"/>
    <meta name="robots" content="{{ version.seo.noIndex|default(false) ? 'noindex' : 'index' }}" />
...
{% endblock seo %}
```

This is a simplified version of one of our layouts, but includes some containers as extra_headers, stylesheets, and SEO metadata.

Then, in the body of the page, you can render the containers like this:

```
{% block body %}

    {% include 'partials/sfs24-header.html.twig' %}
    <main id="main" role="main">
        {% block container %}
            {{ containers.main|raw }}
        {% endblock container %}
    </main>
    {% include 'partials/sfs24-footer.html.twig' %}

{% endblock body %}
```

Here we include templates for the header and footer, and then we render the `main` container in the body of the page.




