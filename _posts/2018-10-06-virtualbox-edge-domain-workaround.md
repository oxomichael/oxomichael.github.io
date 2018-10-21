---
layout: post
categories: vagrant virtualbox microsoft edge domain workaround
date: 2018-10-06 21:00:00 +0200
lang: en
ref: 2018-10-06-virtualbox-edge-domain-workaround
title: "Microsoft Edge workaround with Virtualbox (vagrant) and local domain"
---

Due to network isolation in Windows 10 (is it a bug ?).
Edge can't access domain in VirtualBox network, so we use regedit.exe

Looks for key registry name as `*NdisDeviceType` of your virtual network card (from VirtualBox), and change the value to "0" (value is set to one by default), and reboot.

To find the key, see something like that in the registry  `Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\00XX`.

The change will make your network card in active networks, as a "Public" or "Unidentified" network.
