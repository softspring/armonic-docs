---
title: "Configure CMS Blocks"
description: "Define Armonic CMS blocks with YAML configuration, rendering mode, cache behavior, and form fields."
---

# Configure blocks {#configure-blocks}

A block is a reusable fragment that can be rendered from several pages or templates. Editors update its data in one place. Use a block for a shared call to action, contact details, or another fragment that should remain consistent across the site.

Define a block type in `cms/blocks/<type>/config.yaml` and its frontend template in the same directory. A type defines the form and rendering rules; its editable record is created in the admin area.

## Define a call to action {#define-call-to-action}

```yaml
# cms/blocks/cta/config.yaml
block:
    revision: 1
    singleton: true
    esi: false
    form_fields:
        title:
            type: text
        buttonLabel:
            type: text
        buttonUrl:
            type: url
```

`singleton: true` allows one editable record for this type. `esi: false` renders the example directly, so it does not require an ESI-capable HTTP cache.

Create its template:

```twig
{# cms/blocks/cta/render.html.twig #}
<aside class="call-to-action">
    <h2>{{ title }}</h2>
    <a href="{{ buttonUrl }}">{{ buttonLabel }}</a>
</aside>
```

The fields are available as Twig variables. Keep text escaped. Use a full URL for `buttonUrl`; use a CMS link or route field when editors should choose an application route instead.

Clear the cache, open **Blocks**, and create the **cta** block. Enter its title, label, and URL, then save it:

```bash
php bin/console cache:clear
```

## Render the shared block {#render-block}

In a layout or another Twig template:

```twig
{{ sfs_cms_block_by_type('cta') }}
```

The helper finds the singleton record for that type. The default render template is `@block/cta/render.html.twig`. Use `render_template` when the file is elsewhere.

To let editors insert the block into a page, use the CMS block module. The [small website walkthrough](../user-manual/build-a-small-website.md) shows the complete editing flow.

## Static blocks {#static-blocks}

A static block has no editable per-record fields. For example:

```yaml
# cms/blocks/legal_notice/config.yaml
block:
    revision: 1
    static: true
    singleton: true
    esi: false
```

Create `cms/blocks/legal_notice/render.html.twig` with its content. Static blocks must be singletons and cannot declare `form_fields`, `form_template`, or scheduling.

If you set `render_url`, rendering is delegated to a Symfony route. The route must exist and return the fragment. Do not point it to the block's own render route, which would render recursively.

## Caching and request context {#caching-request-context}

Blocks support `esi`, `ajax`, `cache_type`, `cache_ttl`, and `isolate_request`. Choose these together with the application's HTTP cache setup:

- `esi` and `ajax` cannot both be enabled;
- `isolate_request` can only be set when ESI or AJAX is enabled;
- `cache_type: public` is for output that can be shared between users;
- user-specific output needs an appropriate private or uncached strategy;
- omit `cache_ttl` to keep its default rather than inventing a TTL.

A block containing account or session information must not be cached publicly as if it were shared content. See [HTTP cache](../components/cache.md) before enabling fragment caching.

## Change existing block data {#change-block-data}

`revision` identifies the data format. When changing the stored field structure, increase the revision and provide a compatible data migration where needed. Renaming a field in YAML does not rename values already saved in existing records.
