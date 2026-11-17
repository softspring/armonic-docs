---
title: "CMS Bundle Configuration"
description: "Understand the project-level CMS directory structure and how blocks, contents, layouts, menus, modules, and sites are defined."
---

# Organize CMS configuration {#configuration}

CMS definitions live under `cms/` in the Symfony project directory. If the application is inside an `app/` directory, its definitions are under `app/cms/`; the directory name is relative to `kernel.project_dir`.

## Directory structure {#configuration-structure}

```text
cms/
├── blocks/
├── contents/
├── layouts/
├── menus/
├── modules/
└── sites/
```

Each element has an identifier directory containing `config.yaml` and any required Twig templates or translations. For a hero module:

```text
cms/modules/hero/
├── config.yaml
├── render.html.twig
└── translations/
    └── sfs_cms_modules.en.yaml
```

Add `edit.html.twig` and `form.html.twig` when the module needs a custom editor preview or field layout. They are not mandatory for every dynamic module.

See the configuration guides for [sites](../../configuration/configure-sites.md), [content types](../../configuration/configure-content-types.md), [layouts](../../configuration/configure-layouts.md), [modules](../../configuration/configure-modules.md), [blocks](../../configuration/configure-blocks.md), and [menus](../../configuration/configure-menu.md).

## Override a supplied definition {#configuration-overriding}

CMS collections are loaded in this order:

1. The bundle's built-in collection.
2. Configured external collections.
3. The project's `cms/` directory.

Later definitions with the same identifier override earlier settings according to their configuration schema. Some lists, including site hosts and paths, replace the earlier list rather than appending to it.

For example, `cms/sites/default/config.yaml` customizes the supplied `default` site. A directory with a different identifier creates another site instead.

Twig namespaces follow the same collection precedence, so a project can override a supplied template without editing files in `vendor/`.

## Apply changes {#apply-changes}

Clear the application cache after changing definitions:

```bash
php bin/console cache:clear
```

Open the affected admin screen, create or edit a record, and preview its output. When changing stored data formats, provide the relevant migration instead of relying on a revision number alone. Recompile and review published content after changing render templates.

Fixtures and data import are separate from element definitions. Use the application's documented fixture workflow; the core package does not automatically create a demo website when a configuration directory is added.

## Describe intended use for AI assistants {#ai-instructions}

Store usage guidance in the generic `meta` map under `ai_instructions` to explain its purpose, preferred uses, composition rules
and alternatives. Use a multiline YAML string for longer guidance. The same metadata convention applies across elements:

| Element | Configuration key |
| --- | --- |
| Module | `module.meta.ai_instructions` |
| Content type | `content.meta.ai_instructions` |
| Layout | `layout.meta.ai_instructions` |
| Content or layout container | `content.meta.containers.<id>.ai_instructions` or `layout.meta.containers.<id>.ai_instructions` |
| Menu type | `menu.meta.ai_instructions` |
| Block type | `block.meta.ai_instructions` |
| Site | `site.meta.ai_instructions` |
| Media type | `sfs_media.types.<id>.meta.ai_instructions` in application package configuration |

For example, this is an override fragment for an existing hero module supplied by a collection:

```yaml
# cms/modules/hero/config.yaml
module:
    meta:
        ai_instructions: |
            Use for a prominent introduction to a page or major section.
            Prefer a visually strong background image and a concise heading.
            Keep text readable and respect the selected media type requirements.
```

For an existing paragraph module, describe the editorial preference explicitly:

```yaml
# cms/modules/paragraph/config.yaml
module:
    meta:
        ai_instructions: |
            Use one instance per ordinary paragraph.
            Prefer this module over wysiwyg for normal body copy.
            Use wysiwyg when the content requires richer HTML structure.
```

An override fragment inherits the original definition's revision, form and templates. New definitions still need
all normally required fields. Guidance can be written in any language and is returned as configured; it is not a
translation key. Omitting it preserves existing behavior. Changing guidance does not change stored module data
or require a data revision migration; clear the application cache to reload definitions.

CMS definition loading supplies generic metadata through `MetadataExtension`, without adding AI-specific nodes
to the definition models or replacing an existing `meta` schema. Metadata remains open to other application keys.
Container guidance belongs under the content or layout definition's `meta.containers` map.

The shared CMS serializers expose configured text as `aiInstructions`, even when raw configuration is not
requested. Module and container form views also carry their guidance for schema/prototype consumers. The core
configuration feature does not require the AI or MCP plugins. See [CMS operation guidance](../../components/cms-api-plugin.md#element-instructions)
for its use through the API and MCP, and [media guidance](../media-bundle.md#ai-instructions) for media configuration.

Describe intended use and meaningful constraints. These strings are returned to assistant clients; keep them
focused on editorial and composition guidance. They do not grant permissions, enable disabled modules, change
allowed module lists, relax form validation or authorize publication.
