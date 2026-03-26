---
title: "Collection Form Type"
description: "Use JavaScript collection helpers for add, insert, move, duplicate, copy, paste, and delete interactions in Symfony form UIs."
---

# Collection Form Type {#collection-form-type}

`softspring/collection-form-type` is a browser-side helper for dynamic collection forms.

It does not provide a Symfony PHP form type. Its value is the JavaScript layer that turns collection markup into an interactive UI with actions such as add, insert, move, duplicate, copy, paste, and delete.

This is especially useful in complex admin forms where collection entries behave more like editable blocks than simple repeated inputs.

## Installation {#installation}

Install the Composer package:

```bash
composer require softspring/collection-form-type:^6.0
```

Then import the JavaScript in your asset build:

```js
import '@softspring/collection-form-type/scripts/collection-form-type';
```

This is how `cms-bundle` and `armonic` integrate it.

## When To Use It {#when-to-use-it}

Use this package when your form collection needs interactive behavior such as:

- add a new node at the end
- insert a node at a specific position
- move nodes up or down
- duplicate an existing node
- copy and paste nodes through the clipboard
- keep indexes and names synchronized after reordering

If you only need a plain Symfony `CollectionType` with simple add and remove buttons, this package is usually more than you need.

## DOM Contract {#dom-contract}

The package works through data attributes.

The core markers are:

- `data-collection="collection"` on the collection container
- `data-collection="node"` on each collection item
- `data-collection-action` on action buttons

The collection or the action element must also expose the prototype information through attributes such as:

- `data-prototype`
- `data-prototype-name`
- `data-collection-prototype`
- `data-collection-prototype-name`

The script reads those values and uses them to generate or insert new nodes.

## Supported Actions {#supported-actions}

The package listens to click actions with these values:

- `add`
- `insert`
- `delete`
- `up`
- `down`
- `duplicate`
- `copy`
- `paste`

Example:

```html
<button type="button" data-collection-action="add" data-collection-target="menu_items">
    Add item
</button>
```

## What Each Action Does {#what-each-action-does}

### Add {#action-add}

Adds a new node at the end of the collection.

Use it for the standard “append new item” flow.

### Insert {#action-insert}

Inserts a new node before a specific position.

This is useful when the UI needs insertion bars or “add here” controls between nodes.

### Delete {#action-delete}

Removes the current node and then reindexes the following nodes.

### Up And Down {#action-up-and-down}

Moves the current node in the collection and updates indexes, ids, and full names so the form stays consistent.

### Duplicate {#action-duplicate}

Clones the current node in place and then reindexes the following nodes.

This is useful for content blocks or repeated items that often start from an existing node.

### Copy And Paste {#action-copy-and-paste}

The package can serialize a node to clipboard-friendly JSON, then paste it back into the collection.

This is especially useful in CMS-like UIs where editors reuse a block structure instead of recreating it by hand.

## Lifecycle Events {#lifecycle-events}

The component is event-driven.

For each main action, it dispatches `before` and `after` events such as:

- `collection.node.add.before`
- `collection.node.add.after`
- `collection.node.insert.before`
- `collection.node.insert.after`
- `collection.node.delete.before`
- `collection.node.delete.after`
- `collection.node.duplicate.before`
- `collection.node.duplicate.after`

This is one of the main reasons to use the package: application code can hook into the lifecycle without forking the library.

## Event Context {#event-context}

The custom events expose helper methods to work with:

- collection
- node
- position
- prototype name
- prototype
- original browser event

That makes it possible to customize the default behavior during `before` events, for example:

- change the collection target
- change the insertion position
- replace the prototype HTML
- validate or reject clipboard data

## Copy And Paste Validation {#copy-and-paste-validation}

Copy and paste support uses a dedicated validation event:

- `collection.node.copy.prepare`
- `collection.node.paste.validate`

This gives the application a place to inspect or modify clipboard payloads before the paste happens.

That is useful when:

- only some collection payloads should be accepted
- the application wants to add metadata
- clipboard content should be transformed before insertion

## Index And Name Synchronization {#index-and-name-synchronization}

One of the important jobs of the package is keeping collection nodes consistent after structural changes.

When nodes move or duplicate, the script updates:

- `data-collection-index`
- node ids
- `data-full-name`
- matching references inside the node HTML

Without that step, Symfony collection submissions would quickly become inconsistent after reordering or duplication.

## Typical Integration Pattern {#typical-integration-pattern}

The most common pattern is:

1. render a collection with `data-collection="collection"`
2. render each item with `data-collection="node"`
3. render action buttons with `data-collection-action`
4. import the package JS in the admin asset build
5. listen to the lifecycle events when the project needs custom behavior

This is the approach used by `cms-bundle` for menus, routes, and content modules.

## Extension Patterns {#extension-patterns}

Good extension patterns are:

- listening to `before` events to change prototype or destination
- listening to `after` events to update UI state
- validating paste payloads before insertion
- adding project-specific behavior on top of the generic collection events

This lets the package stay generic while the application owns the domain-specific behavior.

## Current Limits {#current-limits}

- The package depends on browser clipboard support for copy and paste features.
- It relies on correct HTML data attributes and correctly generated prototypes.
- It initializes from browser load events, so the script should be loaded as part of the normal page asset build.
- It is a JavaScript collection helper, not a Symfony PHP form type by itself.
