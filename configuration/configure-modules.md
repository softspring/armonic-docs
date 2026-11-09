---
title: "Configure CMS Modules"
description: "Define Armonic CMS modules with YAML configuration, templates, module type, and module options."
---

# Configure modules {#configure-modules}

Modules are the editable elements inside a layout container. A module combines a form for its data and a Twig template for its output. Editors can reuse the same module type in different pages, with different values.

This example adds a hero heading and subtitle. Create `cms/modules/hero/config.yaml` and `cms/modules/hero/render.html.twig`.

## Define the form fields {#define-form-fields}

```yaml
# cms/modules/hero/config.yaml
module:
    revision: 1
    group: content
    compatible_contents: ['page']
    module_options:
        form_fields:
            title:
                type: translation
            subtitle:
                type: text
                type_options:
                    required: false
```

The default `module_type` is `Softspring\CmsBundle\Form\Module\DynamicFormModuleType`. It builds the fields in `module_options.form_fields`. `translation` stores a value per language; `text` stores a single text value.

Use [dynamic form types](../components/dynamic-form-type.md) for additional field types and validation options. CMS-specific types include [media selection](../bundles/cms-bundle/module/form-types/media.md), [route selection](../bundles/cms-bundle/module/form-types/symfony-route.md), and [translatable fields](../bundles/cms-bundle/module/form-types/translatable.md).

## Render the values {#render-values}

```twig
{# cms/modules/hero/render.html.twig #}
<section class="hero">
    <h1>{{ title|default('')|sfs_cms_trans }}</h1>
    {% if subtitle|default('') %}
        <p>{{ subtitle }}</p>
    {% endif %}
</section>
```

Module data fields are available directly as Twig variables. Keep normal escaping for text. For rich HTML fields, only render unescaped output when that field is intended to contain trusted editorial HTML.

The default template path is `@module/hero/render.html.twig`. Set `render_template` only when you use a different path. Optional `edit_template` and `form_template` let you customize the in-editor preview and field layout.

## Make the module available {#make-module-available}

Clear the cache after adding the files:

```bash
php bin/console cache:clear
```

Open a page using a compatible content type and insert **hero** into a container that allows it. Enter the title, save a draft, and preview the page. Check every language enabled for that page before publishing.

If the module is missing from the picker, check `enabled`, `compatible_contents`, and the container's `allowed_modules`. The `group` setting organizes the picker; it does not grant permissions.

## Evolve a module safely {#evolve-module}

Keep `revision` as an integer and increase it when the saved data format changes. Existing module instances keep their stored data. A revision number alone does not rename or convert fields; provide a `migrate.php` function when a conversion is needed. See [CMS modules](../bundles/cms-bundle/modules.md) for the rendering context and data migration API.

Use a custom Symfony form type in `module_type` when the dynamic fields cannot express your form. Its options belong in `module_options`; their validity is checked by that form type when the form is built.
