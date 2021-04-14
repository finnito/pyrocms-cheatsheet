```
           (                 ,&&&.
            )                .,.&&
           (  (              \=__/
               )             ,'-'.
         (    (  ,,      _.__|/ /|
          ) /\ -((------((_|___/ |
        (  // | (`'      ((  `'--|
      _ -.;_/ \\--._      \\ \-._/.
     (_;-// | \ \-'.\    <_,\_\`--'|
     ( `.__ _  ___,')      <_,-'__,'
      `'(_ )_)(_)_)'
```
# PyroCMS Cheat-sheet

<!-- MarkdownTOC autolink="true" style="ordered" -->

1. [Installing](#installing)
1. [Deploying](#deploying)
1. [PHP Artisan](#php-artisan)
1. [Addons](#addons)
    1. [SPAM Prevention](#spam-prevention)
        1. [Akismet](#akismet)
        1. [reCAPTCHA](#recaptcha)
        1. [Honeypot](#honeypot)
    1. [Sending Mail](#sending-mail)
    1. [Custom Forms](#custom-forms)
        1. [Rendering Forms](#rendering-forms)
        1. [Form Slugs](#form-slugs)
1. [Theming](#theming)
    1. [Laravel Mix](#laravel-mix)
1. [Thanks](#thanks)

<!-- /MarkdownTOC -->

## Installing

```bash
composer create-project pyrocms/pyrocms folder-name
php artisan install --ready

# Remove `anomaly/installer-module` from `composer.json`

composer update

# Set some permissions
chmod -R 755 public/app
chmod -R 755 bootstrap/cache
chmod -R 755 storage
```

## Deploying

```bash
#!/bin/bash

# Get latest from git
git pull origin master

# Set some permissions
sudo chown -R www-data:www-data ./
chmod -R 755 ./public/app
chmod -R 755 ./bootstrap/cache
chmod -R 755 ./storage

# Install latest composer.lock
composer install --profile;

# Pyro chores
php artisan migrate
php artisan assets:clear
php artisan view:clear
php artisan httpcache:clear

# Just in case
composer dump-autoload --profile;
```

## PHP Artisan

```bash
# View all commands
php artisan list

# Install an addon
php artisan addon:install anomaly.module.streams

# Migrate an addon
php artisan migrate --addon=anomaly.module.streams
```

## Addons

### SPAM Prevention

#### Akismet

Install the package:

```bash
composer require nickurt/laravel-akismet
```

Add these lines to your `.env` file, as per your Akismet sign-up information:

```bash
AKISMET_BLOGURL=""
AKISMET_APIKEY=""
```

Then in your form handler (probably), verify the content something like this (check [the documentation](https://akismet.com/development/api/#comment-check) for full details).

```php
$values = $builder->getFormValues();
$akismet->setApiKey(env("AKISMET_APIKEY"));
$akismet->fill([
    "user_ip" => $request->headers->get("origin"),
    "user_agent" => $request->headers->get("user_agent"),
    "referrer" => $request->headers->get("referer"),
    "permalink" => $request->headers->get("origin"),
    "comment_type" => "forum-post",
    "comment_author" => $values["name"],
    "comment_author_email" => $values["email"],
    "comment_content" => $values["job"],
    "blog" => env("AKISMET_BLOGURL")

]);
if (!$akismet->validateKey()) {
    $bag->info("There was an error with Akismet");
    return;
}

if ($akismet->isSpam()) {
    $bag->info("Your log was marked as SPAM. If this is an error, please contact us by phone.");
    return;
}

$bag->success("Thank you for your message. We will get back to you as soon as possible!");
$cache->purge($request->headers->get("referer"));

// At this point the form submission is all good, and you can continue with saving the form entry, or whatever you want to do.
```

#### reCAPTCHA

Start by installing the package. It also requires `guzzlehttp/guzzle`, but this is automatically installed if you use `composer`.

```bash
composer require thrive/recaptcha-field_type
```

Add these lines to your `.env` file, as per your Google reCAPTCHA sign-up information:

```bash
GOOGLE_RECAPTCHA_SITE_KEY=""
GOOGLE_RECAPTCHA_SECRET_KEY=""
```

You can now add it as a field to your form via the Admin panel, or via your form builder like so:

```php
protected $fields = [
    "recaptcha" => [
        "type" => "thrive.field_type.recaptcha",
        "input_view" => "thrive.field_type.recaptcha::input_checkbox",
        "required" => true,
    ],
];
```

__NB__: Thrive makes a rather weird override in the `RecaptchaFieldTypeServiceProvider.php`, which will cause an error.

```php
protected $overrides = [
    'streams::form/standard' => 'theme::partials/standard',
];
```

You can fix this by making an empty view called `standard.twig` in your theme resources: `views/partials/standard.twig`.

#### Honeypot

Start by installed the package.

```bash
composer require fritzandandre/honeypot-field_type
```

You can now add it as a field to your form via the Admin panel, or via your form builder like so:

```php
protected $fields = [
    "honeypot" => [
        "type" => "fritzandandre.field_type.honeypot",
    ],
];
```

Depending on your setup, you may also need to add some CSS like this to hide the honeypot input.

```css
.winnie-the-pooh {
    position: absolute;
    top: -2000000px;
    left: -2000000px;
}
```

### Sending Mail

Add this to the top of your file:

```php
use Illuminate\Support\Facades\Mail;
use Finnito\CanterburyPlumbingAndGasTheme\Mail\YourMailable;
```

Then inside your file you can send mail like this:

```php
Mail::to(env("CONTACT_EMAIL_RECEIPIENTS"))
    ->cc(env("CONTACT_EMAIL_CC"))
    ->send(new YourMailable($values));
```

You need to create an extension of `Mailable`, which might look something like this:

```php
<?php namespace Finnito\CanterburyPlumbingAndGasTheme\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class YourMailable extends Mailable
{
    use Queueable, SerializesModels;

    public $values;

    /**
     * Create a new message instance.
     *
     * @param  \App\Models\Order  $order
     * @return void
     */
    public function __construct($values)
    {
        $this->values = $values;
    }

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('theme::mail.contact')
            ->subject("CP&G: Online Contact")
            ->replyTo($this->values["email"])
            ->with(["values" => $this->values]);
    }
}
```

And last but not least, a twig template to generate your email body should also exist.

```html
<p>
    <strong>Name:</strong> {{ values.name }}<br>
    <strong>Email:</strong> {{ values.email }}<br>
    <strong>Phone:</strong> {{ values.phone }}<br>
</p>

<p>{{ values.job }}</p>
```

### Custom Forms

#### Rendering Forms

You can display custom forms in Twig like so:

```html
<!-- If you need control: -->
{% set form = form('contact_form').get() %}
{{ form.open()|raw }}
{{ form.fields|raw }}
{{ form.actions|raw }}
{{ form.close()|raw }}

<!-- Or more simply: -->
{{ form('contact_form').get()|raw }}
```

#### Form Slugs

Form builders can be bound to a custom slug in a service provider like so:

```php
protected $bindings = [
    "contact_form" => \Finnito\CanterburyPlumbingAndGasTheme\Form\ContactFormBuilder::class,
];
```

## Theming

### Laravel Mix

Generating assets is no longer done with Assetic, but instead with Mix. Setup `webpack.mix.js` in your root directory and run `npm install`, then `npm run watch`

```js
let mix = require('laravel-mix');
mix.sass('addons/shared/finnito/canterbury_plumbing_and_gas-theme/resources/scss/theme/theme.scss', 'public/css');
```

```bash
# Update npm
npm install -g npm

# Install
npm install

# Build assets (omnit "watch" to just build once)
npm run watch
```

---

## Thanks

- [ASCII Art found here](https://www.asciiart.eu/nature/camping)
- Contents auto-generated with [MarkdownTOC](https://packagecontrol.io/packages/MarkdownTOC)