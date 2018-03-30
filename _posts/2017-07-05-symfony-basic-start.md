---
layout: post
title:  "Symfony basic start"
date:   2017-07-05 23:00:00 +0200
categories: php symfony packages
ref: 2017-07-05-symfony-basic-start
---

Basic start of a symfony project (version 3.x)

## Create your project
`$ composer create-project symfony/framework-standard-edition my_project_name`

If you want to use an LTS version you can specify a version.
`$ composer create-project symfony/framework-standard-edition my_project_name "2.8.*"`

## Use Git to store your project
You can see a `.gitignore` file, open it to see which files of your project is ignore.

Simply use these commands to make your initial commit
```
git init
git add .
git commit -m "Initial commit"
```

## Apache with PHP-FPM

### Sample configuration
```
<VirtualHost *:80>
	ServerName symfony.local
	ServerAdmin webmaster@localhost

	DocumentRoot /var/www/project/web
	<Directory /var/www/project/web/>
		Options FollowSymLinks Indexes
		AllowOverride all
		Require all granted
	</Directory>

	# Redirect to local php-fpm if mod_php is not available
	<IfModule !mod_php7.c>
  	<IfModule proxy_fcgi_module>
	# Enable http authorization headers
      	<IfModule setenvif_module>
		SetEnvIfNoCase ^Authorization$ "(.+)" HTTP_AUTHORIZATION=$1
	</IfModule>
	<FilesMatch ".+\.ph(p[3457]?|t|tml)$">
  		#SetHandler "proxy:unix:/run/php/php7.0-fpm.sock|fcgi://localhost"
  		SetHandler "proxy:fcgi://fpm70:9000"
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

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### Production configuration
But in order to use in a production server for better performance:
 - Disabling .htaccess support
 - Disabling other items

```
<VirtualHost *:80>
	ServerName symfony.local
	ServerAdmin webmaster@localhost

	DocumentRoot /var/www/project/web
	<Directory /var/www/project/web/>
    	AllowOverride None
		Order Allow,Deny
		Allow from All

		<IfModule mod_rewrite.c>
			Options -MultiViews
			RewriteEngine On
			RewriteCond %{REQUEST_FILENAME} !-f
			RewriteRule ^(.*)$ app.php [QSA,L]
		</IfModule>
	</Directory>

	# uncomment the following lines if you install assets as symlinks
	# or run into problems when compiling LESS/Sass/CoffeeScript assets
	# <Directory /var/www/project>
	#     Options FollowSymlinks
	# </Directory>

	# optionally disable the RewriteEngine for the asset directories
	# which will allow apache to simply reply with a 404 when files are
	# not found instead of passing the request into the full symfony stack
	<Directory /var/www/project/web/bundles>
		<IfModule mod_rewrite.c>
			RewriteEngine Off
		</IfModule>
	</Directory>

	# Redirect to local php-fpm if mod_php is not available
	<IfModule !mod_php7.c>
  	<IfModule proxy_fcgi_module>
      # Enable http authorization headers
      <IfModule setenvif_module>
      SetEnvIfNoCase ^Authorization$ "(.+)" HTTP_AUTHORIZATION=$1
    </IfModule>

    <FilesMatch ".+\.ph(p[3457]?|t|tml)$">
  		#SetHandler "proxy:unix:/run/php/php7.0-fpm.sock|fcgi://localhost"
  		SetHandler "proxy:fcgi://fpm70:9000"
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

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

It's always interesting to use the same configuration in dev and in production.
I prefer to not discover a configuration problem in production, so i always
keep my dev environment as my production.

## Specific development configuration

I use vagrant or docker for dev environment so i have to tweak `app_dev.php`




## Frontend

### Assetic
`composer require symfony/assetic-bundle`
=> https://symfony.com/doc/current/assetic/asset_management.html

### JQuery
```
composer require components/jquery
composer require components/jqueryui
```

### Bootstrap
`composer require twbs/bootstrap`

### FontAwesome
`composer require fortawesome/font-awesome`

## Dev
```
php bin/console assets:install --symlink
php bin/console assetic:dump
```

## Prod
```
php bin/console assets:install --symlink --env prod
php app/console assetic:dump --env prod
```

Redis Session Handler

Monolog

## Backend

HWIOAuthBundle

Event
http://symfony.com/doc/current/event_dispatcher.html
