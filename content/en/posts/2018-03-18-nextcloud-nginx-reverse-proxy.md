---
translationKey: "2018-03-18-nextcloud-nginx-reverse-proxy"
categories:
- php
- nextcloud
date: "2018-03-18T21:00:00Z"
ref: 2018-03-18-nextcloud-nginx-reverse-proxy
title: Configure Nextcloud with a reverse proxy (Nginx)
---

How to configure Nextcloud with a reverse proxy and add security headers correctly.

## NextCloud with Apache

Our Nextcloud instance will be installed with Apache/PHP-FPM.

In your vhost configuration or in your `.htaccess` file, comment out the following lines:

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

## Configure Nginx

Configure a standard reverse proxy.

In the `server` section, add `client_max_body_size 1G;`.

In the `location` section, add the following headers:
(Since we have removed all headers in the backend (Apache config), these headers must be defined in Nginx)

```nginx
# Specific header config
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
add_header X-Robots-Tag none;
add_header X-Download-Options noopen;
add_header X-Permitted-Cross-Domain-Policies none;
#proxy_hide_header X-Frame-Options;
```

The `X-Frame-Options` header is always defined in the Nextcloud code, so be careful not to declare it twice.
(See: https://github.com/nextcloud/server/blob/master/lib/private/legacy/response.php)