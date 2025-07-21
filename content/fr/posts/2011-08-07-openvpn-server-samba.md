---
translationKey: "2011-08-07-openvpn-server-samba"
categories:
- linux
- openvpn
- server
date: "2011-08-07T11:00:00Z"
ref: 2014-08-17-openvpn-server-samba
title: "Configurer un serveur OpenVPN avec partage Samba"
---

> *Cet article a été initialement écrit en 2011. Il a été revu pour améliorer la clarté et mettre à jour certaines configurations pour des pratiques plus modernes. Les technologies comme OpenVPN et Samba évoluent constamment, consultez toujours la documentation officielle pour les dernières recommandations.*

Ce guide a pour but de mettre en place un serveur OpenVPN simple pour sécuriser l'accès à un réseau local et y faire transiter tout le trafic web.

Le réseau local utilise la plage d'adresses `192.168.0.0/24`. Un serveur sur ce réseau, avec l'adresse IP `192.168.0.10`, hébergera OpenVPN et un serveur DNS (dnsmasq) pour gérer les requêtes DNS locales. Nous l'appellerons "Serveur".

## Installation d'OpenVPN

Commencez par installer le paquet OpenVPN adapté à votre distribution Linux. Un HOWTO détaillé est disponible sur le [site officiel d'OpenVPN](http://openvpn.net/index.php/open-source/documentation/howto.html#quick).

Pour la génération des certificats et des clés, nous utiliserons `easy-rsa`. Sur les systèmes modernes, il est recommandé de l'installer comme un paquet séparé :
```bash
sudo apt-get update
sudo apt-get install easy-rsa
```

Ensuite, copiez le répertoire `easy-rsa` dans un répertoire de travail pour ne pas altérer les fichiers originaux.
```bash
cp -r /usr/share/easy-rsa/ ~/openvpn-ca
cd ~/openvpn-ca
```
> **Note :** Les commandes suivantes sont basées sur `easy-rsa` version 3. Si vous utilisez une version plus ancienne, les commandes peuvent différer (`build-ca`, `build-key-server`, etc.).

## Génération des certificats et clés

### Initialisation de l'autorité de certification (CA)

Créez un fichier `vars` à la racine de `~/openvpn-ca` pour définir les variables de vos certificats.
```
set_var EASYRSA_REQ_COUNTRY    "FR"
set_var EASYRSA_REQ_PROVINCE   "France"
set_var EASYRSA_REQ_CITY       "VotreVille"
set_var EASYRSA_REQ_ORG        "VotreOrganisation"
set_var EASYRSA_REQ_EMAIL      "contact@example.com"
set_var EASYRSA_REQ_OU         "IT"
```

Initialisez la nouvelle infrastructure à clés publiques (PKI) :
```bash
./easyrsa init-pki
```

Construisez le certificat de l'autorité de certification (CA). Choisissez un nom commun (Common Name), par exemple le nom de votre domaine.
```bash
./easyrsa build-ca
```

### Certificat et clé pour le serveur

Générez le certificat et la clé privée pour le serveur OpenVPN.
```bash
./easyrsa build-server-full server nopass
```
Le `Common Name` sera "server". L'option `nopass` crée une clé privée non protégée par un mot de passe.

### Certificat et clé pour un client

Générez un certificat et une clé pour chaque client qui se connectera au VPN.
```bash
./easyrsa build-client-full nom_client nopass
```
Remplacez `nom_client` par un nom unique pour chaque client. Si vous souhaitez protéger la clé par un mot de passe, omettez l'option `nopass`.

### Paramètres Diffie-Hellman

Générez les paramètres Diffie-Hellman. Utilisez une longueur de 2048 bits pour une sécurité adéquate.
```bash
./easyrsa gen-dh
```

### Clé `tls-auth` pour plus de sécurité

Pour renforcer la sécurité, générez une clé `tls-auth` qui aidera à protéger le serveur contre les attaques DoS.
```bash
openvpn --genkey --secret pki/ta.key
```

## Configuration du serveur OpenVPN

Copiez les fichiers générés vers le répertoire de configuration d'OpenVPN.
```bash
sudo cp pki/ca.crt pki/issued/server.crt pki/private/server.key pki/dh.pem pki/ta.key /etc/openvpn/server/
```

Créez le fichier de configuration `/etc/openvpn/server/server.conf`. Vous pouvez vous inspirer des exemples fournis avec OpenVPN.

Voici une configuration de base :
```ini
port 1194
proto udp
dev tun

ca ca.crt
cert server.crt
key server.key
dh dh.pem

server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt

# Pousser les routes pour que les clients accèdent au réseau local
push "route 192.168.0.0 255.255.255.0"

# Rediriger tout le trafic du client à travers le VPN
push "redirect-gateway def1 bypass-dhcp"

# Pousser les serveurs DNS aux clients
push "dhcp-option DNS 10.8.0.1" # DNS interne
push "dhcp-option DNS 8.8.8.8"   # DNS public en fallback

client-to-client
keepalive 10 120

# Sécurité
tls-auth ta.key 0 # Côté serveur
cipher AES-256-GCM
auth SHA256

user nobody
group nogroup
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
verb 3
```

### Activation de l'IP Forwarding

Pour que le serveur puisse router les paquets des clients VPN vers Internet, activez l'IP forwarding.
Pour une activation temporaire :
```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```
Pour rendre le changement permanent, éditez `/etc/sysctl.conf` et décommentez ou ajoutez la ligne :
```
net.ipv4.ip_forward=1
```
Appliquez la modification avec `sudo sysctl -p`.

### Configuration du pare-feu (NAT)

Ajoutez une règle `iptables` pour que le trafic sortant du VPN soit correctement routé.
```bash
sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
```
Remplacez `eth0` par votre interface réseau principale.

Pour rendre cette règle persistante, installez `iptables-persistent` :
```bash
sudo apt-get install iptables-persistent
sudo netfilter-persistent save
```

Démarrez ensuite le serveur OpenVPN :
```bash
sudo systemctl start openvpn-server@server
```

## Configuration d'un client

Sur la machine cliente, installez un client OpenVPN (comme OpenVPN Connect). Créez un fichier de configuration `client.ovpn` avec le contenu suivant :
```ini
client
dev tun
proto udp

remote votre_serveur_ip 1194 # Remplacez par l'IP ou le domaine de votre serveur

resolv-retry infinite
nobind
persist-key
persist-tun

# Sécurité
remote-cert-tls server
cipher AES-256-GCM
auth SHA256
key-direction 1 # Doit être présent avec tls-auth

# Fichiers de clés (à placer dans le même répertoire que le .ovpn)
# ou utiliser la syntaxe <ca>...</ca> pour les inclure dans le fichier
ca ca.crt
cert nom_client.crt
key nom_client.key
tls-auth ta.key 1
```
Vous devrez copier `ca.crt`, `nom_client.crt`, `nom_client.key` et `ta.key` sur la machine cliente.

## Partage de fichiers avec Samba (via WINS)

> **Note :** WINS est une technologie de résolution de noms plus ancienne, principalement pour les anciennes versions de Windows. Dans les réseaux modernes, il est préférable de s'appuyer sur le DNS. Cette section est conservée à titre informatif.

Pour permettre aux clients VPN de découvrir les partages Samba sur le réseau local, vous pouvez utiliser un serveur WINS.

Sur votre serveur Samba principal (qui sera aussi le serveur WINS), modifiez `/etc/samba/smb.conf` :
```ini
[global]
    # ... autres paramètres
    wins support = yes
    name resolve order = wins lmhosts hosts bcast
```

Sur les autres serveurs Samba du réseau, déclarez le serveur WINS :
```ini
[global]
    # ... autres paramètres
    wins server = 192.168.0.10 # IP du serveur WINS
```

Enfin, poussez la configuration du serveur WINS aux clients OpenVPN en ajoutant cette ligne à votre `server.conf` :
```ini
push "dhcp-option WINS 10.8.0.1"
```
Redémarrez les services OpenVPN et Samba pour appliquer les changements.