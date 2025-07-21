---
translationKey: "2017-07-16-ansible-ansistrano-deploy-php-apps"
categories: php deploy ansible ansistrano
date: "2017-07-16T23:00:00Z"
ref: 2017-07-16-ansistrano-deploy-php-apps
title: "Déployer des applications PHP avec Ansistrano"
---

Déployer des applications PHP avec [Ansistrano](https://ansistrano.com/)

# Installer Ansible

> $ sudo apt-get install software-properties-common
> $ sudo apt-add-repository ppa:ansible/ansible
> $ sudo apt-get update
> $ sudo apt-get install ansible

Dans /etc/ansible/ansible.cfg
 [defaults]
 host_key_checking = false

# Installer Ansistrano
> $ ansible-galaxy install carlosbuenosvinos.ansistrano-deploy carlosbuenosvinos.ansistrano-rollback

Mise à jour
Si vous voulez mettre à jour le rôle, vous devez passer le paramètre --force lors de l'installation. Veuillez vérifier la commande suivante :
> $ ansible-galaxy install --force carlosbuenosvinos.ansistrano-deploy carlosbuenosvinos.ansistrano-rollback

# Dépôt Ansible
Vous pouvez organiser votre dépôt de déploiement par entreprise ou par type de projet.
Je l'appelle "ansible" par exemple, avec la structure de répertoires :

apps/
base/
files/
README.md

# Utilisation
Installez votre projet ("ansible" par exemple) depuis git dans le répertoire personnel de l'utilisateur "deploy"

Créez la clé publique pour l'utilisateur "deploy"
> $ ssh-keygen -t rsa
> $ cp -v ~/.ssh/id_rsa.pub ~/ansible/files/authorized_keys.deploy.pub

J'ai créé un playbook pour initialiser l'hôte distant
- base/user-deploy.yml : Créer l'utilisateur deploy avec les permissions
- base/vhosts.yml : Créer

Sur chaque hôte, créez l'utilisateur "deploy"
> $ ansible-playbook -i {inventory} base/user-deploy.yml --become -k --ask-become-pass --user={premier utilisateur créé à l'installation}

```
---
# Utilisateur de déploiement
- name: Préparer et configurer l'utilisateur "deploy"
  hosts: all
  become: yes
  tasks:
   - name: Ajouter l'utilisateur de déploiement
     user: name=deploy comment="Deploy User" groups=adm,sudo,www-data shell=/bin/bash
   - name: Ajouter la clé autorisée à l'utilisateur de déploiement
     authorized_key: user=deploy key="{{item}}"
     with_file:
     - ../files/authorized_keys.deploy.pub
   - name: S'assurer que /etc/sudoers.d est scanné par sudo
     # Une erreur utilise pkexec visudo
     action: lineinfile dest=/etc/sudoers regexp="#includedir\s+/etc/sudoers.d" line="#includedir /etc/sudoers.d" validate="visudo -cf %s"
   - name: Ajouter l'utilisateur de déploiement aux sudoers
     action: 'lineinfile dest=/etc/sudoers.d/deploy state=present create=yes regexp="deploy .*" line="deploy ALL=(ALL) NOPASSWD: ALL" validate="visudo -cf %s"'
   - name: S'assurer que le fichier /etc/sudoers.d/deploy a les bonnes permissions
     action: file path=/etc/sudoers.d/deploy mode=0440 state=file owner=root group=root
```

Vérifier le vhost principal
> $ ansible-playbook -i {inventory} base/vhost.yml

```
---
# Hôte virtuel
- hosts: all
  become: yes
  tasks:
   - name: Créer et vérifier les permissions du répertoire vhost
     file: path=/home/vhosts state=directory mode="u=rwx,g=rwx,o=rx" owner=www-data group=www-data
```

Filtrer pour exécuter uniquement sur un hôte
> $ ansible-playbook -i {inventory} -l 192.168.0.0 playbook.yml


# Gérer un projet à déployer

Nous voulons faire un déploiement progressif, nous devons donc définir comment travailler.
Avec une branche git standard :
- master : production
- develop : pré-production


Voici les commandes pour exécuter un déploiement

Déploiement
> $ ansible-playbook -i hosts -e "ansistrano_release_version=`date -u +%Y%m%d%H%M%SZ`" deploy.yml

Retour en arrière
> $ ansible-playbook -i hosts rollback.yml

Maintenant, allez lire la documentation d'Ansistrano pour voir toutes les étapes du workflow.

```
-- /home/vhosts/my-app.com
|-- current -> /home/vhosts/my-app.com/releases/20100512131539
|-- releases
|   |-- 20100512131539
|   |-- 20100509150741
|   |-- 20100509145325
|-- shared
```

Vous devez effectuer certaines opérations localement et à distance.

Localement :
- Cloner votre projet
- Installer les dépendances
- ... et toutes autres choses

À distance :
- Les fichiers sont automatiquement envoyés (via rsync) à votre emplacement distant
- Nettoyer les fichiers temporaires et chauffer le cache

Exemple avec mon application web
```
/home/deploy/ansible/apps/my-app.com
|-- hosts-prod
|-- deploy.yml
|-- etc/
|-- config/
|-- tasks
|   |-- before-code-update.yml
|   |-- after-code-update.yml
|   |-- before-symlink.yml
|   |-- after-symlink.yml
|   |-- before-cleanup.yml
|   |-- after-cleanup.yml
```

- hosts-prod : contient la définition de votre serveur de production
- deploy.yml : modèle d'ansistrano
- etc/ : fichiers de configuration spécifiques au serveur à placer sur votre serveur distant
- config/ : fichiers d'environnement pour votre projet (les informations secrètes y sont stockées)
- tasks/ : tâches spécifiques

Tous ces fichiers peuvent être spécifiques pour la pré-production.
