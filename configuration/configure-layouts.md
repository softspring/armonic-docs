---
title: "Configure CMS Layouts"
description: "Define page layouts in Armonic with layout config, editable containers, render templates, and translation files."
---

# Configure layouts {#configure-layouts}

A layout defines the editable areas of a page and the HTML used to render them. For example, a landing page can have a header area for CSS or metadata and a main area for editorial content.

Create these files under `cms/layouts/landing/`:

```text
cms/layouts/landing/
├── config.yaml
├── edit.html.twig
└── render.html.twig
```

The directory name is the layout identifier. Use the same container names in the configuration and both templates.

## Define the containers {#define-containers}

```yaml
# cms/layouts/landing/config.yaml
layout:
    revision: 1
    containers:
        headers:
            allowed_modules: ['html_headers', 'css']
        main: ~
```

`headers` accepts only the two named module types. `main: ~` leaves that container unrestricted. Restricting the available modules helps editors choose elements appropriate to each part of the page.

You can add other container names, such as `sidebar`. Each container can define `required` and `allowed_modules`. Module identifiers must match definitions available in your CMS collections.

## Render the published page {#render-page}

The default frontend template is `@layout/landing/render.html.twig`, which resolves to `cms/layouts/landing/render.html.twig`:

```twig
{# cms/layouts/landing/render.html.twig #}
<!doctype html>
<html lang="{{ app.request.locale }}">
<head>
    <meta charset="UTF-8">
    <title>{{ version.seo.metaTitle|default('')|sfs_cms_trans }}</title>
    <meta name="description" content="{{ version.seo.metaDescription|default('')|sfs_cms_trans }}">
    {{ containers.headers|default('')|raw }}
</head>
<body>
    <main>
        {{ containers.main|default('')|raw }}
    </main>
</body>
</html>
```

`containers` contains the rendered HTML for each area. Use `raw` for this CMS-generated HTML; keep Twig's escaping enabled for ordinary text fields. Only trusted editors should have access to modules that accept arbitrary HTML or JavaScript.

In an application with an existing design, extend your own base template and place the containers in its blocks. Load your site's CSS and JavaScript through the asset system used by that application.

## Provide the editor template {#editor-template}

The editor needs form rows for the same containers:

```twig
{# cms/layouts/landing/edit.html.twig #}
{% block body %}
    <section aria-label="Page headers">
        {{ form_row(form.data.headers) }}
    </section>
    <main>
        {{ form_row(form.data.main) }}
    </main>
{% endblock %}
```

These fields let editors insert, reorder, and configure modules. Keep the editor structure close to the frontend structure so the available areas are recognizable.

## Make the layout available to pages {#make-layout-available}

Add the layout identifier to the relevant content type's `allowed_layouts`, and set `default_layout` if new content should use it. See [Configure content types](configure-content-types.md).

Clear the application cache after adding the files:

```bash
php bin/console cache:clear
```

Open a page in the CMS, select the layout, add a module to `main`, and preview it. If you change a template used by published content, recompile and verify the published output; clearing Symfony's cache alone does not replace stored compiled HTML.

## Layout options {#layout-options}

| Option | Purpose |
| --- | --- |
| `revision` | Required integer identifying the layout configuration revision. |
| `enabled` | Makes the layout available; defaults to `true`. |
| `render_template` | Frontend Twig template; defaults to `@layout/<id>/render.html.twig`. |
| `edit_template` | Editor Twig template; defaults to `@layout/<id>/edit.html.twig`. |
| `compatible_contents` | Restricts the content types that can use this layout. |
| `save_compiled` | Controls storage of compiled layout output; defaults to `true`. |
| `containers` | Named editable areas, with optional module restrictions. |
