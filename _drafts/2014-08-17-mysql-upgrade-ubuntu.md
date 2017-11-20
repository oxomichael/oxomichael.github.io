---
layout: post
categories: ubuntu mysql upgrade
date: 2014-08-17 11:00:00 +0200
lang: fr
ref: 2014-08-17-ubuntu-mysql-upgrade
title:  "Upgrade MySQL in Ubuntu"
---

Mettre à jour Ubuntu (passer d'une version à une autre) et redémarrer MySQL.....oups cela ne marche pas...car comme moi, le répertoire des données est situé en dehors du standard et donc apparmor n'est pas content.

Pour faire simple et ne plus avoir de problème :
Sauvegarder votre fichier de configuration mysql

sudo cp /etc/mysql/my.cnf /etc/mysql/my.cnf.backup

Modifier la configuration de Apparmor en ajoutant un fichier

sudo nano /etc/apparmor.d/local/usr.sbin.mysqld

Ce qui ajoute un fichier spécifique pour la machine local contenant des paramètres supplémentaires, sans changer la configuration de base.
Contenu du fichier :

# Site-specific additions and overrides for usr.sbin.mysqld.
# For more details, please see /etc/apparmor.d/local/README.

/dbs/mysql/ r,
/dbs/mysql/** rwk

Et hop faire les mises à jour.
