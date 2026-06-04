---
title: "Mailer Bundle"
description: "Register mail templates, preview and test example emails in admin, decorate Symfony's body renderer, and optionally expose email history through Doctrine."
---

# Mailer Bundle {#mailer-bundle}

The Mailer Bundle adds a small mail-oriented layer on top of Symfony Mailer.

Its current focus is not transport management. It focuses on:

- registering mail templates from different bundles;
- previewing and testing example emails from an admin UI;
- decorating Symfony's body renderer with the translatable renderer from `softspring/mime-translatable`;
- exposing an optional Doctrine-backed email history model and admin screens.

A good way to think about this bundle is: it is the shared infrastructure around translatable, previewable emails in the Softspring ecosystem.

## Installation {#installation}

Install the package:

```bash
composer require softspring/mailer-bundle:^6.0
```

If your application does not use Symfony Flex, enable it manually:

```php
<?php

return [
    // ...
    Softspring\MailerBundle\SfsMailerBundle::class => ['all' => true],
];
```

The bundle expects Symfony Mailer to be configured in the application.

## Main Features {#main-features}

The bundle has three main blocks:

1. a template registry built from tagged template loaders;
2. an admin UI to list templates, preview them, and send test emails;
3. an optional `EmailHistoryInterface` model with Doctrine mappings and admin pages.

It also decorates `twig.mime_body_renderer` so translatable email classes can render correctly through the `softspring/mime-translatable` component.

## Configuration {#configuration}

Configure the bundle under `sfs_mailer`:

```yaml
# config/packages/sfs_mailer.yaml
sfs_mailer:
    entity_manager: default

    templates:
        my_template:
            template: 'emails/my_template.html.twig'
            subject_block: subject
            html_block: html
            text_block: text
            from_email:
                sender_name: 'Acme'
                address: 'noreply@example.com'
            example_context:
                companyName: 'Acme'

    history:
        enabled: true
        class: App\Entity\EmailHistory
```

The three top-level blocks are:

- `entity_manager`
- `templates`
- `history`

### Entity Manager {#entity-manager}

`entity_manager` selects the Doctrine entity manager used by target entity resolution for `EmailHistoryInterface`.

### Template Configuration {#template-configuration}

Each entry inside `sfs_mailer.templates` can define:

- `template`
- `subject_block`
- `html_block`
- `text_block`
- `from_email.sender_name`
- `from_email.address`
- `example_context`

However, the current built-in `ParameterTemplateLoader` does not yet use most of those fields. In the current code, it only creates template entries keyed by the template id and leaves the rest effectively unused.

That means this block should currently be seen as a template registry seed, not as a full declarative email-definition system.

### History Configuration {#history-configuration}

The `history` block enables the email history model and admin controller wiring.

Available options:

- `enabled`
- `class`

When enabled, the configured class must implement `EmailHistoryInterface`.

## Template Registry {#template-registry}

The core service is `Softspring\MailerBundle\Template\TemplateLoader`.

It does not load templates from one place only. Instead, it gathers all services tagged with `sfs_mailer.template_loader`, asks each one for a `TemplateCollection`, and merges them.

This is the key extension point of the bundle.

### Template Object {#template-object}

The `Template` value object currently stores:

- `id`
- `description`
- `class`
- `preview`

Default values matter:

- `class` defaults to `Softspring\MailerBundle\Mime\TranslatableEmail`
- `preview` defaults to `false`

The admin UI uses these fields directly, especially `id`, `class`, and `preview`.

### Built-In Loader {#built-in-loader}

The bundle ships one built-in loader: `ParameterTemplateLoader`.

It reads the `sfs_mailer.templates` container parameter and creates template entries for each configured key.

In the current implementation, this loader:

- sets the template id;
- does not copy `template`, block names, sender data, or example context into the `Template` object;
- does not mark templates as previewable;
- leaves the default `class` untouched.

So, by itself, the parameter loader is not enough to build rich, preview-ready admin templates.

### Tagged Loaders From Other Bundles {#tagged-loaders-from-other-bundles}

The real intended model is that feature bundles contribute their own loaders.

For example, `softspring/user-bundle` provides `MailTemplateLoader`, which registers templates such as:

- `sfs_user.reset_password`
- `sfs_user.register_confirm`
- `sfs_user.invite`

and explicitly sets:

- the email class;
- `preview = true`

That is why the mailer bundle is best understood as a shared registry and admin shell for template providers implemented in other bundles.

### Creating Your Own Loader {#creating-your-own-loader}

To expose emails in the admin UI, create a loader service implementing `TemplateLoaderInterface` and tag it with `sfs_mailer.template_loader`.

Example:

