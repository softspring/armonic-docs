---
title: "CMS frontend and admin assets"
description: "Load the shared admin shell, form widgets, editor scripts, and published assets with Encore or AssetMapper."
related: ["components/components", "use-armonic-standalone-edition"]
---

# Frontend and admin assets {#frontend-and-admin-assets}

The CMS editor uses JavaScript and styles from several packages: the shared admin shell, dynamic collections, media selectors, translated fields, and CMS editing controls. Loading a page's HTML without its widgets does not complete the integration.

## Choose the application asset pipeline {#choose-the-application-asset-pipeline}

Armonic Standalone demonstrates an Encore integration. A Symfony application using AssetMapper consumes the bundles' published `assets/dist/` output where supported. Choose the approach already used by the application and load each entry once.

Package source files live in `assets/`; published browser files live in `assets/dist/` or, for packages that expose direct web assets, `public/`. Do not import editable SCSS as if it were compiled CSS.

## With Webpack Encore {#with-webpack-encore}

An application entry can import package source files from Composer's installed packages. For the shared shell:

```js
// assets/admin.js
import '../vendor/softspring/components/assets/layout/admin-shell.js';
```

```scss
// assets/admin.scss
@import '../vendor/softspring/components/assets/styles/admin.bootstrap5';
@import '../vendor/softspring/components/assets/sidebar/sidebar-pills.bootstrap5';
```

These entries are the shell only. Add the CMS editor, collection, media, translation, and optional plugin entries required by the forms you render. Use the [UI components guide](../../components/components.md) and Armonic Standalone's entries as the complete integration reference; copying only the shell does not enable the content editor.

## With AssetMapper {#with-assetmapper}

Register the bundle classes before building the asset map. Their extensions register published paths when AssetMapper is available. Use the importmap entries declared in each package's root `package.json`, such as `@softspring/components/admin-shell.js` and `@softspring/media-bundle/scripts/media-type.js`.

The application still needs to load its admin importmap entry and the published styles. Check missing imports in strict mode and load the relevant form theme. A registered path alone does not execute JavaScript in the page.

## Check the complete editor {#check-the-complete-editor}

Verify the sidebar, field rendering, adding/removing/reordering modules, media modal, translated fields, live preview, and save action. Inspect network failures and browser errors when one widget is absent.

When using compiled CMS output, a new asset build can leave old asset URLs in stored HTML. Recompile affected content after deploying template or entrypoint changes. See [Content versions and publication](content-versions.md).
