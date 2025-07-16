---
translationKey: "2010-09-07-sftp-openssh"
categories: ["linux", "ssh", "sftp"]
date: "2010-09-07T11:00:00Z"
title: "Setting Up a Secure SFTP Access with OpenSSH"
---

SFTP (SSH File Transfer Protocol) is a secure method for transferring files over a network. Unlike FTP, all data, including credentials, is encrypted. Modern versions of OpenSSH include a built-in SFTP server, allowing you to set up restricted access for file transfers without needing a full SSH shell.

This guide explains how to create a user limited exclusively to SFTP access and confined to their home directory (chroot).

## 1. Create a Dedicated Group

It is recommended to create a specific group for SFTP users. This makes it easier to manage permissions and apply rules in the SSH configuration.

```bash
sudo groupadd sftp
```

## 2. Create an SFTP User

Now, we will create a user who will be a member of this group. This user will not have a valid login shell for security reasons.

```bash
# Create the user, their home directory, and add them to the sftp group
sudo useradd -m -d /home/sftp/my_user -g sftp my_user

# Assign a password to the user
sudo passwd my_user

# Set the user's shell to /bin/false to prevent interactive SSH logins
sudo usermod -s /bin/false my_user
```
- `my_user`: The name of the user to create.
- `-d /home/sftp/my_user`: The user's home directory.
- `-g sftp`: The user's primary group.
- `-s /bin/false`: Prevents the user from opening a shell session.

## 3. Configure the SSH Server

Modify the SSH server configuration file, usually located at `/etc/ssh/sshd_config`, to add the rules for our `sftp` group.

Comment out the existing `Subsystem sftp` line and add the following lines at the end of the file:

```sshd-config
# Comment out this line if it exists
# Subsystem sftp /usr/lib/openssh/sftp-server

# Use the internal SFTP subsystem
Subsystem sftp internal-sftp

# Apply the following rules only for members of the sftp group
Match Group sftp
    # Force the file system root to be the user's home directory
    ChrootDirectory %h
    # Disable X11 forwarding
    X11Forwarding no
    # Disable TCP port forwarding
    AllowTcpForwarding no
    # Force the execution of the internal SFTP command, ignoring any command sent by the client
    ForceCommand internal-sftp
```
- `ChrootDirectory %h`: This is the key directive. It confines the user to their home directory (`%h` is a token that represents the user's home directory).

Don't forget to restart the SSH service to apply the changes:
```bash
sudo systemctl restart sshd
```

## 4. Adjust Directory Permissions

For `ChrootDirectory` to work, the user's home directory must be owned by `root` and not be writable by other users. This is a security constraint of OpenSSH.

Therefore, the user will not be able to write directly into their root (`/home/sftp/my_user`). We need to create subdirectories and give them ownership.

```bash
# The home directory must be owned by root
sudo chown root:root /home/sftp/my_user

# Create subdirectories for the user to transfer files
sudo mkdir /home/sftp/my_user/upload
sudo mkdir /home/sftp/my_user/download

# Give ownership of these subdirectories to the user
sudo chown my_user:sftp /home/sftp/my_user/upload
sudo chown my_user:sftp /home/sftp/my_user/download

# Adjust permissions if necessary
sudo chmod 755 /home/sftp/my_user/upload
sudo chmod 755 /home/sftp/my_user/download
```

## 5. Enhance Logs (Optional)

To get more detailed logs about SFTP activities, you can modify the `Subsystem` line in `sshd_config`:

```sshd-config
Subsystem sftp internal-sftp -l INFO
```
This will log operations such as login, download, or upload of files, which is very useful for monitoring.