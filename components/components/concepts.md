# Concepts

This package provides some template *components* for Symfony projects.

## Structure

The are some components provided by now, like flash-messages and paginator, and some base templates like *base*
 and *layout/admin*.

All of them have a main template and some secondary templates for theming.

You can include or extends those templates, depending on their aim.

For *base* and *layouts* usualy you require to extend them:

```twig
{% extends '@SfsComponents/base.html.twig' %}
```

In the case of *components* you require to include them:

```twig
{% include '@SfsComponents/paginator/table.html.twig' %}
```

## Theming

This package is prepared to contain different template themes, in spite of now it just provides bootstrap5 theme.

You can configure the default theme with a twig global called "sfs_components_theme".

```yaml
# config/packages/twig.yaml
twig:
    globals:
        sfs_components_theme: 'bootstrap5'
```

The default value is 'bootstrap5'.
