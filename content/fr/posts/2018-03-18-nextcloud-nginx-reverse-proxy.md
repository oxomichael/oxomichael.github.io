---
translationKey: "2018-03-18-nextcloud-nginx-reverse-proxy"
categories:
- php
- nextcloud
date: "2018-03-18T21:00:00Z"
ref: 2018-03-18-nextcloud-nginx-reverse-proxy
title: Configurer Nextcloud derrière un reverse proxy Nginx
---

Comment configurer Nextcloud derrière un reverse proxy et ajouter correctement les en-têtes de sécurité.

## Nextcloud avec Apache

Notre instance Nextcloud sera installée avec Apache/PHP-FPM.

Dans la configuration de votre vhost ou dans votre `.htaccess`, commentez les lignes comme ci-dessous :

```apache
<IfModule mod_env.c>
    # Add security and privacy related headers
    # Header set X-Content-Type-Options "nosniff"
    # Header set X-XSS-Protection "1; mode=block"
    # Header set X-Robots-Tag "none"
    # Header set X-Download-Options "noopen"
    # Header set X-Permitted-Cross-Domain-Policies "none"
    SetEnv modHeadersAvailable true
</IfModule>
```

## Configuration de Nginx

Configurez un reverse proxy standard.

Dans la section `server`, ajoutez `client_max_body_size 1G;`.

Dans la section `location`, ajoutez :
(Comme nous avons supprimé tous les en-têtes dans le backend (configuration Apache), ces en-têtes doivent être définis dans Nginx)

```nginx
# Specific header config
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
add_header X-Robots-Tag none;
add_header X-Download-Options noopen;
add_header X-Permitted-Cross-Domain-Policies none;
#proxy_hide_header X-Frame-Options;
```

L'en-tête `X-Frame-Options` est toujours défini dans le code, faites donc attention à ne pas l'avoir en double.
(https://github.com/nextcloud/server/blob/master/lib/private/legacy/response.php)