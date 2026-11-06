---
title: "Create a New Block in Armonic CMS"
description: "Learn how to create a block in Armonic CMS, define its content and settings, and prepare it for use in your pages."
---

# Create a block {#creating-a-new-block-in-armonic-cms}

A block stores reusable content such as a contact invitation. When several pages use the same block record, editors can update that shared content once.

A developer must first define the available block types. If the creation menu has no suitable type, see [Configure blocks](../configuration/configure-blocks.md).

## Choose a block type {#accessing-the-block-creator}

Open **CMS / Blocks**, then **New block** and choose a type. The screenshots use the contact call to action from [Build a small website](build-a-small-website.md).

![Blocks section in the CMS sidebar](.files/armonic-blocks-section.png){.img-fluid}

![New block menu with the configured block types](.files/armonic-new-block-button.png){.img-fluid}

The list is defined by your application. A singleton type allows one record; a reusable type can allow several records with different data.

## Fill in the content {#filling-block-basic-information}

Give the block a recognizable internal name, such as **Contact invitation**, and complete the fields supplied by its type. For the example call to action, enter a heading, message, and contact email.

![New block form with editable call-to-action fields](.files/armonic-new-block-form.png){.img-fluid}

Choose **Create**. Confirm that the new record appears in the list:

![Blocks list after creating the shared record](.files/armonic-new-block-list.png){.img-fluid}

## Use it in a page {#using-block-in-pages}

Open the page's content editor, insert **Block instance**, and select the record you created. Save, preview, and publish the page version.

Creating the record alone does not place it on a page. A developer can also render it directly from a Twig template.

When changing a shared block later, check every important page that uses it. Its edits are independent of page version history, so republishing an older page version does not restore an older block record.