```php
<?php

namespace App\Mailer\Loader;

use App\Mime\WelcomeEmail;
use Softspring\MailerBundle\Template\Loader\TemplateLoaderInterface;
use Softspring\MailerBundle\Template\Template;
use Softspring\MailerBundle\Template\TemplateCollection;

class MailTemplateLoader implements TemplateLoaderInterface
{
    public function load(): TemplateCollection
    {
        $collection = new TemplateCollection();

        $template = new Template();
        $template->setId('app.welcome');
        $template->setClass(WelcomeEmail::class);
        $template->setPreview(true);

        $collection->addTemplate($template);

        return $collection;
    }
}
```

```yaml
services:
    App\Mailer\Loader\MailTemplateLoader:
        tags: ['sfs_mailer.template_loader']
```

This is the most reliable way to integrate application emails with the current bundle.

## Admin Template UI {#admin-template-ui}

The bundle ships an admin controller to inspect registered templates.

Import the routes when you want that UI:

```yaml
_sfs_mailer_templates:
    resource: '@SfsMailerBundle/config/routing/mailer_templates.yaml'
    prefix: /admin/mailer/templates
```

The available routes are:

- `sfs_mailer_templates_search`
- `sfs_mailer_templates_preview`
- `sfs_mailer_templates_test`

### Search Page {#search-page}

`MailerTemplateController::search()` loads the `TemplateCollection` and renders all registered templates.

The search page:

- translates `<template id>.name` for the displayed name;
- uses the template description if available;
- falls back to `<template id>.description` translation when no explicit description exists;
- only shows preview links when `template.preview` is `true`.

### Preview Page {#preview-page}

`MailerTemplateController::preview()` is built for email classes that implement `Softspring\Component\MimeTranslatable\ExampleEmailInterface`.

The controller:

1. loads the template object by id;
2. gets the template email class;
3. checks that the class implements `ExampleEmailInterface`;
4. calls `::generateExample($translator, $locale)`;
5. renders the email through `TranslatableBodyRenderer`;
6. shows the rendered subject and HTML body.

This makes preview support class-driven, not configuration-driven.

In other words, if you want previews, you need an email class with a static example generator.

### Test Send Page {#test-send-page}

`MailerTemplateController::test()` lets an authenticated admin send a test email.

The form contains:

- `locale`
- `toEmail`
- `toName`

The locale choices come from `%kernel.enabled_locales%`.

When the form is submitted, the controller:

1. builds the example email through `ExampleEmailInterface::generateExample()`;
2. sets the target recipient with `to(new Address(...))`;
3. sends the message with `MailerInterface`.

If the Twig template is missing, the controller adds a form error instead of crashing the page.

### Requirements For Preview And Test {#requirements-for-preview-and-test}

For a template to fully work in the admin UI, you usually need:

- a custom template loader that sets the email class;
- `preview = true`;
- an email class implementing `ExampleEmailInterface`;
- a generated example that can render without application-specific runtime state.

Without that, the template may appear in the list but not be previewable or testable.

## Email Classes And Rendering {#email-classes-and-rendering}

The bundle exposes three mail-related classes under `Softspring\MailerBundle\Mime`:

- `TranslatableEmail`
- `ExtendedContextEmail`
- `TranslatableBodyRenderer`

In the current code, all three are deprecated wrappers around classes from `softspring/mime-translatable`.

For new code, prefer the component classes directly:

- `Softspring\Component\MimeTranslatable\TranslatableEmail`
- `Softspring\Component\MimeTranslatable\ExtendedContextEmail`
- `Softspring\Component\MimeTranslatable\TranslatableBodyRenderer`

### Renderer Decoration {#renderer-decoration}

The bundle decorates Symfony's `twig.mime_body_renderer` service with `TranslatableBodyRenderer`.

That means emails rendered through Symfony Mailer use the translatable body renderer automatically once the bundle is enabled.

This is the main runtime integration point of the bundle outside the admin UI.

## Writing Example Emails {#writing-example-emails}

The preview and test screens expect `ExampleEmailInterface`.

A typical email class looks like this:

```php
<?php

namespace App\Mime;

use Softspring\Component\MimeTranslatable\ExampleEmailInterface;
use Softspring\Component\MimeTranslatable\TranslatableEmail;
use Symfony\Contracts\Translation\TranslatorInterface;

class WelcomeEmail extends TranslatableEmail implements ExampleEmailInterface
{
    public static function generateExample(TranslatorInterface $translator, ?string $locale = null): TranslatableEmail
    {
        return new self($translator, $locale);
    }

    public function __construct(TranslatorInterface $translator, ?string $locale = null)
    {
        parent::__construct($translator, $locale);

        $this->htmlTemplate('emails/welcome.html.twig');
        $this->subject('welcome.subject', 'messages');
        $this->setContextParam('name', 'Jane');
    }
}
```

The key point is that `generateExample()` must return a fully renderable email without needing a database lookup or request state.

## Email History {#email-history}

