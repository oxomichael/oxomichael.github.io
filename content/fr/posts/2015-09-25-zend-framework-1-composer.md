---
translationKey: "2015-09-25-zend-framework-1-composer"
categories: php zf1 composer
date: "2015-09-25T11:00:00Z"
ref: 2015-09-25-zend-framework-1-composer
title: Utiliser Composer avec Zend Framework 1
---

# Configurer l'autoloading pour votre application
Si vous maintenez toujours une application utilisant Zend Framework 1 (et oui, certains d'entre nous ont encore des applications qui ne peuvent pas être facilement mises à jour), voici un petit exemple de la manière d'intégrer Composer.

Tout d'abord, créez votre fichier `composer.json` :
```json
{
   "name": "entreprise/projet",
   "description": "Ma description",
   "require": {
       "zendframework/zendframework1": "^1.12"
   },
   "autoload": {
       "classmap": [
           "application/",
           "library/Application/"
        ]
   }
}
```

Nous utiliserons les options `require` et `classmap` pour configurer l'autoloader, ce qui est nécessaire pour cette application héritée.
Pour plus d'options de configuration, consultez la documentation officielle de Composer sur [getcomposer.org](https://getcomposer.org).

# Installation
> $ composer install

Composer lira votre fichier `composer.json`, puis téléchargera et installera le framework ZF1 dans votre répertoire `ROOTDIR/vendor/`.

Ensuite, modifiez votre fichier `public/index.php` pour utiliser l'autoloader de Composer :

```php
// --- Définir le chemin vers le répertoire de l'application
defined('APPLICATION_PATH')
   || define('APPLICATION_PATH', realpath(__DIR__ . '/../application'));

// --- Définir l'environnement de l'application
defined('APPLICATION_ENV')
   || define('APPLICATION_ENV', (getenv('APPLICATION_ENV') ? getenv('APPLICATION_ENV') : 'production'));

// --- Autoload de Composer
require_once realpath(__DIR__ . '/../vendor/autoload.php');

// --- Créer l'application, l'amorcer et l'exécuter
$application = new Zend_Application(APPLICATION_ENV, APPLICATION_PATH . '/configs/application.ini');
$application->bootstrap()->run();
```

L'objectif est de gérer vos bibliothèques depuis Packagist.org ou vos propres dépôts (Git, Subversion, etc.) dans le répertoire `vendor`.
N'oubliez pas d'exclure le répertoire `vendor` de votre système de contrôle de version.

Cependant, comme nous travaillons avec une application ancienne, vous pourriez avoir besoin de gérer des bibliothèques dans le répertoire traditionnel `library` qui ne suivent pas les standards d'autoloading modernes.
Pour les bibliothèques qui ne respectent pas les normes `classmap` ou PSR-*, vous pouvez utiliser l'option `include-path` de Composer :

```json
"include-path": ["library/"],
```

Ensuite, régénérez l'autoloader :
> $ composer dump-autoload