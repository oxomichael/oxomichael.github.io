---
translationKey: "2016-03-02-linux-usb3-uasp-disable"
categories:
- linux
- usb3
- uasp
date: "2016-03-02T21:55:00Z"
ref: 2016-03-02-linux-usb3-uasp-disable
title: "Linux and USB 3.0: Disabling the UASP Protocol for a Device"
---

After purchasing an ICY DOCK Black Vortex enclosure (MB174SU3S-4SB) to house four backup drives, I noticed that intensive use of `rsync` was causing slowdowns and unexpected disconnections.

A quick look at the kernel logs with `dmesg` revealed numerous USB-related errors.

The enclosure is connected via USB 3.0, as confirmed by the `lsusb` command:

```bash
$ lsusb
Bus 006 Device 002: ID 152d:0567 JMicron Technology Corp. / JMicron USA Technology Corp. JMS567 SATA 6Gb/s bridge
```

The `lsusb -t` command shows that the `uas` (USB Attached SCSI) driver is currently in use:

```bash
$ lsusb -t
Bus 06.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/2p, 5000M
   |__ Port 1: Dev 2, If 0, Class=Mass Storage, Driver=uas, 5000M
```

The `uas` driver is supposed to offer better performance, but it can be unstable with certain controllers. To fix this, we will disable it for this specific device to force the system to use the older but more reliable `usb-storage` driver.

To do this, create a configuration file for `modprobe`, for example `/etc/modprobe.d/disable-uas.conf`, with the following content:

```
options usb-storage quirks=152d:0567:u
```

The format of the `quirks` option is `VendorID:ProductID:flags`.
- `152d` (VendorID) and `0567` (ProductID) are the identifiers for our device, obtained earlier.
- The `u` flag (for `UAS_BLACKLIST`) tells the kernel not to use the `uas` driver for this hardware.

After creating the file, update the module dependencies and the initramfs image (on Debian/Ubuntu-based distributions):

```bash
$ sudo depmod -a
$ sudo update-initramfs -u
```

Finally, reboot your machine. The device should now use `usb-storage` and be more stable. Hope this helps!