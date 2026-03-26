---
title: "Mime Translatable"
description: "Build Symfony emails with translated subjects, translation params, and locale-aware body rendering."
---

# Mime Translatable {#mime-translatable}

`softspring/mime-translatable` is a small component that helps you build translated emails on top of Symfony Mime and `TemplatedEmail`.

It is useful when your application mail classes need:

- translated subjects
- one locale per message
- Twig templates that receive translation params
- previewable example emails for admin tooling

This component is intentionally small. It does not create a mailer system by itself. It gives you the email object and renderer behavior that higher-level bundles can reuse.

## Installation {#installation}

```bash
composer require softspring/mime-translatable:^6.0
```

## What The Component Adds {#what-the-component-adds}

The package provides four pieces:

- `TranslatableEmail`
- `TranslatableBodyRenderer`
- `ExtendedContextEmail`
- `ExampleEmailInterface`

Together they cover the most common translation needs for application emails.

## Use TranslatableEmail {#use-translatableemail}

`TranslatableEmail` extends Symfony's `TemplatedEmail`.

It adds two practical features:

- a locale attached to the message
- translation params stored in the email context

That makes it easy to build one email class that knows:

- which locale it should render in
- which placeholders should be used in the translated subject
- which values should be available to Twig templates

### Build A Mail Class {#build-a-mail-class}

The normal pattern is to extend `TranslatableEmail` in each concrete mail class:

```php
use Softspring\Component\MimeTranslatable\TranslatableEmail;
use Symfony\Contracts\Translation\TranslatorInterface;

class WelcomeEmail extends TranslatableEmail
{
    public function __construct(TranslatorInterface $translator, ?string $locale = null)
    {
        parent::__construct($translator, $locale);

        $this->setTranslationParams([
            '%name%' => 'Mery',
        ]);

        $this->htmlTemplate('emails/welcome.html.twig');
        $this->subject('welcome.email.subject', 'messages');
    }
}
```

## Translate The Subject {#translate-the-subject}

`TranslatableEmail::subject()` translates the subject key immediately through Symfony Translation.

It uses:

- the subject key you pass
- the translation params stored with `setTranslationParams()`
- the translation domain you pass
- the message locale, if one exists

That is enough for most real application emails:

```php
$this->setTranslationParams([
    '%name%' => $user->getName(),
]);

$this->subject('register.confirm.email.subject', 'sfs_user');
```

### Important Note About Timing {#important-note-about-timing}

Subject translation happens when you call `subject()`, not later during final body rendering.

So the practical rule is:

1. set translation params first
2. call `subject()` after that

## Pass Translation Params To Twig {#pass-translation-params-to-twig}

`setTranslationParams()` stores the params in a dedicated context block.

That gives you one place to keep the placeholders used by the mail class.

The component stores them under:

- `__translation_params`

This is useful when:

- the subject uses placeholders
- the Twig template also needs the same values
- you want the mail class to own the full translated message contract

## Use The Message Locale During Body Rendering {#use-the-message-locale-during-body-rendering}

`TranslatableBodyRenderer` decorates a normal Symfony body renderer and temporarily switches the translator locale while rendering one `TranslatableEmail`.

That means the subject and the Twig rendering can follow the same message locale.

Without this, the email body would depend only on the global translator locale at render time.

### How To Wire The Renderer {#how-to-wire-the-renderer}

This component is a library, so the application or bundle must wire the service.

The common pattern is to decorate Symfony's `twig.mime_body_renderer`:

```yaml
Softspring\Component\MimeTranslatable\TranslatableBodyRenderer:
    decorates: twig.mime_body_renderer
    arguments:
        - '@Softspring\Component\MimeTranslatable\TranslatableBodyRenderer.inner'
        - '@translator.default'
```

This is the same idea used later by `mailer-bundle`.

## Use ExampleEmailInterface {#use-exampleemailinterface}

`ExampleEmailInterface` is a contract for email classes that can build a standalone example message.

It is especially useful for:

- previews in admin tools
- sending test emails from a back office
- documenting a mail class with a renderable sample

The contract is simple:

```php
public static function generateExample(TranslatorInterface $translator, ?string $locale = null): TranslatableEmail;
```

## Real Usage In Other Bundles {#real-usage-in-other-bundles}

The best examples in this repository are in `user-bundle`.

There, mail classes such as:

- `ConfirmationEmail`
- `InvitationEmail`
- `ResetPasswordEmail`

extend `TranslatableEmail` and implement `ExampleEmailInterface`.

They all follow the same pattern:

1. receive the translator and optional locale
2. put domain data into the email context
3. set translation params
4. choose the Twig template
5. translate the subject key

This is the recommended usage model for your own application emails.

## Extend The Component {#extend-the-component}

The component is meant to be extended through your own mail classes, not through a large inheritance tree inside the library.

The main extension points are:

- extend `TranslatableEmail` for your concrete email classes
- implement `ExampleEmailInterface` when previews matter
- decorate the renderer in your application or bundle

That keeps the component small while still making it easy to integrate in real projects.

## Recommended Usage Pattern {#recommended-usage-pattern}

For most projects, the clean pattern is:

1. create one mail class per email type
2. extend `TranslatableEmail`
3. set all context values in the constructor
4. set translation params there too
5. call `subject()` after the params are ready
6. implement `ExampleEmailInterface` when the mail should be previewable

## Limits And Current Notes {#limits-and-current-notes}

There are a few practical limits to know:

- the component does not auto-register services
- the renderer logic must be wired by the application or a higher-level bundle
- subject translation is eager, not lazy
- `ExampleEmailInterface` is only a contract; preview UIs belong to higher-level bundles such as `mailer-bundle`
