---
title: "CMS Module Preview Texts"
description: "Show live text previews inside module edit templates using form values and editable placeholders in the CMS admin."
---

# Preview texts {#preview-texts}

These examples extend an existing module definition. Its `edit_template` receives the Symfony form view, while `form_template` renders the inputs. Load the CMS admin JavaScript to synchronize inline editing.

```yaml
module:
    module_options:
        form_fields:
            title:
                type: 'text'
```

You can preview it adding to the edit template:

```twig
<h1>{{ form.title.vars.data|default('') }}</h1>
```

It's recommended to be conservative, to prevent errors, so add a *default* filter.

**Using placeholders**

```twig
<h1 data-edit-content-placeholder="Write the title!">{{ form.title.vars.data|default('') }}</h1>
```

**Content editable**

```yaml
module:
    module_options:
        form_fields:
            title:
                type: 'text'
                type_options:
                    attr:
                        data-edit-content-input: 'title'
```

```twig
<h1 contenteditable="true" data-edit-content-target="title">{{ form.title.vars.data|default('') }}</h1>
```

> Important: if you use a *display block* element, when you add a new line it will append a &lt;div&gt; tag.
> Use a inner *inline-block* element to prevent this behaviour.  

> Keep Twig escaping for plain text. Only render `raw` for fields explicitly intended to store trusted editorial HTML. Inline editing writes HTML into its linked field; use the normal form input when a field must remain plain text.

```twig
<h1><span contenteditable="true" data-edit-content-target="title">{{ form.title.vars.data|default('') }}</span></h1>
```

**Work with translated texts**

```yaml
module:
    module_options:
        form_fields:
            title:
                type: 'translatable'
                type_options:
                    type: 'text'
                    type_options:
                        attr:
                            data-edit-content-input: title
```

```twig
{% for locale,localeField in form.title.vars.localeFields %}
    <h1 data-lang="{{ locale }}" contenteditable="true" data-edit-content-target="title.{{ locale }}">{{ localeField.vars.data|default('') }}</h1>
{% endfor %}
```
