---
title: "Time Ago Bundle"
description: "Render short relative past-time messages in Twig using Symfony translations."
---

# Time Ago Bundle {#time-ago-bundle}

`softspring/time-ago-bundle` adds a small Twig API to render relative past-time messages such as `5 minutes ago` or `Hace 1 hora`.

It is useful when your UI needs short, human-friendly timestamps for:

- notifications
- activity feeds
- comments
- dashboard summaries
- recent updates

This bundle is intentionally small. It does one thing: turn a date into a translated `time ago` label for Twig.

## Installation {#installation}

```bash
composer require softspring/time-ago-bundle:^6.0
```

In practice, this bundle expects a Symfony application with:

- Twig enabled
- the Symfony translator available

The bundle does not need extra configuration after installation.

## What You Get {#what-you-get}

After installation, the bundle registers:

- a Twig filter named `time_ago`
- a Twig function named `time_ago`
- built-in translations in English and Spanish

There is no package-specific configuration file to maintain.

## Basic Usage {#basic-usage}

Use the filter when you already have a date value in the template:

```twig
{{ post.publishedAt|time_ago }}
```

Use the function when it reads better in your template:

```twig
{{ time_ago(post.publishedAt) }}
```

Both call the same helper internally, so this is mainly a style choice.

## What Kind Of Input Works {#what-kind-of-input-works}

Today the helper accepts:

- `DateTime`
- strings that PHP can parse into a `DateTime`

Examples:

```twig
{{ time_ago(post.createdAt) }}
{{ time_ago(comment.createdAt|date('c')) }}
```

## How The Output Works {#how-the-output-works}

The bundle returns one main unit only.

Examples of the intended style:

- `Less than a minute ago`
- `5 minutes ago`
- `An hour ago`
- `A day ago`

This makes the output short and easy to scan in UI lists.

It does not try to build longer phrases such as:

- `2 days and 3 hours ago`
- `1 month, 4 days ago`

That simplicity is part of the bundle design.

## Translation Behavior {#translation-behavior}

The bundle uses the translation domain `sfs_timeago`.

It ships with English and Spanish messages, and the active Symfony locale decides which wording is used.

That means the same template can render different text automatically depending on the request locale.

## Overriding The Wording {#overriding-the-wording}

If your project needs a different tone, shorter copy, or product-specific wording, override the translations in your application like any other Symfony translation resource.

The keys are:

- `timeago.seconds`
- `timeago.minutes`
- `timeago.hours`
- `timeago.days`
- `timeago.months`
- `timeago.years`

This is the easiest extension point when:

- you want shorter labels
- you want a different brand voice
- you want to adjust singular or plural wording

## Where This Bundle Fits Well {#where-this-bundle-fits-well}

This bundle works well when:

- the UI needs a quick relative label instead of a full formatted date
- the date is already in the past
- one main unit is enough for the interface
- you want translation-aware output without writing custom Twig code

It is a good fit for compact interfaces where absolute dates would feel too heavy.

## Extension Points {#extension-points}

The bundle is small, but it is still extensible.

### Override Translations {#override-translations}

For most projects, overriding translations is enough.

Use this when you want to change wording but keep the same formatting rules.

### Decorate Or Replace The Helper {#decorate-or-replace-the-helper}

If your application needs different behavior, decorate or replace `Softspring\TimeAgoBundle\Helper\TimeAgoHelper`.

That is the right option when you want to:

- support future dates
- accept more input types
- change rounding or threshold rules
- generate more detailed output

### Replace The Twig API {#replace-the-twig-api}

You can also replace or decorate the Twig extension if you need a different template API.

That is less common, but it is available if your project wants different filter or function names.

## Current Limits {#current-limits}

A few limits matter when you use this bundle today:

- it is designed for past dates expressed as `ago`
- it returns only one main unit
- it does not expose configuration for thresholds or formatting rules
- invalid non-date values return an empty string and may log a warning
- parseable strings are accepted, but malformed strings can still raise an exception
- the current implementation accepts `DateTime`, not generic `DateTimeInterface`
- future dates do not have dedicated wording such as `in 5 minutes`

These points are important if your application needs a broader relative-time component instead of a very small Twig helper.

## Recommended Use {#recommended-use}

Use this bundle when your application needs simple translated `time ago` labels with almost no setup.

If your project needs richer relative-time behavior, keep the bundle as a starting point and extend the helper in the application layer.
