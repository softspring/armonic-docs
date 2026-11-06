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
