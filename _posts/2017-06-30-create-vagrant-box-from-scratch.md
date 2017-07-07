# Create a vagrant box from scratch

## Box Debian Stretch 64

* Install VirtualBox
* Install Vagrant
* Télécharcher le CD d'installation de Debian
* Create virtual machine

### Virtual Machine

#### Hardware
* Name: vagrant-stetch64
* Type: Linux
* Version: vagrant-stretch64
* Memory Size : 512MB
* New Virtual Disk:
  * Type: VMDK
  * Size: 8GB
 * Disable audio
 * DIsable USB
 * Mount ISO CD

#### Installation
Choose Graphical Install
* Select a language
* Select your location
* Configure locales
* Configure the keyboard
* Configure the network
  * Hostname : stretch64
* Set up users and passwords
  * Enter "vagrant" as root password
  * Enter "vagrant" as fullname
  * New user as "vagrant"
  * and also "vagrant" as password
* Configure the clock
* Partition disks
 * Guided - use entire disks
 * Just let SCSIl (0, 0, 0) (sda) - 8.6 GB ATA VBOX HARDDISK as preselected and continue.
 * All files in one partition
 * Let's Finish partitioning and write changes to disk.
* Install the base system
* Configure the package manager
* Software select
  * Please disable every option, except *standard system utilities*.
* Finish installation

#### Configuration
* Install sudo

 > $ su

 > $ apt-get install -y sudo

* Give sudo permission to vagrant

 > $ visudo -f /etc/sudoers.d/vagrant
 
* Add the following line to authorize vagrant use sudo without password
 > `vagrant ALL=(ALL) NOPASSWD:ALL` 

* Exit and disconnect user
* Update and upgrade

 > $ sudo apt-get update && sudo apt-get upgrade

* Install basic packages

 > $ sudo apt-get install -y build-essential module-assistant

 > $ sudo module-assistant prepare

 > $ sudo apt-get install -y zerofree openssh-server

 * SSH Configuration

 Edit /etc/ssh/sshd_config, Uncomment `AuthorizedKeysFile %h/.ssh/authorized_keys`

 > $ mkdir -p /home/vagrant/.ssh

 > $ chmod 0700 /home/vagrant/.ssh

 > $ wget --no-check-certificate \
  https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub \
  -O /home/vagrant/.ssh/authorized_keys

 > $ chmod 0600 /home/vagrant/.ssh/authorized_keys

 > $ chown -R vagrant /home/vagrant/.ssh

* Restart ssh

 > $ sudo service ssh Restart

* Install Guest Tools - Guest Additions CD Image

 > $ sudo mount /dev/cdrom /mnt

 > $ cd /mnt

 > $ sudo ./VBoxLinuxAdditions.run

* Cleaning

 > $ sudo apt-get autoremove && sudo apt-get clean

* Zerofree

  Connect as root

 > $ init 1

 > $ mount -o remount,ro /dev/sda1 /

 > $ zerofree /dev/sda1

* Restart

 > $ shudown -h now

* Pack your machine

 > $ vagrant package --base vagrant-stretch64

## Use your personalize Box

> $ vagrant box add vagrant-stretch64 package.box

> $ vagrant init vagrant-stretch64

> $ vagrant up
 