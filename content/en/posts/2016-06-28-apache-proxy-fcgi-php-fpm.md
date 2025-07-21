---
translationKey: "2016-06-28-apache-proxy-fcgi-php-fpm"
categories:
- apache2
- php-fpm
date: "2016-06-28T21:00:00Z"
ref: 2016-06-28-apache-proxy-fcgi-php
title: Apache2, PHP-FPM
---

Since Apache version 2.4.10, it's possible to achieve performance similar to NGINX,
mainly by using php-fpm with mod_fcgi.

## With Ubuntu 14.04

First, enable `trusty-backports` in `/etc/apt/source.list`,
then give priority to backports packages in the `/etc/apt/preferences` file:

```
Package: *
Pin: release a=trusty-backports
Pin-Priority: 500
```

Then run:

```bash
$ apt-get update && apt-get upgrade
```

Verify that Apache 2.4.10 is installed:

```bash
$ apt-get install apache2
```

Install php-fpm (php5-fpm or php-7.0-fpm):

```bash
$ apt-get install php5-fpm
```

Create a file `/etc/apache2/conf-available/phpfcgi.conf`:

```apache
<FilesMatch "\.php$">
   # Pick one of the following approaches
   # Use the standard TCP socket
   #SetHandler "proxy:fcgi://localhost/:9000"
   # If your version of httpd is 2.4.9 or newer (or has the back-ported feature), you can use the unix domain socket
   #SetHandler "proxy:unix:/path/to/app.sock|fcgi://localhost/"
   SetHandler "proxy:unix:/var/run/php5-fpm.sock|fcgi://localhost/"
</FilesMatch>
# Definition of a suitable proxy configuration.
# The part that is matched with the value of SetHandler is the part
# that follows the "pipe". If you need to make a distinction,
# "localhost" can be changed to a unique server name.
<Proxy "fcgi://localhost/" enablereuse=on max=10>
</Proxy>
# Pass the Authorization header
SetEnvIfNoCase ^Authorization$ "(.+)" HTTP_AUTHORIZATION=$1
```

This configuration file enables the connection to php-fpm for all vhosts. It is possible to separate (chroot) each Apache vhost and redirect it to an appropriate php-fpm pool for more security.

Enable the necessary modules:

```bash
$ a2dismod mpm_prefork php5
$ a2enmod mpm_event proxy_fcgi
$ a2enconf phpfcgi
$ service apache2 restart
```

## Debian Stretch

With Debian, and now PHP 7, when you install `php7.0-fpm`,
a configuration is automatically created in the `/etc/apache2/conf-available/php7.0-fpm.conf` file:

```apache
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

You just need to enable it (`a2enconf php7.0-fpm`).

## Special behavior of php-fpm

It is possible to act on PHP environment variables per project (directory)
by using a `.user.ini` file.
The change will be taken into account at each restart of the php-fpm service or
every 5 minutes.

Here is an example:

```ini
upload_max_filesize=513M
post_max_size=513M
memory_limit=512M
mbstring.func_overload=0
always_populate_raw_post_data=-1
default_charset='UTF-8'
date.timezone='Europe/Paris'
env[APPLICATION_ENV] = "production"
```