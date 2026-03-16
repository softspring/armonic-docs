---
title: "Configure CMS Content Types"
description: "Define Armonic CMS content types with YAML configuration, admin form fields, rendering rules, and entity mapping."
---

# Content Configuration (`app/cms/contents`)

This folder holds the configuration for each content type used in the CMS.  
Each content type lives in its own subfolder (e.g. `project`, `article`, etc.) and is defined using a `config.yaml` file.

These YAML files describe how the content behaves:
- Which fields it has
- How it's edited in the admin panel
- How it’s rendered

---

## Example: `app/cms/contents/project/config.yaml` {#example-appcmscontentsprojectconfigyaml}

```yaml
content:
    revision: 1
    entity_class: 'App\Entity\Cms\ProjectContent'
    extra_fields:
        title:
            type: 'translation'
        client:
            type: 'text'
        description:
            type: 'translation'
            type_options:
                type: 'textarea'
        image:
            type: 'media'
            type_options:
                media_attr:
                    class: 'img-fluid'
                show_thumbnail: true
                media_types:
                    project_image:
                        pictures: _default
    admin:
        create:
            view: '@content/project/admin/create.html.twig'
        update:
            view: '@content/project/admin/update.html.twig'
        list:
            page_view: '@content/project/admin/list-page.html.twig'
```

## Field Reference {#field-reference}

### `content` {#content}

- **`revision`**  
  Used to force cache or schema rebuilds when the structure changes.

- **`entity_class`**  
  *Optional.* Fully qualified class name of the Doctrine entity.  
  Only needed if you want to store the content in a custom database table.  
  If omitted, the system uses default storage.

- **`extra_fields`**  
  Defines custom fields for this content type.

---

### `extra_fields` {#extrafields}

These are examples of fields you can define for your content type, but don't need to be the same we are using here. 
In our case, for example, we have "client" as a text field to define the client name for this project, and show
it in a special way in the frontend. For example, in this page: https://softspring.es/en/works we show the client name in each card, below the image.

- **`title`**
    - `type: translation` — Multilingual field.

- **`client`**
    - `type: text` — Simple text field.

- **`description`**
    - `type: translation` — Multilingual.
    - `type_options.type: textarea` — Shows a textarea in the admin UI.

- **`image`**
    - `type: media` — Image/media field.
    - `type_options`:
        - `media_attr.class`: Adds CSS class (`img-fluid`) on frontend.
        - `show_thumbnail`: Shows a thumbnail preview in the admin.
        - `media_types.project_image.pictures`: Restricts media to a specific type/style (`_default`).

---

### `admin` {#admin}

Defines which templates to use in the admin panel:

- `create.view`: Template for the creation form.
- `update.view`: Template for the edit form.
- `list.page_view`: Template for the content list page.

---

## Adding New Content Types {#adding-new-content-types}

To create a new content type:

1. Create a new folder inside `app/cms/contents`, e.g. `article`.
2. Add a `config.yaml` file with the structure shown above.
3. Define only the fields and templates you need — sensible defaults will apply for anything not defined.
4. You **don’t need to create a custom entity** unless you want to store the content in a separate database table.  
   The system handles generic content storage by default.

This structure is **flexible** and **minimal**. You only configure what’s specific to your content — the system takes care of the rest.
