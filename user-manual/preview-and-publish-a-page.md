---
title: "Preview and Publish a Page"
description: "Review how to preview page changes, publish versions, and roll back to earlier published versions in Armonic CMS."
---

# Preview and publish a page {#previewing-and-publishing-a-page}

Editing creates or updates a content version. Publishing selects the version visitors will see. You can keep working on a draft while an earlier version remains public.

## Preview the current version {#preview-version}

1. Open the page's **Content** editor.
2. Open the arrow next to **Save version** and choose **Save and preview**.
3. Check the page's headings, images, links, shared blocks, and navigation.
4. Switch through the enabled languages and sites when your application exposes those choices.

Use **Edit version** to return to the editor if something needs changing. A preview can use editor-only controls and URLs; send visitors the public page URL after publishing.

## Publish the reviewed version {#publish-version}

Choose **Publish version** in preview, or **Save and publish** in the editor. The available actions depend on your permissions and project configuration.

Open the public route in a separate browser tab and verify the result. A page also needs a valid route for the selected site and locale. Publishing a version does not create a missing route or translate empty language fields.

If the public output is unchanged, confirm which version is published and check the application's cache behavior with a developer.

## Restore an earlier version {#restore-version}

Open the page details with **View**, then use the **Versions** tab. Inspect or preview the earlier version before publishing it again.

Restoring a version changes the page's published content. It does not automatically roll back shared block records, media files, routes, or configuration files changed outside that version.
