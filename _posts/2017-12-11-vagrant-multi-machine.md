---
layout: post
categories: linux vagrant multi machine
date: 2017-12-11 21:00:00 +0200
lang: fr
ref: 2017-12-11-vagrant-multi-machine
title: "Vagrant multi-machine"
---

The purpose is to have several VM to reproduct production environment,
simulate network failure. We start simply with a web machine
and an external database (and specific disk size for that)


## Install vagrant plugins
```
vagrant plugin install vagrant-vbguest
vagrant plugin install vagrant-disksize
```

## Create your multi machine environment
```
cd {yourdir}/workspace/envdev/
mkdir my-multi-machine
cd my-multi-machine
vagrant init
...
```

## Edit your Vagrantfile
```
Vagrant.configure("2") do |config|
	# WEB
	config.vm.define "web", primary: true do |web|
		web.vm.box = "debian/stretch64"
		web.vm.box_check_update = false
		web.vm.network "private_network", ip: "192.168.33.101"
		web.vm.synced_folder ".", "/vagrant", owner: "vagrant", group: "vagrant", type: "virtualbox"
		web.vm.synced_folder "../../", "/home/vhosts", owner: "vagrant", group: "vagrant", type: "virtualbox"
		web.vm.provider "virtualbox" do |vb|
			vb.customize ["modifyvm", :id, "--name", "web"]
			vb.customize ["modifyvm", :id, "--memory", "512"]
			vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
			vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
	  	end
	end
	# DB
	config.vm.define "db" do |db|
		db.vm.box = "debian/stretch64"
		db.disksize.size = "30GB"
		db.vm.box_check_update = false
		db.vm.network "private_network", ip: "192.168.33.102"
		db.vm.synced_folder ".", "/vagrant", owner: "vagrant", group: "vagrant", type: "virtualbox"
		db.vm.provider "virtualbox" do |vb|
			vb.customize ["modifyvm", :id, "--name", "db"]
			vb.customize ["modifyvm", :id, "--memory", "512"]
			vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
			vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  		end
	end
end
```

"web" machine is the primary. When you `vagrant ssh`, you enter in it automatically.
So to go in db, enter `vagrant ssh db`

## More
Read the [doc](https://www.vagrantup.com/docs/multi-machine/)

## DNS
If you want to get name (name.local)  
Set `{subconfig}.vm.hostname = "myname"` and install `apt-get install -y avahi-daemon libnss-mdns`.

You can ping any other VM by using their hostname (plus .local at the end)
