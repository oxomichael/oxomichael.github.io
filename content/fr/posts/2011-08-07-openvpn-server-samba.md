---
translationKey: "2011-08-07-openvpn-server-samba"
categories: linux openvpn server
date: "2011-08-07T11:00:00Z"
ref: 2014-08-17-openvpn-server-samba
title: OpenVPN Server with Samba Share
---

Pour avoir un accès sécurisé, j'ai débuté l'installation d'OpenVPN en commençant simplement.
Je souhaite avoir accès par l'intermédiaire du VPN, au réseau interne et que le flux HTTP passe par lui.
Sur le réseau local, nous sommes en 192.168.0.0/24 avec un serveur dnsmasq pour gérer les requêtes DNS qui me permettent de faire mes propres entrées.
Donc pour récapituler la machine qui a dnsmasq et OpenVPN sera nommé "Serveur", avec sur le réseau local l'IP 192.168.0.10

## Installation
On installe OpenVPN suivant sa distribution et on passe à la configuration.
Un HOWTO est disponible sur http://openvpn.net/index.php/open-source/documentation/howto.html#quick

La première chose à faire est de copier les fichiers servant à la configuration dans votre répertoire personnel afin de ne pas perdre les fichiers générés lors de mise à jour, etc.

`cp /usr/share/doc/openvpn/examples/easy-rsa ~/openvpn/ -R`

L'ensemble des commandes s’exécuteront dans
`cd ~/openvpn/2.0`

## Générer le Certificate Authority (CA) et la clé

Éditer le fichier vars
```
export KEY_COUNTRY=FR
export KEY_PROVINCE=France
export KEY_CITY=ville
export KEY_ORG=societe
export KEY_EMAIL=xxxxxxxxx@xxx.com
```

Initialiser le PKI.
```
. ./vars
./clean-all
./build-ca
```

La commande finale `build-ca` va construire le CA (certificate d'authorité) et la clé en invoquant les paramètres entrées dans le fichier vars. Le seul paramètre a rentrer et le `Common Name` (nom de domaine du serveur ou autre).

## Générer le certificat et la clé privé pour le serveur

`./build-key-server server`

Comme dans l'étape précédente, les paramètres sont prédéfinies, Mais lorsque l'on vous demande le Common Name entrer "server". Les deux demandes suivantes requiert des réponses positives, "Sign the certificate? [y/n]" and "1 out of 1 certificate requests certified, commit? [y/n]".

## Générer un certificat et une clé pour un client

Déterminer une méthode pour nommer les clients

`./build-key nomclient`

Si vous souhaitez protéger la clé par un mot de passe, utilisé le script `build-key-pass`.
Lors de la génération, ne pas oubliez d'utiliser pour le `Common Name`, le même nom que dans l'appel du script et toujours utiliser un nom unique pour chaque client.

## Générer les paramètres Diffie Hellman

`./build-dh`

Output :
```
Generating DH parameters, 1024 bit long safe prime, generator 2
This is going to take a long time
.................+...........................................
...................+.............+.................+.........
......................................
```

## Copie des fichiers pour le serveur

`cp keys/dh1024.pem keys/ca.crt keys/server.crt keys/server.key /etc/openvpn/`

## Création du fichier de configuration pour le serveur

Des exemples de fichier de configuration sont présent dans `/usr/share/doc/openvpn/examples/sample-config-files` et le placer dans `/etc/openvpn/`.
Editer donc le fichier `/etc/openvpn/server.conf`

Quelques possibilités d'options:

    En cas d'utilisation d'Ethernet bridgé, il faut utiliser server-bridge et dev tap au lieu de server et dev tun
    Si le serveur doit écouter sur un port TCP au lieu d'UDP, il faut mettre proto tcp au lieu de proto udp
    Si l'adresse IP virtuelle utilisée doit être différente de 10.8.0.0/24, il faut modifier la directive server. Ne jamais oublier que la plage d'adresse IP virtuelles doit être inutilisée sur les réseaux de chaque côté du VPN.
    Pour que les clients soient capables de s'atteindre à travers le VPN, il faut décommenter la directive client-to-client. Par défaut, les clients ne peuvent atteindre que le serveur.
    Pour augmenter la sécurité, il est possible de décommenter les directives user nobody et group nobody. Diverses options de cryptage et de sécurité peuvent aussi être modifiées. (Le groupe nobody n'existe pas dans ubuntu, il faut donc mettre nogroup)

## Communiquer avec le réseau interne
`push "route 192.168.0.0 255.255.255.0"`

## Activer l'IP Forwarding dans le noyau
> $ echo 1 > /proc/sys/net/ipv4/ip_forward

## Faire passer tout le trafic réseau par le VPN
`push "redirect-gateway def1 bypass-dhcp"`

Comme on fait passer tout le trafic réseau par le VPN, on utilise pousse la configuration DNS pour le client (dnsmasq écoute sur l'adresse IP 10.8.0.1)
`push "dhcp-option DNS 10.8.0.1"`

Après avoir mis toutes ces options, exécuter la commande suivante
> $ iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE

Cette commande fera que le trafic IP sortant du sous-réseau 10.8.0.0/24 vers l’extérieure pourra revenir routé vers le VPN avec la bonne destination.

/etc/network/interfaces
```
iface eth0 inet static
  ...
  post-up /sbin/iptables -t nat -A POSTROUTING -s 10.8.37.0/24 -o eth0 -j MASQUERADE
```

/etc/network/if-up.d/iptables
```
#!/bin/sh
iptables -t nat -A POSTROUTING -s 10.8.250.0/24 -o eth0 -j MASQUERADE
```

Ne pas oublier de redémarrer OpenVPN `/etc/init.d/openvpn restart`

## Création du fichier de configuration pour un client

On va prendre la configuration d'une machine sous winwdows
On installe OpenVPN http://openvpn.net/index.php/open-source/downloads.html
Une nouvelle GUI plus récente est disponible ici, il suffira de copier le fichier dans \bin et de refaire un raccourci.

Dans le répertoire config, copier les fichiers : ca.crt, nomclient.crt, nomclient.key

Copier et modifier le fichier de config présent dans sample-config, client.ovpn dans le repertoire config. Et reproduire la configuration server.

Placer la directive remote suivant votre configuration remote domain.tld 1194 ou remote IP_EXTERNE 1194
Définir les noms de fichier
```
    ca ca.crt
    cert nomclient.crt
    key nomclient.key
```

Lancer la GUI d'OpenVPN, et se connecter. Normalement tout devrait fonctionner.

## Partage réseau
Suite à mon exemple de configuration d'OpenVPN, une question se pose, comment rendre disponible les différents partages Samba.
Il existe le serveur WINS. Voir la doc de samba pour plus d'explication avec un ou plusieurs serveur Samba. Concretement au lieu d'utiliser, dans la plupart des case du broadcast pour lister les serveurs qui propose des partages réseaux, on déclare un serveur

Donc le serveur Samba qui fera office de serveur WINS aura dans sa configuration
```
[global]
	wins support = yes
	name resolve order = wins lmhosts hosts bcast
```

Pour les autres serveurs, afin qu'ils soient déclarés.
```
[global]
	wins server = 192.168.0.1
```

Notre serveur fait donc office de serveur WINS, DNS, OpenVPN et il faut donc modifier un peu la configuration en ajoutant cette ligne

`push "dhcp-option WINS 10.8.0.1"`
