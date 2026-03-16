---
title: "Configure CMS Modules"
description: "Define Armonic CMS modules with YAML configuration, templates, module type, and module options."
---

# Module Configuration (`cms/modules`)

Each module is configured in:

- `cms/modules/<module_name>/config.yaml`

The schema is defined in `cms-bundle/src/Config/Model/Module.php`.

## Minimal Example

```yaml
module:
    revision: 1
```

Only `revision` is required. Other keys are optional and have defaults.

## Practical Example

```yaml
module:
    revision: 1
    enabled: true
    group: 'content'
    render_template: '@module/hero/render.html.twig'
    edit_template: '@module/hero/edit.html.twig'
    form_template: '@module/hero/form.html.twig'
    compatible_contents: ['page', 'article']
    module_type: 'Softspring\CmsBundle\Form\Module\DynamicFormModuleType'
    module_options:
        label_format: 'modules.hero.%%name%%.label'
        form_fields:
            title:
                type: translation
            subtitle:
                type: text
```

## Option Reference

Top-level `module` keys:

- `revision` (required, integer)
- `enabled` (optional, boolean, default: `true`)
- `group` (optional, string, default: `default`)
- `render_template` (optional, default: `@module/<module_name>/render.html.twig`)
- `edit_template` (optional)
- `form_template` (optional)
- `compatible_contents` (optional, array of content type ids)
- `module_type` (optional, default: `Softspring\CmsBundle\Form\Module\DynamicFormModuleType`)
- `module_options` (optional, free-form map passed to the module type)

## What To Configure First

- Set `revision` for every module.
- Set at least `render_template` when you do not follow the default path.
- Use `group` to keep the module picker organized in admin.
- Use `compatible_contents` when a module should only appear in certain content types.
- Use `module_options` according to the `module_type` you chose (keys depend on that type).

## Notes

- `module_options` has no fixed schema at this level; each module type defines what it expects.
- If `enabled` is `false`, the module exists in config but is not available for usage.
- Increase `revision` when module data structure or behavior changes.
