---
title: "CMS Bundle Concepts"
description: "Learn the core concepts behind the Softspring CMS Bundle, including sites, layouts, contents, routes, and modules."
---

# CMS concepts {#concepts}

Armonic separates developer-defined structures from editor-managed records. Developers configure which elements exist; editors combine those elements into content and choose when it becomes public.

## Sites {#sites}

A site defines public hosts or paths, locales, and allowed content types. An application can have one site or several. Content and routes are associated with the sites where they are available.

For example, one application can serve a main website and a separate campaign site, with different languages and page types. A site definition does not automatically create its pages or translate their text.

## Content and versions {#contents}

A content record holds its identity and metadata. A content version holds the editable page composition and the layout used to render it. Publishing selects the version shown publicly, while drafts can continue to be edited separately.

The built-in `page` type covers ordinary website pages. A custom content type supplies a concrete Doctrine entity when content needs domain-specific relations or behavior, such as an article linked to an author.

See [Configure content types](../../configuration/configure-content-types.md) and [Preview and publish](../../user-manual/preview-and-publish-a-page.md).

## Layouts and containers {#layout}

A layout defines the page structure and its editable areas, called containers. For example, a landing layout can expose `main` and `sidebar` containers while its header and footer remain controlled by Twig templates.

A container can restrict which modules editors may insert. This lets developers preserve the design while giving editors control over the content.

## Modules {#modules}

A module is an editable element inside a container: text, an image, a grid, or a project-specific feature. Its configuration defines form fields; its Twig template renders their values.

Each module instance normally belongs to its page version. Two pages using the same module type can have different text and images. See [Configure modules](../../configuration/configure-modules.md).

## Shared blocks and sections {#blocks}

A block is a reusable fragment managed outside a page version. It can have editable fields, or it can render static or controller-provided output. Pages can refer to the same block record so its content is maintained once.

The optional sections plugin provides reusable compositions with their own module editor, versions, and publishing workflow. Choose a section when the shared fragment needs that page-like editing model.

Changing a shared record can affect several pages. Republishing an older page version does not automatically roll back shared blocks, media files, or other records changed independently.

## Routes {#routes}

A CMS route connects an internal identifier to locale-specific public paths. It can render content, redirect to another route or URL, or group child paths.

The route identifier used by a menu or Twig helper is distinct from the public path. For example, identifier `about` can resolve to `/en/about` on one site and another localized path elsewhere.

## Menus {#menus}

A menu stores editable navigation items. A developer places its rendering helper in a layout; editors maintain labels, destinations, and ordering. Creating a menu record does not place it on every page automatically.

## Rendering and caching {#rendering-contents}

Published content is rendered using its layout and module templates. Compiled output can be stored for later requests. ESI and AJAX fragments allow shared elements to be rendered independently when the application is configured for them.

After changing templates or module definitions, verify both previews and published output. Clearing Symfony's container cache does not necessarily replace compiled CMS content or invalidate an HTTP cache.
