---
translationKey: "2017-06-30-create-vagrant-box-from-scratch"
categories: vagrant debian
date: "2017-06-30T21:07:00Z"
ref: 2017-06-30-create-vagrant-box-from-scratch
title: Créer une box Vagrant à partir de zéro
---

## Box Debian Stretch 64

*   Installer VirtualBox
*   Installer Vagrant
*   Télécharger le CD d'installation réseau Debian
*   Créer la machine virtuelle

### Machine Virtuelle

#### Matériel
*   Nom : vagrant-stretch64
*   Type : Linux
*   Version : vagrant-stretch64
*   Taille mémoire : 512Mo
*   Nouveau disque virtuel :
    *   Type : VMDK
    *   Taille : 8Go
*   Désactiver l'audio
*   Désactiver l'USB
*   Monter l'ISO du CD

#### Installation
Choisir l'installation graphique
*   Sélectionner une langue
*   Sélectionner votre emplacement
*   Configurer les locales
*   Configurer le clavier
*   Configurer le réseau
    *   Nom d'hôte : stretch64
*   Configurer les utilisateurs et mots de passe
    *   Entrer "vagrant" comme mot de passe root
    *   Entrer "vagrant" comme nom complet
    *   Nouvel utilisateur "vagrant"
    *   et aussi "vagrant" comme mot de passe
*   Configurer l'horloge
*   Partitionner les disques
    *   Guidé - utiliser le disque entier
    *   Laisser SCSIl (0, 0, 0) (sda) - 8.6 GB ATA VBOX HARDDISK présélectionné et continuer.
    *   Tous les fichiers dans une seule partition
    *   Terminer le partitionnement et écrire les changements sur le disque.
*   Installer le système de base
*   Configurer le gestionnaire de paquets
*   Sélection des logiciels
    *   Veuillez désactiver toutes les options, sauf *utilitaires usuels du système*.
*   Terminer l'installation

#### Configuration
*   Installer sudo

 > $ su

 > $ apt-get install -y sudo

*   Donner les permissions sudo à vagrant

 > $ visudo -f /etc/sudoers.d/vagrant

*   Ajouter la ligne suivante pour autoriser vagrant à utiliser sudo sans mot de passe
 > `vagrant ALL=(ALL) NOPASSWD:ALL`

*   Quitter et déconnecter l'utilisateur
*   Mettre à jour et monter de version

 > $ sudo apt-get update && sudo apt-get upgrade

*   Installer les paquets de base

 > $ sudo apt-get install -y build-essential module-assistant

 > $ sudo module-assistant prepare

 > $ sudo apt-get install -y zerofree openssh-server

 *   Configuration SSH

 Éditer /etc/ssh/sshd_config, décommenter `AuthorizedKeysFile %h/.ssh/authorized_keys`

 > $ mkdir -p /home/vagrant/.ssh

 > $ chmod 0700 /home/vagrant/.ssh

 > $ wget --no-check-certificate \
  https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub \
  -O /home/vagrant/.ssh/authorized_keys

 > $ chmod 0600 /home/vagrant/.ssh/authorized_keys

 > $ chown -R vagrant /home/vagrant/.ssh

*   Redémarrer ssh

 > $ sudo service ssh Restart

*   Installer les additions invité

 > $ sudo mount /dev/cdrom /mnt

 > $ cd /mnt

 > $ sudo ./VBoxLinuxAdditions.run

*   Nettoyage

 > $ sudo apt-get autoremove && sudo apt-get clean

*   Zerofree

  Se connecter en tant que root

 > $ init 1

 > $ mount -o remount,ro /dev/sda1 /

 > $ zerofree /dev/sda1

*   Redémarrer

 > $ shutdown -h now

*   Empaqueter votre machine

 > $ vagrant package --base vagrant-stretch64

## Utiliser votre box personnalisée

> $ vagrant box add vagrant-stretch64 package.box

> $ vagrant init vagrant-stretch64

> $ vagrant up
