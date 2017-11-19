---
layout: post
categories: linux usb3 uasp
date: 2016-03-02 21:55:00 +0200
lang: fr
ref: 2016-03-02-linux-usb3-uasp-disable
title:  "Linux et USB3, Désactiver le protocol UASP"
---

Après avoir acheter un joli ICY DOCK Black Vortex (MB174SU3S-4SB) pour pouvoir mettre 4 disques pour faire des sauvegardes, l'utilisation de rsync intensif provoque des ralentissements/déconnexion.
Un petit tour avec DMESG est on remarque plein d'erreur.

Petite analyse, le disque est branché en USB3 donc

```bash
$ lusb
Bus 006 Device 002: ID 152d:0567 JMicron Technology Corp. / JMicron USA Technology Corp. JMS567 SATA 6Gb/s bridge
```

```bash
$ lusb -t
Bus 06.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/2p, 5000M
   |__ Port 1: Dev 2, If 0, Class=Mass Storage, Driver=uas, 5000M
```

Nous allons donc désactiver l'utilisation du driver uas afin de repasser sur l'usb-storage

Créer un fichier `/etc/modprobe.d/quirks.conf` avec le contenu suivant  
```
options usb-storage quirks=0x152d:0x0567:u
```

Le format de l'option quirks est la suivante `quirks=<VID>:<PID>:u` ou <VID> = VendorId et <PID> = ProductId du périphérique USB. Le drapeau 'u' désactive le driver uas.

Ensuite

```bash
$ depmod -ae
$ update-initramfs -u
```

Puis un reboot. En espérant que cela peut aider.
