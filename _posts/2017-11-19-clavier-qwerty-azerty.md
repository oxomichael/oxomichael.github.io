---
layout: post
categories: clavier qwerty azery
date: 2017-11-19 14:00:00 +0200
lang: fr
ref: 2017-11-19-clavier-ansi-iso
title: "Convertir un clavier QWERTY (ANSI) en version AZERTY (ISO)"
---

Acheter les autocollants puis les coller.
Mais il vous manque les signes supérieur (>) et inférieur (<)

## Windows

Installer AutoHotKey (https://www.autohotkey.com)  
Lire la documentation (je vous met tous ça là, quand même)
1. Right-Click on your desktop.
2. Find "New" in the menu.
3. Click "AutoHotkey Script" inside the "New" menu.
4. Give the script a new name. Note: It must end with a .ahk extension. Ex. MyScript.ahk
5. Find the newly created file on your desktop and Right-Click it.
6. Click "Edit Script".
7. A window should have popped up, probably Notepad. If so, SUCCESS!

Créer un fichier par exemple keymiss.ahk
Saisir le contenu suivant
```
#NoEnv  ; Recommended for performance and compatibility with future AutoHotkey releases.
; #Warn  ; Enable warnings to assist with detecting common errors.
SendMode Input  ; Recommended for new scripts due to its superior speed and reliability.
SetWorkingDir %A_ScriptDir%  ; Ensures a consistent starting directory.

RCtrl::
	Send {U+003C}
Return

+RCtrl::
	Send {U+003E}
Return
```

Enregistrer et exécuter le fichier pour voir le résultat.  
Avec ce script, vous sacrifiez la touche CTRL Droite  
SHIFT + CTRL = >  
CTRL         = <

Ensuite soit on exécute après chaque démarrage ce script ou il est possible de
générer un exécutable (Convert ahk to exe) pour l'ajouter au démarrage sous Windows

1. Barre de recherche (menu démarrer, fenêtre de l'explorateur Windows, fenêtre Exécuter): %appdata% faites Entrer,
2. Aller sur Windows/menu démarrer/démarrage
3. Placez le raccourcis du programme a faire démarrer dans le dossier Démarrage.
4. Le tour est joué

ou Exécuter (shell:startup)

Version anglaise : "C:\Users\*****\AppData\Roaming\Microsoft\Windows\ Start Menu\Programs\Startup"
Version française : "C:\Utilisateurs\*****\AppData\Roaming\Microsoft\W indows\Menu Démarrer\Programmes\Démarrage"

## Linux

### Xmodmap

On génère le fichier de map
```bash
$ xmodmap -pke > ~/.Xmodmap
```

On trouve la clef a remap
```bash
$ xev | awk -F'[ )]+' '/^KeyPress/ { a[NR+2] } NR in a { printf "%-3s %s\n", $5, $8 }'
```

Une bonne aide est d'aller voir la doc d'ArchLinux [https://wiki.archlinux.org/index.php/xmodmap]
On prend la ligne
```
keycode  94 = less greater less greater bar brokenbar bar brokenbar less greater bar NoSymbol less greater lessthanequal greaterthanequal
```
Et on remplace dans la ligne `keycode 52` par exemple
(on affecte des nouvelles fonctions sur la touche `w` )
```
keycode  52 = w W z Z less greater guillemotleft less w W guillemotleft leftdoublequotemark
```

On modifie le fichier Xmodmap et on teste avec
```bash
xmodmap ~/.Xmodmap
```

### Xkb

Sinon il y a aussi xkb
```
sudo nano /usr/share/X11/xkb/symbols/fr
```
Suppression du cache
```
rm -rf /var/lib/xkb/*
```

### Wayland
@todo
