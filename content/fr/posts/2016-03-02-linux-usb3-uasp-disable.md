---
translationKey: "2016-03-02-linux-usb3-uasp-disable"
categories:
- linux
- usb3
- uasp
date: "2016-03-02T21:55:00Z"
ref: 2016-03-02-linux-usb3-uasp-disable
title: "Linux et USB 3.0 : Désactiver le protocole UASP pour un périphérique"
---

Après avoir acheté un boîtier ICY DOCK Black Vortex (MB174SU3S-4SB) pour y placer quatre disques de sauvegarde, j'ai constaté que l'utilisation intensive de `rsync` provoquait des ralentissements et des déconnexions intempestives.

Un rapide coup d'œil aux logs du noyau avec `dmesg` a révélé de nombreuses erreurs liées à l'USB.

Le boîtier est branché en USB 3.0, ce qui est confirmé par la commande `lsusb` :

```bash
$ lsusb
Bus 006 Device 002: ID 152d:0567 JMicron Technology Corp. / JMicron USA Technology Corp. JMS567 SATA 6Gb/s bridge
```

La commande `lsusb -t` montre que le pilote `uas` (USB Attached SCSI) est actuellement utilisé :

```bash
$ lsusb -t
Bus 06.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/2p, 5000M
   |__ Port 1: Dev 2, If 0, Class=Mass Storage, Driver=uas, 5000M
```

Le pilote `uas` est censé offrir de meilleures performances, mais il peut être instable avec certains contrôleurs. Pour résoudre ce problème, nous allons le désactiver pour ce périphérique spécifique afin de forcer le système à utiliser le pilote `usb-storage`, plus ancien mais plus fiable.

Pour ce faire, créez un fichier de configuration pour `modprobe`, par exemple `/etc/modprobe.d/disable-uas.conf`, avec le contenu suivant :

```
options usb-storage quirks=152d:0567:u
```

Le format de l'option `quirks` est `VendorID:ProductID:flags`.
- `152d` (VendorID) et `0567` (ProductID) sont les identifiants de notre périphérique, obtenus plus haut.
- Le flag `u` (pour `UAS_BLACKLIST`) indique au noyau de ne pas utiliser le pilote `uas` pour ce matériel.

Après avoir créé le fichier, mettez à jour les dépendances des modules et l'image initramfs (sur les distributions basées sur Debian/Ubuntu) :

```bash
$ sudo depmod -a
$ sudo update-initramfs -u
```

Enfin, redémarrez votre machine. Le périphérique devrait maintenant utiliser `usb-storage` et être plus stable. En espérant que cela puisse aider !