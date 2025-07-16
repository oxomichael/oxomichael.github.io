---
translationKey: "2010-07-14-pure-ftpd-virtual-users"
title: "Pure-FTPd and Virtual Users"
date: "2010-07-14T11:00:00Z"
---

Pure-FTPd is a secure and easy-to-configure FTP server. One of its most useful features is the management of **virtual users**. Unlike system users, virtual users exist only for the FTP service. They are stored in a dedicated database, which avoids creating real accounts on the operating system, thereby improving security.

This guide will show you how to configure Pure-FTPd to use a virtual user database.

## 1. Virtual User Files

The configuration of virtual users relies on two main files, usually located in `/etc/pure-ftpd/`:

-   `pureftpd.passwd`: A text file containing user information in a readable format. You can edit this file manually if needed.
-   `pureftpd.pdb`: The binary database generated from the `pureftpd.passwd` file. This is the file Pure-FTPd uses to authenticate users.

It is advisable to back up these two files regularly.

## 2. Configuring Pure-FTPd

Pure-FTPd is configured via individual files in the `/etc/pure-ftpd/conf/` directory. Each file represents a configuration option.

For example, to chroot all users (confining them to their home directory), you create a `ChrootEveryone` file with the value `yes`.

```bash
echo "yes" > /etc/pure-ftpd/conf/ChrootEveryone
```

This method adds the `-A` parameter to the Pure-FTPd startup command after reloading. To discover all available options, consult the manual:

```bash
man pure-ftpd-wrapper
```

Pay attention to case sensitivity when creating configuration files.

## 3. Managing Virtual Users

The `pure-pw` command is the primary tool for managing virtual users.

### Create a User

To add a new user:

```bash
pure-pw useradd my_user -u ftpuser -d /home/ftp/my_user
```

-   `my_user`: The name of the virtual user.
-   `-u ftpuser`: Associates the virtual user with a real system user (`ftpuser` in this example). Uploaded files will belong to `ftpuser`.
-   `-d /home/ftp/my_user`: The user's home directory.

**Note**: For the home directory to be created automatically on the first login, ensure the `/etc/pure-ftpd/conf/CreateHomeDir` file contains `yes`.

### Modify a User

To change the password:

```bash
pure-pw passwd my_user
```

To modify a user's information (e.g., their directory):

```bash
pure-pw usermod my_user -d /home/ftp/another_folder
```

### Update the Database

**Important**: After each user addition or modification with `pure-pw` (or by editing `pureftpd.passwd`), you must update the `.pdb` database:

```bash
pure-pw mkdb
```

## 4. Enable Database Authentication

For Pure-FTPd to use our virtual user database, we need to enable the `PureDB` authentication method. This is done by creating a symbolic link in the `/etc/pure-ftpd/auth/` directory.

```bash
cd /etc/pure-ftpd/auth/
ln -s ../conf/PureDB 50puredb
```

The name `50puredb` defines the priority order. Lower numbers are checked first.

## 5. Disable Other Authentications (Optional)

To enhance security, it is recommended to disable other authentication methods, such as system user authentication via PAM. To do this, modify the corresponding configuration file:

```bash
echo "no" > /etc/pure-ftpd/conf/PAMAuthentication
```

Don't forget to reload the Pure-FTPd configuration for the changes to take effect. The method varies depending on your system (e.g., `service pure-ftpd restart` or `systemctl restart pure-ftpd`).