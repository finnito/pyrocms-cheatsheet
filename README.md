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

<!-- MarkdownTOC autolink="true" style="ordered" -->

1. [PyroCMS Cheat-sheet](#pyrocms-cheat-sheet)
    1. [Installing](#installing)
    1. [Deploying](#deploying)
    1. [Thanks](#thanks)

<!-- /MarkdownTOC -->


# PyroCMS Cheat-sheet

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

---

## Thanks

- [ASCII Art found here](https://www.asciiart.eu/nature/camping)
- Contents auto-generated with [MarkdownTOC](https://packagecontrol.io/packages/MarkdownTOC)