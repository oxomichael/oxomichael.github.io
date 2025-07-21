---
translationKey: "2014-08-17-mysql-upgrade-ubuntu"
categories: ubuntu mysql upgrade
date: "2014-08-17T11:00:00Z"
ref: 2014-08-17-ubuntu-mysql-upgrade
title: Upgrade MySQL in Ubuntu
---

Updating Ubuntu (from one version to another) and restarting MySQL.....oops it doesn't work...because for me, the data directory is located outside the standard path, and so apparmor is not happy.

To keep it simple and avoid further issues:
Backup your MySQL configuration file:

```bash
$ sudo cp /etc/mysql/my.cnf /etc/mysql/my.cnf.backup
```

Modify the Apparmor configuration by adding a file:

```bash
$ sudo nano /etc/apparmor.d/local/usr.sbin.mysqld
```

This adds a machine-specific file for local settings containing additional parameters, without changing the base configuration.
File content:

```
# Site-specific additions and overrides for usr.sbin.mysqld.
# For more details, please see /etc/apparmor.d/local/README.

/dbs/mysql/ r,
/dbs/mysql/** rwk
```

And then, run the updates.