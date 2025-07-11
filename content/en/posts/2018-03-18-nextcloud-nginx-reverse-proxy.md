---
translationKey: "2018-03-18-nextcloud-nginx-reverse-proxy"
categories: php nextcloud
date: "2018-03-18T21:00:00Z"
ref: 2018-03-18-nextcloud-nginx-reverse-proxy
title: Configure Nextcloud with a reverse proxy (nginx)
---

How to configure Nextcloud with a reverse proxy and add secure header correctly.

## NextCloud with Apache
Our nextcloud instance will be install with Apache/PHP-FPM
In your vhost config or in .htaccess comment some line as below
```
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

## Configure Nginx
Configure a standard reverse proxy

In `server` section add `client_max_body_size 1G;`

In `location` section add
(As we have remove all headers in the backend (apache config) - These headers must be define in nginx)
```
# Specific header config
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
add_header X-Robots-Tag none;
add_header X-Download-Options noopen;
add_header X-Permitted-Cross-Domain-Policies none;
#proxy_hide_header X-Frame-Options;
```

The header `X-Frame-Options` is always define in code so take care to not have twice.
(https://github.com/nextcloud/server/blob/master/lib/private/legacy/response.php)
