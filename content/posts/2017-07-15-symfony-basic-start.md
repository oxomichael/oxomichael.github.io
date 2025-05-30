---
categories: php symfony packages
date: "2017-07-15T23:00:00Z"
lang: en
ref: 2017-07-15-symfony-basic-start
title: Symfony basic start
---
Basic start of a symfony project (version 3)

# Create your project
`$ composer create-project symfony/framework-standard-edition my_project_name lts`

Use LTS version, if you are not reckless.

# Apache
I choose to use Apache with php-fpm.

## Sample configuration

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

## Production configuration

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

# Create your Boilerplate with interesting package  

## Backend

FOS User Bundle
HWIOAuthBundle

## Frontend
Assetic  
`$ composer require symfony/assetic-bundle`  
Documentation: https://symfony.com/doc/current/assetic/asset_management.html

JQuery  
`$ composer require components/jquery`

Bootstrap  
`$ composer require twbs/bootstrap`

FontAwesome  
`$ composer require fortawesome/font-awesome`
