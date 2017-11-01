---
layout: post
categories: php zf1 composer
date: 2015-09-15 11:00:00 +0200
lang: en
ref: 2017-07-16-ansistrano-deploy-php-apps
title:  "Composer and Zend Framework version 1"
---

# Define your apps for autoloading
You're still working with Zend Framework 1 (and yes, still apps running without you being able to make an evolution), so here's a small example of use.

First, create your file composer.json
```json
{
   "name": "company/projet",
   "description": "ma description",
   "require": {
       "zendframework/zendframework1": "^1.12"
   },
   "autoload": {
       "classmap": [
       "application/",
       "library/Application/",
        ]
   },
}
```

Use the `require` and `classmap` option to create our autoload (again we have an old application).
See the composer documentation to add more configuration options (http://getcomposer.org).

# Install
> $ composer install

Your composer.json file is read, the zf1 framework is download and install in your ROOTDIR/vendor/

Now, change your `public/index.php` to use the autoload in your project

```php
// --- Define path to application directory
defined('APPLICATION_PATH')
   || define('APPLICATION_PATH', realpath(__DIR__ . '/../application'));

// --- Define application environment
defined('APPLICATION_ENV')
   || define('APPLICATION_ENV', (getenv('APPLICATION_ENV') ? getenv('APPLICATION_ENV') : 'production'));

// --- Composer autoload
require_once realpath(__DIR__ . '/../vendor/autoload.php');

// --- Create application, bootstrap, and run
$application = new Zend_Application(APPLICATION_ENV, APPLICATION_PATH . '/configs/application.ini');
$application->bootstrap()->run();
```

The purpose is to have your libraries goes in the `vendor` directory from packagist.org, your own repositories (subversion, git, ...).
Exclude this directory from your repositories.

But remember to work with an older app, so how to use the standard ZF1 layout
and libraries in "library" directory.
Use a composer option for that library which not respect classmap or psr-* :

```
"include-path": ["library/"],
```

And generate your autoload.
> $ composer dumpautoload
