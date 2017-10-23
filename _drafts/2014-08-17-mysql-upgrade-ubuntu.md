Mettre à jour Ubuntu (passer d'une version à une autre) et redémarrer MySQL.....oups cela ne marche pas...car comme moi, le répertoire des données est situé en dehors du standard et donc apparmor n'est pas content.

Pour faire simple et ne plus avoir de problème :
Sauvegarder votre fichier de configuration mysql
1

sudo cp /etc/mysql/my.cnf /etc/mysql/my.cnf.backup

Modifier la configuration de Apparmor en ajoutant un fichier
1

sudo nano /etc/apparmor.d/local/usr.sbin.mysqld

Ce qui ajoute un fichier spécifique pour la machine local contenant des paramètres supplémentaires, sans changer la configuration de base.
Contenu du fichier :
1
2
3
4
5

# Site-specific additions and overrides for usr.sbin.mysqld.
# For more details, please see /etc/apparmor.d/local/README.

/dbs/mysql/ r,
/dbs/mysql/** rwk

Et hop faire les mises à jour.
