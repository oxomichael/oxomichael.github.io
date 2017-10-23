Composer et Zend Framework version 1
Soumis par michael le ven, 25/09/2015 - 11:06

Vous travaillez encore avec le Zend Framework 1 (et oui encore des applications qui tournent sans que vous puissiez faire une évolution), alors voici un petit exemple d'utilisation.
Premièrement créer son fichier composer.json
1
2
3
4
5
6
7
8
9
10
11
12
13

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

On utilise "require" et l'option "classmap" pour réaliser, notre autoload (oui encore une fois c'est une veille application). Voir la doc de composer pour ajouter plus d'options de configuration (http://getcomposer.org).
Première Installation
1

composer install

Lecture de composer.json, téléchargement et installation du zf1 dans ROOTDIR/vendor/
Modifions maintenant le fichier public/index.php pour l'utilisation dans le projet
1
2
3
4
5
6
7
8
9
10
11
12
13
14

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

L'objectif est donc d'avoir les librairies qui se télécharge dans le dossier vendor soit depuis packagist.org ou depuis vos propres repositories subversion, git,....
Ce dossier est donc à exclure dans le gestionnaire de version.
Mais comment faire si vous avez des librairies dans "library" pour respecter la structure d'un projet ZF v1
Il existe pour composer (voir la doc) une option pour les librairies hors, classmap ou psr-* :
1

"include-path": ["library/"],

Il vous reste ensuite uniquement à générer votre référentiel autoload
1

composer dumpautoload
