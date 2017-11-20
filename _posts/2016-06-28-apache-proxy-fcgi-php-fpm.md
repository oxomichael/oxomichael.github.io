---
layout: post
categories: apache2 php-fpm
date: 2016-06-28 21:00:00 +0200
lang: en
ref: 2016-06-28-apache-proxy-fcgi-php
title:  "Apache2, PHP-FPM"
---

A partir d'Apache version 2.4.10, il est possible d'avoir des performances identiques à NGINX.
Principalement en utilisant php-fpm avec le mod_fcgi.

## Avec Ubuntu 14.04
Tout d'abord activer trusty-backports dans /etc/apt/source.list,
puis autoriser les paquets backports en priorité dans le fichier `/etc/apt/preferences`
```
Package: *
Pin: release a=trusty-backports
Pin-Priority: 500
```

Ensuite exécuter
```bash
$ apt-get update && apt-get upgrade
```

Vérifier que Apache 2.4.10 est bien installé
```bash
$ apt-get install apache2
```

Installer php-fpm (php5-fpm ou php-7.0-fpm)
```bash
$ apt-get install php5-fpm
```

Créer un fichier /etc/apache2/conf-available/phpfcgi.conf
```
<FilesMatch "\.php$">
   # Pick one of the following approaches
   # Use the standard TCP socket
   #SetHandler "proxy:fcgi://localhost/:9000"
   # If your version of httpd is 2.4.9 or newer (or has the back-ported feature), you can use the unix domain socket
   #SetHandler "proxy:unix:/path/to/app.sock|fcgi://localhost/"
   SetHandler "proxy:unix:/var/run/php5-fpm.sock|fcgi://localhost/"
</FilesMatch>
# Définition d'une configuration de mandataire qui convient.
# La partie qui est mise en correspondance avec la valeur de
# SetHandler est la partie qui suit le "pipe". Si vous devez faire
# une distinction, "localhost" peut être changé en un nom de serveur
# unique.
<Proxy "fcgi://localhost/" enablereuse=on max=10>
</Proxy>
# Passer le header Authorization
SetEnvIfNoCase ^Authorization$ "(.+)" HTTP_AUTHORIZATION=$1
```

Ce fichier de configuration active la connexion à php-fpm pour tout les vhosts, il est possible de séparer (chroot) chaque vhosts d'apache et de le rediriger vers un php-fpm approprié pour plus de sécurité.  
Activer les modules nécessaires
```bash
$ a2dismod mpm_prefork php5
$ a2enmod mpm_event
$ a2enconf phpfcgi
$ service apache2 restart
```

## Debian Stretch
Avec debian, et maintenant php 7, lorsque que l'on installe php7.0-fpm,
une conf est automatique créer dans le fichier `/etc/apache2/conf-available/php7.0-fpm.conf`
```
# Redirect to local php-fpm if mod_php is not available
<IfModule !mod_php7.c>
<IfModule proxy_fcgi_module>
    # Enable http authorization headers
    <IfModule setenvif_module>
    SetEnvIfNoCase ^Authorization$ "(.+)" HTTP_AUTHORIZATION=$1
    </IfModule>

    <FilesMatch ".+\.ph(p[3457]?|t|tml)$">
        SetHandler "proxy:unix:/run/php/php7.0-fpm.sock|fcgi://localhost"
    </FilesMatch>
    <FilesMatch ".+\.phps$">
        # Deny access to raw php sources by default
        # To re-enable it's recommended to enable access to the files
        # only in specific virtual host or directory
        Require all denied
    </FilesMatch>
    # Deny access to files without filename (e.g. '.php')
    <FilesMatch "^\.ph(p[3457]?|t|tml|ps)$">
        Require all denied
    </FilesMatch>
</IfModule>
</IfModule>
```

Il suffit donc de l'activer (`a2enconf php7.0-fpm`)

## Fonctionnement particulier de php-fpm
Il est possible d'agir sur des variables d'environnement de PHP par projet (répertoire),
en utilisant un fichier `.user.ini`  
La modification sera prise en compte à chaque redémarrage du service php-fpm ou
 toutes les 5 minutes.

Voilà un exemple:  
```
upload_max_filesize=513M
post_max_size=513M
memory_limit=512M
mbstring.func_overload=0
always_populate_raw_post_data=-1
default_charset='UTF-8'
date.timezone='Europe/Paris'
env[APPLICATION_ENV] = "production"
```
