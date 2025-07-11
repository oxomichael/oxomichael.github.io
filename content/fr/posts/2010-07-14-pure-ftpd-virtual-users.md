---
translationKey: "2010-07-14-pure-ftpd-virtual-users"
title: PureFTPd et utilisateurs virtuels
date: "2010-07-14T11:00:00Z"
---

## Fichier de configuration des utilisateurs virtuels
```
pureftpd.passwd
pureftpd.pdb
```
Remarque : Utiliser la sauvegarde de ces fichiers, ils sont à placer dans `/etc/pure-ftpd/`

## Configuration de pure-ftpd
Pour chaque options de configuration,il suffit de créer un fichier portant le nom de la variable de configuration et en placeur la valeur sur la première ligne de ce fichier.

Par exemple, en créant le fichier `/etc/pure-ftpd/conf/ChrootEveryone` et en mettant "yes" sur sa première ligne, cela permet de "Chrooter" les utilisateurs. Pour info, cela ajoutera le paramètre "-A" sur la ligne de commandes après avoir rechargé la configuration.

> $ echo "yes" > /etc/pure-ftpd/conf/ChrootEveryone

Pour avoir la liste des fichiers de configuration qu’il est possible de créer ou de modifier et la syntaxe pour les renseigner, il faut faire "man pure-ftpd-wrapper". Il faut faire attention à la casse des caractères lors de la création des fichiers.
Création et gestion des utilisateurs virtuels

Création d'un utilisateur virtuel :
> $ pure-pw useradd toto -u ftpuser -d /LeDossierDeToto

Remarque : Le dossier "LeDossierDeToto" indiqué sera créé automatiquement à la première connexion si le fichier `/etc/pure-ftpd/conf/CreateHomeDir` contient "yes"

Changer le mot de passe:
`pure-pw passwd toto`

Modifier un utilisateur:
`pure-pw usermod toto -d /UnautreDossierPourToto`

Il est possible également de modifier directement le fichier `/etc/pure-ftpd/pureftpd.passwd`
ATTENTION : Après chaque création ou modification d’un utilisateur, il faut générer la base de données avec la commande suivante:
> $ pure-pw mkdb

Pour finir, il faut créer un lien symbolique pour activer l’authentification des utilisateurs virtuels:
```
cd /etc/pure-ftpd/auth/
ln -s ../conf/PureDB 50puredb
```

Remarque : Le dossier `/etc/pure-ftpd/auth/` contient un lien symbolique pour chaque méthode d'authentification acceptée. Pour désactiver l’accès FTP aux comptes du système Linux, il faut mettre "no" dans le fichier `/etc/pure-ftpd/conf/PAMAuthentication` et recharger la configuration