The bundle includes an optional email history model.

This part contains:

- `EmailHistoryInterface`
- `Model\EmailHistory`
- `Entity\EmailHistory`
- Doctrine mappings for the model and entity
- admin routes and templates to browse stored history

### Enabling History {#enabling-history}

Enable it in configuration:

```yaml
sfs_mailer:
    history:
        enabled: true
        class: App\Entity\EmailHistory
```

When history is enabled, the bundle:

- validates that the configured class implements `EmailHistoryInterface`;
- enables the default entity mapping for `Softspring\MailerBundle\Entity\EmailHistory`;
- resolves `EmailHistoryInterface` to your configured class;
- registers the mail history admin controller.

### Implementing The Entity {#implementing-the-entity}

The bundle ships `Softspring\MailerBundle\Model\EmailHistory` as a mapped superclass and `Softspring\MailerBundle\Entity\EmailHistory` as a ready entity.

If you want an application entity, the usual approach is:

```php
<?php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Softspring\MailerBundle\Entity\EmailHistory as BaseEmailHistory;
use Softspring\MailerBundle\Model\EmailHistoryInterface;

#[ORM\Entity]
class EmailHistory extends BaseEmailHistory implements EmailHistoryInterface
{
}
```

The mapped model stores:

- `id`
- `status`
- `templateId`
- serialized `message`
- `createdAt`
- `lastStatusAt`

### What The History Model Exposes {#what-the-history-model-exposes}

Besides the raw serialized message, the model provides helper accessors such as:

- `getMessageSubject()`
- `getMessageFrom()`
- `getMessageTo()`
- `getMessageReplyTo()`
- `getMessageDate()`
- `getMessageBodyHtml()`
- `getMessageBodyText()`

This is what the admin history templates use to show message details.

### History Status Values {#history-status-values}

`EmailHistoryInterface` defines four statuses:

- `STATUS_PENDING`
- `STATUS_IN_PROGRESS`
- `STATUS_SENT`
- `STATUS_FAILED`

These values are intended for your own history-writing logic.

## Admin History UI {#admin-history-ui}

Import the history routes when you want the backoffice screens:

```yaml
_sfs_mailer_history:
    resource: '@SfsMailerBundle/config/routing/mailer_history.yaml'
    prefix: /admin/mailer/history
```

The available routes are:

- `sfs_mailer_history_search`
- `sfs_mailer_history_details`

### Search Screen {#history-search-screen}

`MailerHistoryController::search()` loads all history rows ordered by `createdAt DESC`.

The page shows:

- message date
- template id
- recipients
- subject
- senders
- status

### Details Screen {#history-details-screen}

`MailerHistoryController::details()` loads one history record by id and renders the stored HTML body and message metadata.

This UI is read-only. The bundle does not provide history filters, retries, or resend actions.

## Important Limitations {#important-limitations}

These limitations come directly from the current code and are worth knowing before you design around the bundle.

### Template Configuration Is Only Partially Implemented {#template-configuration-is-only-partially-implemented}

The `templates` configuration block accepts fields such as `template`, block names, sender info, and example context, but the built-in `ParameterTemplateLoader` currently does not transfer those values into the `Template` object.

So configuration alone does not yet describe a full email template runtime model.

### ParameterTemplateLoader Reuses One Template Instance {#parametertemplateloader-reuses-one-template-instance}

The current loader creates one `Template` object outside the loop and reuses it for every configured key.

That means parameter-based template registration should be treated carefully, especially if you expect multiple distinct template objects in the final collection.

### History Is A Model And UI, Not A Full Logging Pipeline {#history-is-a-model-and-ui-not-a-full-logging-pipeline}

The bundle includes history entities, mappings, an event wrapper, and admin screens, but it does not currently register a built-in listener that captures every sent email and persists history automatically.

If you enable history, you still need your own application logic to create and update `EmailHistoryInterface` records.

### Event Surface Is Minimal {#event-surface-is-minimal}

The repository contains `EmailHistoryEvent`, but `SfsMailerEvents` is currently empty and the bundle does not expose a richer built-in event workflow around sending, storing, or status changes.

### Admin Template Redirect On Missing Template {#admin-template-redirect-on-missing-template}

When the template id is not found, the template controller redirects to the mail history search route, not to the template list route.

This is part of the current implementation and can feel odd in a project where history is disabled.

## Practical Recommendations {#practical-recommendations}

For the current `6.0` codebase, the most robust way to use the bundle is:

1. create email classes based on `softspring/mime-translatable`;
2. implement `ExampleEmailInterface` for the emails you want to preview;
3. register them through your own tagged template loader;
4. use the admin UI for preview and test sending;
5. if you need history, implement and persist `EmailHistoryInterface` yourself.

That matches the real shape of the package better than trying to express the whole mail system through `sfs_mailer.templates` configuration alone.
