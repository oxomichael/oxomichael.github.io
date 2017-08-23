---
layout: post
title:  "Ansistrano Deployement"
date:   2017-07-15 23:00:00 +0200
categories: symfony packages
---

# Installation Ansible

> $ sudo apt-get install software-properties-common  
> $ sudo apt-add-repository ppa:ansible/ansible  
> $ sudo apt-get update  
> $ sudo apt-get install ansible  

In /etc/ansible/ansible.cfg  
 [defaults]  
 host_key_checking = false  

# Installation Ansistrano
> $ ansible-galaxy install carlosbuenosvinos.ansistrano-deploy carlosbuenosvinos.ansistrano-rollback

Update
If you want to update the role, you need to pass --force parameter when installing. Please, check the following command:
> $ ansible-galaxy install --force carlosbuenosvinos.ansistrano-deploy carlosbuenosvinos.ansistrano-rollback

# Usage
Install your project ("ansible" for example) from git in the user "deploy" home directory

Create the public key for the "deploy" user
> $ ssh-keygen -t rsa
> $ cp -v ~/.ssh/id_rsa.pub ~/ansible/files/authorized_keys.deploy.pub

I have created to playbook to init remote host
- base/user-deploy.yml : Create user deploy with permission
- base/vhosts.yml : Create

On each host create the user "deploy"
> $ ansible-playbook -i {inventory} base/user-deploy.yml --become -k --ask-become-pass --user={first user create on install}

```
---
# User Deploy
- name: Prepare and configure user "deploy"
  hosts: all
  become: yes
  tasks:
   - name: Add deploy user
     user: name=deploy comment="Deploy User" groups=adm,sudo,www-data shell=/bin/bash
   - name: Adding authorized key to deploy user
     authorized_key: user=deploy key="{{item}}"
     with_file:
     - ../files/authorized_keys.deploy.pub
   - name: Ensure /etc/sudoers.d is scanned by sudo
     # A mistake use pkexec visudo
     action: lineinfile dest=/etc/sudoers regexp="#includedir\s+/etc/sudoers.d" line="#includedir /etc/sudoers.d" validate="visudo -cf %s"
   - name: Add deploy user to the sudoers
     action: 'lineinfile dest=/etc/sudoers.d/deploy state=present create=yes regexp="deploy .*" line="deploy ALL=(ALL) NOPASSWD: ALL" validate="visudo -cf %s"'
   - name: Ensure /etc/sudoers.d/deploy file has correct permissions
     action: file path=/etc/sudoers.d/deploy mode=0440 state=file owner=root group=root
```

Check main vhost
> $ ansible-playbook -i {inventory} base/vhost.yml

```
---
# Vhost
- hosts: all
  become: yes
  tasks:
   - name: Create and check vhost directory perms
     file: path=/home/vhosts state=directory mode="u=rwx,g=rwx,o=rx" owner=www-data group=www-data
```

Filter to execute only on one host
> $ ansible-playbook -i {inventory} -l 192.168.0.0 playbook.yml

# Manage a project to deploy

We want to make progressive deployment, so we have to define how to work.
With standard git branch :
- master : production
- develop : staging


Récupération de la branch "master" du projet dans /home/vhosts/deploy/export

Deploying
> $ ansible-playbook -i hosts -e "ansistrano_release_version=`date -u +%Y%m%d%H%M%SZ`" deploy.yml

Rolling back
> $ ansible-playbook -i hosts rollback.yml
