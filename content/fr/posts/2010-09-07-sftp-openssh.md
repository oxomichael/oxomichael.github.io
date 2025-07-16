---
translationKey: "2010-09-07-sftp-openssh"
categories: ["linux", "ssh", "sftp"]
date: "2010-09-07T11:00:00Z"
title: "Configurer un accès SFTP sécurisé avec OpenSSH"
---

SFTP (SSH File Transfer Protocol) est une méthode sécurisée pour transférer des fichiers sur un réseau. Contrairement à FTP, toutes les données, y compris les identifiants, sont chiffrées. Les versions modernes d'OpenSSH intègrent un serveur SFTP, ce qui permet de mettre en place des accès restreints au transfert de fichiers sans avoir besoin d'un shell SSH complet.

Ce guide explique comment créer un utilisateur limité exclusivement à un accès SFTP et confiné à son répertoire personnel (chroot).

## 1. Créer un groupe dédié

Il est recommandé de créer un groupe spécifique pour les utilisateurs SFTP. Cela facilite la gestion des permissions et l'application des règles dans la configuration SSH.

```bash
sudo groupadd sftp
```

## 2. Créer un utilisateur SFTP

Nous allons maintenant créer un utilisateur qui sera membre de ce groupe. Cet utilisateur n'aura pas de shell de connexion valide pour des raisons de sécurité.

```bash
# Crée l'utilisateur, son répertoire personnel, et l'ajoute au groupe sftp
sudo useradd -m -d /home/sftp/mon_user -g sftp mon_user

# Assigne un mot de passe à l'utilisateur
sudo passwd mon_user

# Définit le shell de l'utilisateur à /bin/false pour interdire les connexions SSH interactives
sudo usermod -s /bin/false mon_user
```
- `mon_user` : Le nom de l'utilisateur à créer.
- `-d /home/sftp/mon_user` : Le répertoire personnel de l'utilisateur.
- `-g sftp` : Le groupe principal de l'utilisateur.
- `-s /bin/false` : Empêche l'utilisateur d'ouvrir une session shell.

## 3. Configurer le serveur SSH

Modifiez le fichier de configuration du serveur SSH, généralement situé à `/etc/ssh/sshd_config`, pour y ajouter les règles pour notre groupe `sftp`.

Commentez la ligne `Subsystem sftp` existante et ajoutez les lignes suivantes à la fin du fichier :

```sshd-config
# Commentez cette ligne si elle existe
# Subsystem sftp /usr/lib/openssh/sftp-server

# Utilisez le sous-système SFTP interne
Subsystem sftp internal-sftp

# Applique les règles suivantes uniquement pour les membres du groupe sftp
Match Group sftp
    # Force la racine du système de fichiers à être le répertoire personnel de l'utilisateur
    ChrootDirectory %h
    # Interdit le tunneling X11
    X11Forwarding no
    # Interdit la redirection de ports TCP
    AllowTcpForwarding no
    # Force l'exécution de la commande SFTP interne, ignorant toute commande envoyée par le client
    ForceCommand internal-sftp
```
- `ChrootDirectory %h` : C'est la directive clé. Elle confine l'utilisateur à son répertoire personnel (`%h` est un jeton qui représente le répertoire home de l'utilisateur).

N'oubliez pas de redémarrer le service SSH pour appliquer les changements :
```bash
sudo systemctl restart sshd
```

## 4. Ajuster les permissions des répertoires

Pour que le `ChrootDirectory` fonctionne, le répertoire personnel de l'utilisateur doit appartenir à `root` et ne doit pas être inscriptible par d'autres utilisateurs. C'est une contrainte de sécurité d'OpenSSH.

L'utilisateur ne pourra donc pas écrire directement dans sa racine (`/home/sftp/mon_user`). Nous devons créer des sous-répertoires et lui en donner la propriété.

```bash
# Le répertoire personnel doit appartenir à root
sudo chown root:root /home/sftp/mon_user

# Créez des sous-répertoires pour que l'utilisateur puisse transférer des fichiers
sudo mkdir /home/sftp/mon_user/upload
sudo mkdir /home/sftp/mon_user/download

# Donnez la propriété de ces sous-répertoires à l'utilisateur
sudo chown mon_user:sftp /home/sftp/mon_user/upload
sudo chown mon_user:sftp /home/sftp/mon_user/download

# Ajustez les permissions si nécessaire
sudo chmod 755 /home/sftp/mon_user/upload
sudo chmod 755 /home/sftp/mon_user/download
```

## 5. Améliorer les logs (Optionnel)

Pour obtenir des logs plus détaillés sur les activités SFTP, vous pouvez modifier la ligne `Subsystem` dans `sshd_config` :

```sshd-config
Subsystem sftp internal-sftp -l INFO
```
Cela enregistrera les opérations comme l'ouverture de session, le téléchargement ou l'envoi de fichiers, ce qui est très utile pour le suivi.