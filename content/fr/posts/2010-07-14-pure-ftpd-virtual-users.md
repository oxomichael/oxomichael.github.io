---
translationKey: "2010-07-14-pure-ftpd-virtual-users"
title: "Mise en place de Pure-FTPd avec des utilisateurs virtuels"
date: "2010-07-14T11:00:00Z"
---

Pure-FTPd est un serveur FTP sécurisé et facile à configurer. Une de ses fonctionnalités les plus utiles est la gestion des **utilisateurs virtuels**. Contrairement aux utilisateurs système, les utilisateurs virtuels n'existent que pour le service FTP. Ils sont stockés dans une base de données dédiée, ce qui permet de ne pas créer de vrais comptes sur le système d'exploitation et d'améliorer ainsi la sécurité.

Ce guide vous montrera comment configurer Pure-FTPd pour utiliser une base de données d'utilisateurs virtuels.

## 1. Fichiers des utilisateurs virtuels

La configuration des utilisateurs virtuels repose sur deux fichiers principaux, généralement situés dans `/etc/pure-ftpd/`:

-   `pureftpd.passwd`: Un fichier texte contenant les informations des utilisateurs dans un format lisible. C'est ce fichier que vous pouvez modifier manuellement si besoin.
-   `pureftpd.pdb`: La base de données binaire générée à partir du fichier `pureftpd.passwd`. C'est ce fichier que Pure-FTPd utilise pour authentifier les utilisateurs.

Il est conseillé de sauvegarder régulièrement ces deux fichiers.

## 2. Configuration de Pure-FTPd

La configuration de Pure-FTPd se fait via des fichiers individuels dans le répertoire `/etc/pure-ftpd/conf/`. Chaque fichier représente une option de configuration.

Par exemple, pour "chrooter" tous les utilisateurs (les confiner à leur répertoire personnel), on crée un fichier `ChrootEveryone` avec la valeur `yes`.

```bash
echo "yes" > /etc/pure-ftpd/conf/ChrootEveryone
```

Cette méthode ajoute le paramètre `-A` à la commande de lancement de Pure-FTPd après rechargement. Pour découvrir toutes les options disponibles, consultez le manuel :

```bash
man pure-ftpd-wrapper
```

Faites attention à la casse lors de la création des fichiers de configuration.

## 3. Gestion des utilisateurs virtuels

La commande `pure-pw` est l'outil principal pour gérer les utilisateurs virtuels.

### Créer un utilisateur

Pour ajouter un nouvel utilisateur :

```bash
pure-pw useradd mon_utilisateur -u ftpuser -d /home/ftp/mon_utilisateur
```

-   `mon_utilisateur` : Le nom de l'utilisateur virtuel.
-   `-u ftpuser` : Associe l'utilisateur virtuel à un utilisateur système réel (`ftpuser` dans cet exemple). Les fichiers téléversés appartiendront à `ftpuser`.
-   `-d /home/ftp/mon_utilisateur` : Le répertoire personnel de l'utilisateur.

**Note** : Pour que le répertoire personnel soit créé automatiquement à la première connexion, assurez-vous que le fichier `/etc/pure-ftpd/conf/CreateHomeDir` contienne `yes`.

### Modifier un utilisateur

Pour changer le mot de passe :

```bash
pure-pw passwd mon_utilisateur
```

Pour modifier les informations d'un utilisateur (par exemple, son répertoire) :

```bash
pure-pw usermod mon_utilisateur -d /home/ftp/un_autre_dossier
```

### Mettre à jour la base de données

**Important** : Après chaque ajout ou modification d'utilisateur avec `pure-pw` (ou en éditant `pureftpd.passwd`), vous devez impérativement mettre à jour la base de données `.pdb` :

```bash
pure-pw mkdb
```

## 4. Activer l'authentification par base de données

Pour que Pure-FTPd utilise notre base d'utilisateurs virtuels, il faut activer la méthode d'authentification `PureDB`. Cela se fait en créant un lien symbolique dans le dossier `/etc/pure-ftpd/auth/`.

```bash
cd /etc/pure-ftpd/auth/
ln -s ../conf/PureDB 50puredb
```

Le nom `50puredb` définit l'ordre de priorité. Les nombres les plus bas sont testés en premier.

## 5. Désactiver les autres authentifications (Optionnel)

Pour renforcer la sécurité, il est recommandé de désactiver les autres méthodes d'authentification, comme l'authentification des utilisateurs système via PAM. Pour ce faire, modifiez le fichier de configuration correspondant :

```bash
echo "no" > /etc/pure-ftpd/conf/PAMAuthentication
```

N'oubliez pas de recharger la configuration de Pure-FTPd pour que les changements soient pris en compte. La méthode varie selon votre système (ex: `service pure-ftpd restart` ou `systemctl restart pure-ftpd`).