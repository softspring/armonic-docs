---
title: "Using CMS blocks"
description: "Choose static, editable, singleton, and repeated blocks and render them from Symfony Twig templates."
related: ["configure-blocks", "bundles/cms-bundle/content-versions"]
---

# Using blocks {#using-blocks}

Blocks hold content shared across pages. Use one for contact details or a campaign call to action that editors should update once. Define its fields in `cms/blocks/<type>/config.yaml`, then create the editable record in the admin.

Start with the complete [call-to-action example](../../configuration/configure-blocks.md). A block's fields become variables in its render template; they are not page module fields.

## Render a singleton {#render-a-singleton}

```twig
{{ sfs_cms_block_by_type('cta') }}
```

The type must exist and, for an editable block, its record must have been created. A singleton means one record for that type. It does not mean one copy of its HTML on the entire site.

## Render one of several records {#render-one-of-several-records}

Set `singleton: false` when editors need several records of one type. Select the specific record instead of relying on a type-only lookup:

```twig
{% set block = sfs_cms_block_find({id: selectedBlockId}) %}
{% if block %}
    {{ sfs_cms_block(block) }}
{% endif %}
```

Here `selectedBlockId` is a block identifier supplied by the application or the selected CMS field. A module can store the selection with a `blockInstance` field:

```yaml
module:
    module_options:
        form_fields:
            shared_block:
                type: blockInstance
```

The submitted model value is a block object. In that module's render template, pass it directly to `sfs_cms_block(shared_block)`. Use `blockStatic` to choose a static block type instead.

## Choose a rendering mode {#choose-a-rendering-mode}

Direct rendering (`esi: false`, `ajax: false`) is the simplest starting point. ESI lets a compatible HTTP cache request the fragment separately. AJAX loads the fragment in the browser and needs the CMS frontend JavaScript.

ESI and AJAX cannot both be enabled on one block. `isolate_request` applies only to those fragment modes. Keep public fragments independent of session or account data; a shared cache must not reuse one visitor's private output for another.

## Static and scheduled blocks {#static-and-scheduled-blocks}

A static block has template content and no editable record fields. It must be a singleton and cannot declare a form or scheduling. Editable block types can enable `schedulable` when their records should be available only during configured periods.

Changes to a shared block can affect several pages without a new page version. Preview its usages before changing or deleting it, and check compiled output and fragment caching when a public page still shows the previous value.
