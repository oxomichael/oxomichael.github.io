---
categories: linux ssh sftp
date: "2010-09-07T11:00:00Z"
lang: fr
ref: 2010-09-07-openssh-sftp
title: SFTP et OpenSSH
---

Les versions récentes de openssh permettent de créer un accès sftp pour transmettre des fichiers.
Il n'est donc plus nécessaire d'installer un serveur ftp.

## Ajouter un nouveau groupe

`$ groupadd sftp`

Pour ajouter un utilisateur seulement pour le sftp

`$ useradd -d /home/sftp/{username} {username} passwd {username} usermod -g sftp {username} usermod -s /bin/false {username}`

## Configuration `/etc/ssh/sshd_config`

```
#Subsystem sftp /usr/lib/openssh/sftp-server
Subsystem sftp internal-sftp
Match group sftp
X11Forwarding no
ChrootDirectory %h
AllowTcpForwarding no
ForceCommand internal-sftp
```

## Réglages des permissions

Un répertoire pour recevoir les fichiers "recv" Un répertoire pour déposer les fichiers "send"

```
$ mkdir /home/sftp/{username}/recv
$ mkdir /home/sftp/{username}/send
$ chown user:sftp -R /home/{username}
$ chown {username}:{username} /home/{username}/recv
$ chown {username}:{username} /home/{username}/send
$ chmod 755 /home/sftp/{username}/recv
$ chmod 755 /home/sftp/{username}/send
```

Attention : il faut respecter les permissions suivantes pour la sécurité et il est impossible d'écrire dans le répertoire parent

`$ chown root:root /home/sftp/username`

Gestion des logs avec informations supplémentaires  
`Subsystem sftp internal-sftp -l INFO`
