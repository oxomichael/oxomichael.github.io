---
layout: post
categories: linux ubuntu eclipse
date: 2014-08-17 11:00:00 +0200
lang: en
ref: 2016-05-12-install-eclipse-ubuntu
title:  "Install Eclipse on Ubuntu"
---

## Eclipse OXYGEN
- Download the new oomph installer.
- Install in your home

## Create a desktop entry with Icon
`$ gedit .local/share/applications/eclipse.desktop`

```
[Desktop Entry]
Name=Eclipse
Type=Application
Exec=/home/USERNAME/eclipse/eclipse
Terminal=false
Icon=/home/USERNAME/eclipse/icon.xpm
Comment=Integrated Development Environment
NoDisplay=false
Categories=Development;IDE;
Name[en]=Eclipse
Name[fr]=Eclipse
```

## Eclipse Mars.2
- Uncompress the Eclipse archive in your home  
- Edit eclipse.ini File
```
-startup
plugins/org.eclipse.equinox.launcher_1.3.100.v20150511-1540.jar
--launcher.library
plugins/org.eclipse.equinox.launcher.gtk.linux.x86_64_1.1.300.v20150602-1417
-product
org.eclipse.epp.package.php.product
--launcher.defaultAction
openFile
-showsplash
org.eclipse.platform
--launcher.XXMaxPermSize
256m
--launcher.defaultAction
openFile
--launcher.GTK_version
2
--launcher.appendVmargs
-vmargs
-Dosgi.requiredJavaVersion=1.8
-Xms256m
-Xmx1024m
```
