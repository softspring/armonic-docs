---
title: "Configure CMS Blocks"
description: "Define Armonic CMS blocks with YAML configuration, rendering mode, cache behavior, and form fields."
---

# Block Configuration (`cms/blocks`)

Each block type is configured in:

- `cms/blocks/<block_name>/config.yaml`

The schema is defined in `cms-bundle/src/Config/Model/Block.php`.

## Minimal Example

```yaml
block:
    revision: 1
```

This is the only required key. Everything else has defaults.

## Practical Examples

Static singleton block rendered through a route:

```yaml
block:
    revision: 1
    static: true
    singleton: true
    esi: true
    render_url: user_dropdown
```

Configurable (non-static) block with form fields:

```yaml
block:
    revision: 1
    render_template: '@block/cta/render.html.twig'
    form_template: '@block/cta/form.html.twig'
    form_fields:
        title:
            type: text
        buttonLabel:
            type: text
        buttonUrl:
            type: url
```

## Option Reference

Top-level `block` keys:

- `revision` (required, integer)
- `render_template` (optional, default: `@block/<block_name>/render.html.twig`)
- `form_template` (optional)
- `esi` (optional, default: `true`)
- `ajax` (optional, default: `false`)
- `isolate_request` (optional, `true|false|null`, default: `null`)
- `cache_type` (optional, `public|private`, default: `public`)
- `cache_ttl` (optional, integer, default: `false`)
- `singleton` (optional, default: `true`)
- `static` (optional, default: `false`)
- `schedulable` (optional, default: `false`)
- `render_url` (optional)
- `form_fields` (optional map of fields)

### `form_fields`

Each field entry supports:

- `type` (required)
- `type_options` (optional map)

Example:

```yaml
form_fields:
    heading:
        type: translation
    columns:
        type: choice
        type_options:
            choices:
                '2 columns': 2
                '3 columns': 3
```

## Validation Rules (Important)

The configuration model enforces these constraints:

1. If `static: true`, then `singleton` must also be `true`.
2. If `static: true`, `form_fields` cannot be defined.
3. If `static: true`, `form_template` cannot be defined.
4. If `static: true`, `schedulable` cannot be `true`.
5. `isolate_request` can only be set when at least one of `esi` or `ajax` is enabled.
6. You cannot enable both `esi: true` and `ajax: true` at the same time.

## How To Choose Options

- Use `static: true` for blocks that have no per-instance editable data.
- Keep `singleton: true` when the block should exist only once globally.
- Use `form_fields` only for non-static blocks that editors need to customize.
- Use `render_url` when rendering should be delegated to a Symfony route/controller.
- Use `cache_type` and `cache_ttl` to control cache behavior for expensive blocks.
- Use `isolate_request: true` only when you need to avoid user-session context in sub-requests.

## Notes

- `revision` is typically increased when block structure/behavior changes and migration logic is needed.
- Most projects start with defaults and only override values that are functionally necessary.
