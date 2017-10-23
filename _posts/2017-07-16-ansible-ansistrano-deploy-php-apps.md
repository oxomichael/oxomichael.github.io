---
layout: post
categories: php deploy ansible ansistrano
date: 2017-07-16 23:00:00 +0200
lang: en
ref: 2017-07-16-ansistrano-deploy-php-apps
title:  "Ansistrano Deployement"
---

Deploy PHP Apps with [Ansistrano](https://ansistrano.com/)

# Install Ansible

> $ sudo apt-get install software-properties-common  
> $ sudo apt-add-repository ppa:ansible/ansible  
> $ sudo apt-get update  
> $ sudo apt-get install ansible  

In /etc/ansible/ansible.cfg  
 [defaults]  
 host_key_checking = false  

# Install Ansistrano
> $ ansible-galaxy install carlosbuenosvinos.ansistrano-deploy carlosbuenosvinos.ansistrano-rollback

Update
If you want to update the role, you need to pass --force parameter when installing. Please, check the following command:
> $ ansible-galaxy install --force carlosbuenosvinos.ansistrano-deploy carlosbuenosvinos.ansistrano-rollback

# Ansible repository

You could organize your deployment repository by company or by type of project.
I call it "ansible" for example, with directory structure :

apps/  
base/  
files/  
README.md

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


Following are the commands to execute a deployement

Deploying
> $ ansible-playbook -i hosts -e "ansistrano_release_version=`date -u +%Y%m%d%H%M%SZ`" deploy.yml

Rolling back
> $ ansible-playbook -i hosts rollback.yml

Now go read the documentation of Ansistrano to see all the step in the workflow.

```
-- /home/vhosts/my-app.com
|-- current -> /home/vhosts/my-app.com/releases/20100512131539
|-- releases
|   |-- 20100512131539
|   |-- 20100509150741
|   |-- 20100509145325
|-- shared
```

You have to make some operation locally and remotely.

Locally :
- Clone your project
- Install dependencies
- ... and all other things

Remotely :
- Files are automatically sent (via rsync) in your remote place
- Clean temporary files and warmup cache

Sample with my web app
```
/home/deploy/ansible/apps/my-app.com
|-- hosts-prod
|-- deploy.yml
|-- etc/
|-- config/
|-- tasks
|   |-- before-code-update.yml
|   |-- after-code-update.yml
|   |-- before-symlink.yml
|   |-- after-symlink.yml
|   |-- before-cleanup.yml
|   |-- after-cleanup.yml
```

hosts-prod : contains your production server definition  
deploy.yml :  
etc/ : specific server config files to be place in your remote server  
config/ : environment files for your project (secret information is store here)  
tasks/ : specific tasks  

All these files could be specific for staging.  
